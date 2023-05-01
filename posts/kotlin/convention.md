---
layout: post
title:  "Kotlin convention 번역"
date:   2022-09-03 02:02:59
author: Inhyuk
category: kotlin
tags: kotlin
cover:  "/assets/instacode.png"
name: convention.md
---

[원문](https://kotlinlang.org/docs/coding-conventions.html)에 대한 번역

Source code organization
====================

Direcotry structure
-----------

- 기본적인 kotlin 프로젝트에서 공통적인 root가 있으면 제거한다.
  - ex) org.example.kotlin이 공통된 directory이면 해당 root directory를 생략한다. org.example.kotlin.network.socket -> network/socket

Source file names
-----------------

- kotlin 파일이 하나의 클래스 혹은 interface를 갖고 있으면 대상의 이름으로 파일 이름을 설정
- 여러 class혹은 최상위 선언만 갖고 있는 경우에는 파일의 요소들을 설명하는 이름으로 파일 이름 설정
- 파일 이름은 **upper camel case**(=pascal case)를 이용. ex) *ProcessDeclartions.kt*
- *Util*과 같이 의미 없는 파일 이름은 지양

Class layout
-----------

- class 내부는 다음과 같은 순서로 정렬되어야 한다
  1. Property 선언 및 초기화 block
  2. 부가적인 생성자
  3. method
  4. companion object
  5. 내부에서 쓰이지 않는 nested class

- method를 alphabet순서로 정렬하지 말자. method와 확장 메서드를 분리하지말자
- 연관되어있는 대상들은 비슷한 위치에 놓아서 읽는 사람이 위에서 아래로만 읽어도 이해할 수 있도록 한다
- 일관성 있는 순서를 결정하여 유지한다. ex) high-level이 위에 또는 low-level이 위에
- nested class는 해당 클래스를 사용하는 코드 바로 아래에 둔다.
- nested class가 내부에서 쓰이지 않고 외부에서만 쓰인다면 companion object 아래에 놓는다

Interface implementation layout
--------------

- interface를 구현하는 클래스의 내부에서 구현 method의 순서는 interface에서 method 정의들의 순서를 유지한다.

- - -

Naming rules
--------------

package, class
----------

- package 이름: lowercase만 사용. underscore 사용 금지. 복수의 단어로 이루어지는 package 이름은 지양하고 필요시 underscore없이 두 단어를 붙인다. 원하면 camel case로 써도 된다. ex) my_home -> myhome or myHome
- class & object 이름: 대문자로 시작하는 camel case. ex) MySampleObject

function, property, variable
-------

- 소문자로 시작하는 camel case 이용.
- underscore 사용 금지
- 예외적으로 factory functions(instance 생성에 사용되는 method)의 이름은 abstract class(or interface)의 이름과 같게 생성

```kt
interface Foo
class FooImpl: Foo
fun Foo(): Foo { return FooImpl() }
```

 test methods
 ----------

 - test method의 이름은 아래와 같이 생성
 - 아래의 방식은 오직 test에서만 사용하자

```kt
@Test
fun `test A returns B`(){
  //impl
}
```

 property
 --------

 - *const* property, getter가 없는 immutable 최상위 property: 대문자와 underscore로 이루어진 이름 생성

```kt
const val MAX_COUNT = 8
val NAME = "user name"
```

- 최상위 property라도 immutable이 아니면 기존의 variable처럼 적시
- enum constant의 경우에는 대문자로 이루어진 *snake case** 혹은 *upper camel case* 이용

```kt
enum class Color{
  RED, SKY_BLUE
}
enum class Food{
  Coke, TomatoPasta
}
```

- backing property: 두 property가 하나는 공개, 하나는 비공개로 되어 있어서 하나를 숨기거나 다른 기능을 추가하기 위한 property 쌍. private property는 underscore로 시작하는 동일한 이름으로 명명.

Choose good names
-----------

