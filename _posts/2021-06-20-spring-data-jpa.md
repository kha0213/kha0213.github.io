---
title:  "Spring Data JPA"

toc: true
toc_label: "go page"
toc_icon: "book"
toc_sticky : true

categories:
  - jpa
tags:
  - jpa
  - study
last_modified_at: 2021-06-20T00:40:00-00:00
---

# Spring Data JPA
Spring Data JPA를 사용하면 JPA 기반 Repository를 쉽게 구현할 수 있다.   
인터페이스만 작성을 하면 Spring이 자동으로 구현을 해주기 때문에 반복적인 작업이 상당히 줄어든다.   

특징으로는 다음과 같다.
1. Spring 및 JPA 기반 Repository를 위한 복잡한 기능을 지원한다.
2. Querydsl을 통한 type-safe한 JPA 쿼리를 제공한다.
3. DomainClassConverter를 통해 domain 클래스를 적절하게 변환해준다.
4. 페이징 처리, 동적 쿼리 실행의 추상화로 간단히 쓸 수 있다.
5. @Query주석이 달린 쿼리의 유효성 검사를 실행한다.
6. XML기반 엔티티 매핑도 지원한다.
7. @EnableJpaRepositories로 JavaConfig 기반 Repository를 구성한다. (SpringBoot에서는 어노테이션 없이 사용 가능하다.)

