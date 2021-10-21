---
description: 이 글은 '처음 배우는 스프링 부트2' (김영재 저)' 책 내용을 정리한 글입니다.
---

# 스프링 부트 테스트하기

스프링 부트에서는 각종 테스트를 위한 어노테이션 기반 기능을 제공

### @SpringBootTest

* 통합 테스트를 제공하는 기본적인 스프링 부트 테스트 어노테이션
* 애플리케이션이 실행될 때의 설정을 임의로 바꾸어 테스트 진행 가능하며, 여러 단위 테스트를 하나의 통합된 테스트로 수행할 때 적합
  * 장점 : 실제 구동되는 애플리케이션과 똑같이 애플리케이션 컨텍스트를 로드하여 테스트하므로 하고 싶은 테스트를 모두 수행 가능
  * 단점 : 애플리케이션에 설정된 빈을 모두 로드하기 때문에 규모가 클수록 느려짐
* @SpringBootTest 어노테이션을 사용하려면 @RunWith(SpringRunner.class)를 붙여서 사용해야 함

```java
@RunWith(SpringRunner.class)
@SpringBootTest(value = "value=test", properties = {"property.value=propertyTest"}, 
classes = {SpringBootTestApplication.class}, webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class SpringBootTestApplicationTests {

    @Value("${value}")
    private String value;

    @Value("${property.value}")
    private String propertyValue;

    @Test
    public void contextLoads() {
        assertThat(value, is("test"));
        assertThat(propertyValue, is("propertyTest"));
    }
}
```

* 파라미터
  * value : 테스트가 실행되기 전 적용할 프로퍼티 주입. 기존의 프로퍼티를 오버라이드
  * properties : 테스트가 실행되기 전 {key = value} 형식으로 프로퍼티 추가
  * classes : 애플리케이션 컨텍스트에 로드할 클래스 지정. 따로 지정하지 않으면 @SpringBootConfiguration을 찾아서 로드
  * webEnvironment : 애플리케이션이 실행될 때의 웹 환경을 설정

### @WebMvcTest

* MVC를 위한 테스트로 웹에서 테스트하기 힘든 컨트롤러를 테스트하는 데 적합
* 웹상에서 요청과 응답에 대해 테스트 가능, 시큐리티 혹은 필터까지 자동으로 테스트하며 수동으로 추가/삭제 가능
* @WebMvcTest 어노테이션을 사용하면 @Controller, @ControllerAdvice, @JsonComponent와 WebMvcConfigurer, HandlerMethodArgumentResolver만 로드

```java
@RunWith(SpringRunner.class)
@WebMvcTest(BookController.class)
public class BookControllerTest {

    @Autowired
    private MockMvc mvc;

    @MockBean
    private BookService bookService;

    @Test
    public void Book_MVC_테스트() throws Exception {
        Book book = new Book("Spring Boot Book", LocalDateTime.now());
        given(bookService.getBookList()).willReturn(Collections.singletonList(book));
        mvc.perform(get("/books"))
                .andExpect(status().isOk())
                .andExpect(view().name("book"))
                .andExpect(model().attributeExists("bookList"))
                .andExpect(model().attribute("bookList", contains(book)));
    }
}
```

* 인터페이스를 구현하는 구현체는 만들지 않고 목(Mock) 데이터를 이용하여 테스트
* MockMvc를 사용하면 해당 URL의 상태값, 반환값에 대한 테스트를 수행

### @DataJpaTest

* JPA 관련 테스트 설정만 로드하는 어노테이션
* 데이터소스의 설정이 정상적인지, JPA를 사용하여 데이터를 제대로 생성, 수정, 삭제하는지 등의 테스트 가능
* 내장형 데이터베이스를 사용하여 실제 데이터베이스를 사용하지 않고 테스트 데이터베이스로 테스트 가능
* 기본적으로 인메모리 임베디드 데이터베이스를 사용하지만 @ActiveProfiles, @AutoConfigureTestDatabase 어노테이션 설정으로 변경 가능
* JPA 테스트가 끝날 때마다 자동으로 테스트에 사용한 데이터를 롤백하며, 데이터베이스를 선택하여 사용 가능
* TestEntityManager 사용시 persist, flush, find 등의 기본적인 JPA 테스트 가능

