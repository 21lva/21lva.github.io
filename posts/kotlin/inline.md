
# Value class 

- value class: kotlin 1.5에 추가. 
- value class는 inline class를 대체한다.

## 사용하는 이유 예시

- value class가 어떻게 사용되는지를 예시를 들면서 알아보자

```kotlin
data class Member(
	val name: String,
)

data class Bag(
	val memberName: String,
)
```

- 위와 같이 member의 name은 2 곳에 위치해 있다. *Bag, Member*. 
- 여기에서 memberName에 어떤 로직(e.g. validation)이 추가가 된다면 두 클래스에 모두 그 로직을 구현해야 하는데 이는 필요없는 중복을 만들 뿐이다.
- 결국, DDD등 여러 방법론에서는 *VO*라는 값 객체를 사용하도록 권장한다. 
- 상황에 따라서 *wrapper class*가 필요할 때 사용한다.

```kotlin
data class MemberName(val value: String) {
	init {
		require(value.length >= 2) //a validation logic
	}
}

data class Member(
	val name: MemberName,
)

data class Bag(
	val memberName: MemberName,
)
```

- 이렇게 만들면 모든 문제가 해결되는 것 같지만 불필요한 객체를 만드는 경우가 많아진다는 것이 문제이다. 
- 이때 필요한 것이 *Value class*이다. 

```kotlin
@JvmInline value class MemberName(private val value: String) {
	init {
		require(value.length >= 2)
	}
}
```

- *@JvmInline*은 JVM bakcend에서 사용하기 위해 필요한 annotation.

- 아래는 *value class*를 이용하는 함수와 그 함수를 JAVA로 확인했을 때의 모습이다
```kotlin
// kotlin 함수
fun sayName(memberName: MemberName) {  
    println(memberName)  
}

// java 코드
public static final void sayName_iIYq_E4/* $FF was: sayName-iIYq-E4*/(@NotNull String memberName) {  
   Intrinsics.checkNotNullParameter(memberName, "memberName");  
   MemberName var1 = MemberName.box-impl(memberName);  
   System.out.println(var1);  
}
```

- 즉, value class를 쓰면 bytecode로 변환할 때 value class의 객체가 아니라 객체의 유일한 프로퍼티를 받는 함수로 변환한다. 
- 함수뒤에 이상한 숫자와 알파벳이 붙었다. 이를 *Mangling*이라고 한다. 
	- value class를 받는 함수와 value class의 프로퍼티와 같은 타입을 매개변수로 받는 두 함수가 오버로딩으로 되어있어도 바이트코드에서는 두 함수를 구별하기 위해 맹글링한다.
	
## value class의 몇 특성

- value class는 하나의 property만을 가질 수 있다.

![[Pasted image 20230326215226.png]]

- value class는 data class와 달리 *componentN, copy*를 지원하지 않는다.

```kotlin
val name1 = MemberName("abc")  
name1.equals()  
name1.toString()  
name1.hashCode()  
name1.componentN() //Not exists  
name1.copy() //Not exists
```

- value class는 identity(\=\=\=)를 제공하지 않는다. data class에서는 제공한다. 참조 값을 비교한다. 

![[Pasted image 20230326215102.png]]

- value class는 val property만 가능하다

![[Pasted image 20230326215400.png]]

- value class의 property는 primary constructor에서 생성 및 초기화 되어야 한다

![[Pasted image 20230326215639.png]]

## value class에서의 상속

- value class에서도 인터페이스 상속을 지원한다. 
- class를 상속할 수는 없다.

```kotlin
interface X {  
    fun say()  
}  
  
@JvmInline value class MemberName(private val value: String): X {  
    init {  
        require(value.length >= 2)  
    }  
  
    override fun say() {  
        println(value)  
    }  
}
```

## Generic

- value class는 바이트코드로 변환될 때 property의 타입으로 변환되는 것이 기본이지만 아래와 같이 몇 가지 경우에는 unbox되지 않는다

