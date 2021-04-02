---
title:  "JPA Error just for purpose of providing stack trace"

categories: 
  - jpa
tags:
  - error
  - study
last_modified_at: 2021-04-02T00:13:00-00:00
---

## Error log
![error-log]({{ site.baseurl }}/assets/images/study/jpa-error-pk.png)   
em.persist에서 에러가 났는데 알고 보니 간단한 오류이다.   

먼저 설정이다.
```java
// member entity
@Entity
@Data --Lombok
public class Member {
    @Id @GeneratedValue
    private Long id;
    @Column
    private String name;
    @Column
    private Integer age;
}

// main
EntityManagerFactory emf = Persistence.createEntityManagerFactory("simple-jpa-application");
EntityManager em  = emf.createEntityManager();
EntityTransaction tx = em.getTransaction();

try {
  tx.begin();
  Member member = new Member();
  member.setId(1L);
  member.setName("Long1");
  member.setAge(20);
  em.persist(member); // ERROR!!!!!

  Member findMember = em.find(Member.class, member.getId());
  System.out.println("findMember == member" + (findMember == member));
  System.out.println("findMember equals member" + (findMember.equals(member)));
  tx.commit();
} catch (Exception e) {
  tx.rollback();
} finally {
  em.close();
}
emf.close();
```

## Cause of error
Entity 생성전략이 pk가 @GeneratedValue이다. 비어 있어야 하는데 메인 메서드에서 set을 하였다.   
member.setId(1L); 부분을 없애니 정상 작동 하였다.   
