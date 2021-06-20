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

👍TIP : 이름 전략   
[공식문서 : 이름 규칙](https://docs.spring.io/spring-data/jpa/docs/2.5.1/reference/html/#repositories.query-methods.query-creation)   

😊TeacherRepository.java   
```java
public interface TeacherRepository extends JpaRepository<Teacher, Long> {
    Teacher findById(long id);
    List<Teacher> findByNameContains(String name); // name을 포함하는 Teacher List 리턴
}
```

👌Test.java   
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

💻console   
```markdown
// 이름에 teacher만 있는 teacher조회
teacher = Teacher(id=4, name=teacherA, age=50, subject=Subject(id=1, title=math))
teacher = Teacher(id=5, name=teacherB, age=33, subject=Subject(id=2, title=english))
```

## Reference
[공식 https://spring.io/projects/spring-data-jpa#overview](https://spring.io/projects/spring-data-jpa#overview)   
[가이드 https://spring.io/guides/gs/accessing-data-jpa/](https://spring.io/guides/gs/accessing-data-jpa/)   
# ERROR CODE   
가장 많이 하는 실수이다. test 클래스를 만들 때 @Transactional을 붙이지 않아 에러났다.
💻console   
```markdown
javax.persistence.TransactionRequiredException: 
No EntityManager with actual transaction available for current thread 
- cannot reliably process 'persist' call
•••
```
   
