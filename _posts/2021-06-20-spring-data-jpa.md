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
### 1-1 Spring
1. pom.xml에 spring-data-jpa와 h2 database의 의존성을 추가한다.     
2. xml 또는 java config 로 base-package를 지정한다.   
<br>   
   
😊root-context.xml   
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:jpa="http://www.springframework.org/schema/data/jpa"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
    https://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/data/jpa
    https://www.springframework.org/schema/data/jpa/spring-jpa.xsd">

  <jpa:repositories base-package="com.test.repositories" />

</beans>
```

jpa:repositories 패키지의 하위 패키지에 있는 JPARepository를 인식한다.    
(boot는 SpringBootApplication이 기본 JPARepository 패키지)    

😊ApplicationConfig.java   

```java
@Configuration
@EnableJpaRepositories
@EnableTransactionManagement
class ApplicationConfig {

  @Bean
  public DataSource dataSource() {

    EmbeddedDatabaseBuilder builder = new EmbeddedDatabaseBuilder();
    return builder.setType(EmbeddedDatabaseType.HSQL).build();
  }

  @Bean
  public LocalContainerEntityManagerFactoryBean entityManagerFactory() {

    HibernateJpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
    vendorAdapter.setGenerateDdl(true);

    LocalContainerEntityManagerFactoryBean factory = new LocalContainerEntityManagerFactoryBean();
    factory.setJpaVendorAdapter(vendorAdapter);
    factory.setPackagesToScan("com.test.entity");
    factory.setDataSource(dataSource());
    return factory;
  }

