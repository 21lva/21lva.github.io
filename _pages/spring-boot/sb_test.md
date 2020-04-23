---
layout: post
title:  "Test"
name: sb_test.md
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

자주 사용하는 method
-------------------

- assertEquals(a,b): 객체 a,b의 값이 일치함을 확인
- assertArrayEquals(a,b): 배열 a,b의 값이 일치함을 확인
- assertSame(a,b) : 객체 a,b가 같은 객체임을 확인. 두 객체의 레퍼런스가 동일한가를 확인
- assertTrue(a): 조건 a가 참인가 확인
- assertNotNull(a) : 객체 a가 null이 아님을 확인

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

@WebMvcTest
============

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

- - -

@DataJpaTest
============

- JPA 관련 테스트
- 데이타 source, JPA를 이용한 생성, 수정, 삭제 동작 여부 등을 테스트
- 내장형 DB를 이용하여 실제 DB를 사용하지 않고 테스트
- @Entity 가 달린 class를 scan하여 JPA repository 구성

- - -

@RestClientTest
================

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

- - -

@JsonTest

- json의 serialize, deserialize 등을 테스트할 때 사용
- BasicJsonTest가 bean으로 등록
- jackson, gson등의 classpath에 존재하면 자동으로 등록
