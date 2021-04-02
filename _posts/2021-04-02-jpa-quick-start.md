---
title:  "What is JPA"

categories: 
  - jpa
tags:
  - jpa
  - study
last_modified_at: 2021-04-02T00:40:00-00:00
---

### What is JPA?
JPA는 Java Persistence API의 약자이다.   
자바의 영속적인 API란 뜻인데 간단히 말해서 자바진영의 ORM 기술 표준(인터페이스) 이다.   
(TIP : JPA는 인터페이스고 구현체는 보통 Java진영에선 Hibernate를 쓴다.)    

**Note:** ORM 이란? Object Relational Mapping 즉 객체와 관계형 데이터베이의 매핑이다.     
객체는 객체대로 설계   
관계형 데이터베이스는 관계형 데이터베이스대로 설계   
ORM 프레임워크가 중간과정에서 매핑   
{: .notice--info} 

### Why Use JPA?
객체지향적으로 개발하기 위해서이다다.   
기존의 mybatis의 개발은 결국 sql문에 의해 CRUD하기 때문에 Java 객체를 SQL로, SQL을 자바 객체로 변환하는 과정을 반복해야 했다.   
직관적으로 이해하기도 힘들고 결국 데이터가 중요하기 때문에 RDB에 의존적인 개발을 하여야 했다.   
결국 설계부터 객체를 테이블에 맞추어서 모델링 하기 때문에 Java의 특성인 객체지향의 장점을 살리기 어렵다.   
JPA는 API로 RDB와 매핑하기 때문에 개발자는 CRUD보다 비즈니스로직에 집중할 수 있다.   

### How JPA Works   
![jpa_works]({{ site.baseurl }}/assets/images/study/how_jpa_works.png)
DB와 직접적인 연결을 하는건 JDBC API이다. JPA는 JDBC AP를 의존하므로 개발자는 JAVA Application에서 간단한 JPA를 호출함으로 JDBC와 Query문을 해결할 수 있다.   

* JPA 역할   
1. Entity 분석
2. SQL 생성
3. JDBC API 사용
4. 패러다임 불일치 해결   

간단히 비교를 해봐도 설정만 간단히 하면 JPA에서는 간단한 CRUD는 1~2 줄이면 끝나는데 mybatis는 반복해서 사용하는 경우가 많이 있다.
* JPA
```java
String memberId = "123a";
Member member = jpa.find(Member.class, memberId);
```
* mybatis
```java
String memberId = "123a";
memberService.findById(memberId);
-- MemberServiceImpl Class
public Member findById(String memberId) {
    return dao.selectOne("memberQuery.findById",memberId);
}
```
```xml
<!-- memberQuery.xml -->
<select id="findById" resultType="Member">
  SELECT * FROM MEMBER WHERE MEMBERID = #{memberId}
</select>
```

### TIP
ORM 프레임워크는 객체와 관계형데이터베이스를 매핑하는 것이기 때문에 결국 둘 다 잘 알아야 한다.   
하지만 제대로 공부하기만 한다면 훨씬 생산성 좋고 반복작업을 줄여주는 훌륭한 프레임워크가 될 것이다.

## Reference
[참고 : https://www.vogella.com/tutorials/JavaPersistenceAPI/article.html](https://www.vogella.com/tutorials/JavaPersistenceAPI/article.html)   
[참고 : https://www.tutorialspoint.com/jpa/jpa_quick_guide.htm](https://www.tutorialspoint.com/jpa/jpa_quick_guide.htm)   
[참고 : Heee's 블로그 https://gmlwjd9405.github.io/2019/08/03/reason-why-use-jpa.html](https://gmlwjd9405.github.io/2019/08/03/reason-why-use-jpa.html)    
