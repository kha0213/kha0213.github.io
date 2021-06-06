---
title:  "JPA toy project"

toc: true
toc_label: "go page"
toc_icon: "book"
toc_sticky : true

categories:
  - jpa
tags:
  - jpa
  - toy_project
last_modified_at: 2021-06-06T00:40:00-00:00
---

# 

## Reference
  
# ERROR CODE
1. h2 Database may be already in use: null •••
h2에서 8080포트를 쓰는데 springboot 내장 톰캣도 8080포트를 써서 에러
해결방법 → application.properties 에서 server.port=8181로 톰캣 서버 변경

2. A file path that is implicitly relative to the current working directory is not allowed in the database URL "jdbc:h2:tcp://localhost:9092/toyproject". Use an absolute path, ~/name, ./name, or the baseDir setting instead. [90011-200].   
intellij와 h2 db 연동 실패    
해결방법 → java bean 만들고 h2정보의 url을 ~/ 이나 ./으로 해야 한다.       

```java
@Configuration
public class H2ServerConfig {

    @Bean
    public Server h2DatabaseServer() throws SQLException {
        return Server.createTcpServer().start();
    }
}
```   

![h2setting]({{ site.baseurl }}/assets/images/study/h2setting.png)    
