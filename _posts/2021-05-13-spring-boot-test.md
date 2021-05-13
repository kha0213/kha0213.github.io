---
title:  "Spring Boot Test"

categories:
  - spring
tags:
  - springBoot
  - test
last_modified_at: 2021-05-13T00:40:00-00:00
---

# Spring Boot Test



# Error Log
```java
@ExtendWith(SpringExtension.class) //junit4 의 RunWith대신
@SpringBootTest
@WebMvcTest(controllers = HelloController.class)
class HelloControllerTest {
    @Autowired
    private MockMvc mvc;

    @Test
    public void hello연결테스트() throws Exception {
        String hello = "hello";

        mvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string(hello));
    }
}
```
Junit4책을 보고 Junit5로 만들었다. 하지만 위와 같이 하면 에러가 발생한다.   
💻console   
```markdown 
Configuration error: found multiple declarations of @BootstrapWith for test class [com.study.awsboot.web.HelloControllerTest]: [@org.springframework.test.context.BootstrapWith(value=org.springframework.boot.test.context.SpringBootTestContextBootstrapper), @org.springframework.test.context.BootstrapWith(value=org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTestContextBootstrapper)]
java.lang.IllegalStateException: Configuration error: found multiple declarations of @BootstrapWith for test class
```
읽어보니 @BootstrapWith가 중복되어 에러가 발생한다는 것이다. @SpringBootTest와 @WebMvcTest 둘 다 @BootstrapWith를 가지고 있었다. 여기선 MVC테스트이니 @SpringBootTest 을 주석처리하니 에러가 사라졌다.