- class 이름은 클래스를 설명하는 명사 혹은 명사 구문.
- method 이름은 동작을 설명하는 동사 혹은 동사 구문. 해당 method가 객체를 변경하는지 return 값이 존재하는지등을 설명하는 것이 좋다. ex) sort, sorted: 전자는 in-place, 후자는 sorted된 새로운 collection 제공
- Manager와 Wrapper와 같이 의미 없는 이름은 피하자.
- 약자를 사용하는 경우
  - 약자가 2개 이하: 전부 대문자로 사용. ex) IOStream
  - 약자가 2개 초과: 앞글자만 대문자로 사용: ex) HttpInputStream

- - -

Formatting
==========

Indentation
--------

- indentation: 스페이스 4개. tab 사용 금지
- curl brace: opening 괄호를 한 줄 띄지 말고 사용. closing 괄호는 opening 되는 첫 글자와 수직으로 정렬되도록 혼자 위치 시킨다.
- semicolon은 optional. *semicolon을 사용하지 않는 경우가 많기 때문에 띄어쓰기가 중요하다*

```kt
//wrong
if(elment==null)
{

}

//right
if (elment == null) {
}
```

Horizontal whitespace
----------------

- 이항연산(ex. +,-)의 연산자는 다른 대상과 붙이지 않는다. ex) a+b -> a + b
- *range to*는 붙인다. ex) 0..i
- unary 연산자는 띄어쓰지 않는다. ex) a ++ -> a++
- control flow 키워드(if, when, for, while)는 띄어쓴다. 뒤에 붙는 brace도 띄어쓴다.

```kt
if (a + b == 1) {

}
```

- primary 생성자, method 선언, method 호출에서는 opening 소괄호를 띄지 않는다.

```kt
class A(val x: Int)

fun foo(x: Int) { ... }

fun bar() {
    foo(1)
}
```

- opeing 소괄호, 대괄호 후에는 공간을 두지 않는다. closing 소괄호, 대괄호의 앞도 마찬가지
- . 과 ?. 의 양옆에는 공간을 두지 않는다. ex) foo.bar()?.bar()
- 단문 주석처리를 위한 *//*의 뒤에는 공간을 둔다. ex) // This is a comment
- <> 괄호 양옆에는 띄우지 않는다. ex) Pair<String, String>
- :: 양 옆에 공간을 두지 않는다. ex) String::class
- nullable 타입 선언을 위한 ? 앞에 공간을 두지 않는다. ex) String?

Colon
------

- 다음과 같은 경우에 Colon *앞*에 공간을 둔다
  1. type과 super type을 구분하기 위해서 사용될 때
  2. super class의 생성자 혹은 다른 클래스의 생성자에 위임하기 위해 사용될 때
  3. object keyword 이후에 사용될 때

```kt
// case 1.
abstract class Foo<out T : Any>

// case 2
class FooImpl : Foo() {
  constructor() : this()
}

// case 3
val x = object : Foo()
```

Class headers
---------

- class의 primary 생성자는 한 줄로 쓰는 것이 원칙이나 길어질 경우에는 아래와 같이 하나의 대상이 한 줄에 나타나도록 줄을 띄어쓴다. closing 소괄호는 따로 한 줄로 쓴다. 상속을 받은 경우에는 closing 소괄호 옆에 붙인다. 다중 상속은 띄어쓴다

```kt
class Persion(id: Int)

class Persion(
  id: Int,
  name: String,
  age: Int,
) : Human(),
    KotlinMaker() {

}
```

- class header가 길어지면 클래스 opening 중괄호를 상속 대상의 마지막 혹은 그 다음줄에 쓴다.

```kt
class Persion(
  id: Int,
  name: String,
  age: Int,
) : Human(),
    KotlinMaker() {

}

class Persion(
  id: Int,
  name: String,
  age: Int,
) : Human(),
    KotlinMaker()
{

}
```

- indent할 때 4개의 space를 사용해야 한다. constructor의 property와 class 내부의 property가 같은 indentation을 갖게 해준다.

