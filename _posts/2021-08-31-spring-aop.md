---
title: "Spring AOP"

toc: true
toc_label: "go page"
toc_icon: "book"
toc_sticky : true

sidebar:
  nav: "sidebar-study"
categories:
  - spring
tags:
  - aop
  - study
last_modified_at: 2021-08-31T00:40:00-00:00
---

# What is AOP
AOP (Aspect Oriented Programming) 란 관점지향 프로그래밍이다.   
관점지향이란 애플리케이션에서 공통되는 부가기능을 핵심 기능에서 분리하여 모듈화 할 수 있는 것을 말한다.    
![aop-concept]({{ site.baseurl }}/assets/images/study/aop/aop-concept.png)   

그림에서 로그인 서비스, 판매 서비스, 다운로드 서비스는 애플리케이션의 핵심 기능이다.    
해당 서비스를 이용 할 때 로그를 남기고 싶다고 모든 서비스에 로그를 달았다가는 나중에 로그를 변경해야 할 때 상당히 곤란한 상황에 처할 수 있다.   
따라서 AOP는 부가기능을 따로 관리하여 모듈화 할 수 있게 하는 것을 말한다.   

👍TIP : 부가기능이란?   
로깅, 트랜잭션, 보안 등 서비스에 꼭 필요하지만 핵심기능이 아닌 애플리케이션의 전 영역에서 나타날 수 있는 기능을 말한다.   
(횡단관심사라고도 불리기도 한다.)   
* 중복코드가 발생하기 쉽다.   
* 핵심 비즈니스 로직과 함께 있으면 코드가 복잡해진다.   

😊간단한 예를 들어서 알아보자.   
```java
public class MarketService {
	public void buy(int amount) {
		long startTime = System.currentTimeMillis();
		try {
			// Actual method body...
		} finally {
			long endTime = System.currentTimeMillis() - startTime;
			System.out.println("running time: " + endTime);
		}
	}
}
```
어떤 물건을 사는 메소드가 있다고 하였을 때 메소드와 이루어지는데 걸린 시간을 보고 싶으면 위와 같이 하면 된다.   
하지만 시간을 확인하는 로직은 buy라는 메소드와 어울리지도 않고 핵심 비즈니스 기능이 아닌 부가기능이며.
시간 확인을 원하는 메소드 마다 반복적으로 같은 코드가 작성될 수밖에 없다.   
이러한 핵심 비즈니스가 아닌 시간을 재는 기능을 따로 모듈화 하여 관리할 때 AOP는 유용한 기능이다.   

👍TIP : Spring AOP는 프록시 패턴을 통해 구현 한다.

# Quick Start AOP
간단한 구현 방법 확인 후 개념을 이해해보자.   
아래는 Spring boot 일 때의 적용하는 방법이다.

## 1. 의존성 주입
😊pom.xml
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

