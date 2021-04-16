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
|String         |name               |테이블 명                    |                    |✔            |  
|Index[]        |indexes            |테이블의 인덱스               |                   |✔             |    
|String         |catalog            |테이블의 카탈로그             |                   |✔              |    
|String         |schema             |테이블의 스키마               |                   |✔              |   
|UniqueConstraint[]|uniqueConstraints|유일성 제약조건              |                   |✔              |   


### @Id
Target(value={METHOD,FIELD})   
Entity와 매핑되는 Table의 primary key를 말한다.   
사용 가능한 Type으로는 String, java.util.Date, java.sql.Date, java.math.BigDecimal, java.math.BigInteger 가 있다.   
| Type 			| Element 			|   
|:--------------|:------------------|
|String         |name               |   

### @Column
Target(value={METHOD,FIELD})   
테이블의 Column과 매핑되는 정보를 입력하는 annotation이다.   

### @ManyToOne
Target(value={METHOD,FIELD})    

### @JoinColumn
Target(value={METHOD,FIELD})   
기본적인 Column의 속성은 다 가지고 있다.   


### @GeneratedValue
Target(value={METHOD,FIELD})   
기본키 생성전략을 사용한다. (기본은 AUTO이다.)   


### @Enumerated
Target(value={METHOD,FIELD})   
enum 타입을 사용한다는 주석이다.   
enum은 ordinal로 하는 경우가 거의 없고 String으로 해줘야한다.   
(기본이 ordinal이라서 항상 바꿔줘야 한다.)   

### @Temporal
Target(value={METHOD,FIELD})   
날짜 타입 (java.util.Date, java.util.Calendar) 을 매핑할 때 사용한다.   
👏**Tip: TemporalType는 DATE, TIME, TIMESTAMP 3가지 이다.**   

### @Lob
데이터베이스 BLOB, CLOB 타입과 매핑한다. 필드 타입이 문자면 CLOB 나머지는 BLOB이다.

### @Transient
Entity에서 매핑하지 않는 필드이다. 데이터 베이스에 저장, 조회가 되지 않는다.

### @Access
JPA가 엔티티에 접근하는 방식을 나타낸다. 필드접근, 프로퍼티 접근(Getter)이다. (기본은 @ID를 설정한 방식이다.)

## Jpa&hibernate config setting (persistence.xml)
jpa의 세팅방법에 대해서 알아보자.   
[Jpa 공식 문서 : https://docs.oracle.com/cd/E16439_01/doc.1013/e13981/cfgdepds005.htm#TopLinkJDBC](https://docs.oracle.com/cd/E16439_01/doc.1013/e13981/cfgdepds005.htm#TopLinkJDBC)   
[Hibernate 공식 문서 : https://docs.jboss.org/hibernate/core/3.5/reference/en/html/session-configuration.html](https://docs.jboss.org/hibernate/core/3.5/reference/en/html/session-configuration.html)   

### Example
```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence"
             version="2.1">
    <persistence-unit name="simple-jpa-application">
        <properties>
            <!-- 필수 속성 -->
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/jpastudy"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>

            <!-- 옵션 -->
            <!-- 콘솔에 하이버네이트가 실행하는 SQL문 출력 -->
            <property name="hibernate.show_sql" value="true"/>
            <!-- SQL 출력 시 보기 쉽게 정렬 -->
            <property name="hibernate.format_sql" value="true"/>
            <!-- 쿼리 출력 시 주석(comments)도 함께 출력 -->
            <property name="hibernate.use_sql_comments" value="true"/>
            <!-- JPA 표준에 맞춘 새로운 키 생성 전략 사용 -->
            <property name="hibernate.id.new_generator_mappings" value="true"/>
            <!-- 애플리케이션 실행 시점에 데이터베이스 테이블 자동 생성 -->
            <property name="hibernate.hbm2ddl.auto" value="create"/>
            <!-- 이름 매핑 전략 설정 - 자바의 카멜 표기법을 테이블의 언더스코어 표기법으로 매핑
             ex) lastModifiedDate -> last_modified_date -->
            <property name="hibernate.ejb.naming_strategy" value="org.hibernate.cfg.ImprovedNamingStrategy"/>
        </properties>
    </persistence-unit>
</persistence>
```

