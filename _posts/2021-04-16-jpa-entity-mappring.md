---
title:  "JPA Entity and Mapping Annotation"

categories: 
  - jpa
tags:
  - jpa
  - study
  - entity
last_modified_at: 2021-04-16T00:40:00-00:00
---

## What is JPA Entity?
Entity는 JPA진영에서 Database의 테이블의 한 로우 해당하는 객체이다.   
Entity로 인해 객체와 테이블이 Mapping되어 관리되는 ORM의 핵심이라고 볼 수 있다.      

## How to use Entity
Java에서 클래스 위에 annotation으로 @Entity를 선언하면 DB의 테이블로 자동 매핑이 된다.   

![entity-table-mapping]({{ site.baseurl }}/assets/images/study/entity-table-mapping.png)    

Entity class의 한 객체는 테이블의 한 튜플과 같다.   
![Tuple-entityRow]({{ site.baseurl }}/assets/images/study/Tuple-entityRow.png)    

## Requirements for Entity Classes
[공식 문서 : https://docs.oracle.com/javaee/7/tutorial/persistence-intro001.htm#BNBQA](https://docs.oracle.com/javaee/7/tutorial/persistence-intro001.htm#BNBQA)

1. @Entity annotation이 있어야 한다.   
2. Class는 public or protected 이어야 하고 no-argument constructor (매개변수 없는 생성자함수)를 가져야 한다.   
3. Class, method, persistent instance variables 에 final 선언을 해서는 안된다.   
    👏**Tip: java에서 final 선언!**   
    1. Class : 상속 받지 못한다.   
    2. method : Override 금지   
    3. persistent instance variables : table과 매핑되는 변수에는 final 금지 !   
    (@Transient 하면 매핑 안되고 순수 java 에서 쓰는 변수가 된다. 이것만 final 가능하다.)   
4. Entity instance가 detached 된 상태에서 session bean 같은 인터페이스 형태로 전달되는 경우 직렬화 가능한 인터페이스를 구현해야 한다.   
5. Entity class는 entity와 non-entity class를 모두 상속 받을 수 있으며 non-entity class는 entity class를 상속받을 수 있다.   
   👏**Tip: non-entity class**

    🔗Non-entity
    [https://www.logicbig.com/tutorials/java-ee-tutorial/jpa/non-entity-super-class.html](https://www.logicbig.com/tutorials/java-ee-tutorial/jpa/non-entity-super-class.html)   
    non-entity class는 abstract class나 concrete class이다.
    @Entity 되어있는 class 를 상속받더라도 @Entity 가 없으면 concrete class 이고 추상클래스는 당연히 non-entity class이다.
    non-entity class는 영속화 될 수 없고 EntityManager나 쿼리 작업에서 사용 할 수 없다. 그리고 테이블과 관련된 annotation과 mapping 정보가 무시된다.   

6. Persistent instance variables 는 private, protected이거나 package-private로 선언되어야 한다. (Entity class의 method로만 접근 가능)   
이유 : 그래야 proxies에서 interception이이 가능하기 때문에   

## JPA annotation Type
[공식 문서 : https://docs.oracle.com/javaee/7/api/javax/persistence/package-summary.html](https://docs.oracle.com/javaee/7/api/javax/persistence/package-summary.html)   

Jpa는 간단한 annotation과 속성들 정의로 Table과 매핑을 한다.  annotation 의미를 알아보자.   
### Example
```java
@Entity
@Table(name="TEAM")
public class Team {
    @Id
    private Long id;
    @Column(name = "teamName", unique=true, nullable=true, length=50)
    private String name;
    @ManyToOne(targetEntity = Member.class, optional = false)
    private Member member;
}
```
### @Entity
Target(value=TYPE)   
ORM의 핵심 annotation이고 속성으로 name을 줄 수 있다. ( 😀default name은 class name)   

### @Table
Target(value=TYPE)   
| Type 			| Element 			| Description               | Default           | Optaional     |
|:--------------|:------------------|:--------------------------|:------------------|:--------------|
|String         |name               |테이블 명                   |                    |✔              |
|Index[]        |indexes            |테이블의 인덱스               |                   |✔              |
|String         |catalog            |테이블의 카탈로그              |                   |✔              |
|String         |schema             |테이블의 스키마               |                   |✔              |
|UniqueConstraint[]|uniqueConstraints|유일성 제약조건              |                   |✔              |


### @Id
Target(value={METHOD,FIELD})   
Entity와 매핑되는 Table의 primary key를 말한다. 사용 가능한 Type으로는 String, java.util.Date, java.sql.Date, java.math.BigDecimal, java.math.BigInteger 가 있다.   

