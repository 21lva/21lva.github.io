---
layout: post
title:  "Query DSL + Spring Data"
date:   2022-07-30 02:02:59
author: Inhyuk
category: spring
tags: spring
cover:  "/assets/instacode.png"
name: query_dsl.md
---

- Spring Data를 사용하여 data를 가져오는 방법에서 디테일한 query를 하고 싶으면 **Query DSL**을 사용하면 된다
- 여기서는 Spring Data Mongo를 사용해서 예시를 보여준다

아래는 gradle 설정이다

```kt
import org.jetbrains.kotlin.gradle.tasks.KotlinCompile

plugins {
	id("org.springframework.boot") version "2.7.4"
	id("io.spring.dependency-management") version "1.0.14.RELEASE"
	kotlin("jvm") version "1.6.21"
	kotlin("plugin.spring") version "1.6.21"
}

group = "com.example"
version = "0.0.1-SNAPSHOT"
java.sourceCompatibility = JavaVersion.VERSION_1_8

repositories {
	mavenCentral()
}

dependencies {
	implementation("org.springframework.boot:spring-boot-starter-data-mongodb-reactive")
	implementation("org.springframework.boot:spring-boot-starter-webflux")
	implementation("com.fasterxml.jackson.module:jackson-module-kotlin")
	implementation("io.projectreactor.kotlin:reactor-kotlin-extensions")
	implementation("org.jetbrains.kotlin:kotlin-reflect")
	implementation("org.jetbrains.kotlin:kotlin-stdlib-jdk8")
	implementation("org.jetbrains.kotlinx:kotlinx-coroutines-reactor")

	testImplementation("de.flapdoodle.embed:de.flapdoodle.embed.mongo:3.4.9")
	testImplementation("org.springframework.boot:spring-boot-starter-test")
	testImplementation("io.projectreactor:reactor-test")
}

tasks.withType<KotlinCompile> {
	kotlinOptions {
		freeCompilerArgs = listOf("-Xjsr305=strict")
		jvmTarget = "1.8"
	}
}

tasks.withType<Test> {
	useJUnitPlatform()
}
```

- 아래는 entity와 repository이다

```kt
@Document("users")
data class User (
    val name: String,
    val age: Int,
    var id: String? = null,
)

@Repository
interface UserRepository: ReactiveMongoRepository<User, String> {
    fun findByName(name: String): Flux<User>
}
```

- 위의 내용을 바탕으로 간단한 테스트를 작성해보자.

```kt
@Suppress("ReactiveStreamsUnusedPublisher")
@DataMongoTest
@TestPropertySource(properties = ["spring.mongodb.embedded.version=3.4.9"])
@ComponentScan(basePackages = ["com.example.querydsl.data",])
class UserTest {
    @Autowired
    lateinit var repository: UserRepository

    @BeforeEach
    fun `init before each test`(){
        repository.deleteAll().block()
    }

    @Test
    fun `test getByName`(){
        repository.save(User("u1", 1))
            .block()

        val user1 = "u1"
        val user2 = "u2"

        StepVerifier.create(repository.findByName(user1))
            .assertNext { assertEquals(user1,it.name) }
            .verifyComplete()

        StepVerifier.create(repository.findByName(user2))
            .verifyComplete()
    }
}
```

- 잘 동작하는 것을 확인할 수 있다. 그러나, query의 조건이 조금만 더 복잡해지면 spring data에서 기본으로 제공하는 query 조건으로 확인할 수 없다.

Query DSL
=========

- spring data에서 다양한 query 작업을 쉽게 해주는 기능
- **Query DSL**을 위한 몇 가지 설정을 하자

- 가장 먼저 할 일은 *annotation* 처리를 위한 **kapt** 기능을 추가하는 것이다.

```
// build.gradle.kts
plugins {
	id("org.springframework.boot") version "2.7.4"
	id("io.spring.dependency-management") version "1.0.14.RELEASE"
	kotlin("jvm") version "1.6.21"
	kotlin("plugin.spring") version "1.6.21"
  kotlin("kapt") version "1.7.10"
}
```

- *anntotationProcessor*를 추가하자

```
// build.gradle.kts
kapt {
	annotationProcessor("com.querydsl.apt.morphia.MorphiaAnnotationProcessor")
}
```

- 이제 query-dsl 기능을 추가할 것이다.

```
// build.gradle.kts
dependencies {

  // 생략

  compileOnly("org.mongodb.morphia:morphia:1.3.2" )
	implementation("com.querydsl:querydsl-mongodb")
	{
		exclude("org.mongodb", "mongo-java-driver")
	}
	implementation("com.querydsl:querydsl-apt")

	kapt("com.querydsl:querydsl-apt")
	kapt("org.springframework.boot:spring-boot-configuration-processor")

  // 생략

}
```

- user entity에 Morphia에서 적용된 *@Entity*를 추가하자. Repsitory도 *ReactiveQueryDslPredicateExecutor*를 상속받도록 하자

```
import org.mongodb.morphia.annotations.Entity
import org.springframework.data.mongodb.core.mapping.Document

@Document("users")
@Entity
data class User (
    val name: String,
    val age: Int,
    var id: String? = null,
)
```

```kt
@Repository
interface UserRepository: ReactiveMongoRepository<User, String>, ReactiveQuerydslPredicateExecutor<User>{
    fun findByName(name: String): Flux<User>
}
```

- 그 후에 빌드를 하면 *build/generated/source/kapt/main/com/example/querydsl/data* 밑에 *QUser*라는 파일이 생성되어 있는 것을 확인할 수 있다.

- 사용은 다음과 같이 할 수 있다.
  - 실제로 아래와 같이 코드를 짜면 안된다. 이렇게 하면 되는 것을 보여주기 위한 코드이다.

```kt
@Test
fun `test getByName 2`(){
    repository.save(User("u1", 1))
        .block()

    val user1 = "u1"
    val user2 = "u2"

    val pred1 = QUser.user.name.eq(user1)
    val pred2 = QUser.user.name.eq(user2)

    StepVerifier.create(repository.findAll(pred1))
        .assertNext { assertEquals(user1,it.name) }
        .verifyComplete()

    StepVerifier.create(repository.findAll(pred2))
        .verifyComplete()
}
```