```java
@RunWith(SpringRunner.class)
@DataJpaTest
public class BookJpaTest {
    private final static String BOOT_TEST_TITLE = "Spring Boot Test Book";

    @Autowired
    private TestEntityManager testEntityManager;

    @Autowired
    private BookRepository bookRepository;

    @Test
    public void Book저장하기_테스트() {
        Book book = Book.builder().title(BOOT_TEST_TITLE).publishedAt(LocalDateTime.now()).build();
        testEntityManager.persist(book);
        assertThat(bookRepository.getOne(book.getIdx()), is(book));
    }
}
```

### @RestClientTest

* REST 관련 테스트를 도와주는 어노테이션
* REST 통신의 데이터형으로 사용되는 JSON 형식의 응답에 대한 테스트 가능
* @RestController로 데이터 반환을 JSON 형식으로 하도록 설정, RestTemplate을 사용하여 비동기 요청을 처리한 후 테스트

```java
@RestController
public class BookRestController {

    @Autowired
    private BookRestService bookRestService;

    @GetMapping(path = "/rest/test", produces = MediaType.APPLICATION_JSON_VALUE)
    public Book getRestBooks() {
        return bookRestService.getRestBook();
    }
}

@Service
public class BookRestService {

    private final RestTemplate restTemplate;

    public BookRestService(RestTemplateBuilder restTemplateBuilder) {
        this.restTemplate = restTemplateBuilder.rootUri("/rest/test").build();
    }

    public Book getRestBook() {
        return this.restTemplate.getForObject("/rest/test", Book.class);
    }
}
```

```java
@RunWith(SpringRunner.class)
@RestClientTest(BookRestService.class)
public class BookRestTest {

    @Rule
    public ExpectedException thrown = ExpectedException.none();

    @Autowired
    private BookRestService bookRestService;

    @Autowired
    private MockRestServiceServer server;

    @Test
    public void rest_테스트() {
        this.server.expect(requestTo("/rest/test"))
                .andRespond(withSuccess(new ClassPathResource("/test.json", getClass()), MediaType.APPLICATION_JSON));
        Book book = this.bookRestService.getRestBook();
        assertThat(book.getTitle()).isEqualTo("테스트");
    }

    @Test
    public void rest_error_테스트() {
        this.server.expect(requestTo("/rest/test"))
                .andRespond(withServerError());
        this.thrown.expect(HttpServerErrorException.class);
        this.bookRestService.getRestBook();
    }
}
```

* 파라미터
  * @Rule : @Before, @After 어노테이션에 상관없이 하나의 테스트 메서드가 끝날 때마다 정의한 값으로 초기화
  * MockRestServiceServer : 클라이언트와 서버 사이의 REST 테스트를 위한 객체. 내부에서 RestTemplate을 바인딩

### @JsonTest

* Gson, Jackson API의 테스트를 제공
* 문자열로 나열된 JSON 데이터를 객체로 변환하여 변환된 객체값 테스트 또는 그 반대를 테스트

```java
@RunWith(SpringRunner.class)
@JsonTest
public class BookJsonTest {

    @Autowired
    private JacksonTester<Book> json;

    @Test
    public void json_테스트() throws Exception {
        Book book = Book.builder()
                .title("테스트")
                .build();

        String content = "{\"title\":\"테스트\"}";
        assertThat(this.json.parseObject(content).getTitle()).isEqualTo(book.getTitle());
        assertThat(this.json.parseObject(content).getPublishedAt()).isNull();

        assertThat(this.json.write(book)).isEqualToJson("/test.json");
        assertThat(this.json.write(book)).hasJsonPathStringValue("title");
        assertThat(this.json.write(book)).extractingJsonPathStringValue("title").isEqualTo("테스트");
    }
}
```