Modifier order
----------

- 여러 종류의 modifier를 사용한다면 다음과 같은 순서로 위치 시킨다.

```
public / protected / private / internal
expect / actual
final / open / abstract / sealed / const
external
override
lateinit
tailrec
vararg
suspend
inner
enum / annotation / fun // as a modifier in `fun interface`
companion
inline / value
infix
operator
data

ex)
private final lateinit var name: String
```

- annotation은 모든 modifier 앞에 놓는다

```kt
@Named("Food")
private val foo: Foo
```

- library를 작성하는 경우가 아니라면 public과 같은 redundant한 modifier는 뺀다.

Annotations
----------

- annotation들은 대상의 위에 적는다. 이 때 indentation은 대상과 같다.
- argument가 없는 복수의 annotation들은 annotation들 끼리 한 줄에 위치시켜도 된다
- argument가 없는 하나의 annotation은 대상과 같은 줄 가장 앞에 놓는다.

```kt
@Named("Food")
private val foo: Foo
@Named @JvmField
private val foo: Foo
@JvmField private val foo: Foo
```

File annotations
--------------

- file annotations는 file 주석 뒤에 놓는다. 순서는 다음과 같다
  1. file comment (if necessary)
  2. file annotation (if necessary)
  3. blank
  4. package

```kt
/* License */
@file:JvmName("FooBar")

package foo.bar
```

Functions
---------

- 함수 signature(선언문)가 한 줄로 쓸 수 없다면 다음과 같이 쓴다

```kt
fun longMethodName(
    argument: ArgumentType = defaultValue,
    argument2: AnotherArgumentType,
): ReturnType {
    // body
}
```

- 함수의 매개변수 앞에는 4개의 space를 위치시킨다.
- 한 줄로 이루어진 함수는 *expression body*를 사용한다

```kt
fun foo() = 1
```

Expression bodies
-----------

- 한 줄로 쓸 수 없는 expression body로 이루어지는 함수는 *=* 다음에 한 줄 띄고, 정규 indentation(= 4 spaces)를 위치 시킨다

```kt
fun f(x: String, y: String, z: String) =
    veryLongFunctionCallWithManyWords(andLongParametersToo(), x, y, z)
```

Properties
---------

- 간단한 read-only property는 한 줄로 쓴다

```
val isEmpty: Boolean get() = size == 0
```

- 복잡한 property의 *get*, *set*은 다른 줄에 적는다

```kt
val foo: String
    get() {

    }
    set() {

    }
```

- 초기화가 매우 긴 경우에는 *expresion bodis*에 나온 것과 비슷하게 한다.

```kt
private val defaultCharset: Charset? =
    EncodingRegistry.getInstance().getDefaultCharsetForPropertiesFiles(file)
```

Control flow statements
-------------------

- *if*나 *when*의 조건문이 한 줄로 쓸 수 없는 경우에는 다음과 같이 한다.
  - 각 조건은 한 줄로 쓰고 다음 조건이 있는 경우에 && 을 붙인다.
  - 조건문의 closing 소괄호는 본문 opening 중괄호와 같은 위치에 놓는다

```kt
if (!component.isSyncing &&
    !hasAnyKotlinRuntimeInScope(module)
) {
    return createKotlinNotConfiguredPanel(module)
}
```

- *else, catch, finally*, *do-while*의 *while* 과 같은 keyword는 앞의 closing 중괄호와 같은 줄에 놓는다

```kt
if (condition) {
    // body
} else {
    // else part
}

try {
    // body
} finally {
    // cleanup
}
```

- *when*에서 분기를 한 줄로 쓸 수 없는 경우에는 다음 조건과 한 줄 띄운다.
- 분기문이 한 줄인 경우에는 붙여쓰고, 괄호는 지양한다

