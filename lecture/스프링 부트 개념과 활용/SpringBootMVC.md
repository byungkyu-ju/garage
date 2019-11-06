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
  - https://mvnrepository.com/artifact/com.fasterxml.jackson.dataformat/jackson-dataformat-xml dependency 추가
  
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
