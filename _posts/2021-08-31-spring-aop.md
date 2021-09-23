---
title: "Spring AOP"

toc: true
toc_label: "go page"
toc_icon: "book"
toc_sticky : true

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
따라서 AOP는 로깅, 트랜잭션, 보안 등 꼭 필요하지만 핵심기능이 아닌 부가기능을 따로 관리하여 모듈화 할 수 있게 하는 것을 말한다.   

# AOP Terminology
AOP에 사용 되는 용어 정리이다.   
![Program_Execution]({{ site.baseurl }}/assets/images/study/aop/Program_Execution.webp)    

1. Aspect : 
   로깅, 트랜잭션 관리와 같은 여러 곳에서 사용하는 작업 클래스이다. Aspect는 Spring XML에 정의된 클래스이거나 @Aspect 로 정의한 클래스 이다.
   
2. Advice : 
   Aspect가 무엇을 언제 할 지를 정의한다.   
   **Note:**   
   Aspect는 다섯 종류의 Advice를 가진다.   
   {: .notice--info}   
   
    * before : 대상이 호출되기 전에 Advice 기능을 수행한다.
    * after : 대상이 호출된 후에 Advice 기능을 수행한다.
    * after-returning : 대상이 성공적으로 호출된 후에 Advice 기능을 수행한다.
    * after-throwing : 대상이 예외를 던진 후에 Advice 기능을 수행한다.
    * around : Advice가 대상 작업을 감싸서 호출 전과 호출 후에 할 기능을 정의한다.   
    
3. Join Point : 
   Advice를 적용할 수 있는 곳을 Join Point 라고 한다.
   즉 애플리케이션 실행에 Aspect를 끼워 넣을 수 있는 지점을 말한다. 이러한 조인 포인트는 메소드 호출 지점이나 예외 발생, 필드 값 수정 등이 있다.    
   
4. PointCut : 
   Aspect가 어디서 Join Point 할 지를 말한다.   
   간단하게 클래스나 메소드 명을 직접 사용해도 되고 정규표현식이나 다양한 연산자를 이용하여 동적으로 결정 할 수도 있다.   
   
5. Introduction :
   기존 클래스에 코드 변경 없이도 새 메소드나 새 멤버 변수를 추가하는 기능이다.   
   
6. Weaving : 
    타깃 객체에 Aspect를 적용해서 새로운 프록시 객체를 생성하는 절차이다.   



# AOP Terminology
