---
title:  "Spring transaction"

toc: true
toc_label: "go page"
toc_icon: "book"
toc_sticky : true

classes: wide

categories:
  - spring
tags:
  - transaction
last_modified_at: 2022-07-04T00:40:00-00:00
---

## 1. 트랜잭션(Transaction) 이란?
더 이상 쪼갤 수 없는 논리적인 작업단위 이다.
가장 많이 예를 드는 것으로 송금 예제가 있다.
A가 B에게 5000원을 송금하려고 할 때 비즈니스적으로는 두 단계가 있다.   
1. A의 계좌에서 5000원 인출   
2. B의 계좌에 5000원 입금   
이 두 단계가 동시에 실행되거나 동시에 실행되지 않아야 한다. 만약 1만 실행되면 A는 5000원이 없어졌는데 그 돈은 사라져 버려 심각한 논리적인 오류가 발생한다.

따라서 이렇게 1, 2 작업을 하나의 트랜잭션 단위로 묶어야 한다.   

스프링은 선언적 트랜잭션으로 아래 코드와 같이 원하는 곳에 @Transactional 어노테이션만 붙이면 트랜잭션이 가능하게 된다.
(class 위에 @Transactional 을 사용하면 안에 있는 모든 메서드들마다 각각 트랜잭션 적용이 된다.)
```java
@Service
public class RemitService {
    private RemitRepository remitRepository;
    
    // 여기의 모든 메서드는 Transactional 이 작용된다.
    @Transactional
    public void remit(Member from, Member to, int money) {
        // from의 계좌에서 돈을 money만큼 인출하고
        remitRepository.withdraw(from, money);
        // to계좌에 money 만큼 입금한다
        remitRepository.deposit(to, money);
    }
}
```

스프링은 트랜잭션 기능을 어노테이션 하나로 적용 가능하도록 하였는지 지금부터 이 원리와 방법을 알아보자.

## 2. Transaction without a @Transactional annotation
만약 스프링이 제공하는 Transaction 기능을 @Transactional 없이 코드로 직접 구현해보면 어떻게 될까? 그 기능을 단순하게 알아보자.

```java
@Service
public class RemitService {
    private RemitRepository remitRepository;
    private DataSource ds;

    public void remit(Member from, Member to, int money) {
        Connection con = null;
        try {
            con = ds.getConnection();
            con.setAutoCommit(false);

            // from의 계좌에서 돈을 money만큼 인출하고
            remitRepository.withdraw(from, money);
            // to계좌에 money 만큼 입금한다
            remitRepository.deposit(to, money);
            con.commit();
        } catch (Exception e) {
            con.rollback();
            throw e;
        } finally {
            if (con != null) {
                try {
                    con.setAutoCommit(true);
                    con.close();
                } catch (Exception e) {
                    log.error("error: " , e);
                }
            }
        }
    }
}
```

코드가 완성되었다. 실제로는 두 줄에 불과한 코드가 열 줄 넘게 되었을 뿐 아니라 반복적인 코드가 사용되었고 또한 
서비스에서 DataSource를 주입받으며 결합도가 높아졌다. 직접 Connection 객체를 닫음으로써 Connection Pool도 고려되지 않았다.
또한 이 소스는 DataSource를 다룰 때고 만약 JPA를 사용한다면 코드가 또 달라진다.

```java
@Service
public class RemitService {
    private RemitRepository remitRepository;
    private final EntityManager em;
    // 여기의 모든 메서드는 Transactional 이 작용된다.
    @Transactional
    public void remit(Member from, Member to, int money) {
        EntityTransaction tx = em.getTransaction();
        tx.begin();
        try {
            // from의 계좌에서 돈을 money만큼 인출하고
            remitRepository.withdraw(from, money);
            // to계좌에 money 만큼 입금한다
            remitRepository.deposit(to, money);
            tx.commit();
        } catch (Exception e) {
            tx.rollback();
            throw e;
        }
    }
}
```

코드량이 줄어들긴 했지만 역시 EntityManager에 의존하게 되고 try catch를 포함하여 비즈니스 코드보다 트랜잭션을 위한 코드가 더 많아졌다.   
스프링은 이것을 어떻게 추상화하여 @Transactional 로 줄였는지 알아보자.

## @Transactional의 기능
스프링의 트랜잭션의 핵심은 PlatformTransactionManager을 통해 이루어진다. (리액티브는 ReactiveTransactionManager을 이용한다.)
😊 PlatformTransactionManager.java
```java
public interface PlatformTransactionManager extends TransactionManager {
    TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;
    void commit(TransactionStatus status) throws TransactionException;
    void rollback(TransactionStatus status) throws TransactionException;
}
```
트랜잭션은 간단히 말해서 동시에 커밋이거나 롤백하는 기능이다. 그래서 트랜잭션 시작지점에 getTransaction으로 상태를 가져오고 commit이나 rollback을 적용하게 해 놓았다.
스프링은 AbstractPlatformTransactionManager 추상 클래스로 getTransaction(), commit(), rollback()을 정해놓고 상속받는 각 TransactionManager마다
doGetTransaction(), doCommit() 등을 오버라이드 하게 하였다.
만약 DataSource를 사용하게 된다면 DataSourceTransactionManager (혹은 JdbcTransactionManager) 가 자동으로 스프링 빈으로 등록되어 사용된다.

😊 DataSourceTransactionManager.java
```java
	@Override
    protected Object doGetTransaction() {
        DataSourceTransactionObject txObject = new DataSourceTransactionObject();
        txObject.setSavepointAllowed(isNestedTransactionAllowed());
        ConnectionHolder conHolder =
        (ConnectionHolder) TransactionSynchronizationManager.getResource(obtainDataSource());
        txObject.setConnectionHolder(conHolder, false);
        return txObject;
    }
    
	@Override
	protected void doCommit(DefaultTransactionStatus status) {
		DataSourceTransactionObject txObject = (DataSourceTransactionObject) status.getTransaction();
		Connection con = txObject.getConnectionHolder().getConnection();
		if (status.isDebug()) {
			logger.debug("Committing JDBC transaction on Connection [" + con + "]");
		}
		try {
			con.commit();
		}
		catch (SQLException ex) {
			throw translateException("JDBC commit", ex);
		}
	}

	@Override
	protected void doRollback(DefaultTransactionStatus status) {
		DataSourceTransactionObject txObject = (DataSourceTransactionObject) status.getTransaction();
		Connection con = txObject.getConnectionHolder().getConnection();
		if (status.isDebug()) {
			logger.debug("Rolling back JDBC transaction on Connection [" + con + "]");
		}
		try {
			con.rollback();
		}
		catch (SQLException ex) {
			throw translateException("JDBC rollback", ex);
		}
	}
```

## Reference
[https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html)

## Error Experience
* No EntityManager with actual transaction available for current thread - cannot reliably process 'persist' call
  - 테스트에서 @Transactional 을 선언 안 해줘서 에러

   