# Getting Start Spring Data JPA
## 1. 프로젝트 세팅
spring boot에 spring-data-jpa와 h2 database의 의존성을 추가한다.    
😊참고 제 프로젝트 세팅   
[spring-boot setting](https://kha0213.github.io/jpa/jpa-toy-project/#project-setting)

## 2. 엔티티 관계 설정
기본적인 엔티티를 추가한다.
![entity]({{ site.baseurl }}/assets/images/study/jpql_table.png)   

😊Teacher.java

```java
@Entity
public class Teacher {
    @Id
    @GeneratedValue
    @Column(name = "teacher_id")
    private Long id;

    private String name;

    private int age;

    @ManyToOne(fetch = LAZY)
    @JoinColumn(name = "subject_id")
    private Subject subject;
    
    // getter,setter,Constructor •••
}
```

😊Subject.java   
```java
@Entity
public class Subject {
    @Id @GeneratedValue
    @Column(name = "subject_id")
    private Long id;

    private String title;

    @OneToMany(mappedBy = "subject")
    private List<Teacher> teacherList = new ArrayList<>();
    
    // getter,setter,Constructor •••
}
```

## 3. 인터페이스 설정
Entity를 만들었으면 JpaRepository를 상속받는 인터페이스를 만들어보자.     
그 인터페이스에 규칙에 맞는 메서드 명만 지으면 구현은 Spring이 해준다.   
(작동원리는 다음 장에서 알아보자!)   

👍TIP : 이름 전략   
[공식문서 : 이름 규칙](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.query-creation)   

😊TeacherRepository.java   
```java
public interface TeacherRepository extends JpaRepository<Teacher, Long> {
    Teacher findById(long id);
    List<Teacher> findByNameContains(String name); // name을 포함하는 Teacher List 리턴
}
```

😊Test.java   
```java
@Test
void 이름으로찾기() {
    List<Teacher> teachers = teacherRepository.findByNameContains("teacher");
    for (Teacher teacher : teachers) {
        System.out.println("teacher = " + teacher);
    }
    Assertions.assertThat(teachers.size()).isEqualTo(2);
}
```

🔑query   
```sql
-- JPQL
select
    generatedAlias0 
from
    Teacher as generatedAlias0 
where
    generatedAlias0.name like :param0 escape :param1

-- SQL
select
    teacher0_.teacher_id as teacher_1_1_,
    teacher0_.age as age2_1_,
    teacher0_.name as name3_1_,
    teacher0_.subject_id as subject_4_1_ 
from
    teacher teacher0_ 
where
    teacher0_.name like ? escape ?
```

💻console   
```markdown
// 이름에 teacher만 있는 teacher조회
teacher = Teacher(id=4, name=teacherA, age=50, subject=Subject(id=1, title=math))
teacher = Teacher(id=5, name=teacherB, age=33, subject=Subject(id=2, title=english))
```
   
**Note:**
JpaRepository의 자동구현 기능을 사용하려면 @EnableAutoConfiguration 의 하위 패키지여야 한다.   
(springBoot는 @SpringBootApplication 하위 패키지면 된다)   
{: .notice--info}   

# How does Spring Data JPA Repository work?
Spring Data JPA는 JPA를 추상화하여 사용하기 간편하게 만든 것이다. 대부분의 경우 EntityManager를 직접 다루지 않는다. (물론 사용할 수도 있다.)   
JPA를 추상화한 Repository와 여러 어노테이션을 기반한 기능으로 JPA를 보다 간단하게 사용가능하다.   
![spring-data-jpa-ex]({{ site.baseurl }}/assets/images/study/spring-data-jpa-ex.png)   
위의 예제에서 JpaRepository를 상속받고 findByNameContains() 라는 메소드 명만 지었는데 실제로 동작이 되었다.   
🤔 구현체를 알아 보자.   

```java
•••
System.out.println("teacherRepository = " + teacherRepository.getClass().getName());
```

💻console   
```markdown
teacherRepository = com.sun.proxy.$Proxy83
```   
class를 보니 프록시에서 만들어 준 구현체를 사용하고 있다. 디버그로 확인해보자.
![jpaRepositoryEx]({{ site.baseurl }}/assets/images/study/capture/jpaRepositoryEx.png)    
spring boot의 기본 구현체는 SimpleJpaRepository 이다. 분석해보자.   

😊SimpleJpaRepository.java   

```java
@Repository
@Transactional( 
readOnly = true // 각각 메소드마다 @Transactional 따로 거는 듯하다. 전체적인 dirty checking 생략때문에 하는듯
)
public class SimpleJpaRepository<T, ID> implements JpaRepositoryImplementation<T, ID> {
    private static final String ID_MUST_NOT_BE_NULL = "The given id must not be null!";
    private final JpaEntityInformation<T, ?> entityInformation; // T인 엔티티 정보
    private final EntityManager em;             // 엔티티 매니저
    private final PersistenceProvider provider; // Hibernate 같은 공급자
    @Nullable
    private CrudMethodMetadata metadata; 
    private EscapeCharacter escapeCharacter; // 기본 \이다.
    
    
    // Repository 구현체에서 엔티티 매니저를 사용한다.
    public T getById(ID id) {
        Assert.notNull(id, "The given id must not be null!");
        return this.em.getReference(this.getDomainClass(), id);
    }

    •••
    
    @Transactional
    public <S extends T> S save(S entity) {
        Assert.notNull(entity, "Entity must not be null.");
        if (this.entityInformation.isNew(entity)) {
            this.em.persist(entity); // 새로운 객체면 persist 아니면 merge 한다.
            return entity;
        } else {
            return this.em.merge(entity);
        }
    }
    
    // 기본적인 Query는 Criteria를 사용한다.
    protected <S extends T> TypedQuery<S> getQuery(@Nullable Specification<S> spec, Class<S> domainClass, Sort sort) {
        CriteriaBuilder builder = this.em.getCriteriaBuilder();
        CriteriaQuery<S> query = builder.createQuery(domainClass);
        Root<S> root = this.applySpecificationToCriteria(spec, domainClass, query);
        query.select(root);
        if (sort.isSorted()) {
            query.orderBy(QueryUtils.toOrders(sort, root, builder));
        }

        return this.applyRepositoryMethodMetadata(this.em.createQuery(query));
    }
```

기본적인 JPA 구현체는 엔티티매니저를 사용하고 Criteria와 em.createQuery() 로 쿼리를 만들어 준다.   
사용자 Repository 세팅은 JpaRepositoryFactory에서 한다.

😊JpaRepositoryFactory.java   
```java
    //RepositoryFactorySupport 추상 클래스를 상속받아 JpaRepositoryFactory 에서 getTargetRepository 으로 세팅한다.
    protected final JpaRepositoryImplementation<?, ?> getTargetRepository(RepositoryInformation information) {
        JpaRepositoryImplementation<?, ?> repository = this.getTargetRepository(information, this.entityManager);
        repository.setRepositoryMethodMetadata(this.crudMethodMetadataPostProcessor.getCrudMethodMetadata());
        repository.setEscapeCharacter(this.escapeCharacter);
        return repository;
    }
```



## Reference
[공식 https://spring.io/projects/spring-data-jpa#overview](https://spring.io/projects/spring-data-jpa#overview)   
[가이드 https://spring.io/guides/gs/accessing-data-jpa/](https://spring.io/guides/gs/accessing-data-jpa/)   
[Hibernate 가이드](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html)   
# ERROR CODE   
가장 많이 하는 실수이다. test 클래스를 만들 때 @Transactional을 붙이지 않아 에러났다.
💻console   
```markdown
javax.persistence.TransactionRequiredException: 
No EntityManager with actual transaction available for current thread 
- cannot reliably process 'persist' call
•••
```
   