  @Bean
  public PlatformTransactionManager transactionManager(EntityManagerFactory entityManagerFactory) {

    JpaTransactionManager txManager = new JpaTransactionManager();
    txManager.setEntityManagerFactory(entityManagerFactory);
    return txManager;
  }
}
```
@EnableJpaRepositories : xml의 &lt;jpa:repositories&gt; 와 동일기능. 해당 패키지 하위의 패키지를 JpaRepository로 인식한다.   
entityManagerFactory의 리턴 객체로 LocalContainerEntityManagerFactoryBean를 사용해야 한다. (EntityManagerFactory 생성 외에도 예외 변환 매커니즘에 사용되기 때문에)   

👍TIP   
기본적으로 JPA repository는 Spring Bean이고 싱글톤이며 애플리케이션 로딩 시점에 초기화 된다.   
로딩 시점에 검증, 메타데이터 분석 (구현체 자동생성) 을 한다.   
EntityManagerFactory는 생성시 시간이 오래 걸리기 때문에 Spring Framework의 백그라운드 스레드에서 동작한다.   
이를 효과적으로 활용하려면 JPA저장소를 가능한 한 늦게 초기화 해야 한다.   

👍TIP : Bootstrap Mode
Spring Data JPA 2.1부터 Bootstrap Mode 모드를 지원한다. 아래 3가지 변수를 지정 할 수 있다.   
* DEFAULT (default) : @LAZY 달지 않는 한 실제 객체를 가져온다. 
* LAZY : 모든 Repository의 Bean에게 LAZY를 선언하고 프록시 객체가 주입되도록 한다. 
* DEFERRED : 기본적으로 작동방식은 LAZY랑 비슷하지만 ContextRefreshedEvent 할 때 저장소 초기화 트리거에서 검증한다. (애플리케이션 로딩 전에 검증한다.)

### 1-2 Spring Boot
spring boot는 기본설정이 되어있기 때문에 의존성만 추가하면 된다.   
pom.xml에 spring-data-jpa와 h2 database의 의존성을 추가한다.    
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

# JPA Repository Function

## 1. Query Creation
위의 예제에서 findByNameContains() 메서드 구현 방법이다.   
JPARepository 인터페이스만 상속받고 지정된 메서드명만 선언하면 구현은 Spring에게 할당하는 방식이다.   
쿼리를 아예 작성 안 해도 Spring이 해주기 때문에 편리하나 기본적인 쿼리 위주로 사용한다.   


👍TIP : 이름 전략 : find•••By~ (•••에는 아무거나 들어가도 된다.)   
[공식문서 : 이름 규칙](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.query-creation)   

<table>
    <tr>
        <th>Keyword</th>
        <th>Sample</th>
        <th>JPQL snippet</th>
    </tr>
    <tr>
        <td>Distinct</td>
        <td>findDistinctByLastnameAndFirstname</td>
        <td>select distinct … where x.lastname = ?1 and x.firstname = ?2</td>
    </tr>
    <tr>
        <td>And</td>
        <td>findByLastnameAndFirstname</td>
        <td>… where x.lastname = ?1 and x.firstname = ?2</td>
    </tr>
    <tr>
        <td>Or</td>
        <td>findByLastnameOrFirstname</td>
        <td>… where x.lastname = ?1 or x.firstname = ?2</td>
    </tr>
    <tr>
        <td>Is, Equals</td>
        <td>findByFirstname,<br>findByFirstnameIs,<br>findByFirstnameEquals</td>
        <td>… where x.firstname = ?1</td>
    </tr>
    <tr>
        <td>Between</td>
        <td>findByStartDateBetween</td>
        <td>… where x.startDate between ?1 and ?2</td>
    </tr>
    <tr>
        <td>LessThan</td>
        <td>findByAgeLessThan</td>
        <td>… where x.age &lt; ?1</td>
    </tr>
    <tr>
        <td>LessThanEqual</td>
        <td>findByAgeLessThanEqual</td>
        <td>… where x.age &lt;= ?1</td>
    </tr>
    <tr>
        <td>GreaterThan</td>
        <td>findByAgeGreaterThan</td>
        <td>… where x.age &gt; ?1</td>
    </tr>
    <tr>
        <td>GreaterThanEqual</td>
        <td>findByAgeGreaterThanEqual</td>
        <td>… where x.age &gt;= ?1</td>
    </tr>
    <tr>
        <td>After</td>
        <td>findByStartDateAfter</td>
        <td>… where x.startDate &gt; ?1</td>
    </tr>
    <tr>
        <td>Before</td>
        <td>findByStartDateBefore</td>
        <td>… where x.startDate &lt; ?1</td>
    </tr>
    <tr>
        <td>IsNull, Null</td>
        <td>findByAge(Is)Null</td>
        <td>… where x.age is null</td>
    </tr>
    <tr>
        <td>IsNotNull, NotNull</td>
        <td>findByAge(Is)NotNull</td>
        <td>… where x.age not null</td>
    </tr>
    <tr>
        <td>Like</td>
        <td>findByFirstnameLike</td>
        <td>… where x.firstname like ?1</td>
    </tr>
    <tr>
        <td>NotLike</td>
        <td>findByFirstnameNotLike</td>
        <td>… where x.firstname not like ?1</td>
    </tr>
    <tr>
        <td>StartingWith</td>
        <td>findByFirstnameStartingWith</td>
        <td>… where x.firstname like ?1 (parameter bound with appended %)</td>
    </tr>
    <tr>
        <td>EndingWith</td>
        <td>findByFirstnameEndingWith</td>
        <td>… where x.firstname like ?1 (parameter bound with prepended %)</td>
    </tr>
    <tr>
        <td>Containing</td>
        <td>findByFirstnameContaining</td>
        <td>… where x.firstname like ?1 (parameter bound wrapped in %)</td>
    </tr>
    <tr>
        <td>OrderBy</td>
        <td>findByAgeOrderByLastnameDesc</td>
        <td>… where x.age = ?1 order by x.lastname desc</td>
    </tr>
    <tr>
        <td>Not</td>
        <td>findByLastnameNot</td>
        <td>… where x.lastname &lt;&gt; ?1</td>
    </tr>
    <tr>
        <td>In</td>
        <td>findByAgeIn(Collection ages)</td>
        <td>… where x.age in ?1</td>
    </tr>
    <tr>
        <td>NotIn</td>
        <td>findByAgeNotIn(Collection ages)</td>
        <td>… where x.age not in ?1</td>
    </tr>
    <tr>
        <td>True</td>
        <td>findByActiveTrue()</td>
        <td>… where x.active = true</td>
    </tr>
    <tr>
        <td>False</td>
        <td>findByActiveFalse()</td>
        <td>… where x.active = false</td>
    </tr>
    <tr>
        <td>IgnoreCase</td>
        <td>findByFirstnameIgnoreCase</td>
        <td>… where UPPER(x.firstname) = UPPER(?1)</td>
    </tr>
</table>

## 2. Named Query
named Query는 로딩시점에 EntityManager에 쿼리를 저장한다. 실행 시 해당 쿼리를 구성하여 실행 할 수 있다.
* 정적 쿼리를 미리 지정해서 사용할 수 있다.   
* <strong>애플리케이션 로딩 시점</strong>에 문법 오류를 확인 할 수 있다.

😊Teacher.java    
```java
@Entity
@NamedQuery(
        name = "Teacher.findByName",
        query = "select t from Teacher as t where t.name = :name"
)
@NamedNativeQuery( // Native 쿼리도 가능하다.
    name = "Teacher.findNativeByName",
    query = "select name from Teacher where name = :name"
)
public class Teacher {
    •••
```
NamedQuery는 관례상 엔티티명.메소드명을 적어주는것이 좋다. (Spring Data JPA에서 자동으로 찾아간다.)   
위와 같이 선언하면 JPA에서는 아래와 같이 사용 할 수 있다.

😊Test.java
```java
@Test
void namedQueryInJpa() {
    List<Teacher> teachers = em.createNamedQuery("Teacher.findByName",Teacher.class)
        .setParameter("name","teacherA")
        .getResultList();
    assertThat(teachers.size()).isEqualTo(1);
    assertThat(teachers.get(0).getName()).isEqualTo("teacherA");
}

@Test
void namedNativeQueryInJpa() {
    //Teacher.findNativeByName 은 안된다.
    List<Teacher> teachers = em.createNativeQuery("select * from Teacher where name = :name",Teacher.class)
        .setParameter("name","teacherA")
        .getResultList();
    assertThat(teachers.size()).isEqualTo(1);
}

```
Spring Data Jpa에서는 더 간단하게 사용가능하다.      
👍TIP : 꼭 동적변수에는 @Param 어노테이션을 붙여야 한다.   
<br>

😊TeacherRepository.java   
```java
public interface TeacherRepository extends JpaRepository<Teacher, Long> {
    Teacher findById(long id);
    List<Teacher> findByNameContains(String name);