## 2. @EnableAspectJAutoProxy 적용
proxy를 적용하고자 하는 패키지에 있는 Bean 클래스에 어노테이션을 작성해 준다.    
👍TIP: (@SpringBootApplication 안에 EnableAspectJAutoProxy가 있어서 명시 안 해주어도 된다. )
```java
@EnableAspectJAutoProxy
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

## 3. AOP 적용
Spring Component에 @Aspect 어노테이션을 적용하고 AOP를 적용하면 끝이다.   
@Around나 ProceedingJoinPoint 에 대한 사용법은 아래서 알아보자.   
```java
@Aspect
@Slf4j
@Component
public class LogAspect {
    @Around("execution(* com.example.demo.service.MarketService.*(..))")
    public Object logging(ProceedingJoinPoint pjp) throws Throwable {
        log.info("start - " + pjp.getSignature().getDeclaringTypeName() + " / " + pjp.getSignature().getName());
        // 실제 target의 메서드 실행
        Object result = pjp.proceed();
        log.info("finished - " + pjp.getSignature().getDeclaringTypeName() + " / " + pjp.getSignature().getName());
        return result;
    }
}
```



# AOP Terminology
AOP에 사용 되는 개념 정리이다.   
![Program_Execution]({{ site.baseurl }}/assets/images/study/aop/Program_Execution.webp)

* Aspect : 
   로깅, 트랜잭션 관리와 같은 여러 곳에서 사용하는 작업 클래스이다. Aspect는 Spring XML에 정의된 클래스이거나 @Aspect 로 정의한 클래스 이다.

* Advice : 
   Aspect가 무엇을 언제 할 지를 정의한다.    
   
   **Note:**   
   Aspect는 다섯 종류의 Advice를 가진다.   
   {: .notice--info}   
   
    1. before : 대상이 호출되기 전에 Advice 기능을 수행한다.
    2. after : 대상이 호출된 후에 Advice 기능을 수행한다.
    3. after-returning : 대상이 성공적으로 호출된 후에 Advice 기능을 수행한다.
    4. after-throwing : 대상이 예외를 던진 후에 Advice 기능을 수행한다.
    5. around : Advice가 대상 작업을 감싸서 호출 전과 호출 후에 할 기능을 정의한다.   

* Join Point : 
   Advice를 적용할 수 있는 곳을 Join Point 라고 한다.   
   즉 애플리케이션 실행에 Aspect를 끼워 넣을 수 있는 지점을 말한다. 이러한 조인 포인트는 메소드 호출 지점이나 예외 발생, 필드 값 수정 등이 있다.    
  (spring aop에서 주로 사용하는 것은 메소드 호출 지점이다.)   
* PointCut : 
   Aspect가 어디서 Join Point 할 지를 말한다.   
   간단하게 클래스나 메소드 명을 직접 사용해도 되고 정규표현식이나 다양한 연산자를 이용하여 동적으로 결정 할 수도 있다.      

* Introduction :
   기존 클래스에 코드 변경 없이도 새 메소드나 새 멤버 변수를 추가하는 기능이다.   

* Weaving : 
    타깃 객체에 Aspect를 적용해서 새로운 프록시 객체를 생성하는 절차이다.   

😊간단한 예를 들어서 알아보자.   
애플리케이션에서 com.yl.order 패키지에 있는 모든 메서드의 시작과 끝에 로그를 출력하는 aop를 만드려고 한다.   
Aspect는 로그를 출력하는 작업을 의미하는 것이고, Advice는 around즉 대상 작업 호출 전과 호출 후를 의미하는 것이다.   
PointCut은 com.yl.order 패키지 하위에 있는 모든 메서드 호출시 이다.
(Join Point는 Advice가 적용 가능 한 모든 것을 의미하는 추상적인 개념이다.)   
그리고 spring aop는 프록시를 통해 동작하므로 실제 동작은 Weaving을 통해서 하는 것을 알 수 있다.   

# Spring AOP
각 언어마다 AOP가 있지만 Spring framework에서는 Spring AOP가 주로 사용된다.   
Spring AOP는 순수 자바로 구현되어 특별한 컴파일 과정이 필요 없다.   (AspectJ는 컴파일 과정에 바이트 코드를 삽입하여 class파일을 변경한다.)   
Spring Framework의 AOP 기능은 프록시 패턴으로 구현한다.   
(프록시 패턴이라 final class이거나 static 메서드에는 Spring AOP를 사용하지 못한다.)   
Spring IoC 컨테이너와 함께 사용된다.   
따라서 Spring AOP는 Bean객체에만 적용이 가능하다.   
그래서 AspectJ에 비해 기능이 약하나 포괄적으로 사용이 가능하고 @AspectJ 어노테이션을 통해 AspectJ도 쉽게 사용 가능하다.    

👍TIP : 우아한 Tech에 잘 정리되어 있어서 Spring AOP와 AspectJ의 차이를 확인해 보았다.   
[우아한Tech-Spring AOP](https://www.youtube.com/watch?v=Hm0w_9ngDpM)   
<table>
    <tr>
        <th></th>
        <th>Spring AOP</th>
        <th>AspectJ</th>
    </tr>
    <tr>
        <td>목표</td>
        <td>간단한 AOP 기능 제공</td>
        <td>완벽한 AOP 기능 제공</td>
    </tr>
    <tr>
        <td>join point</td>
        <td>메서드 레벨만 지원</td>
        <td>생성자, 필드, 메서드, 객체 등 모두 지원</td>
    </tr>
    <tr>
        <td>weaving</td>
        <td>런타임 시에만 가능</td>
        <td>compile-time, post-compile, load-time 제공</td>
    </tr>
    <tr>
        <td>대상</td>
        <td>Spring Container가 관리하는 Bean</td>
        <td>모든 Java Object에 가능</td>
    </tr>
</table>


# Pointcut
Pointcut 표현식에 대해 알아보자.
Spring AOP는 AspectJ 표현식과 문법이 같다. (문법만 차용해서 쓰는 것일 뿐 소스는 다르다.)
pointcut을 사용하면 Advice가 적용될 비즈니스 메서드를 쉽게 필터링 할 수 있다.


1. execution : 실행시 원하는 문법으로 조인프인트를 적용가능하다. (가장 많이 쓰인다.)

```markdown
.. : 0개 이상.   
(*) : 1개의 매개변수.   
! : 부정어   
* : 모든 것을 뜻하는 일종의 와일드카드이다. (ex) User* 이면 User로 시작하는 모든 것이다.
```

![aop-execution]({{ site.baseurl }}/assets/images/study/aop/aop-execution.png)    
위와 같은 공식을 사용하면 어떠한 패턴에도 적용가능하다. 예를 들어 확인해 보자.   

😊example
```java
// 1. 매개변수 없고 리턴타입 String, public인 hello 메서드에 적용 가능하다. (여러 클래스에서도 해당 조건을 만족하면 hello()메서드가 전부 적용된다.)
@Around("execution(public String hello())")