```kotlin
// 코틀린 코드
interface X {  
    fun say()  
}  
  
@JvmInline value class MemberName(private val value: String): X {  
    init {  
        require(value.length >= 2)  
    }  
  
    override fun say() {  
        println(value)  
    }  
}  

fun sayName(memberName: MemberName) {  
    println(memberName)  
}  
  
fun <T> sayNameByGeneric(memberName: T) {  
    println(memberName)  
}  
  
fun sayNameByInterface(memberName: X) {  
    println(memberName)  
}  
  
fun sayNameByNullable(memberName: MemberName?) {  
    println(memberName?:"")  
}

// decomplie 코드
public static final void sayName_iIYq_E4/* $FF was: sayName-iIYq-E4*/(@NotNull String memberName) {  
   Intrinsics.checkNotNullParameter(memberName, "memberName");  
   MemberName var1 = MemberName.box-impl(memberName);  
   System.out.println(var1);  
}  
  
public static final void sayNameByGeneric(Object memberName) {  
   System.out.println(memberName);  
}  
  
public static final void sayNameByInterface(@NotNull X memberName) {  
   Intrinsics.checkNotNullParameter(memberName, "memberName");  
   System.out.println(memberName);  
}  
  
public static final void sayNameByNullable_59LW_jU/* $FF was: sayNameByNullable-59LW-jU*/(@Nullable String memberName) {  
   Object var1 = memberName != null ? (memberName != null ? MemberName.box-impl(memberName) : null) : "";  
   System.out.println(var1);  
}
```

- 1번 함수: 기본적인 동작이다. unbox를 통해서 property의 값을 이용
- 2번 함수: Generic을 이용한다. object로 unboxing되었다
- 3번 함수: Interface(X)를 이용.
- 4번 함수: nullable. 1번과 비슷하지만 내부에서 property를 다시 Object 변수에 매핑해서 사용한다


- Kotlin 1.8 이후부터는 value calss에 generic을 이용할 수 있다

```kotlin
@JvmInline value class Name<T>(private val value: T)

fun f(name: Name<String>) {  
    println(name)  
}

// bytecode
public static final void f_KKnTRB0/* $FF was: f-KKnTRB0*/(@NotNull Object name) {  
   Name var1 = Name.box-impl(name);  
   System.out.println(var1);  
}
```

## vs Typealias


- value class와 typealias는 비슷해보이지만 몇 가지 차이점이 존재한다. typealias와 달리 value class는 실제 타입이다. 

```kotlin
typealias TypeName = String  
  
  
fun TypeName.getName() {  
	TODO()
}  
  
fun getName2(aa: TypeName) {  
	TODO()
}  
  
fun test(){  
    val a1: TypeName = "12"  
    a1.getName()  
    getName2(a1)  
  
    "ab".getName() // You want?  
    getName2("abc") //you want?  
}
```
- 위 코드에서 typealias 하나 만들고 해당 type에 확장 함수를 만들었다. 의도한 로직은 해당 타입에만 확장함수가 동작하는 것이다. BUT 코드에서 볼 수 있듯이 기존 타입(String)인 모든 대상에 확장함수가 지정되는 부작용이 발생한다. 이는 확장함수가 아닌 일반 함수에서도 마찬가지이다

```kotlin
@JvmInline value class ValueName(private val value: String)  
  
  
fun ValueName.getName() {  
    TODO()  
}  
  
fun getName2(aa: ValueName) {  
}  
  
fun test(){  
    val a1 = ValueName("12")  
    a1.getName()  
    getName2(a1)  
  
    "ab".getName() // ERROR  
    getName2("abc") // ERROR  
}
```

- 반면에 value class를 사용하면 원하는 동작을 보여준다. String클래스가 아닌 value class에만 함수가 동작하도록 제한할 수 있다.

- 그렇다면 typealias는 아무 쓸모 없는 것인가? 그렇지 않다. Type을 선언하는 것이 아니라 긴 타입을 줄여쓰기만을 원하거나, 함수형 타입에 이름을 정의하고 싶을 때 사용한다

```kotlin
typealias LongPair = Pair<String, Pair<String, Map<String, Long>>>  
typealias F1 = (String) -> String
```


## Reference

- https://kotlinlang.org/docs/inline-classes.html
- https://velog.io/@dhwlddjgmanf/Kotlin-1.5%EC%97%90-%EC%B6%94%EA%B0%80%EB%90%9C-value-class%EC%97%90-%EB%8C%80%ED%95%B4-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90
- https://quickbirdstudios.com/blog/kotlin-value-classes/