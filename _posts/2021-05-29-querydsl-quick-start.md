---
title:  "Querydsl quick start"

toc: true
toc_label: "go page"
toc_icon: "book"
toc_sticky : true

categories:
  - jpa
tags:
  - jpa
  - querydsl
last_modified_at: 2022-02-14T00:40:00-00:00
---

# What is Querydsl
querydsl은 Java framework이며 SQL문법과 비슷하지만 java문법으로 type-safe 인 쿼리를 생성할 수 있다.   
querydsl은 JPQL 및 Criteria 쿼리의 대안으로 동적 쿼리와 JPQL의 표현을 type-safe 한 형식으로 결합한다.   

# Querydsl quick start
1. 의존성 주입한다.   
참고 : [https://mvnrepository.com/artifact/com.querydsl/querydsl-jpa](https://mvnrepository.com/artifact/com.querydsl/querydsl-jpa)      
   
😀gradle   
   
```markdown
plugins {
    id "com.ewerk.gradle.plugins.querydsl" version "1.0.10"   
}
dependencies {
	implementation 'com.querydsl:querydsl-jpa'
}

//맨 아래 querydsl 추가 시작
//Entity에 대한 쿼리 만들어 줄 폴더
def querydslDir = "$buildDir/generated/querydsl"
querydsl {
	jpa = true
	querydslSourcesDir = querydslDir
}
sourceSets {
	main.java.srcDir querydslDir
}
configurations {
	querydsl.extendsFrom compileClasspath
}
compileQuerydsl {
	options.annotationProcessorPath = configurations.querydsl
}
//querydsl
```

2. Entity를 만들고 Gradle compile 하면 지정된 폴더에 QEntity가 생성된다.   

```java
// Member.class
@Entity
@Getter @Setter
public class Member {
    @Id @GeneratedValue
    private Long id;
	•••
}
```

엔티티를 만들면 미리 지정한 build/generated/QMember로 class가 만들어 지는것을 볼 수 있다.   

```java
@Generated("com.querydsl.codegen.EntitySerializer")
public class QMember extends EntityPathBase<Member> {

    private static final long serialVersionUID = 1133991934L;

    public static final QMember member = new QMember("member1");
	•••
```

3. application.yml   

```yaml
spring:
  datasource:
    url: jdbc:mariadb:tcp://localhost/~/querydsl
    username: sa
    password:
    driver-class-name: org.h2.Driver

  jpa:
    hibernate:
      ddl-auto: create
    properties:
      hibernate:
        # show_sql: true
        format_sql: true
logging.level:
  org.hibernate.SQL: debug
# org.hibernate.type: trace
```
   
4. gradle build   
querydsl 사용하려면 꼭 gradle compilequerydsl을 해야 한다. (엔티티 내용을 변경했을 때 컴파일 해야 QEntity가 변경이 된다.)   
   
5. Querydsl start   
준비는 끝났다.   
EntityManager로 JPAQueryFactory를 만들고 QEntity로 쿼리를 실행하면 된다.   
👍Tip EntityManager가 Thread Safe 하기 때문에 JPAQueryFactory 역시 Thread Safe 하다.   
😀example   
   
```java
// em은 EntityManager
JPAQueryFactory queryFactory = new JPAQueryFactory(em);
QMember m = QMember.member; // 기본 인스턴스
// QMember m = new QMember("m") // 별칭 주는 방법 (셀프조인이 아니면 잘 쓰이지 않는다.)
final Member findMember = queryFactory
    .select(m)
    .from(m)
    .where(m.username.eq("member1"))
    .fetchOne();
assertThat(findMember.getUsername()).isEqualTo("member1"); // 성공
```

## Reference
[공식 홈페이지: http://querydsl.com/](http://querydsl.com/)     
[공식 레퍼런스:http://querydsl.com/static/querydsl/5.0.0/reference/html_single/](http://querydsl.com/static/querydsl/5.0.0/reference/html_single/)   
[https://www.baeldung.com/querydsl-with-jpa-tutorial](https://www.baeldung.com/querydsl-with-jpa-tutorial)   