```kt
private fun parsePropertyValue(propName: String, token: Token) {
    when (token) {
        is Token.ValueToken ->
            callback.visitValue(propName, token.value)

        Token.LBRACE -> { // ...
        }
    }
}

when (foo) {
    true -> bar() // good
    false -> { baz() } // bad
}
```

Method calls
-----------

- 매개변수 문장이 긴 함수를 호출하는 경우에는 다음과 같이 한다
  1. opening 소괄호 후에 한 줄 띄운다
  2. 매개변수를 여러개 묶어서 한 줄 씩 쓴다. 이 때 각 문장은 정규 indentation한다.
  3. named argument로 매개변수를 사용하는 경우에는 *=* 좌우로 공간을 놓는다.

```kt
drawSquare(
    x = 10, y = 10,
    width = 100, height = 100,
    fill = true
)
```

Wrap chained calls
------------------

- wrapping chained 호출은 *., ?.*를 호출하는 쪽의 다음 줄에 놓는다.
- chained calls을 띄우고 앞에는 4 spaces
- 첫 번째 chain은 한 줄 띄우는 것이 좋지만, 필요시 앞에 붙여도 된다.

```kt
val anchor = owner
    ?.firstChild!!
    .siblings(forward = true)
    .dropWhile { it is PsiComment || it is PsiWhiteSpace }

val anchor = owner?.firstChild!!
    .siblings(forward = true)
    .dropWhile { it is PsiComment || it is PsiWhiteSpace }
```

Lambdas
--------

- lambda 표현식에서는 중괄호 좌우로 공간이 있어야 한다.
- lambda의 매개변수 뒤에 오는 화살표는 매개변수와 띄어쓴다. body가 길어지면 화살표 다음 줄부터 본문을 쓴다.
- lambda 표현식이 하나인 경우에는 () 없이 사용한다.
- lambda에 label을 부여하는 경우에는 label과 lambda의 opening 중괄호를 붙인다

```
appendCommaSeparated(properties) { prop ->
    val propertyValue = prop.get(obj)  // ...
}

list.filter { it > 10 }

fun foo() {
    ints.forEach lit@{
        // ...
    }
}
```

- lambda의 매개변수가 한 줄로 쓰기에 너무 길면 매개변수와 화살표를 한 줄에 하나씩 쓰고 그 다음 줄부터 본문을 쓴다.

```kt
foo {
   context: Context,
   environment: Env
   ->
   context.configureEnv(environment)
}
```

Trailing commas
---------------

- trailing commas: 마지막 매개변수 뒤에 찍는 commas. trailing commas를 사용하면 다음과 같은 장점이 있다.
  - git과 같은 version control에서 ,가 찍혀서 변했다는 내용없이 변한 대상만 볼 수 있다
  - 새로운 대상을 add하거나 reorder하기 쉽다
  - 객체 초기화 같은 코드 생성을 쉽게할 수 있다. 마지막 요소에도 comma를 놓을 수 있다.
- trailing commas는 선택이다. 없어도 됨. 그런데 추천은 한다.
- Intellij IDEA에서 trailing commas를 사용하고 싶으면 **Settings/Preferences** > **Editor** > **Code Style** > **Kotlin** > **Other** 에서 **Using trailing comma** 옵션을 선택한다.

- - -

Documemtation comments
=================

- 긴 주석은 /\*\*를 맨 앞에 놓는다. 각 줄도 \*로 시작한다.

```kt
/**
 * This is a documentation comment
 * on multiple lines.
 */
```

- *@param*, *@return* 을 주석에 적지 말도록 한다. 매개변수와 반환값에 대해서 주석에 바로 적는다. *@param*, *@return* 는 매개변수와 반환값에 대한 설명이 너무 길 경우에만 사용한다.

```kt
// Avoid doing this:

/**
 * Returns the absolute value of the given number.
 * @param number The number to return the absolute value for.
 * @return The absolute value.
 */
fun abs(number: Int): Int { /*...*/ }

// Do this instead:

/**
 * Returns the absolute value of the given [number].
 */
fun abs(number: Int): Int { /*...*/ }
```

