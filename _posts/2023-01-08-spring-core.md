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

![spring-overview]({{ site.baseurl }}/assets/images/study/spring/spring-overview.png)

스프링의 특징으로는 다음과 같습니다.
1. Dependency injection : Spring Framework는 종속성 주입을 사용하여 응용프로그램의 개체 간 종속성을 관리하므로 응용프로그램을 보다 쉽게 테스트하고 수정할 수 있습니다.
2. Aspect-oriented programming : Spring Framework는 측면 지향 프로그래밍(AOP)을 지원합니다. 이를 통해 교차 관심사(예: 로깅 또는 보안)를 정의하고 비즈니스 로직을 복잡하게 만들지 않고 애플리케이션에 적용할 수 있습니다.    
3. Data access and transaction management : Spring Framework는 데이터 액세스 및 트랜잭션 관리를 지원하므로 데이터베이스 및 기타 데이터 소스에 저장된 데이터와 함께 작동하는 애플리케이션을 보다 쉽게 구축할 수 있습니다.   
4. MVC web framework : Spring 프레임워크에는 모델 뷰 컨트롤러 설계 패턴을 사용하여 웹 애플리케이션을 쉽게 구축할 수 있는 웹 MVC 프레임워크가 포함되어 있습니다.   
5. Lightweight : Spring Framework는 가볍고 최소의 설치 공간을 가지고 있어 다양한 애플리케이션에 쉽게 통합할 수 있습니다.   
6. Testability : Spring Framework는 통합 테스트, 모의 객체 및 테스트 컨텍스트 프레임워크를 지원하여 매우 테스트 가능하도록 설계되었습니다.   
7. Flexibility : Spring Framework는 매우 유연하며, 애플리케이션에 필요한 기능만 사용할 수 있는 모듈식 설계입니다.   

## Spring ioc container
![spring-ioc-container]({{ site.baseurl }}/assets/images/study/spring/spring-ioc-container.png)
스프링 컨테이너는 내부에 존재하는 애플리케이션 빈의 생명주기를 관리합니다.
스프링에서 관리하는 자바 객체들의 공간을 말합니다.   
스프링 컨테이너에서 관리하는 자바 객체를 빈(Bean)이라고 하는데, 스프링 컨테이너에서는 이 빈의 생성부터 소멸까지를 개발자 대신 관리해주는 곳이라고 할 수 있습니다.   
즉, Bean 생성, 관리, 제거 등의 역할을 담당합니다.   
DI Container, IOC Container, Spring Container 다 같은 뜻입니다.

![application-context]({{ site.baseurl }}/assets/images/study/spring/application-context.png)
**Note:** ApplicationContext   
ApplicationContext 가 일반적으로 ioc container에 해당되며
애플리케이션을 구성하는 빈의 생명주기를 관리하고 종속성 주입, 트랜잭션 관리 및 메시징과 같은 서비스를 담당합니다.
{: .notice--info}  

```java
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
		MessageSource, ApplicationEventPublisher, ResourcePatternResolver {
    •••    
}
```

* 싱글톤 빈
![di-container]({{ site.baseurl }}/assets/images/study/spring/di-container.png)   
IOC 컨테이너에 있는 빈들은 기본적으로 싱글톤 입니다. (싱글톤 아니게 설정 시 생명주기를 컨테이너에서 관리해 주지 않습니다.)
따라서 빈들은 모두 무상태성으로 관리해야 합니다.
*  특정 클라이언트가 값을 변경할 수 있는 필드가 있으면 안됩니다.
*  가급적 읽기만 가능해야 합니다.
*  필드 대신에 자바에서 공유되지 않는, 지역변수, 파라미터, ThreadLocal 등을 사용해야 합니다.
*  스프링 빈의 필드에 공유 값을 설정하면 정말 큰 장애가 발생할 수 있습니다.

### Bean의 등록방법
스프링은 다양한 Bean 등록방법을 지원합니다.   
![beanFactory]({{ site.baseurl }}/assets/images/study/spring/ac-beanFactory.png)      

1. @Component 어노테이션으로 등록합니다.
```java
@Component
public class MyBean1 {
    public void print() {
        System.out.println("MyBean1.print : " + this.getClass().getName());
    }
}
```

2. @Configuration의 @Bean 어노테이션으로 등록합니다.
```java
@Configuration
public class BasicConfiguration {
    @Bean
    public MyBean2 myBean2() {
        return new MyBean2();
    }
}
```

3. xml 설정으로 등록 할 수 있습니다.   
```xml
<beans>
  <context:annotation-config/>
  <bean class="com.example.springbasic.MyBean3"/>
</beans>
```

