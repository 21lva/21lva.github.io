

# Spring Boot + Spring Data Mongo + Kotlin Sealed Class 사용 시 주의 사항


- spring boot, spring data reactive mongo를 사용하면서 sealed class를 사용하면서 겪은 문제와 해결 방법을 적어놓는다.
- 테스트용 코드 환경은 JVM_11, kotlin, local mongodb(6.0.5)이다.
- spring 의존성은 아래와 같다
```
//build.gradle.kts
dependencies {  
   implementation("org.springframework.boot:spring-boot-starter-data-mongodb-reactive")  
   implementation("org.springframework.boot:spring-boot-starter-webflux")  
   implementation("com.fasterxml.jackson.module:jackson-module-kotlin")  
   implementation("io.projectreactor.kotlin:reactor-kotlin-extensions")  
   implementation("org.jetbrains.kotlin:kotlin-reflect")  
   implementation("org.jetbrains.kotlinx:kotlinx-coroutines-reactor")  
   testImplementation("org.springframework.boot:spring-boot-starter-test")  
   testImplementation("io.projectreactor:reactor-test")  
}
```

- MongoDB용 spring property 설정은 아래와 같다
```yml
spring:  
  data:  
    mongodb:  
      host: localhost  
      port: 27017  
      database: local
```

## 부모의 멤버 필드 오버라이드 시 

- 부모 클래스(sealed class)의 멤버 필드를 오버라이드할 때 몇 가지를 주의해야한다

```kotlin
@Document("parents")  
sealed class Parent(  
        @Id open val id: String?,  
) {  
    open var nn: String = "kkk"  
}  
  
data class C1(  
        override val id: String?,  
        val x: Long,  
        override var nn: String,  
): Parent(id)  
  
data class C2(  
        override val id: String?,  
        val x: Long,  
        val y: Long,  
        override var nn: String,  
): Parent(id)

// TEST
@Test  
fun `test save`() {  
   val c1 = C1(null, 1, "abc")  
   val c2 = C2(null, 2, 3, "cdf")  
   parentRepository.saveAll(listOf(c1, c2)).blockLast()  
     
   val list2 = parentRepository.findAll().collectList().block()  
     
   assertEquals(2, list2!!.size)  
}
```

- 위의 코드를 실행하면 아래와 같은 에러가 발생한다

```
Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'mongoMappingContext' defined in class path resource [org/springframework/boot/autoconfigure/data/mongo/MongoDataConfiguration.class]: Invocation of init method failed; nested exception is org.springframework.data.mapping.MappingException: Ambiguous field mapping detected! Both private java.lang.String com.example.demo.Parent.nn and private java.lang.String com.example.demo.C1.nn map to the same field name nn! Disambiguate using @Field annotation!
```

- 부모와 자식 클래스에 같은 멤버변수가 존재하면서 몽고디비 입장에서는 매핑하기가 애매하기 때문이다. 
- 아래와 같이 annotation을 달자

```kotlin
import org.springframework.data.annotation.Id  
import org.springframework.data.annotation.Transient  
import org.springframework.data.mongodb.core.mapping.Document  
  
@Document("parents")  
sealed class Parent(  
        @Id open val id: String?,  
) {  
    open var nn: String = "kkk"  
}  
  
data class C1(  
        override val id: String?,  
        val x: Long,  
        @Transient override var nn: String,  
): Parent(id)  
  
data class C2(  
        override val id: String?,  
        val x: Long,  
        val y: Long,  
        @Transient override var nn: String,  
): Parent(id)
```

- 이와 같은 일이 발생하는 이유는 Spring data mongo에서 *ReadConversion* 방식때문이다. 

## Duplicate Key error

- 위와 같이 고치더라도, 아래처럼 테스트를 시행하면 에러가 발생한다

```kotlin
data class C1(  
        override val id: String?,  
        val x: Long,  
        @Transient override var nn: String,  
        @Transient override var mm: String,  
): Parent(id)  
  
data class C2(  
        override val id: String?,  
        val x: Long,  
        val y: Long,  
        @Transient override var nn: String,  
        @Transient override var mm: String,  
): Parent(id)

// TEST code
@Test  
fun `test save2`() {  
   val c1 = C1(null, 1, "abc", "123")  
   val c2 = C2(null, 2, 3, "cdf", "456")  
   parentRepository.saveAll(listOf(c1, c2)).blockLast()  
  
   val list2 = parentRepository.findAll().collectList().block()  
   list2?.forEach {  
      parentRepository.save(it).block()   // 여기서 에러 발생
   }  
   val list3 = parentRepository.findAll().collectList().block()  
  
   assertEquals(2, list2!!.size)  
}
```

- 에러 문구는 다음과 같다