- - -

Avoid redundant contstrutors
========================

- 일반적으로 kotlin의 특정 문장이 선택적이고 IDE에서 불필요하다고 표시하면 지우는게 좋다.
- 단순히 "명료성"을 위해서 불필요한 문장을 쓰는 것을 지양하자

Unit return type
---------------

- 함수의 반환 타입이 *Unit*인 경우에는 적지 않는다


Semicolons
--------

- 꼭 필요한 경우가 아니면 빼자

String templates
------------

- 단순한 변수를 string에 포함하기 위해서 중괄호를 쓰지말자.

```
println("$name has ${children.size} children") //  name에는 중괄호를 사용하지 말자
```

- - -

Idiomatic(관용적) use of language features
========================

Immutability
----------------


- mutable 대신 immutable을 최대한 이용하자.
- 지역 변수는 최대한 *var* 대신 *val*을 사용하자
- collection을 선언할 경우에는 immutable type(*Collection, List, Set, Map*)로 선언하자.
- collection을 전달하는 factory 함수의 반환 타입은 immutable type으로 하자.

```kt
// Bad: use of mutable collection type for value which will not be mutated
fun validateValue(actualValue: String, allowedValues: HashSet<String>) { ... }

// Good: immutable collection type used instead
fun validateValue(actualValue: String, allowedValues: Set<String>) { ... }

// Bad: arrayListOf() returns ArrayList<T>, which is a mutable collection type
val allowedValues = arrayListOf("a", "b", "c")

// Good: listOf() returns List<T>
val allowedValues = listOf("a", "b", "c")
```

Default parameter values
------------------

- overload 함수보다는 default 매개변수를 제공하는 함수를 쓰자.

```kt
// Bad
fun foo() = foo("a")
fun foo(a: String) { /*...*/ }

// Good
fun foo(a: String = "a") { /*...*/ }
```

Type aliases
-----------

- 어떤 타입(함수 타입, 매개변수 타입 등)이 여러번 사용된다면 해당 타입을 *type alias* 하자

```
typealias MouseClickHandler = (Any, MouseEvent) -> Unit
typealias PersonIndex = Map<String, Person>
```

- 이름 충돌을 피하기 위해 private, internal을 선호한다면, *import ... as ...*를 이용하자

Lambda
--------------

- 다른 lambda에 nested되어 있지 않고 매개변수가 하나라면 paramter에 이름을 붙이지말고 *it*을 이용하자. 만약 다른 lambda의 nested되어있다면 해당 lambda의 매개변수에는 이름을 붙여서 사용해야한다.
- lambda에서 복수의 labeled return을 하지 않는다. 하나의 반환만 하도록 재설계하고, 재설계가 불가능하다면 lambda 대신에 익명 함수를 사용하자.
- lambda의 마지막 반환에 *return* 키워드를 붙이지 않는다.

Named arguments
--------

- 같은 primitive type의 매개변수를 여러 개 받는 함수를 호출할 때는 named argument로 호출하는 것이 좋다. 물론 그 의미가 충분하다면 사용하지 않아도 된다.

Conditional statements
------------------

- *if, when*의 결과를 *return*한다면 한 번만 쓰자

```kt
return when(x) {
    0 -> "zero"
    else -> "nonzero"
}

return if (x) foo() else bar()
```

- binary condition이면 *when* 대신 *if*를 사용하자.
- nullable Boolean은 null check 말고 boolean check만 해도 충분하다. ex) if (value == true)

Loops
------

- *for, while*과 같은 loop보다 high-order functions(e.g. filter, map)을 쓰자
- 복수의 복잡한 high-order functions를 사용한다면 각 단계에서 발생할 수 있는 cost를 고려해야한다. 너무 복잡하고 cost가 높다면 loop를 사용하는 것이 나을 수도 있다.

- open 범위에는 *until*를 사용하자

