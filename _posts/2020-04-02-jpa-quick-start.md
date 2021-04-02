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
* ORM 이란? Object Relational Mapping 즉 객체와 관계형 데이터베이의 매핑이다.   

### Why Use JPA?
객체지향적으로 개발하기 위해서입니다.   
기존의 mybatis의 개발은 결국 sql문에 의해 CRUD하기 때문에 Java 객체를 SQL로, SQL을 자바 객체로 변환하는 과정을 반복해야 했다.   
직관적으로 이해하기도 힘들고 결국 데이터가 중요하기 때문에 RDB에 의존적인 개발을 하여야 했다.   
결국 설계부터 객체를 테이블에 맞추어서 모델링 하기 때문에 Java의 특성인 객체지향의 장점을 살리기 어렵다.
JPA는 API로 RDB와 매핑하기 때문에 개발자는 CRUD보다 비즈니스로직에 집중할 수 있다.




## Reference
[참고 : https://www.tutorialspoint.com/jpa/jpa_quick_guide.htm](https://www.tutorialspoint.com/jpa/jpa_quick_guide.htm)
[참고 : Heee's 블로그 https://gmlwjd9405.github.io/2019/08/03/reason-why-use-jpa.html](https://gmlwjd9405.github.io/2019/08/03/reason-why-use-jpa.html)   