```
Write operation error on server localhost:27017. Write error: WriteError{code=11000, message='E11000 duplicate key error collection: local.parents index: _id_ dup key: { _id: ObjectId('641dbc9b151c195eaf50188c') }', details={}}.; nested exception is com.mongodb.MongoWriteException: Write operation error on server localhost:27017. Write error: WriteError{code=11000, message='E11000 duplicate key error collection: local.parents index: _id_ dup key: { _id: ObjectId('641dbc9b151c195eaf50188c') }', details={}}.; nested exception is org.springframework.dao.DuplicateKeyException: Write operation error on server localhost:27017. Write error: WriteError{code=11000, message='E11000 duplicate key error collection: local.parents index: _id_ dup key: { _id: ObjectId('641dbc9b151c195eaf50188c') }', details={}}.; nested exception is com.mongodb.MongoWriteException: Write operation error on server localhost:27017. Write error: WriteError{code=11000, message='E11000 duplicate key error collection: local.parents index: _id_ dup key: { _id: ObjectId('641dbc9b151c195eaf50188c') }', details={}}.
```
- 부모와 자식의 key(*id 필드*)가 중복되어 있다고 나온다. 
- 해결방법은 위와 비슷하다

```kotlin
data class C1(  
        @Transient override val id: String?,  
        val x: Long,  
        @Transient override var nn: String,  
        @Transient override var mm: String,  
): Parent(id)  
  
data class C2(  
        @Transient override val id: String?,  
        val x: Long,  
        val y: Long,  
        @Transient override var nn: String,  
        @Transient override var mm: String,  
): Parent(id)
```

## private 멤버 변수 문제

- 아래의 코드에서처럼 data class의 생성자가 아닌 다른 멤버로 변수가 존재하는 경우에는 에러가 발생한

```kotlin
@Document("a")  
data class F(  
        @Id private val id: String?,  
        private val p: Parent,  
)  
  
sealed class Parent(  
        protected open var mm: String = "mm"  
) {  
    protected open var nn: String = "nn"  
}  
  
data class C1(  
        private val x: Long,  
        @Transient override var nn: String,  
        @Transient override var mm: String,  
): Parent(mm) {  
    @Transient private val ff: String = "12"  
}  
  
data class C2(  
        private val x: Long,  
        @Transient override var nn: String,  
        @Transient override var mm: String,  
): Parent(mm) {  
    @Transient private val ff: String = "34"  
}

//TEST
@Test  
fun `test save2`() {  
   val c1 = C1(1, "abc", "123")  
   val c2 = C2(2,  "cdf", "456")  
   val f1 = F(null, c1)  
   val f2 = F(null, c2)  
   fRepository.saveAll(listOf(f1, f2)).collectList().block()  
  
   val list1 = fRepository.findAll().collectList().block()  
   list1?.forEach {  
      fRepository.save(it).block()  
   }  
   val list2 = fRepository.findAll().collectList().block()  
  
   assertEquals(2, list1!!.size)  
}
```

- 에러 문구는 다음과 같다

```
No accessor to set property private final java.lang.String com.example.demo.C1.ff
```


