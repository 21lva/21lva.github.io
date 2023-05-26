
- Spring의 Annotation 정리

- - -

### @EnableAutoConfiguration
- spring boot에서 제공해주는 auto configruation 기능을 사용하겠다는 annotation
- 주로 @SpringBootApplication을 통해서 사용한다.

### @Configuration & @AutoConfiguration

- spring boot에서 bean 생성, 설정 파일 주입등에 사용되는 기능
- @AutoConfiguration으로 설정하면 @Configuration 후에 "read" 된다.
- @AutoConfiguration에서 @AutoConfigureAfter를 이용하여 특정 @AutoConfiguration 다음에 read할 수 있게 한다
- @AutoConfiguration에서는 @ConditionalXXXXX 를 통해서 특정 조건에 따라 bean을 생성하도록 할 수 있다
- @AutoConfiguration은 주로 application 개발자가 아니라 framework 개발자가 사용한다

#### @CondtionalXXXX
- 위에서 "생성"대신 "read"라고한 이유가 있다.
- 추가로 @ConditaonlOnBean(XClass:class)라고 하면 해당 Bean이 없을 때 생성하기 때문에 순서에 맞출것 같지만 아니다. 해당 @Bean의 정의를 읽을 때 bean이 없으면 정의를 읽는다. 즉, 순서가 맞지 않으면 의도와 다르게 bean이 없다고 하면서 bean을 생성하지 않는다.
- @AutoConfigureXXX도 결국 Config 파일을 읽는 순서를 정의할 뿐이지 bean이 생성되는 순서에는 관련 없다([spring boot - @AutoConfigureAfter not working as desired - Stack Overflow](https://stackoverflow.com/questions/42284478/autoconfigureafter-not-working-as-desired))
- 

```kt
//오류가 발생하는 코드
@Configuration  
class AConfig {  
@PostConstruct  
fun f() {  
logger.info("AConfig")  
}  
  
@Bean  
@ConditionalOnBean(B::class)  
fun a(): A {  
logger.info("A")  
return A()  
}  
  
private val logger by lazy {  
LoggerFactory.getLogger(this::class.java)  
}  
}  
  
  
@Configuration  
class BConfig {  
@PostConstruct  
fun f() {  
logger.info("BConfig")  
}  
  
@Bean  
fun b(): B {  
logger.info("B")  
return B()  
}  
  
private val logger by lazy {  
LoggerFactory.getLogger(this::class.java)  
}  
}  
  
class A  
class B  
  
  
@Component  
class Test {  
@Autowired lateinit var a: A  
@Autowired lateinit var b: B  
}
```

### @Bean, @Service, @Repository, @Component

- @Component: Component scan의 대상이 되고 DI 되기 위한 대상을 표시하는 가장 기본적인 annotation.
- @Bean: @Configuration 내에서 bean을 정의할 때 사용하는 annotation.
- @Service, @Repository: @Component와 비슷한 기능(spring)을 하지만 application에서 사용할때 그 의미를 명확하게 하고 다른 라이브러리등에서 분리된 기능을 적용하기 위해서 사용.
	- @Repository: DDD에서 의미하는 repoistory, DAO등을 아우르는 persistence layer의 component에 적용하는 annotation
	- @Service: Business 로직과 관련된 작업들을 하는 component를 위한 annotation

### @ContextConfiguration

- Integration test를 위해서 application context를 load하는데 이때 로드에 필요한 정보를 설정해 xml 파일, class등을 지정할때 사용된다.
- 여러개의 ContextConfiguration중 하나만 적용 가능하다.
- 아래의 순서대로 적용한다
	1. @ContextConfiguration
	2. Nested @Configuration
	3. @SpringBootApplication의 @SpringBootConfiguration
	2. @NestedTestConfiguration

### @Transactional

- 해당 테스트 혹은 해당 테스트 클래스 이후에 DB를 rollback한다. 
- @DataJpaTest같은 annotation에 포함되어 있다.


### @BootStrapWith

- 

### @ExtendWith

- test class, test interface, test method등에 사용될 extension을 등록하는 annotation. 
- 복수의 @ExtendWith 사용 가능. 
- annotated된 parameter는 test class의 생성자, method, @BeforeAll, @BeforeEach 등의 메서드에서 사용할 수 있다.
- test의 이벤트, life cycle에 관여.
- custom extension은 BeforeAllCallback, BeforeEachCallback, BeforeTestExecutionCallback, AfterTestExecutionCallback, AfterEachCallback, AfterAllCallback을 구현해서 만들 수 있다.


```
@ExtendWith(TestExtension::class)  
class ATest {  
  
@Test  
fun a1() {  
logger.info("start test 1!")  
}  
  
@Test  
fun a2() {  
logger.info("start test 2!")  
}  
  
private val logger by lazy {  
LoggerFactory.getLogger(this::class.java)  
}  
}  
  
  
class TestExtension: BeforeTestExecutionCallback {  
override fun beforeTestExecution(context: ExtensionContext?) {  
logger.info("Before test execution!!")  
}  
  
private val logger by lazy {  
LoggerFactory.getLogger(this::class.java)  
}  
}
```

- 테스트 결과는 아래와 같다

```
2023-05-26 19:21:16.141       INFO                   [    Test worker] c.e.d.TestExtension                      : Before test execution!!
2023-05-26 19:21:16.164       INFO                   [    Test worker] c.e.d.ATest                              : start test 1!
2023-05-26 19:21:16.175       INFO                   [    Test worker] c.e.d.TestExtension                      : Before test execution!!
2023-05-26 19:21:16.178       INFO                   [    Test worker] c.e.d.ATest                              : start test 2!
```

#### SpringExtension

- SpringExtension 클래스는 BeforeAllCallback, AfterAllCallback, TestInstancePostProcessor, BeforeEachCallback, AfterEachCallback, BeforeTestExecutionCallback, AfterTestExecutionCallback을 구현한 extension이다.
- test에서 사용되는 ApplicaitonContext를 관리한다.

### @DirtiesContext

- test에서 사용하는 applicationContext의 생명주기를 결정해서 context cache을 언제 지워야하는지를 결정하는 annotation
- 특히, test 내부에서 context를 변경할때 해당 context의 변경이 언제까지 유지되어야하는지를 결정할 때 사용할 수 있다.
- 종류
	- BEFORE_CLASS: 테스트 클래스가 시작하기전에 context 재생성
	- BEFORE_EACH_TEST_METHOD: 각 테스트 메서드가 실행되기 전에 재생성.
	- BEFORE_METHOD: method에 지정하는 annotation. 해당 테스트 메서드가 실행되기 전에 재생성
	- AFTER_METHOD: method에 지정. 해당 테스트 메서드가 실행된 후에 재생성. 기본값.
	- AFTER_EACH_TEST_METHOD: 각 테스트 메서드가 실행된 후에 재생성
	- AFTER_CLASS: 테스트 클래스가 실행된 후에 context 재생성

### @Import

- Component class(주로 @Configuration)를 import하도록 지정하는 annotation
- Configuration위에 Configuration을 Import하면서 계층화된 Configuration을 형성할 수 있다.

### @Inherited & @Repeatble
- @Inherited: 해당 annotation이 서브 클래스에도 상속된다.
- @Repeatable: 해당 annotation을 여러번 적용 가능하게 만든다. 


