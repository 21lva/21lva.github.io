---
layout: post
title:  "Gradle"
date:   2022-07-30 02:02:59
author: Inhyuk
category: gradle
tags: gradle
cover:  "/assets/instacode.png"
name: gradle.md
---

gradle
======

- 모든 종류의 소프트웨어를 위한 오픈 소스 빌드 자동화 툴.
- 특징
  - 높은 성능: 필요한 **task**들만 수행해서 오버헤드를 줄인다. cache 기능을 제공
  - JVM 기반: JDK에서 설치된 JVM 기반으로 동작
  - task로 동작: gradle은 **task**들의 DAG(Directed Acyclic Graphs)로 이루어져있다. task들은 이미 작성된 plugin과 사용자가 직접 작성하는 build script들로 나뉨. task는 Actions, Inputs, Outputs로 구성된다
  - fixed build phases: gradle은 3개의 phases로 이루어짐.
    1. Initialization: 환경을 설정.
    2. Configuration: task들의 DAG를 만들고 설정. 어떤 순서에 어떤 task들이 시작되는지 설정.
    3. ExecutionL task들을 성해진 순서에 맞게 실행

기본 구조
--------

- gradle 프로젝트를 생성하면 구조는 다음과 같다
  - setting.gradle.kts: build name과 subprojects를 설정하는 파일
  - gradle/: gradle wrapper 파일들이 존재하는 디렉토리
  - gradlew: gradle wrapper start script
  - gradle.properties: gradle 빌드할 때 필요한 환경설정. system properties, gradle properties로 나뉜다 ex) jvmargs
  - build.gradle.kts: 현재 project 빌드를 위한 스크립트. 서브 프로젝트마다 하나씩 존재

- gradle wrapper: 빌드도구를 실행할 수 있는 jar과 그 스크립트 파일을 함께 관리하는 방식. 여러 개발자들이 동일한 gradle 환경(버젼 등)을 사용할 수 있게 해주는 역할을 한다.

```
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradle.properties
├── gradlew
├── gradlew.bat
├── settings.gradle.kts
└── app
    ├── build.gradle.kts
```

#### setting.gradle.kts

```
rootProject.name = "untitled1" //project 이름
include("app") //같이 빌드 될 subproject
```

#### app/build.gradle.kts

```
import org.jetbrains.kotlin.gradle.tasks.KotlinCompile

plugins {
    kotlin("jvm") version "1.6.21"
    application //appliction을 생성하기 위한 plugin 추가
}

group = "org.example"
version = "1.0-SNAPSHOT"

repositories {
    mavenCentral() // dependency를 위한 maven repository
}

dependencies {
    testImplementation(kotlin("test"))
}

tasks.test {
    useJUnitPlatform() //test라는 task에서는 JUnitPlatform 사용
}

tasks.withType<KotlinCompile> {
    kotlinOptions.jvmTarget = "1.8" //Kotlin compile 시에 사용될 JVM version은 1.8
}

application {
    mainClass.set("MainKt") //application의 main class 정의
}
```

주요 개념
-------

- project: gradle의 빌드대상. gradle 빌드는 1개 이상의 project로 이루어짐
- task: gradle project에서의 작업 단위. ex) compile, jar 생성
- plugin: 추가적인 task들을 사용하기 위한 확장 기능. task들의 집합이라고 보면 된다. ex) java, org.springframework.boot
- dependency: 의존성 정보.
- configuration: dependency가 적용될 scope 설정. ex) compileOnly, api, implementation, runtimeOnly
- source set: 소스 파일과 리소스들의 논리적 그룹. ex) main, test

plugin
-------

- plugin에는 script plugin과 binary plugin으로 나뉜다.
  - script plugin: 빌드를 조작하는 일반적인 빌드 스크립트.
  - binary plugin: **Plugin** interface를 구현하여 프로그래밍 적으로 만들어진 클래스.
- plugin을 사용하려면 *resolve* 후에 *apply*해야함
  - resolve: plugin을 포함하는 jar를 script class의 경로에 추가하는 작업. resolve후에는 빌드 스크립트에서 plugin API 사용 가능. script plugin과 일부 binary plugin은 apply시에 자동으로 resolve됨.
  - apply: project의 *Plugin.apply(plugin)*을 통해서 실제로 사용하도록 하는 작업. binary plugin의 경우에는 *id*를 이용하여 플러그인을 적용하거나 특별한 PluginDependenciesSpec 구현체를 이용한다.