## Bean 의 생성과정
Bean의 생성과 소멸은 Spring Container에 의해 이루어집니다. 그 과정을 알아봅시다.
데이터베이스 커넥션 풀이나, 네트워크 소켓처럼 애플리케이션 시작 시점에 필요한 연결을 미리 해두고,   
애플리케이션 종료 시점에 연결을 모두 종료하는 작업을 진행하려면, 객체의 초기화와 종료 작업이 필요합니다.
스프링 Bean도 위와 같은 원리로 초기화 작업과 종료 작업이 나눠서 진행됩니다.   
간단하게 말하면 객체 생성 -> 의존관계 주입이라는 라이프사이클을 가집니다.

즉, 스프링 빈은 의존관계 주입이 다 끝난 다음에야 필요한 데이터를 사용할 수 있는 준비가 완료됩니다.

![spring-bean-container]({{ site.baseurl }}/assets/images/study/spring/spring-bean-container.png)
스프링 컨테이너 에서 Component-Scan을 통해 Bean을 등록합니다. 
기본 Bean 이름은 @Component면 클래스 이름 (첫 글자 소문자) @Bean이면 메서드 명이 됩니다.      

![spring-bean-container2]({{ site.baseurl }}/assets/images/study/spring/spring-bean-container2.png)
위와 같이 @Bean 어노테이션이 붙어있는 메서드로부터 Bean을 만들어 컨테이너에 저장합니다.   

![spring-bean-container3]({{ site.baseurl }}/assets/images/study/spring/spring-bean-container3.png)   
이 단계에서 객체의 생성이 일어납니다. 
* 생성자 주입 : 객체의 생성과 동시에 의존관계 주입이 일어남
* setter, field 주입 : 객체의 생성 -> 의존관계 주입으로 라이프 사이클이 나눠져 있습니다.

그래서 위와 같이 코드에 작성한 의존관계를 보고 `IoC 컨테이너에서 의존성 주입을 해줍니다.`   

* Bean life cycle
<pre>
스프링 컨테이너 생성 -> 스프링 빈 생성 -> 의존관계 주입 -> 초기화 콜백 -> 사용 -> 소멸 전 콜백 -> 스프링 종료
</pre>

`스프링은 의존관계 주입이 완료되면 스프링 빈에게 콜백 메소드를 통해서 초기화 시점을 알려주는 다양한 기능을 제공합니다.` 또한 `스프링은 스프링 컨테이너가 종료되기 직전에 소멸 콜백`을 제공합니다.   

### Bean life cycle callback
스프링은 크게 3가지 방법으로 빈 생명주기 콜백을 지원합니다.   
- @PostConstruct, @PreDestroy 애노테이션 지원 (추천)
- 인터페이스(InitializingBean, DisposableBean)
- xml에 초기화 메소드, 종료 메소드 지정

1. 애노테이션 (추천)
```java
@Component
public class MyBean1 {
    @PostConstruct
    public void init() {
        System.out.println("MyBean1.init 시작");
    }

    @PreDestroy
    public void destroy() {
        System.out.println("MyBean1.destroy 종료");
    }
}
```

2. 인터페이스 구현
```java
@Component
public class MyBean2 implements InitializingBean, DisposableBean {
    @Override
    public void destroy() {
        System.out.println("MyBean2.destroy 종료");
    }

    @Override
    public void afterPropertiesSet() {
        System.out.println("MyBean2.afterPropertiesSet 시작");
    }
}
```

3. xml설정
```xml
<beans>
    <bean id="myBean" class="com.example.MyBean" init-method="init" destroy-method="cleanup"/>
</beans>
```

### EventListener 
스프링 기동 시 ApplicationEvent를 상속받은 개체들이 
- ApplicationStartingEvent - 수신기 및 초기화자 등록을 제외한 모든 처리 전에 응용프로그램 실행이 시작될 때 게시됩니다.
- ApplicationEnvironmentPreparedEvent - 컨텍스트에서 사용할 환경을 알고 있지만 컨텍스트가 생성되기 전에 게시됩니다.
- ApplicationPreparedEvent - 새로 고침이 시작되기 직전에 게시되지만 빈 정의가 로드된 후에 게시됩니다.
- ContextRefreshedEvent - 응용프로그램 컨텍스트가 초기화되거나 새로 고쳐질 때 게시됩니다.
- ApplicationReadyEvent - 응용 프로그램이 요청을 처리할 준비가 되었을 때 게시됩니다. (스프링이 구동 된 이후 실행)
- ContextStartedEvent - 응용 프로그램 컨텍스트가 start() 메서드를 사용하여 시작될 때 게시됩니다.
- ContextStoppedEvent - stop() 메서드를 사용하여 응용 프로그램 컨텍스트가 중지될 때 게시됩니다.
- ContextClosedEvent - close() 메서드를 사용하여 응용 프로그램 컨텍스트를 닫을 때 게시됩니다.
이러한 이벤트는 특정 순서로 게시되지만 사용되는 컨텍스트에 따라 순서가 달라질 수 있습니다.   
Spring 애플리케이션에서 사용자 지정 이벤트 및 수신기를 정의할 수도 있습니다.      



## Reference
[https://docs.spring.io/spring-framework/docs/current/reference/html/core.html](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html)   
[https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html](https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html)   

