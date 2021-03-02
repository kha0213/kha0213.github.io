---
title:  "Spring Boot Banner"
header:
  teaser: "/assets/images/portfolio/crowd-fund-teaser.png"
categories: 
  - study
tags:
  - spring-boot
  - study
  - banner
last_modified_at: 2021-03-12T20:57:00-00:00
---

## Spring Banner Docs
[https://docs.spring.io/spring-boot/docs/](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-banner)   

### Overview
Spring Boot Application 이 실행 될 때 보이는 banner에 대해 알아보겠습니다.   
   
   
#### 1. banner.txt
* resources 밑에 banner.txt를 만들면 바로 text 배너가 적용이 됩니다.   

↓ banner.txt   
```console
■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
■ Hello~                            ■
■ Young Long Spring Boot            ■
■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
```


![banner]({{ site.baseurl }}/assets/images/study/banner1.png)

* banner.txt에서 사용 가능한 변수 (일부 변수는 MANIFEST.MF가 있어야 한다.)   
    * application.version : ${application.version}    
    * application.formatted-version : ${application.formatted-version}   
    * spring-boot.version : ${spring-boot.version}   
    * spring-boot.formatted-version : ${spring-boot.formatted-version}   


![banner]({{ site.baseurl }}/assets/images/study/banner2.png)
   

#### 2. application.properties 설정
* application.properties에 banner 위치를 변경 할 수도 있습니다.   
↓ application.properties   
```yaml   
spring.banner.location=classpath:/custom-banner.txt
```
* 이미지로 배너를 만들 수 있습니다.
↓ application.properties   
```properties
spring.banner.image.location=classpath:long-banner.jpg
spring.banner.image.width=100
spring.banner.image.height=100
spring.banner.image.margin=0
spring.banner.image.invert=1
```
   

![alt]({{ site.baseurl }}/assets/images/study/banner3.png)
#### 3. SpringApplication banner config   
1. SpringApplication에서 배너가 보이지 않게 할 수도 있습니다..
↓ Study01Application.java   
```java
SpringApplication app = new SpringApplication(Study01Application.class);
app.setBannerMode(Banner.Mode.OFF);
app.run(args);
```

2. banner.txt없이 배너 생성.
만약 app과 banner.txt 둘 다 설정하면 banner.txt로 적용됩니다.   
↓ Study01Application.java   
```java
SpringApplication app = new SpringApplication(Study01Application.class);
app.setBanner(new Banner() {
    @Override
    public void printBanner(Environment environment, Class<?> sourceClass, PrintStream out) {
        out.println("============");
        out.println("Create Banner!");
        out.println("============");
    }
});
app.run(args);
```
