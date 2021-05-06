---
title:  "JPA Composite Primary Key"

categories:
  - jpa
tags:
  - jpa
  - study
  - jpa primary key
last_modified_at: 2021-05-03T00:40:00-00:00
---

# JPA Composite Primary Key
JPA에서 Primary Key가 단일 키 일때는 @ID annotation으로 쉽게 가능하였다.   
하지만 실무에서는 복합키인 경우도 많고 JPA에서는 이 기능을 지원하기 때문에 학습해 보았다.   
크게 두 가지 방식이 있는데 @IdClass를 사용하는 방법과 @EmbeddedId를 사용하는 방법에 대해 알아보겠다.   

[참고 : 복합키의 4가지 조건 ](https://www.programmergirl.com/jpa-hibernate-identifiers/)   
복합키를 구성 하기 위한 필수 조건이 있다.   
1. @EmbeddedId or @IdClass 의 annotation을 붙여야한다.
2. public 의 no-args constructor 가 있어야 한다.
3. Serializable 를 implement 받아야 한다.
4. equals() 와 hashCode() method를 override해야 한다. (Id Class도)   
* (복합키의 일부 컬럼에 @GeneratedValue는 사용하지 못한다. 고 나와있지만 실제로 TEST 시 사용 가능한 경우도 있었다. )

## 1. @IdClass 
[참고 : https://www.objectdb.com/java/jpa/entity/id](https://www.objectdb.com/java/jpa/entity/id)   
[참고 : https://www.objectdb.com/java/jpa/persistence/retrieve](https://www.objectdb.com/java/jpa/persistence/retrieve)    
[우아한 기술 블로그 : https://woowabros.github.io/experience/2019/01/04/composit-key-jpa.html](https://woowabros.github.io/experience/2019/01/04/composit-key-jpa.html)    

간단히 Long ID와 String name으로 복합 키를 구성하는 테이블 Users를 만들어 보겠다.    
```java
@Entity
@Getter @Setter
@EqualsAndHashCode // 필수
@NoArgsConstructor
@AllArgsConstructor
@IdClass(UsersId.class) // UsersId라는 식별자 클래스에 @Id 속성이 다 있어야 한다.
public class Users implements Serializable {
    @Id
    @Column(name = "ID")
    private Long id;
    @Id
    @Column(name = "NAME")
    private String name;

    private int age;
}

// UsersId Class
@Getter
@Setter
@EqualsAndHashCode
@NoArgsConstructor
@AllArgsConstructor
public class UsersId implements Serializable {
    private Long id; // Users Class의 필드 이름이 꼭 같아야 한다.
    private String name; // Users Class의 필드 이름이 꼭 같아야 한다.
}

//main
public static void logic(EntityManager em, EntityTransaction tx) throws Exception {
    Users user1 = new Users(1L,"userA",31);
    Users user2 = new Users(2L,"userB",39);
    Users user3 = new Users(3L,"userC",17);
    em.persist(user1);
    em.persist(user2);
    em.persist(user3);

    UsersId id2 = new UsersId(2L, "userB");
    final Users findUser = em.find(Users.class, id2);
    System.out.println("findUser = " + findUser);
}

// 결과 : findUser = Users(id=2, name=userB, age=39)
```

IdClass의 필드 명과 Entity의 @Id 필드명이 다르면 Property of @IdClass not found in entity 에러가 발생한다.   
IdClass 보다는 EmbeddedId 방법이 조금 더 객체지향적이다. Embeddable class를 재사용 할 수 있기 때문이다.   

## 2. @EmbeddedId
Embeddable Object를 Id로 사용하는 방식이다.    
```java
@Embeddable
public class UsersId implements Serializable {
    private Long id;
    private String name;
}
```
UsersId는 마찬가지로 구성하였고 위의 4가지 조건은 만족해야한다.    
@Embeddable만 추가하면 ID역할 뿐만 아니라 다른 엔티티에서 필드로도 활용 가능하다.   
```java
public class Users implements Serializable {
    // 식별자 클래스를 별도로 만들 것이다.
//    @Id
//    @Column(name = "ID")
//    private Long id;
//    @Id
//    @Column(name = "NAME")
//    private String name;

    @EmbeddedId
    private UsersId usersId;

    private int age;
}
```
Users 객체도 복잡한 구성 필요 없이 Id로 사용할 필드에 @EmbeddedId annotation만 추가하면 끝이다.   

```java
// Main
public static void logic(EntityManager em, EntityTransaction tx) throws Exception {
    UsersId id1 = new UsersId(1L, "userA");
    Users user = new Users(id1,33);
    em.persist(user);

    final Users findUser = em.find(Users.class, id1);
    System.out.println("findUser = " + findUser);
}
```
결과 : findUser = Users(usersId={1,userA}, age=33)   

[참고 : 복합키로 매핑시 주의사항](https://medium.com/@SlackBeck/jpa-joincolumns-%EC%82%AC%EC%9A%A9%EC%8B%9C-%EC%A3%BC%EC%9D%98-%EC%82%AC%ED%95%AD-7bc22b98ed9b)   

## 3. How use IdClass and EmbeddedId
둘 다 장단점이 있다.   
IdClass는 @Id annotation을 여러 개 사용하고 또 식별자 클래스에 각각의 필드명을 맞춰줘야 한다는 점.   
EmbeddedId는 JPQL에서 객체 탐색을 더 많이 한다는 점이 있다.   
해당 식별자 클래스가 의미가 있어 다른곳에서 사용하면 EmbeddedId, 의미 없이 복합키로서 존재하면 IdClass를 사용하면 될 것이다.   
```java
// IdClass
select u.name from Users u
// EmbeddedId
select u.UsersId.name from Users u
```

# Error log
* Property of @IdClass not found in entity ••• [field name] 
  - IdClass에서의 field name과 entity의 field name이 다를 때 나타나는 에러
* org.hibernate.PropertyAccessException: Could not set field value [Value] value by reflection [IdClass]
  - IdClass에서 field가 entity와 같은 타입의 변수여야 한다. (이름도 같아야 한다.)
* No part of a composite identifier may be null   
  - 복합키 중 하나가 null일 때 발생한다.