- kotlin의 val은 final이기 때문에 setter로 값을 변경해줄 수 없다는 의미이다.
	- private 여부는 상관 없다.
	- [Spring Data 2.1 이후로 final변수를 저장하기 위해서 reflection을 사용하지 않는다](https://github.com/spring-projects/spring-data-commons/issues/1810?focusedCommentId=182289&page=com.atlassian.jira.plugin.system.issuetabpanels%253Acomment-tabpanel#comment-182289)

- 해결방법은 val을 DB에 저장할지 말지에 따라 달라진다
	1. DB에 저장을 원하지 않는 경우: @Transient로 해결
	2. DB에 저장을 원하는 경우: 새로운 생성자을 만들고 @PersistenceConstructor를 설정 / data class를 다시 설정. 보통은 후자를 추천한다. @PersistenceContstructor는 deprecated되었다. 

```kotlin
data class C1(  
        private val x: Long,  
        @Transient override var nn: String,  
        @Transient override var mm: String,  
): Parent(mm) {  
    @PersistenceConstructor  
    constructor(x: Long, nn: String, mm: String, ff: String): this(x, nn, mm) {  
    }  
    val ff: String = "13"  
}
```

- 잘 저장되어 있는 것을 볼 수 있다.
![[Pasted image 20230325021452.png]]

## isXXX 메서드 저장 문제

- 이 문제는 Spring Data Mongo에서 기본적으로 제공하는 Conversion을 이용하면 나타나지는 않지만, Custom한 Conversion을 만들면 문제가 발생할 수 있다. 특히, 이 때 *JacksonMapper*를 이용하면 생길 수 있다.

- 아래의 객체를 저장하는 CustomConversion을 만들어보자
```kotlin
@JsonAutoDetect(fieldVisibility = JsonAutoDetect.Visibility.ANY) // private 객체를 이용하려면 필요
@Document("a")  
data class F(  
        @Id private val id: String?,  
        private val p: Parent,  
) {  
    fun isA(): Boolean = true  
}  
  
@JsonAutoDetect(fieldVisibility = JsonAutoDetect.Visibility.ANY)  
sealed class Parent(  
        protected open var mm: String = "mm"  
) {  
    protected open var nn: String = "nn"  
    abstract fun isB(): Boolean  
}  
  
data class C1(  
        private val x: Long,  
        @Transient override var nn: String,  
        @Transient override var mm: String,  
): Parent(mm) {  
    @PersistenceConstructor  
    constructor(x: Long, nn: String, mm: String, ff: String): this(x, nn, mm) {  
    }  
    val ff: String = "13"  
    override fun isB(): Boolean = true  
}  
  
data class C2(  
        private val x: Long,  
        @Transient override var nn: String,  
        @Transient override var mm: String,  
): Parent(mm) {  
    @PersistenceConstructor  
    constructor(x: Long, nn: String, mm: String, ff: String): this(x, nn, mm) {  
    }  
    val ff: String = "12"  
    override fun isB(): Boolean = false  
}
```

- Conversion 코드는 다음과 같다
```kotlin
@Configuration  
class FMongoConfig {  
    @Bean  
    fun customConversion(): MongoCustomConversions =  
            MongoCustomConversions(listOf(FReadingConverter(), FWritingConverter()))  
  
    @ReadingConverter  
    class FReadingConverter: Converter<Document, F> {  
        override fun convert(source: Document): F? {  
            val value = mapper.readValue<F>(source.toJson(), F::class.java)  
            return value  
        }  
    }  
  
    @WritingConverter  
    class FWritingConverter: Converter<F, Document> {  
        override fun convert(source: F): Document? {  
            return Document.parse(mapper.writeValueAsString(source))  
        }  
    }  
  
    companion object {  
        private val mapper = jacksonObjectMapper()  
                .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false)  
                .setDefaultPropertyInclusion(JsonInclude.Include.NON_NULL)  
    }  
}
```

- WritingConverter를 이용해보면 mapper가 넘겨주는 string이 아래와 같은 것을 볼 수 있다
```
// 결과
{"p":{"x":1,"nn":"abc","mm":"123","ff":"13","b":true},"a":true}
```

- isXX로 만들어진 함수를 field로 인식해서 is는 떼고 변수로 포함시킨다. 
- 이 현상을 방지하기 위해서는 isXXX 함수 앞에 @JsonIgnore을 붙이면 된다.

- 또한 HttpRequest를 위해서 @JsonNaming이라던지 @JsonValue를 붙이는 경우가 있다.

```kotlin
@JsonAutoDetect(fieldVisibility = JsonAutoDetect.Visibility.ANY)  
@Document("a")  
data class F(  
        @Id private val id: String?,  
        private val p: Parent,  
) {  
    @JsonIgnore fun isA(): Boolean = true  
  
    @JsonValue  
    override fun toString() = "ffff"  
}
```

- mapper로 string으로 변환해 보면 아래와 같이 된다.
```
"ffff"
```

- Conversion에서 mixIn을 이용하면 이런 annotation을 제거할 수 있다.
```kotlin
// conversion의 일부
companion object {  
    private abstract class IgnoreToStringMixIn {  
        @JsonValue(false)  
        abstract override fun toString(): String  
    }  
    private val mapper = jacksonObjectMapper()  
            .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false)  
            .setDefaultPropertyInclusion(JsonInclude.Include.NON_NULL)  
            .addMixIn(F::class.java, IgnoreToStringMixIn::class.java)  
}
```

## sealed class의 생성자 문제

- 위 코드는 writer는 제대로 동작한다. 그러나 reader는 문제가 발생한다
```
Failed to convert from type [org.bson.Document] to type [com.example.demo.parent.F] for value 'Document{{_id=641deaa974cb3a516ad4adac, p=Document{{x=1, nn=abc, mm=123, ff=13}}}}'; nested exception is com.fasterxml.jackson.databind.exc.InvalidDefinitionException: Cannot construct instance of `com.example.demo.parent.Parent` (no Creators, like default constructor, exist): abstract types either need to be mapped to concrete types, have custom deserializer, or contain additional type information
```

- 생성자가 없다는 말이다. sealed class는 생성자가 없다.
- 이럴 때에는 아래의 코드와 같이 annotation을 설정해준다
```kotlin
@JsonAutoDetect(fieldVisibility = JsonAutoDetect.Visibility.ANY)  
@JsonTypeInfo(use = JsonTypeInfo.Id.NAME, property = "type", include = JsonTypeInfo.As.PROPERTY)  
@JsonSubTypes(value = [  
    JsonSubTypes.Type(value = C1::class, name = "C1"),  
    JsonSubTypes.Type(value = C2::class, name = "C2"),  
])  
sealed class Parent(  
        protected open var mm: String = "mm"  
) {  
    protected open var nn: String = "nn"  
    @JsonIgnore abstract fun isB(): Boolean  
}
```

- 실행해보면 잘되는 것을 알 수 있다. 
- DB를 확인해보면 아래와 같다. MongoDB에 type이라는 값을 넣어서 이것을 기준으로 어떤 자식 클래스인지 판단하는 것이다.
![[Pasted image 20230325033239.png]]