```
//apply script plugin
apply(from="script_plugin.kts")

//apply binary plugin
plugins {
    kotlin("jvm") version "1.6.21" // a core plugin
    id("com.jfrog.bintray") version "1.8.5" // a community plugin
}
```

- 특정 프로젝트에만 plugin을 적용하고 싶으면 다음과 같이 *subproject*에 적용한다.

```
//settings.gradle.kts
rootProject.name="mother"

include("child1")
include("child2")

//build.gradle.kts
plugins {
    id("org.gradle.sample.hello") version ("1.0.0") apply false //apply=false: 현재 프로젝트에는 plugin 적용하지 않고 classpath에만 넣겠다는 의미
}

subprojects {
    if (name.startsWith('child1')) {
        apply {
            plugin("org.gradle.sample.hello")
        }
    }
}
```

plugin들
---------------

#### 기본 plugin

- 공통된 작업을 위해 gradle에서 기본 제공되는 plugin
- 제공되는 task들
  - clean: 빌드 디렉토리 삭제
  - check: 빌드 검증 및 테스트
  - assemble: 배포 대상 생성
  - build: check + assemble

#### java plugin

- java application을 빌드하기 위한 기능을 추가하는 plugin
- sources sets:
  - main: 프로젝트의 production code를 위한 source set.
  - test: 테스트를 위한 source set.
- tasks: compileJava, jar, test
- comptibility: java와 관련된 property를 설정하기 위한 java extension. sourceCompatibility(java source compile에 사용될 java version. 기본값은 gradle JVM 버전), targetCompatibility(byte code가 대상으로 할 java version. 기본값은 sourceCompatibility)

#### java library plugin

- java plugin을 확장하여 java library에 관한 기능을 추가로 제공하는 plugin
- configuration: api, implementation, compileOnly, testImplementation 등 제공

#### kotlin plugin

- kotlin을 위해서 java library와 비슷한 기능을 제공하는 plugin

```
plugins {
  kotlin("jvm") version "1.7.0" //jvm을 위한 kotlin plugin
}
```

- 기본적인 sourceSets의 구조는 다음과 같다

```
project
  - src
    - main(root)
      - kotlin
      - java
```

- default convention이 아닌 다른 sourceSets를 사용하고 싶으면 바꿀 수 있다

```
sourceSets.main {
    this.java.srcDirs("src/main/myJava", "src/main/myKt")
}
```

- 더 자세한 내용은 [kotlin 공식 gradle plugin](https://kotlinlang.org/docs/gradle.html)에서 볼 수 있다.

compileOnly, api, implementation, runtimeOnly
-------------------

- Classpath: 클래스나 jar 파일이 존재하는 위치. compile time classpath, runtime classpath로 나뉜다.
  - compile time classpath: 컴파일용도로 사용되는 클래스나 jar가 위치.
  - runtime classpaht: 런타임(실행 중)에 필요한 클래스와 jar가 위치.

- compileOnly: compile time classpath에만 추가. ex) lombok
- runtimeOnly: runtime classpath에 추가. ex) DB 관련 dependency
- implementation vs api
  - A(내가 작성한 코드)->B(외부 라이브러리)->C(다른 외부 라이브러리)라고 해보자. 이 때, 컴파일 타임에는 A와 B에 대한 경로만 갖고 있으면 되지만 런타임시에는 C에 대한 경로도 갖고있어야함. 즉, 이 프로젝트는 compiletime path에는 A,B가 runtime classpath에는 C가 있어야함. 이 때 보통 *implementation(B)* 혹은 *api(B)*를 하게 되는데 둘의 기능에는 약간의 차이가 있다.
  - implementation(B): compile path에 B만 들어감. 컴파일 타임에는 C에대한 정보를 모르니 compile 시에 C에 변화가 있더라도 무시해도 된다.
  - api(B): compile path에 B와 C가 전부 들어간다. C가 수정되는 경우에 compile을 다시 해야한다.


