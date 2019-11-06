# 3.SpringBootMVC

## 1.Spring Web MVC

### 1.MockMVC

- @WebMvcTest를 사용해서 자동으로 bean을 등록해 주기 때문에 bean에 등록된 기능을 바로 사용할 수 있음

```code
* UserControllerTest.java

@WebMvcTest(UserController.class)
public class UserControllerTest {

  @Autowired
  MockMvc mockMvc;

  @Test
  public void hello() throws Exception {
    mockMvc.perform(get("/hello"))
        .andExpect(status().isOk())
        .andExpect(content().string("hello"));
  }
}

* UserController.java
@RestController
public class UserController {

  @GetMapping("/hello")
  public String hello() {
    return "hello";
  }

}
```  

## 2.Spring Boot MVC

### 1.HttpMessageConverters

- HTTP 요청 본문을 객체로 변경하거나, 객체를 HTTP응답 본문으로 변경할 때 사용.
  - @RequestBody, @ResponseBody와 함께 사용
  - HttpMessageConverters를 통해서 값이 리턴될 때 Composition(객체) type 일 경우 기본은 JSON으로 리턴, String/int와 같은 경우 StringMessageConverter가 사용됨
  - RestController를 사용할 경우 bean에 등록된 viewResolver를 찾지 않고, 바로 MessageConverter를 사용하게 됨

```code
* UserControllerTest.java

@Test
public void createUser_JSON() throws Exception {
String userJson = "{\"username\":\"bk\", \"password\":\"1234\"}";
mockMvc.perform(post("/users/create")
    .contentType(MediaType.APPLICATION_JSON)
    .accept(MediaType.APPLICATION_JSON)
    .content(userJson))
    .andExpect(status().isOk())
    .andExpect(jsonPath("$.username", is(equalTo("bk"))))
    .andExpect(jsonPath("$.password", is(equalTo("1234")))
    );
}

* UserController.java

  @PostMapping("/users/create")
  public User create(@RequestBody User user){
    return user;
  }

* User.java

public class User {

  private Long id;
  private String username;
  private String password;

  //set-get generate
}

```

### 2.ContentNegotiatingViewResolver

- ViewResolver중의 하나. 들어오는 요청의 accept header에 따라 응답이 달라짐.
  - JSON으로 요청을 받고, XML로 응답하는 코드
  - 실행시 HttpMediaTypeNotAcceptableException 메시지가 나타난다면, MediaType을 처리할 HttpMessageConverter가 없는것.
  - 기본은 HttpMessageConvertersAutoConfiguration이 처리함
  - <https://mvnrepository.com/artifact/com.fasterxml.jackson.dataformat/jackson-dataformat-xml> dependency 추가
  
  ```code
  * UserControllerTest.java

  @Test
  public void createUser_XML() throws Exception {
    String userJson = "{\"username\":\"bk\", \"password\":\"1234\"}";
    mockMvc.perform(post("/users/create")
        .contentType(MediaType.APPLICATION_JSON) //요청은 JSON
        .accept(MediaType.APPLICATION_XML) //응답은 XML
        .content(userJson))
        .andExpect(status().isOk())
        .andExpect(xpath("/User/username").string("bk"))
        .andExpect(xpath("/User/password").string("1234")
        );
  }

  ``

### 3.Static resource support

- SpringBoot Web MVC 가 제공하는 정적 리소스 지원기능
  - 정적 리소스 맵핑 “ /**”
- 기본 리소스 위치
  - classpath:/static
  - classpath:/public
  - classpath:/resources/
  - classpath:/META-INF/resources
  - 예) “/hello.html” => /static/hello.html
- spring.mvc.static-path-pattern: 맵핑 설정 변경 가능
- spring.mvc.static-locations: 리소스 찾을 위치 변경 가능
- 지정된 static경로부터만 읽게 할 경우

```code
* application.properties

spring.mvc.static-path-pattern=/static/**
```  

- WebMvcConfigurer의 addResourceHandlers 로 커스터마이징 가능  

```code
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
  registry.addResourceHandler("/m/**")
    .addResourceLocations("classpath:/m/")
    .setCachePeriod(20); //별도의 캐싱옵션 지정
```

### 4.웹 JAR

- jquery나 bootstrap을 jar로 dependency에 추가하여 리소스를 참고할 수 있도록 할 수 있음
  - <https://mvnrepository.com/artifact/org.webjars.bower/jquery/3.4.1>

  ```code
    <script src="webjars/jquery/3.4.1/dist/jquery.min.js"></script>
    <script>
    $(function() {
        console.log("ready!");
    });
    </script>
  ```

### 5.Spring HATEOS

- Hypermedia As The Engine Of Application State
- 서버: 현재 리소스와 연관된 링크 정보를 클라이언트에게 제공한다.
- 클라이언트: 연관된 링크 정보를 바탕으로 리소스에 접근한다.
- 연관된 링크 정보
  - Relation
  - Hypertext Reference)
- <https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-hateoas/2.2.0.RELEASE> dependency 추가

```code
* SampleControllerTest.java

@WebMvcTest(SampleController.class)
public class SampleControllerTest {

  @Autowired
  MockMvc mockMvc;

  @Test
  public void helloSample() throws Exception {
    mockMvc.perform(get("/helloSample"))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$._links.self").exists());
  }

}

* SampleController.java

@RestController
public class SampleController {

  @GetMapping("/helloSample")
  public EntityModel helloSample() {
    Hello hello = new Hello();
    hello.setPrefix("hey, ");
    hello.setName("bk");

    EntityModel helloEntityModel = new EntityModel<>(hello);
    helloEntityModel.add(linkTo(methodOn(SampleController.class).helloSample()).withSelfRel());

    return helloEntityModel;
  }

}

```

- EntityModel에 담긴 값을 확인하면 관련된 링크정보가 리소스에 담겨있고, 클라이언트도 해당 리소스를 사용할 수 있게 됨 -> HATEOS의 기본

- Object Mapper 제공 (객체 <-> JSON)
  - ObjectMapper가 내장되어있음
  - Custom이 필요한 경우 spring.jackson.* / Jackson2ObjectMapperBuilder 사용

- LinkDiscovers 제공
  - xPath를 확장해서 만든 HATEOS용 Client API
  - 클라이언트 쪽에서 링크 정보를 Rel 이름으로 찾을때 사용할 수 있는 XPath 확장 클래스

### 6.CORS

- Cross-Origin Resource Sharing
- Origin?
  - URI 스키마 (http, https)
  - hostname (testurl.com, localhost)
  - 포트 (8080, 18080)
- SpringBoot에서는 @CrossOrigin으로 간단하게 설정을 할 수 있게 됨

- 해당 method만 허용하는 경우

```code

@CrossOrigin(origins = "http://localhost:18080")
@GetMapping("/helloSample")
  public EntityModel helloSample() {
    Hello hello = new Hello();
    hello.setPrefix("hey, ");
    hello.setName("bk");

    EntityModel helloEntityModel = new EntityModel<>(hello);
    helloEntityModel.add(linkTo(methodOn(SampleController.class).helloSample()).withSelfRel());

    return helloEntityModel;
  }


```

- 별도의 설정파일을 만드는 경우

```code
* WebConfig.java

@Configuration
public class WebConfig implements WebMvcConfigurer {

  @Override
  public void addCorsMappings(CorsRegistry registry) {
    registry.addMapping("/**")
        .allowedOrigins("http://localhost:18080");
  }
}

```  