```kt
for (i in 0..n - 1) { /*...*/ }  // bad
for (i in 0 until n) { /*...*/ }  // good
```

Strings
------

- *string concatenation*보다 *string templates*를 사용하자
- \\n을 사용하는 것보다 multiline strings을 쓰자
- multiple line strings를 쓸 때는 indentation을 조심하자. 이 때 사용할 수 있는 메서드는 2개가 있다
  - trimIndent: 모든 문장의 indent를 모두 삭제
  - trimMargin: | 이전까지의 모든 indent 삭제

```kt
println("""
    Not
    trimmed
    text
    """
       )

println("""
    Trimmed
    text
    """.trimIndent()
       )

println()

val a = """Trimmed to margin text:
          |if(a > 1) {
          |    return a
          |}""".trimMargin()

println(a)

//결과
    Not
    trimmed
    text

Trimmed
text

Trimmed to margin text:
if(a > 1) {
    return a
}
```

Functions vs Properties
---------------

- 매개변수가 없는 메서드는 read-only properties와 혼용해서 쓰는 경우가 많다. 다음은 property를 쓰는 것이 좋은 상황이다.
  - throw가 필요 없는 경우
  - 계산하는 비용이 싼 경우 혹은 첫 번째 동작을 cache하는 경우
  - object의 상태가 변하지 않는 이상 같은 값을 전달하는 경우

Extension functions
----------------

- 어떤 객체에 주로 동작하는 함수가 있으면 그 객체의 확장함수로 만드는 것에 대해서 고려하자.
- API가 오염되는 것을 방지하기 위해 확장함수의 가시성을 최소화 하자. private한 지역 확장함수, 멤버 확장함수, top-level 확장함수를 이용하면 된다.

Infix functions
-------------

- 비슷한 역할을 하는 두 객체에 동작하는 함수만 *infix*로 만들자. ex) and, or, zip. add는 두 객체의 역할이 다르기 때문에 하지 않는 것이 좋다.
- 협력하는 두 대상 중 하나라도 함수에서 변경이 이루어진다면 절대로 infix로 만들지 말자.

Factory functions
-------------

- 어떤 클래스 하나를 위해서 factory 함수를 만드는 경우에는 class와 이름이 같도록 함수를 만들지 말자. *fromXXX*와 같이 factory 함수라는 것을 알 수 있도록 하자.

```kt
class Point(val x: Double, val y: Double) {
    companion object {
        fun fromPolar(angle: Double, radius: Double) = Point(...)
    }
}
```

- 같은 superclass의 생성자를 호출하는 overloaded된 여러 생성자가 있고 default argument를 이용하여 하나의 생성자로 줄일 수 없으면, 이 생성자들을 factory functions로 바꾸자.

Platform types
----------

- platform type의 표현식을 반환하는 함수나 메서드는 그 type을 명시해야한다.
- platform type의 표현식으로 초기화 되는 property는 type을 명시해야한다.
- platform type의 표현식으로 초기화 되는 지역 변수는 type을 명시하지 않아도 된다.

```kt
fun apiCall(): String = MyJavaApi.getProperty("name")

class Person {
    val name: String = MyJavaApi.getProperty("name")
}

fun main() {
    val name = MyJavaApi.getProperty("name")
    println(name)
}
```

Scope functions
----------------

- kotlin에서는 어떤 객체의 문맥에서 동작하는 block code를 제공한다. e.g. *let, run, with, apply, also*
- [Scope functions](https://kotlinlang.org/docs/scope-functions.html)를 보고 잘 사용하자.

- - -

Coding conventions for libraries
==============

- library를 만들 때는 몇 가지 지켜야할 수칙이 있다
1. member의 가시성을 신경써서 필요 없는 것을 공개하지 않도록 하자
2. 반환 타입과 property 타입을 명시하자
3. [KDoc](https://kotlinlang.org/docs/kotlin-doc.html#block-tags) comments를 이용하자.