annotation processor
--------------

[참고자료](https://tomgregory.com/annotation-processors-in-gradle-with-the-annotationprocessor-dependency-configuration/)

- complie 중에 새로운 코드을 만들 수 있게 해주는 기능
- lombok, querydsl 등에서 사용
- annotation processor들은 *javax.anntation.processing.AbstractProcessor*를 extend.

- gradle java compliation task에서는 *javac*를 통해서 compile 진행한다. 이 때, annotation processing을 위해 *--processor-path {path}*라는 옵션을 추가할 수 있다. javac는 이때 제공된 annoation processor로 class들을 생성한다. 이는 다음과 같은 과정을 통해서 이루어진다.
  1. gradle 파일(ex. build.gradle.kts)에 anntationProcessor로 명시된 파일을 읽는다.
  2. gradle이 *javac ~~~~ --processor-path {path}*호출
  3.

- 예시를 위해서 build.kotlin.kts를 아래와 같이 수정한다

```kotlin
dependencies {
    annotationProcessor("org.mapstruct:mapstruct-processor:1.5.1.Final")
}

tasks.withType<JavaCompile> {
    doFirst {
        println("AnnotationProcessorPath for $name is ${options.annotationProcessorPath?.files?:"no file"}")
    }
}
```

- 터미널에서 컴파일 해보자

```
user2@AL01983481  ~/private/gradle_example/untitled  ./gradlew compileJava
~생락~
> Task :compileJava
AnnotationProcessorPath for compileJava is [/Users/user_1/.gradle/caches/modules-2/files-2.1/org.mapstruct/mapstruct-processor/1.5.1.Final/2bdc76f0222d41dba87daba05be5c9f20029e23/mapstruct-processor-1.5.1.Final.jar]
```

- kotlin에서 사용하고자 한다면 **KAPT**를 이용하자.

```
plugins {
    kotlin("jvm") version "1.6.21"
    application
    kotlin("kapt") version "1.7.0"
}

dependencies {
    implementation("org.mapstruct:mapstruct:1.4.2.Final")
    kapt("org.mapstruct:mapstruct-processor:1.4.2.Final")
}
```

- *mapperStruct*를 이용하여 *Entity* 에서 *Dto*로 변환하는 mapper를 만들어보자.

```Kotlin
@org.mapstruct.Mapper
interface BaseMapper {
    fun toDto(baseE: BaseE): BaseDto
}

class BaseE(
    val name: String
)

data class BaseDto(
    val name: String
)
```

- 이 파일을 빌드하면 다음과 같은 파일이 *build/generated/kapt/main/BaseMapperImpl.java*로 만들어진다

```java

import javax.annotation.processing.Generated;

@Generated(
    value = "org.mapstruct.ap.MappingProcessor",
    date = "2022-07-03T16:56:46+0900",
)
public class BaseMapperImpl implements BaseMapper {

    @Override
    public BaseDto toDto(BaseE baseE) {
        if ( baseE == null ) {
            return null;
        }

        String name = null;

        name = baseE.getName();

        BaseDto baseDto = new BaseDto( name );

        return baseDto;
    }
}
```

task
----

- 먼저 **Task**를 생성해보자

```
//생성
task("sampleTask"){
    println("This is a sample task.")//이 문장은 Task를 create 할 때 실행된다
    doLast {
      println("This is a sample task.")//이 문장은 Task를 실행할 때 실행된다
    }
}

//실행
./gradlew sampleTask

//결과
> Configure project :
This is a sample task.
This is a sample task.

BUILD SUCCESSFUL in 5s
```

- 의존성 있는 task는 다음과 같이 만들 수 있다.

```
//생성
task("task1"){
    doLast {
      println("This is the task 1.")
    }
}

task("task2"){
    dependsOn("task1")
    doLast {
      println("This is the task 2.")
    }
}

//실행
./gradlew task2

//결과
> Configure project :
This is the task 1.
This is the task 2.

BUILD SUCCESSFUL in 5s
```

- 순서 있는 task는 *mustRunAfter*를 이용한다

```
//생성
task("task1"){
    println("T1")
    doLast {
        println("Task1")
    }
}.mustRunAfter("task2")

task("task2"){
    println("T2")
    doLast {
        println("Task2")
    }
}


//실행
./gradlew -q task1 task2

//결과
T1
T2
Task2
Task1

//실행. dependsOn과 달리 task2가 없어도 진행된다. 단지 있으면 정해진 순서대로 하라는 말.
./gradlew -q task1

//결과
T1
T2
Task1
```

- *defaultTasks("taskName1", "taskName2", ...)*로 지정한 Task들은 Task가 주어지지 않으면(ex. gradlew -q) 실행된다.

```
//생성
defaultTasks("task1", "task2")

task("task1"){
    doLast {
      println("This is the task 1.")
    }
}

task("task2"){
    doLast {
      println("This is the task 2.")
    }
}

//실행
./gradlew -q

//결과
This is the task 1.
This is the task 2.
```

- 특정 task가 대상일 때 어떤 동작을 실행하게 시킬 수도 있다. 이 동작을 이용하면 version이름을 변경한다든지를 할 수 있다.

```
//생성
task("task1"){
    doLast {
        println("This is the task 1.")
    }
}

task("task2"){
    doLast{
        println("This is the task 2.")
    }
}

gradle.taskGraph.whenReady {
    if(this.hasTask(":task1")){
        println("Contains Task1")
    } else {
        println("Not contains Task1")
    }

//실행
./gradlew -q task2

//결과
Not contains Task1
This is the task 2.

//실행
./gradlew -q task1

//결과
Contains Task1
This is the task 1.
```

- *onlyIf*: 어떤 조건을 만족해야만 task 수행

```
//생성
task("task1"){
    doLast {
        println("Task1")
    }
    onlyIf {
        project.tasks.any { task -> task.name == "task2" }
    }
}

//실행
./gradlew -q task1

//결과: task2가 없기 때문에 task1도 수행되지 않는다
```

- 파일 복사(*from*) 붙여넣기(*to*)

```
//생성
task("task1"){
    copy {
        from("./build.gradle.kts")
        mkdir("./tmp")
        into("./tmp/")
    }
}

//실행
./gradlew -q task1 && ll tmp

//결과
total 8
-rw-r--r--  1 user2  staff   655B  7  8 14:56 build.gradle.kts
```

- - -

Multi project
=============

- gradle의 multi project는 하나의 root project와 다수의 하위 프로젝트로 구성된다. 하위 프로젝트는 다시 다른 하위 프로젝트들을 포함한다.
- 단일 프로젝트에서의 작업 요청은 다중 프로젝트 빌드의 모든 프로젝트가 구성(생성)됨.

```
//root project
task("task1"){
    println("configure task1")
    doLast {
        println("start task1")
    }
}


//child1 project
task("sub-task1"){
    println("configure sub-task1")
    doLast{
        println("start sub-task1")
    }
}

//child2 project
task("sub-task2"){
    println("configure sub-task2")
    doLast{
        println("start sub-task2")
    }
}
```

task들을 다양한 방법으로 수행해보자.

```
//root project의 task
./gradlew -q task1
configure task1
configure sub-task1
configure sub-task2
start task1

//sub project의 task
./gradlew -q :child1:sub-task1
configure task1
configure sub-task1
configure sub-task2     //관련 없는 project인 child2도 구성됨을 알 수 있다.
start sub-task1
```

- - -

Gradle Kotlin DSL
===============

- gradle 5.0 부터 지원하기 시작한 Kotlin 기반 DSL. groovy DSL보다 약간 더 자유롭지는 않은 방식.

KAPT(Kotlin Annotation Processing Tool)
=====

- kotlin에서 **annotation** 처리를 위해서 제공하는 툴
- Hilt, Room, Databinding 기능이 필요하다면 기존의 annotationProcessor 대신에 kapt가 필요. kotlin c로 컴파일 되기 때문.

- 사용하기 위해서는 plugin 추가해야한다

```kt
plugins {
  kotlin("kapt") version "1.5.30"
}
```

- 추가 후 빌드하면 이제 kapt를 사용할 수 있다.

```kt
dependencies {
  kapt("com.google.dagger:hilt-android-compiler:$hilt_version")
```
