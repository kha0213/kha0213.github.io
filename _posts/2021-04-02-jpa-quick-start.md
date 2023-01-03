---
title:  "What is JPA"

categories: 
  - jpa
tags:
  - jpa
  - study
last_modified_at: 2021-04-02T00:40:00-00:00
---

## What is JPA?
JPA는 Java Persistence API의 약자이다.   
자바진영의 ORM 기술 표준(인터페이스) 이다.   
(TIP : JPA는 인터페이스고 구현체는 보통 Java진영에선 Hibernate를 쓴다.)    
기존 EJB의 Entity Bean을 대체하는 기술이다.   

<br>

**Note:** ORM 이란? Object Relational Mapping 즉 객체와 관계형 데이터베이의 매핑이다.     
객체는 객체대로 설계   
관계형 데이터베이스는 관계형 데이터베이스대로 설계   
ORM 프레임워크가 중간과정에서 매핑   
{: .notice--info}   

<br>

다음 4가지 영역으로 구성되어 있다.   



### 1. The Java Persistence API
javax.persistance 패키지로 정의 된 API 그 자체를 말한다.   
[javax.persistance docs](https://docs.oracle.com/javaee/7/api/javax/persistence/package-summary.html)   
### 2. The query language
JPQL (Java Persistence Query Language)   
SQL이 (Structured Query Language) 테이블을 대상으로 질의하는 것이라면 JPQL은 객체를 대상으로 쿼리를 질의한다.   


[쿼리문 비교]   
```sql
-- SQL
SELECT * FROM MEMBER WHERE ID LIKE 'A%';
```
```java
//JPQL
String jpql = "select m from Member m where m.id like 'A%'";
```

### 3. The Java Persistence Criteria API
java 에서 sql 쿼리를 만들 때 String이 아닌 객체화 하여 IDE 도움을 받을 수 있게 한 것을 말한다.   
``` java
// criteriaQuery example
criteriaQuery.select(root)
  .where(root.get("title")
  .in(titles));
```
IDE의 도움으로 compile시점 또는 build 시점에서 error를 확인 할 수 있어 좋다.   
(다만 Criteria API는 너무 복잡하고 직관적이지 않아 Query Dsl의 사용 빈도가 더 높다.)   
### 4. Object/relational mapping metadata
객체와 관계형 데이터 베이스를 mapping 하기 위하여 필요한 metadata를 말한다.   
(클래스와 테이블, 속성 및 열, 연결 및 외래 키, Java 유형 및 SQL 유형 등)   

<br>

## Why Use JPA?
객체지향적으로 개발하기 위해서이다.   
기존의 mybatis의 개발은 결국 sql문에 의해 CRUD하기 때문에 Java 객체를 SQL로, SQL을 자바 객체로 변환하는 과정을 반복해야 했다.   
직관적으로 이해하기도 힘들고 결국 데이터가 중요하기 때문에 RDB에 의존적인 개발을 하여야 했다.   
결국 설계부터 객체를 테이블에 맞추어서 모델링 하기 때문에 Java의 특성인 객체지향의 장점을 살리기 어렵다.   
JPA는 API로 RDB와 매핑하기 때문에 개발자는 CRUD보다 비즈니스로직에 집중할 수 있다.   

<br>

## How JPA Works   
![jpa-structured]({{ site.baseurl }}/assets/images/study/jpa-structured.png)   
JPA는 JDBC의 반복적인 작업과 DataSource의 connect작업을 대부분 추상화 하여 간단한 메서드로 작업이 가능하게 하였다.   
이제 java 개발자는 jpa만 사용하여 crud의 대부분을 할 수 있다.   
**Note:** 물론 통계 쿼리 같은 복잡한 쿼리는 sql문을 위해 jpa에서도 JDBC와 mybatis를 같이 사용 가능하다.
{: .notice--info}    

<br>

![jpa-mapping]({{ site.baseurl }}/assets/images/study/jpa-mapping.png)   
개발에서 상당히 오랜시간 걸리는 지루한 작업이 객체와 관계형 데이터베이스 매핑이다.   
id, password, name이 있는 객체라 하더라도
JPA에서 기본적인 CRUD는 1~2 줄이면 끝나는데 mybatis는 파일도 나누어져 있고 훨씬 많은 작업을 해야 한다.   

* JPA
``` java
String memberId = "123a";
Member member = jpa.find(Member.class, memberId);
```
<br>
* mybatis
```java
String memberId = "123a";
memberService.findById(memberId);
//MemberServiceImpl Class
public Member findById(String memberId) {
    return dao.selectOne("memberQuery.findById",memberId);
}
```
```xml
<!-- memberQuery.xml -->
<select id="findById" parameterType="String" resultType="Member">
  SELECT * FROM MEMBER WHERE ID = #{id}
</select>
```
![jpa-hibernate-structured]({{ site.baseurl }}/assets/images/study/jpa-hibernate-structured.png)   
![jpa_works]({{ site.baseurl }}/assets/images/study/how_jpa_works.png)   
DB와 직접적인 연결을 하는건 JDBC API이다. JPA는 JDBC AP를 의존하므로 개발자는 JAVA Application에서 간단한 JPA를 호출함으로 JDBC와 Query문을 해결할 수 있다.   

* JPA 역할   
1. Entity 분석
2. SQL 생성
3. JDBC API 사용
4. 패러다임 불일치 해결   



## TIP
ORM 프레임워크는 객체와 관계형데이터베이스를 매핑하는 것이기 때문에 결국 둘 다 잘 알아야 한다.   
하지만 제대로 공부하기만 한다면 훨씬 생산성 좋고 반복작업을 줄여주는 훌륭한 프레임워크가 될 것이다.

## Reference

[https://docs.oracle.com/javaee/7/tutorial/partpersist.htm](https://docs.oracle.com/javaee/7/tutorial/partpersist.htm)  
[https://www.vogella.com/tutorials/JavaPersistenceAPI/article.html](https://www.vogella.com/tutorials/JavaPersistenceAPI/article.html)    
[https://www.tutorialspoint.com/jpa/jpa_quick_guide.htm](https://www.tutorialspoint.com/jpa/jpa_quick_guide.htm)   
[Heee's 블로그 https://gmlwjd9405.github.io/2019/08/03/reason-why-use-jpa.html](https://gmlwjd9405.github.io/2019/08/03/reason-why-use-jpa.html)    
[https://velog.io/@modsiw](https://velog.io/@modsiw/JPAJava-Persistence-API%EC%9D%98-%EA%B0%9C%EB%85%90)   