// 2. 매개변수가 0개 이상인 hell로 시작하는 메서드)
@Around("execution(* hell*(..))")

// 3. aopController 안의 매개변수가 1개인 모든 메서드
@Before("execution(* com.example.springmvc.restController.aopController.*(*))")

// 4. com.example.springmvc 패키지의 모든 클래스의 적용 (서브패키지는 미포함)
@After("execution(* com.example.springmvc.*.*(..))")

// 4. com.example.springmvc 패키지의 모든 클래스의 적용 (서브패키지도 포함)
@After("execution(* com.example.springmvc..*.*(..))")
```

👍TIP : 만약 해당 클래스의 자식 클래스에 같은 메서드가 있으면 다 적용 가능하다. (interface의 메서드에 적으면 구현체에 모두 적용)

2. within : 타입패턴 내에 해당하는 모든 것들을 포인트 컷 한다. 
execution의 Type 부분만 활용한다고 생각하면 된다.   
간단하니 예를 들어 알아보자.   
   
😊example      
```java
// com.example.springmvc 인터페이스의 모든 메서드
@After("within(com.example.springmvc)")


// com.example.spring으로 시작하는 패키지의 모든 메서드
@After("within(com.example.spring*.*)")

// com.example.springmvc 패키지 및 하위 패키지의 모든 메서드
@After("within(com.example.springmvc..*)")
```

👍TIP : 인터페이스를 적으면 구현체에 적용되지 않는다. execution과 문법이 조금 다르다.

3. args : 인자가 주어진 타입의 인스턴스인 조인 포인트로 매칭.
기본 문법은 execution의 args 부분과 같다.

```java
// 매개변수가 하나이고 String 으로 넘어온 모든 메서드
@After("args(String)")

// 매개변수가 하나인 모든 메서드
@After("args(*)")
```

다만 args로만 조인포인트를 설정하면 스프링 기본 빈들 중 final 클래스는 프록시를 생성하지 못해 에러가 날 수 있다.    
execution과 조합해서 사용하도록 하자.   

👍TIP : execution과 args의 차이점
execution 은 파라미터 타입이 정확하게 매칭되어야 한다. execution 은 클래스에 선언된 정보를
기반으로 판단한다.
args 는 부모 타입을 허용한다. args 는 실제 넘어온 파라미터 객체 인스턴스를 보고 판단한다.

4. bean : bean 이름으로 포인트 컷 한다.
AspectJ 문법이 아닌 spring bean 이름으로 포인트 컷 하는 문법이다. 해당 bean의 이름을 가진 클래스의 모든 메서드를 조인포인트 설정이 가능하다.   
```java
// orderService이름의 빈이거나 (대소문자 구분하니 주의하자) Repository로 끝나는 이름의 빈의 모든 메서드가 적용 대상이다.
@After("bean(orderService) || bean(*Repository)")
```


4. pointcut : @Pointcut로 미리 지정하면 여러곳에서 재활용 할 수 있다.

```java
// 포인트컷으로 패턴을 만들고 해당 메서드 명으로 다른 Advice에서 활용 가능하다.
@Pointcut("execution(* hello(..))")
public void helloPoint() {}

// @Pointcut의 메서드 명만 적어주면 된다. (다른 클래스에 있으면 경로도 필수)
@Around("helloPoint()")
```
   
**Note:**      
@Pointcut으로 사용하는 helloPoint 메서드 안의 내용은 비워 두는 것이 원칙이며 (설령 로직 있어도 실행되지 않음) Pointcut 하기 위한 메서드 명만 의미가 있다.
(메서드 바디는 의미 없음)
{: .notice--warn}




## Reference
[https://docs.spring.io/spring-framework/docs/2.5.x/reference/aop.html](https://docs.spring.io/spring-framework/docs/2.5.x/reference/aop.html)    
[우아한Tech-Spring AOP](https://www.youtube.com/watch?v=Hm0w_9ngDpM)    