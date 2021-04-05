---
title:  "JPA Entity Manager"

categories: 
  - jpa
tags:
  - jpa
  - study
last_modified_at: 2021-04-05T00:40:00-00:00
---

## What is the Entity Manager?
[참고 https://docs.jboss.org/hibernate/entitymanager/3.6/reference/en/html_single/](https://docs.jboss.org/hibernate/entitymanager/3.6/reference/en/html_single/)   
[API https://docs.oracle.com/javaee/7/api/javax/persistence/EntityManager.html](https://docs.oracle.com/javaee/7/api/javax/persistence/EntityManager.html)   
EntityManager는 EntityManagerFactory에서 생성되며 Entity를 관리하는 객체이다.    
특정 작업의 데이터베이스에 접근하는데 사용된다. (TIP. begin으로 시작하고 commit 혹은 rollback으로 끝나는)   
Entity를 create, delete, select, update 등 쿼리하는데 사용된다.   

**Note:** 
EntityManager는 thread safe하지 않아서 (동시성 이슈) 공유하면 안된다. (하나의 작업당 하나의 EntityManager 사용)   
{: .notice--info}   
   
### Entity Manager Example
``` java
EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("jpa-study");
EntityManager entityManager = entityManagerFactory.createEntityManager();
// 보통 예외처리 하여 에러시 commit()이 아닌 rollback를 한다.
entityManager.getTransaction().begin();

Student student = new Student("young long");
// persist로 영속화 한다. (쓰기 지연 sql로 DB에 저장된다.)
entityManager.persist(student);
entityManager.getTransaction().commit();
entityManager.close();
entityManagerFactory.close();
```

### Entity Manager is thread safe in spring?

```java
// Spring에서는 @PersistenceContext로 의존성 주입을 한다.   
@Repository
public class ExampleRepositoryImpl implements ExampleRepository {
    @PersistenceContext
    private EntityManager em;
    
    @Override
    public void save(Member member) {
        em.persist(member);
    }
}
```

Spring에서는 EntityManager를 Proxy를 통해 감싸고 EntityManager 메소드 호출 시 마다 Proxy를 통해 EntityManager를 생성 하여 Thread-Safety를 보장 한다.   
[참고 Yoo Young-mo blog](https://medium.com/@SlackBeck/spring-container%EB%8A%94-jpa-entitymanager%EC%9D%98-thread-safety%EB%A5%BC-%EC%96%B4%EB%96%BB%EA%B2%8C-%EB%B3%B4%EC%9E%A5%ED%95%A0%EA%B9%8C-1650473eeb64#.8ejrwpwnw)   


## What is the Entity Manager Factory?
Entity Manager Factory은 Entity Manager 인스턴스화를 지원하는 것이다.   
Entity Manager의 생성과 DB연결을 효율적으로 할 수 있도록 도와준다.   

**Note:**
Entity Manager Factory는 특정 데이터베이스에 맞게 구성하며 만드는 비용이 크기 때문에 DB당 하나씩만 생성한다.   
{: .notice--info}   

### Entity Manager Factory Example
``` java
// Persistence.xml의 설정으로 불러온다
EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpa-study");
```

## Reference

[https://www.objectdb.com/api/java/jpa/EntityManagerFactory](https://www.objectdb.com/api/java/jpa/EntityManagerFactory)   
[https://www.objectdb.com/api/java/jpa/EntityManager](https://www.objectdb.com/api/java/jpa/EntityManager)   