    //NamedQuery
    List<Teacher> findByName(@Param("name") String name);
    //NamedNativeQuery
    List<Teacher> findNativeByName(@Param("name") String name);
}
```

😊Test.java   
```java
@Test
void namedQuery() {
    List<Teacher> teachers = teacherRepository.findByName("teacherA");
    assertThat(teachers.size()).isEqualTo(1);
    assertThat(teachers.get(0).getName()).isEqualTo("teacherA");
}

@Test
void namedNativeQuery() {
    List<Teacher> teachers = teacherRepository.findNativeByName("teacherA");
    assertThat(teachers.size()).isEqualTo(1);
    assertThat(teachers.get(0).getName()).isEqualTo("teacherA");
}
```

## 3. Query Annotation
NamedQuery는 어플리케이션 로딩시점에 에러를 확인 할 수 있다는 장점이 있으나 엔티티 위에 어노테이션으로 작성해야 한다는 부담이 있다.   
(여러개면 @NamedQueries 안에 작성 필요하다.)    
복잡한 실무에서는 한 번 더 검색을 해야만 찾을 수 있다. 하지만 @Query는 NamedQuery와 같은 기능에 더 간단한 작성으로 구현 가능하다.   
(익명의 NamedQuery라고 보면 된다.)
<br>   

😊TeacherRepository.java    
```java
@Query("select t from Teacher as t where t.age > :age")
List<Teacher> findQueryByAgeGreaterThan(@Param("age") int age);
```
이걸로 선언 끝이다.   

😊Test.java   
```java
@Test
void findQueryByAgeGreaterThanTest() {
    List<Teacher> teachers = teacherRepository.findQueryByAgeGreaterThan(40);
    assertThat(teachers.size()).isEqualTo(2);
}
```

😊Query.java   
```java
public @interface Query {
    String value() default "";
    String countQuery() default ""; // Page에서 countQuery위한 것
    String countProjection() default "";
    boolean nativeQuery() default false;
    String name() default "";
    String countName() default "";
}
```

## 4. Return Type
[🧷 참고 : 공식문서](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repository-query-return-types)    
Spring Data Jpa는 다양한 <strong>리턴 타입</strong>을 제공한다. 상황에 맞게 원하는 타입을 변환하는작업이 간단하다.   
<br>

<table>
    <tr>
        <th>Return type</th>
        <th>Description</th>
    </tr>
    <tr>
        <td>void</td>
        <td>리턴이 없다.</td>
    </tr>
    <tr>
        <td>
            Primitives<br>
            Wrapper types
        </td>
        <td>
            기본형이나 Wrapper 클래스 다 가능하다. (int, Integer, Long...)
        </td>
    </tr>
    <tr>
        <td>T, Optional (one result)</td>
        <td>기본 엔티티 리턴도 가능하다. (여러건시 에러)</td>
    </tr>
    <tr>
        <td>Collection</td>
        <td>List, Collection, Iterator처럼 컬렉션.</td>
    </tr>
    <tr>
        <td>
            Future<br>
            CompletableFuture
        </td>
        <td>@Async 와 함께 사용하면 동기화가 가능하다.</td>
    </tr>
    <tr>
        <td>Page, Slice</td>
        <td>페이징을 위한 객체 리턴이 가능하다. (Pageable 파라미터 필요)</td>
    </tr>
</table>

Page와 Slice 리턴을 알아보자. Page는 count 쿼리가 추가로 실행되고 Slice는 +1 조회로 다음페이지 존재 여부만 알 수 있다.   

### 4-1 Page
😊TeacherRepository.java
```java
Page<Teacher> findPageAllBy(Pageable pageable);
```
기존 문법대로 find~~By 로 조건 없이 조회하였다.   
😊Test.java
```java
@Test
void findPageAllBy() {
    teacherRepository.deleteAll(); // 기본메서드
    List<Teacher> teachers = new ArrayList<>();
    for (int i = 0; i < 100; i++) {
        teachers.add(new Teacher("tea"+i, i));
    }
    teacherRepository.saveAllAndFlush(teachers); // 기본 메서드
    // 페이지는 0부터 시작이다. 3페이지이고 1페이지당 5개씩, 정렬하여 조회하였다.
    Page<Teacher> teacherPage = teacherRepository.findPageAllBy(PageRequest.of(2, 5, Sort.by("age").ascending()));

    System.out.println("teacherPage = " + teacherPage.getTotalElements()); // 총 행 갯수
    System.out.println("teacherPage.getTotalPages() = " + teacherPage.getTotalPages()); // 총 페이지 수
    for (Teacher teacher : teacherPage.getContent()) { // List 출력이다.
        System.out.println("teacher = " + teacher);
    }
```

🔑query   
```sql
    /* select
        generatedAlias0 
    from
        Teacher as generatedAlias0 
    order by
        generatedAlias0.age asc */ 
    select
        teacher0_.teacher_id as teacher_1_1_,
        teacher0_.age as age2_1_,
        teacher0_.name as name3_1_,
        teacher0_.subject_id as subject_4_1_ 
    from
        teacher teacher0_ 
    order by
        teacher0_.age asc limit ? offset ?

    /* select
        count(generatedAlias0) 
    from
        Teacher as generatedAlias0 */ 
    select
        count(teacher0_.teacher_id) as col_0_0_ 
    from
        teacher teacher0_
```

count 쿼리까지 추가로 나간 것을 알 수 있다. (count는 자동 성능 최적화 지원)

💻console   
```markdown
teacherPage = 100
teacherPage.getTotalPages() = 20
teacher = Teacher(id=17, name=tea10, age=10, subject=null)
teacher = Teacher(id=18, name=tea11, age=11, subject=null)
teacher = Teacher(id=19, name=tea12, age=12, subject=null)
teacher = Teacher(id=20, name=tea13, age=13, subject=null)
teacher = Teacher(id=21, name=tea14, age=14, subject=null)

```

Slice는 기존 쿼리보다 +1 하여 다음 페이지 존재 여부만 체크한다. (count 없어 성능 유리)   

😊TeacherRepository.java   
```java
Slice<Teacher> findSliceAllBy(Pageable pageable);
```

😊Test.java   

```java
@Test
void sliceTest() {
    teacherRepository.deleteAll();
    List<Teacher> teachers = new ArrayList<>();
    for (int i = 0; i < 20; i++) {
        teachers.add(new Teacher("tea"+i, i));
    }
    teacherRepository.saveAllAndFlush(teachers);
    Slice<Teacher> teacherSlice = teacherRepository.findSliceAllBy(PageRequest.of(3, 4, Sort.by("age").ascending()));
    Slice<TeacherDto> dtos = teacherSlice.map(t -> new TeacherDto(t.getName(),t.getAge(),""));
    for (TeacherDto dto : dtos) {
        System.out.println("dto = " + dto);
    }
}
```
🔑query   
```sql
/* select
        generatedAlias0
    from
        Teacher as generatedAlias0
    order by
        generatedAlias0.age asc */ 
select
    teacher0_.teacher_id as teacher_1_1_,
    teacher0_.age as age2_1_,
    teacher0_.name as name3_1_,
    teacher0_.subject_id as subject_4_1_
from
    teacher teacher0_
order by
    teacher0_.age asc limit ? offset ?
```
💻console   
```markdown
dto = TeacherDto(name=tea12, age=12, subjectName=)
dto = TeacherDto(name=tea13, age=13, subjectName=)
dto = TeacherDto(name=tea14, age=14, subjectName=)
dto = TeacherDto(name=tea15, age=15, subjectName=)
```

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

👍TIP : 엔티티 상태 탐지 전략 (신규 or 기존)   
* Version-Property and Id-Property inspection (default): 
  기본적으로 Spring Data JPA는 1. Version-Property를 탐지하고 2. @ID를 탐지한다. 
  (✨null이면 새로운 엔티티)   
* Implementing Persistable: 만약 엔티티가 Persistable을 구현했다면 isNew() 라는 메서드를 정의하여 판별한다.   
* Implementing EntityInformation: SimpleJpaRepository를 상속받는 클래스를 생성하고 getEntity를 Override하여 커스터마이징 가능하다. (JpaRepositoryFactory 구현도 Bean 등록)   

## Reference
[공식 https://spring.io/projects/spring-data-jpa#overview](https://spring.io/projects/spring-data-jpa#overview)   
[가이드 https://spring.io/guides/gs/accessing-data-jpa/](https://spring.io/guides/gs/accessing-data-jpa/)   
[Hibernate 가이드](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html)   

# ERROR CODE   
1. TransactionRequiredException : No EntityManager with actual transaction    
💻console      
```markdown
javax.persistence.TransactionRequiredException: 
No EntityManager with actual transaction available for current thread 
- cannot reliably process 'persist' call
•••
```
가장 많이 하는 실수이다. test 클래스를 만들 때 @Transactional을 붙이지 않아 에러났다.    

2. IllegalArgumentException: Could not locate named parameter   
😊Test.java     

```java
@Test
void namedNativeQueryInJpa() {
    List<Teacher> teachers = em.createNativeQuery("Teacher.findNativeByName",Teacher.class)
            .setParameter("name","teacherA")
            .getResultList();
    assertThat(teachers.size()).isEqualTo(1);
}
```

💻console     

```markdown
java.lang.IllegalArgumentException: Could not locate named parameter [name], expecting one of []

	at org.hibernate.query.internal.ParameterMetadataImpl.getNamedParameterDescriptor(ParameterMetadataImpl.java:229)
	at org.hibernate.query.internal.ParameterMetadataImpl.getQueryParameter(ParameterMetadataImpl.java:198)
•••	
```
em.createNativeQuery에서 정의된 이름이 아니라 직접 쿼리를 넣으니 테스트 성공했다.   

3.ClassCastException : cannot be cast to class    
 
```markdown
java.lang.ClassCastException: class [Ljava.lang.Object; 
cannot be cast to class com.example.springdatajpa.entity.Teacher 
([Ljava.lang.Object; is in module java.base of loader 'bootstrap'; 
com.example.springdatajpa.entity.Teacher is in unnamed module of loader 'app')
```

em.createNativeQuery에서 두번째 인자로 리턴 클래스를 주지 않아 발생했다.   
   
4. IllegalStateException: Paging query needs to have a Pageable parameter!   

```markdown
Caused by: java.lang.IllegalArgumentException: 
Paging query needs to have a Pageable parameter! 
Offending method public abstract org.springframework.data.domain.Page 
com.example.springdatajpa.repository.TeacherRepository.findPageAll()
```
Page로 리턴받았는데 Pageable 파라미터를 받지 않았다.(Page, Slice리턴은 필수로 받아야한다.)   