---
layout: post
title:  "Test"
name: sb_test.md
category: spring-boot
---

- spring boot에서는 기본적인 test 환경 제공
- spring-boot-starter-test 모듈을 이용
- [여러 테스트들](http://wonwoo.ml/index.php/post/1926)
- [참고자료](https://meetup.toast.com/posts/124)

- - -

JUnit4
==========================

자주 사용하는 Annotation
---------------------

- @Test : 선언된 메서드는 테스트를 수행하는 메소드
  - jUnit은 각각의 테스트가 서로 영향을 주지 않고 독립적으로 실행됨을 원칙으로 @Test마다 객체를 생성
- @Ignore : 선언된 메서드는 테스트를 실행하지 않음
- @BeforeClass : 테스트 전체 시작 전에 시행되는 메서드
- @Before : @Test 메서드가 실행되기 전에 반드시 실행되는 메서드. 각 테스트마다 한 번씩 실행
  - @Test메서드에서 공통으로 사용하는 코드를 @Before 메서드에 선언하여 사용
- @After가 선언된 메서드는 @Test 메소드가 실행된 후 실행. 각 테스트마다 한 번씩 실행
- @AfterClass : 전체 테스트 시행 후에 한 번 시행되는 메서드

- @RunWith(JUnit5에서는 @ExtendWith): test runner를 확장하는 방법. 스프링 테스트에서는 *@RunWith(SpringRunner.class)*로 사용.

JUnit4 vs JUnit5
-----------------

- JUnit5는 JUnit Platform, JUnit Jupiter, JUnit Vintage를 포함한다
  - JUnit Platform: JVM에 동작하는 framework. 테스트를 발견, 생성, 결과를 전달하는 인터페이스를 정의
  - JUnit Jupiter: JUnit Platform에서 정의한 인터페이스를 구현.
  - JUnit Vintage: Junit3, JUnit4 기반 테스트를 위한 기능 제공
- @ExtendWith 에서는 Extension 구현체를 지정하는데 이 때는 메타 어노테이션을 지원하고 중복사용을 허용한다. *@SpringBootTest*에는 이미 *@ExtendWith(SpringExtension.class)*를 포함한다
  - Extension: test class와 메서드의 기능을 확장. 테스트의 이벤트, 생명 주기등에 관여.


- - -

@SpringBootTest
===============

- 통합 테스트를 제공하는 기본적인 spring boot test annotation
- 여러 단위 테스트를 하나의 통합된 테스트로 수행
- 설정된 bean을 모두 load -> 느려질 수 있음

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringBootTestApplicationTests{

  @Test//이 annotaion과 관련되 bean만 로드해서 사용
  public void contextLoads(){
    //생략
  }
}
```

- @RunWith : **JUnit** 의 내장 runner가 아닌 정의된 runner class 사용
  - @SpringBootTest 를 사용하기 위해서는 SpringRunner.class(SpringJUnit4ClassRunner를 상속함) 를 사용해야 한다

- SpringBootTest의 parameters
  - value: test가 실행되기 전에 적용할 프로퍼티. 기존의 property 오버라이딩. properties와 함께 사용 불가
  - properties : 테스트가 실행되기 전에 {key=value} 형식으로 프로퍼티 추가. value와 함께 사용 불가
  - classes: applicationContext에 로드할 class를 지정. Defalut로 @SpringBootConfiguration 을 찾아서 로드
  - webEnvironment: application이 실행될 때의 web 환경을 설정. Default로는 Mock servlet로드

- 특정 환경을 이용하여 테스트하고 싶으면 **@ActiveProfiles("local")** 과 같이 적용

- - -

Slice test
===========

- layer별로 독립적으로 테스트 하기 위한 annotation

@WebFluxText
------------

- spring webflux에서 controller, router, handler가 제대로 동작하는지 확인하는 테스트
- @Controller, @ControllerAdvice, @JsonComponent, Converter(GenericConverter), WebFluxConfigurer등을 scan. 일반 @Component, @Bean 등은 scan하지 않는다.
- Webflux의 Router 방식으로 사용하면 Router가 @Bean으로 등록되는데 이때는 @Import를 통해서 Router Bean을 등록해서 사용해야 한다

```kt
@WebFluxTest
@Import(SampleRoute::class)
class SampleRouteTest
```

- controller를 등록할 때 주의할 점은 주입할 대상을 명시하지 않으면 모든 controller들이 자동으로 등록되는 데 이때 controller가 의존한 다른 bean(@Service, @Component)등은 등록하지 않아서 에러가 발생한다. 컨트롤러를 명시하고 의존 대상들은 Mock처리하던지 해야 한다.

```
// 예외 발생
@WebFluxTest
class SampleControllerTest {
    // 생략
    // 생략
}

// 정상 작동
@WebFluxTest(controllers = [SampleController::class])
class SampleControllerTest {
    @MockBean
    lateinit var service: SampleService
    // 생략
    // 생략
}
```

@WebMvcTest
------------

- MVC를 위한 테스트
- web에서 테스트하기 힘든 controller를 테스트하기에 적합
- @Controller, @WebMvcConfigurer 와 같은 MVC관련 설정만 로드


```java
//VO
@Data
public class Book{
  private String title;
  private LocalDateTime publishedAt;
}
```

```java
//Service
public interface BookService{
  List<Book> getBookList();
}
```

```java
//Controller
@Controller
public class BookController{
  @Autowired
  private BookService bookService;
  @GetMapping("/books")
  public String getBookList(Model model){
    model.addAttribute("bookList",bookService.getBookList());
    return "book";
  }
}
```

- Test를 위해서 service의 interface는 Mock을 이용한다

```java
@RunWith(SpringRunner.class)
@WebMvcTest(BookController.class)
public class BookControllerTest{
  @Autowired
  private MockMvc mvc;

  @MockBean
  private BookService bookService;

  @Test
  public void Book_MVC_Test() throws Exception{
    Book book = new Book("Spring Boot",LocalDateTime.now);
    book.setTitle("SpringBoot");
    book.setPublishedAt("SpringBoot");
    given(bookService.getBookList()).willReturn(Collections.singletonList(book));

    mvc.perform(get("/books"))
      .andExpect(status().isOk())
      .andExpect(view().name("book"))
      .andExpect(model().attributeExists("bookList"))
      .andExpect(model().attribute("bookList",contains(book)));
  }
}
```

- WebMvcTest에는 test하고자 하는 class를 넣는다
- test 대상이 아닌 service는 mock을 이용하여 대체한다
- given() 메서드를 이용하여 메서드의 값을 미리 정한다
- test는 다음 단계로 진행된다
  1. status().isOk() : HTTP 상태 값이 200인지 테스트
  2. view().name("book") : 반환되는 값의 이름이 "book"인지 테스트
  3. model().attributeExists("bookList") : "bookList"라는 attribute가 존재하는지 테스트
  4. model().attribute("bookList",contains(book)) : bookList라는 attribute가 book 객체를 포함하는지 여부 테스트


@DataJpaTest
--------------

- JPA 관련 테스트
- 데이타 source, JPA를 이용한 생성, 수정, 삭제 동작 여부 등을 테스트
- 내장형 DB를 이용하여 실제 DB를 사용하지 않고 테스트
- @Entity 가 달린 class를 scan하여 JPA repository 구성


@RestClientTest
---------------

- 클라이언트의 **REST** 관련 테스트할 때 이용
- 예상되는 JSON이 제대로 반환되는지를 테스트

```java
//controller
@RestController
public class BookRestController{
  @Autowired
  private BookRestService bookRestService;

  @GetMapping(path="/rest/test",produces=MediaType.APPLICATION_JSON_VALUE)
  public Book getRestBooks(){
    return bookRestService.getRestBook();
  }
}
```

```java
//Service
@Service
public class BookRestService{
  private final RestTemplate restTemplate;

  public BookRestService(RestTemplateBuilder restTemplateBuilder){
    //RestTemplateBuilder를 이용하여 RestTemplate 생성
    this.restTemplate = restTemplateBuilder.rootUri("/rest/test").build();
  }

  public Book getRestBook(){
    //getForObject() 메서드를 이용하여 URI에 요청을 보내고 응답을 Book 객체로 받아옴
    return this.restTemplate.getForObject("/rest/test",Book.class);
  }
}
```

- 테스트하는 클래스를 생성하자

```java
@RunWith(SpringRunner.class)
@RestClientTest(BookRestService.class)
public class BookRestTest{
  //Before, After와 관련 없이 하나의 테스트 메서드가
  //끝날때마다 rule을 적용한다
  @Rule
  public ExpectedException thrown = ExpectedException.none();

  @Autowired
  private BookRestService bookRestService;

  //client와 server 사이의 REST 테스트를 위한 객체
  //RestTemplate을 binding하여 실제로 통신이 이루어지게 함
  @Autowired
  private MockRestServiceServer server;

  //메서드 요청에 대한 응답과 기대값이 같은지를 테스트
  @Test
  public void rest_test(){
    //서버를 "/rest/test"로 요청이 오면
    //test.json 파일로 응답을 하게 설정(/test/resource에 정의되어 있음)
    this.server.expect(requestTo("/rest/test"))
                .andRespond(withSuccess(new ClassPathResource("/test.json",getClass())),MediaType.APPLICATION_JSON);

    //getRestBook()을 통해서 this.server로 부터 test.json을 받는다
    Book book = this.bookRestService.getRestBook();
    assertThat(book.getTitle()).isEqualTo("TEST");  
  }

  @Test
  public void rest_error_test(){
    this.server.expect(requestTo("/rest/test"))
      .andRepond(withServerError());
    this.thrown.expect(HttpServerErrorException.class);
    this.bookRestService.getRestBook();
  }
}
```

@JsonTest
------------

- json의 serialize, deserialize 등을 테스트할 때 사용
- BasicJsonTest가 bean으로 등록
- jackson, gson등의 classpath에 존재하면 자동으로 등록

- - -

MockMvc
========

- controller를 테스트 하기 위해서는 요청경로, 매핑,  입력값 검사 등의 기능을 하기 때문에 단위 테스트가 아닌 spring 기능을 사용하는 통합적인 테스트를 진행해야 한다
- MockMvc : MVC controller를 테스트 하기 위해 사용. 서버를 배포하지 않고도 controller에게 request를 보내고 response를 받는 역할을 한다

- 흐름
  - MockMvc는 DispathSevlet(TestDispatchSevlet)에 request
  - DispatchServlet은 request을 평소처럼 해당하는 controller의 method 호출
  - MockMvc는 controller의 결과 값(reponse)를 검사


```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
public class MyControllerTest {

    @Autowired
    public MyController myController;
    private MockMvc mockMvc;

    @Before
    public void createController() {
        mockMvc = MockMvcBuilders.standaloneSetup(myController).build();
    }

    @Test
    public void restTest() throws Exception {
        RequestBuilder reqBuilder = MockMvcRequestBuilders.get("/api/hi");
        mockMvc.perform(reqBuilder).andExpect(MockMvcResultMatchers.status().isOk()).andDo(MockMvcResultHandlers.print());
    }
}
```

아래는 controller의 모습이다

```java
@RestController
@RequestMapping("api")
public class MyController {
    private MyService myService;

    @Autowired
    MyController(MyService myservice){
        this.myService = myservice;
    }

    @GetMapping("hi")
    public String pringHi(){
        return "hi";
    }

    @GetMapping("hello")
    public String aaa(){
        return this.myService.getVo().getName();
    }

}
```

- 검증하는 부분은 다음과 같다
  - status: HTTP 상태 코드를 검증
  - header: 응답 헤더의 상태를 검증
  - cookie: 쿠키 상태를 검증
  - content: 응답 본문 내용을 검증
  - view: 컨트롤러가 반환한 뷰 이름을 검증
  - forwardedUrl: 이동 대상 경로를 검증
  - redirectedUrl: 리다이렉트 대상의 경로 또는 URL을 검증
  - model: 스프링 MVC model 상태를 검증
  - request: 서블릿 3.부터 지원되는 비동기 처리의 상태나 요청 스코프의 상태, 세션 스코프의 상태를 검증한다.

- 실행 결과 출력 방법
- log: 실행 결과를 디버깅 레벨에서 로그로 출력. logger의 이름은 org.springframework.test.web.servlet.result
- print: 결과를 출력 대상에 출력.기본값은 표준 출력

- - -

Test With Embedded Database
=======================

- integration test에서 DB를 사용하는 방법은 3가지가 있다.
  1. Mock
  2. embedded DB 사용
  3. test용 container에 DB 띄어서 사용
  4. 원격 혹은 local DB 사용

- 각각에는 장단점이 있지만, 비즈니스 로직 혹은 SQL query 등을 테스트 및 개발하기 위한 용도로는 embedded DB를 사용하는 것이 적합하다.
- 예시로 embedded mongo DB를 사용해보자. 먼저 해야할 일은 의존성을 추가하는 일이다
- 아래의 예시는 간단하게 만들었다. 코루틴 테스트를 이렇게 하면 안된다

```
testImplementation("de.flapdoodle.embed:de.flapdoodle.embed.mongo")
```

- 아래와 같이 entity, service, repository를 설정하자

```kt
@Service
class SampleService(private val sampleRepository: SampleRepository){
    suspend fun getA(name: String): User {
        return sampleRepository.findByName(name) ?: throw Exception("No user")
    }
}

@Document("users")
class User(
    val name: String,
    val age: Int,

    @Id var id: String? = null,
)

@Repository
interface SampleRepository: ReactiveMongoRepository<User, String>{
    suspend fun save(user: User): User?
    suspend fun findByName(name: String): User?
}
```

- 테스트 파일은 아래와 같이 설정하면 잘 동작하는 것을 알 수 있다.

```kt
@Suppress("ReactiveStreamsUnusedPublisher")
@ContextConfiguration(classes = [SpringdemoApplication::class, SampleService::class, SampleRepository::class])
@DataMongoTest
@TestPropertySource(properties = ["spring.mongodb.embedded.version=3.4.6"])
class SampleServiceTest {
    @Autowired
    lateinit var sampleService: SampleService
    @Autowired
    lateinit var sampleRepository: SampleRepository

    @BeforeEach
    fun `clear db`(){
        sampleRepository.deleteAll().block()
    }

    @Test
    fun `test getUser`(){
        val name = "na-1"
        val user = User(name, 11)


        runBlocking {
            sampleRepository.save(user)
            val user = sampleService.getA(name)
            assertEquals(name, user.name)
            assertEquals(11, user.age)
        }
    }
}
```

- - -

Test with MockWebServer
==============

- integration test에서 또 하나 필요한 것이 다른 서버와의 통신이다. 그러나 다른 서버가 본인이 관리하는 서버가 아니면 이러한 테스트를 진짜 서버에 전송하는 것은 좋지 못하다. 이 때 필요한 것이 **MockWebServer**이다
- 아래의 예시는 간단하게 만들었다. 코루틴 테스트를 이렇게 하면 안된다
- 먼저 의존성을 설정하자

```
testImplementation("com.squareup.okhttp3:mockwebserver")
```

- 테스트 대상은 다음과 같다
- WebClient: HTTP Request를 전송하는 클라이언트. Reactor 기반. 쓰레드, 동시성 문제업시 비동기 형태로 사용이 가능하다. 논블러킹.

```
@Service
class SampleService(private val client: WebClient){
    suspend fun send(): Boolean {
        return client.get()
            .retrieve()
            .onStatus(HttpStatus::isError){
                Mono.error(Exception("Error"))
            }
            .toEntity(String::class.java)
            .map { true }
            .awaitFirst()
    }
}
```

- 다음은 테스트 코드이다

```
@Suppress("ReactiveStreamsUnusedPublisher")
class SampleServiceTest {
    private lateinit var sampleService: SampleService
    private lateinit var webServer: MockWebServer

    @BeforeEach
    fun `init`(){
        webServer = MockWebServer()
        webServer.start()
        val client = WebClient.builder().baseUrl("http://localhost:${webServer.port}").build()

        sampleService = SampleService(client)
    }

    @AfterEach
    fun `clear`(){
        webServer.shutdown()
    }

    @Test
    fun `test truth`(){
        webServer.enqueue(MockResponse().setResponseCode(200))

        runBlocking {
            assertEquals(true, sampleService.send())
        }
    }

    @Test
    fun `test error`() {
        webServer.enqueue(MockResponse().setResponseCode(400))

        runBlocking {
            assertThrows <Exception>{
                sampleService.send()
            }
        }
    }
}
```

- request body값에 따라 다른 응답을 보이는 서버를 만들고 싶으면 다음과 같이 하면 된다

```
mockWebServer.dispatcher = object : Dispatcher() {
    override fun dispatch(request: RecordedRequest?): MockResponse {
        val bodyString = request!!.body.readUtf8()
        return MockResponse()
            .setResponseCode(
                when (bodyString.contains("Fail") || bodyString.contains("Succ") ) {
                    true -> 200
                    else -> 400
                }
            ).addHeader("Content-Type", "application/json")
    }
}
```

- - -

Test container
==========
