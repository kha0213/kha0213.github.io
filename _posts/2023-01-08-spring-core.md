---
title:  "Spring core"

toc: true
toc_label: "go page"
toc_icon: "book"
toc_sticky : true

classes: wide

categories: 
  - spring
tags:
  - spring
  - ioc
last_modified_at: 2023-01-08T00:10:00-00:00
---
# Spring
스프링 프레임워크(Spring Framework)는 오픈 소스 응용 프로그램 프레임워크이며 자바 플랫폼을 위한 inversion of control 컨테이너를 제공합니다.   
강력한 엔터프라이즈 애플리케이션을 구축하기 위한 포괄적인 기능 세트를 제공하며, 시간이 지남에 따라 애플리케이션을 더 쉽게 개발하고 유지 관리할 수 있도록 설계되었습니다.

![spring-overview]({{ site.baseurl }}/assets/images/study/spring/spring-overview.jpg)

스프링의 특징으로는 다음과 같습니다.
1. Dependency injection : Spring Framework는 종속성 주입을 사용하여 응용프로그램의 개체 간 종속성을 관리하므로 응용프로그램을 보다 쉽게 테스트하고 수정할 수 있습니다.
2. Aspect-oriented programming : Spring Framework는 측면 지향 프로그래밍(AOP)을 지원합니다. 이를 통해 교차 관심사(예: 로깅 또는 보안)를 정의하고 비즈니스 로직을 복잡하게 만들지 않고 애플리케이션에 적용할 수 있습니다.    
3. Data access and transaction management : Spring Framework는 데이터 액세스 및 트랜잭션 관리를 지원하므로 데이터베이스 및 기타 데이터 소스에 저장된 데이터와 함께 작동하는 애플리케이션을 보다 쉽게 구축할 수 있습니다.   
4. MVC web framework : Spring 프레임워크에는 모델 뷰 컨트롤러 설계 패턴을 사용하여 웹 애플리케이션을 쉽게 구축할 수 있는 웹 MVC 프레임워크가 포함되어 있습니다.   
5. Lightweight : Spring Framework는 가볍고 최소의 설치 공간을 가지고 있어 다양한 애플리케이션에 쉽게 통합할 수 있습니다.   
6. Testability : Spring Framework는 통합 테스트, 모의 객체 및 테스트 컨텍스트 프레임워크를 지원하여 매우 테스트 가능하도록 설계되었습니다.   
7. Flexibility : Spring Framework는 매우 유연하며, 애플리케이션에 필요한 기능만 사용할 수 있는 모듈식 설계입니다.   

## Spring ioc container
스프링 컨테이너는 내부에 존재하는 애플리케이션 빈의 생명주기를 관리합니다.
스프링에서 관리하는 자바 객체들의 공간을 말합니다.   
스프링 컨테이너에서 관리하는 자바 객체를 빈(Bean)이라고 하는데, 스프링 컨테이너에서는 이 빈의 생성부터 소멸까지를 개발자 대신 관리해주는 곳이라고 할 수 있습니다.   
즉, Bean 생성, 관리, 제거 등의 역할을 담당합니다.   

![spring-ioc-container]({{ site.baseurl }}/assets/images/study/spring/spring-ioc-container.png)

ApplicationContext 가 일반적으로 ioc container에 해당되며 
애플리케이션을 구성하는 빈의 생명주기를 관리하고 종속성 주입, 트랜잭션 관리 및 메시징과 같은 서비스를 담당합니다.

```java
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
		MessageSource, ApplicationEventPublisher, ResourcePatternResolver {
    •••    
}
```

### Bean의 등록방법
1. @Component 어노테이션으로 등록한다.
```java
@Component
public class MyBean1 {
    public void print() {
        System.out.println("MyBean1.print : " + this.getClass().getName());
    }
}
```

2. @Configuration의 @Bean 어노테이션으로 등록한다.
```java
@Configuration
public class BasicConfiguration {
    @Bean
    public MyBean2 myBean2() {
        return new MyBean2();
    }
}
```

3. xml 설정으로 등록 할 수 있다.
```xml
<beans>
  <context:annotation-config/>
  <bean class="com.example.springbasic.MyBean3"/>
</beans>
```

## Bean life cycle


## EventListener 
스프링 기동 시 ApplicationEvent를 상속받은 개체들이 
- ApplicationStartingEvent - 수신기 및 초기화자 등록을 제외한 모든 처리 전에 응용프로그램 실행이 시작될 때 게시됩니다.
- ApplicationEnvironmentPreparedEvent - 컨텍스트에서 사용할 환경을 알고 있지만 컨텍스트가 생성되기 전에 게시됩니다.
- ApplicationPreparedEvent - 새로 고침이 시작되기 직전에 게시되지만 빈 정의가 로드된 후에 게시됩니다.
- ContextRefreshedEvent - 응용프로그램 컨텍스트가 초기화되거나 새로 고쳐질 때 게시됩니다.
- ApplicationReadyEvent - 응용 프로그램이 요청을 처리할 준비가 되었을 때 게시됩니다. (스프링이 구동 된 이후 실행)
- ContextStartedEvent - 응용 프로그램 컨텍스트가 start() 메서드를 사용하여 시작될 때 게시됩니다.
- ContextStoppedEvent - stop() 메서드를 사용하여 응용 프로그램 컨텍스트가 중지될 때 게시됩니다.
- ContextClosedEvent - close() 메서드를 사용하여 응용 프로그램 컨텍스트를 닫을 때 게시됩니다.
이러한 이벤트는 특정 순서로 게시되지만 사용되는 컨텍스트에 따라 순서가 달라질 수 있습니다. Spring 애플리케이션에서 사용자 지정 이벤트 및 수신기를 정의할 수도 있습니다.



## Reference
[https://docs.spring.io/spring-framework/docs/current/reference/html/core.html](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html)   
[https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html](https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html)   

