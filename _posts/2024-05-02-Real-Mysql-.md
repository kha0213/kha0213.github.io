---
title:  "Real Mysql"

toc: true
toc_label: "go page"
toc_icon: "book"
toc_sticky : true

classes: wide

categories: 
  - mysql
tags:
  - mysql
last_modified_at: 2024-05-02T00:08:00-00:00
---
# Real My Sql

# ch4. 아키텍처

## MySQL 엔진 아키텍처
### MySQL의 구조
[mysql-architecture-diagram](https://dev.mysql.com/doc/refman/8.0/en/pluggable-storage-overview.html#mysql-architecture-diagram)   
![mysql-architecture]({{ site.baseurl }}/assets/images/study/architecture/mysql-architecture.png)

**Note:**      
MySQL Document Store를 사용하면 SQL 관계형 테이블 및 스키마 없는 JSON 컬렉션으로 작업할 수 있다.    
[document_store](https://www.mysql.com/products/enterprise/document_store.html)    
{: .notice--info}

MySQL은 대부분 프로그래밍 언어로부터 접근 방법을 모두 지원한다.(JDBC, ODBC 등)    
MySQL 서버는 **MySQL 엔진**과 **스토리지 엔진** 으로 구분된다.    
MySQL 서버에서 MySQL 엔진은 하나지만 스토리지 엔진은 여러 개를 동시에 사용할 수 있다.   

#### MySQL 엔진
MySQL엔진은 Connection Handler, SQL 파서, 전처리기, SQL 옵티마이저로 나뉜다.
- Connection Handler : 커넥션 및 쿼리 요청을 처리 담당
- SQL 인터페이스 : DML, DDL, Procedure, View 등 SQL 인터페이스 제공 담당
- SQL 파서(parser) : SQL문법 오류 탐지 및 SQL 쿼리 문장을 MySQL이 처리하기 좋은 토큰 단위로 나눠서 트리 형태로 파싱 하는 작업 담당
- SQL 옵티마이저(optimizer) : 쿼리를 얼마나 낮은 비용으로 효율적으로 처리할지 결정하는 역할 수행
  (쿼리 재작성, 스캔 순서 조정 및 인덱스 선택 작업 수행)
- 캐시와 버퍼 : 성능 향상을 위한 보조 저장소 기능 담당

#### 스토리지 엔진 
실제로 데이터를 처리하는 곳이다. 스토리지 엔진을 지정하면 지정된 스토리지 엔진이 작업들을 진행한다.
- 스토리지 엔진은 MySQL 엔진과 플러그인 형태로 연동/분리 가능하고 핸들러 API(핸들러 요청)를 통해 스토리지 엔진에 읽기/쓰기 요청이 가능

### MySQL 스레딩 구조
![mysql-threading]({{ site.baseurl }}/assets/images/study/architecture/mysql-threading.png)   
MySQL 서버는 프로세스 기반이 아닌 스레드 기반으로 작동하며, 포그라운드 스레드와 백그라운드 스레드로 구분할 수 있다.   

#### 포그라운드 스레드(클라이언트 스레드)

포그라운드 스레드는 최소 MySQL 서버에 접속된 클라이언트 수만큼 존재하며, 각 클라이언트 사용자의 요청 쿼리 문장을 처리한다.

커넥션이 종료되면, 해당 커넥션을 담당하던 스레드는 스레드 캐시로 되돌아 가며, 스레드 캐시에 일정 개수 이상의 스레드가 존재할 경우 스레드를 종료시켜 스레드 개수를 일정하게 유지한다.

포그라운드 스레드는 데이터를 데이터 버퍼나 캐시로부터 가져오며, 이 곳들에 데이터가 없는 경우 직접 디스크의 데이터나 인덱스 파일로부터 데이터를 가져온다.

- MyISAM 테이블은 디스크 쓰기 작업까지 포그라운드 스레드에서 처리한다.
- InnoDB 테이블은 데이터 버퍼나 캐시까지만 포그라운드 스레드에서 처리하고 디스크 쓰기 작업은 백그라운드 스레드가 처리한다.

#### 백그라운드 스레드(InnoDB)

- 인서트 버퍼를 병합하는 스레드
- 로그를 디스크로 기록하는 스레드
- InnoDB 버퍼 풀의 데이터를 디스크에 기록하는 스레드
- 데이터를 버퍼로 읽어 오는 스레드
- 잠금이나 데드락을 모니터링하는 스레드

**Note:**      
가장 중요한 스레드는 **로그 스레드**와 버퍼의 데이터를 디스크로 내려쓰는 작업을 하는 **쓰기 스레드**이다.   
{: .notice--info}

데이터를 쓰는 아주 많은 작업은 백그라운드로 처리하기 때문에 디스크를 최적으로 사용할 수 있을 만큼 스레드 개수를 설정하는 것이 좋다.

### 메모리 할당 및 구조

MySQL에서 사용되는 메모리 공간은 글로벌 메모리 영역, 로컬 메모리 영역으로 구분할 수 있다.

#### 글로벌 메모리 영역

클라이언트 스레드 수와 무관하게 하나의 메모리 공간을 할당 받는다. 생성된 모든 글로벌 영역은 모든 스레드에 의해 공유된다.

- 테이블 캐시
- InnoDB 버퍼 풀
- InnoDB 어댑티브 해시 인덱스
- InnoDB 리두 로그 버퍼

***어댑티브 해시 인덱스*** 는 사용자가 수동으로 생성하는 인덱스가 아니라 InnoDB 스토리지 엔진에서 사용자가 자주 요청하는 데이터에 대해 자동으로 생성하는 인덱스   

#### 로컬 메모리 영역

세션 메모리 영역이라고도 하며, MySQL 클라이언트 스레드가 쿼리를 처리하는데 사용하는 메모리 영역이다.

로컬 메모리는 각 클리언트 스레드별로 독립적으로 할당되며 절대 공유되어 사용되지 않는다.

- 정렬 버퍼(Sort buffer)
- 조인 버퍼
- 바이너리 로그 캐시
- 네트워크 버퍼

### 플러그인 스토리지 엔진 모델

MySQL의 독특한 구조 중 하나는 플러그인 모델이다.

MySQL은 이미 기본적으로 많은 스토리지 엔진을 가지고 있다. 하지만 사용자의 요구 조건을 만족시키기 위해 부가적인 기능을 더 제공하는 스토리지 엔진이 필요할 수 있으며, 이를 위해 직접 스토리지 엔진을 개발하는 것도 가능하다.

**MySQL 엔진이 각 스토리지 엔진에게 데이터를 읽어오거나 저장하도록 명령하려면 반드시 핸들러를 통해야 한다**

### 컴포넌트

MySQL 8.0 부터 기존 플러그인 아키텍처를 대체하기 위해 컴포넌트 아키텍처가 지원된다.

- 플러그인은 오직 MySQL 서버와 인터페이스할 수 있고, 플러그인끼리는 통신 불가
- 플러그인은 MySQL 서버의 변수나 함수를 직접 호출하기 때문에 안전하지 않음
- 플러그인은 상호 의존 관계를 설정할 수 없어 초기화가 어려움

컴포넌트는 이러한 단점들을 보완하여 만들었다.

### 쿼리 실행 구조
SQL 요청 -> 쿼리 파서 -> 전처리기 -> 옵티마이저 -> 쿼리 실행기 -> 스토리지엔진
#### 쿼리 파서
쿼리 파서는 사용자 요청으로 들어온 쿼리 문장을 토큰으로 분리해 트리 형태의 구조로 만들어 내는 작업이다.
기본 적인 문법 오류는 이 과정에서 발견된다.
#### 전처리기
파서 과정에서 만들어진 파서 트리를 기반으로 쿼리 문장의 문제점을 확인한다.
각 토큰을 테이블 이름, 컬럼 이름, 또는 내장 함수와 같은 개체를 매핑해 객체의 존재 여부와 접근 권한 등을 확인하는 과정을 수행한다.
#### 옵티마이저
**옵티마이저는 사용자의 요청으로 들어온 쿼리 문장의 효율적인 처리를 결정하는 역할을 맡으며, DBMS의 두뇌이다.**
#### 실행 엔진
실행 엔진과 핸들러는 DBMS의 손과 발에 비유할 수 있다.
실행 엔진은 만들어진 계획대로 각 핸들러에게 요청해서 받은 결과를 또 다른 핸들러의 입입력으로 연결하는 역할을 수행한다.
#### 핸들러(스토리지 엔진)
MySQL 실행 엔진의 요청에 따라 데이터를 디스크에 저장하고 읽어오는 역할을 담당한다.
핸들러는 곧 스토리지 엔진을 의미한다.
#### 복제
2대 이상의 DBMS를 나눠서 데이터를 저장하는 방식이며, 사용하기 위한 최소 구성은 Master / Slave 구성을 하여야 한다.
#### 쿼리 캐시
SQL의 실행결과를 메모리에 캐시하여 동일 요청일 경우 즉시 결과 반환한다.
하지만 Select 만 있는 서비스가 아니면 오히려 동시 처리 성능이 떨어졌고 많은 버그의 원인이 되어 **Mysql 8.0 에 기능이 완전히 제거**되었다.
#### 스레드 풀
MySQL의 스레드 풀은 내부적으로 사용자의 요청을 처리하는 스레드 개수를 줄여서 동시 처리되는 요청이 많다 하더라도 MySQL 서버의 CPU가 제한된 개수의 스레드 처리에만 집중할 수 있게 해서 서버의 자원 소모를 줄이는 것이 목적이다. 
Percona Server의 스레드 풀 플러그인은 선순위 큐와 후순위 큐를 이용해 특정 트랜잭션이나 쿼리를 우선적으로 처리할 수 있는 기능도 제공한다.

#### 트랜잭션 지원 메타데이터
MySQL 최신 버전부터는 테이블의 구조 정보나 스토어드 프로그램의 코드 관련 정보를 InnoDB의 테이블에 저장하도록 되었다.   
MySQL서버가 작동하는 데 기본적으로 필요한 테이블들을 묶어서 **시스템 테이블**이라는 테이블에 저장한다. 대표적으로 사용자 인증과 권한에 관련된 테이블들이 있다.   

## InnoDB 스토리지 엔진 아키텍처
InnoDB 스토리지 엔진은 MySQL 스토리지 엔진 중 가장 많이 사용되는 스토리지 엔진이다.   
InnoDB는 레코드 기반의 잠금을 제공하며, 덕분에 높은 동시성 처리가 가능하고 안정적이며 성능이 뛰어나다.   
![mysql-innodb-architecture-8-0]({{ site.baseurl }}/assets/images/study/architecture/mysql-innodb-architecture-8-0.png)   

### 프라이머리 키에 의한 클러스터링
InnoDB의 모든 테이블은 기본적으로 pk를 기준으로 클러스터링 되어 저장된다. 이 말은 pk값 순서대로  디스크에 저장된다는 뜻이며, 모든 세컨더리 인덱스는 레코드 주소 대신 pk값을 논리적인 주소로 사용한다.
pk가 클러스터링 인덱스이기 때문에 pk를 이용한 레인지 스캔은 상당히 빨리 처리되고 결과적으로 쿼리의 실행 계획에서 pk는 기본적으로 다른 보조 인덱스에 비해 비중이 놓게 설정된다.

### 외래 키 지원
외래 키에 대한 지원은 InnoDB 스토리지 엔진 레벨에서 지원하는 기능이다.
외래 키는 데이터베이스 서버 운영의 불편함 때문에 서비스용 데이터베이스에서 생성하지 않는 경우도 자주 있지만, 개발 환경의 데이터베이스에는 좋은 가이드 역할을 할 수 있다.
InnoDB 외래 키는 부모 테이블과 자식 테이블 모두 해당 컬럼에 인덱스 생성이 필요하고, 변경 시 참조된 테이블에 데이터가 존재하는지 체크해야 한다.

### MVCC(Multi Version Concurrency Control)
레코드 레벨의 트랜잭션을 지원하는 DBMS가 제공하는 기능이며, 이 기능의 가장 큰 목적은 잠금을 사용하지 않는 일관된 읽기의 제공에 있다. → InnoDB는 Undo log를 통해 구현
여기서 멀티 버전은 하나의 레코드에 대해 여러 개의 버전이 동시에 관리된다는 의미다.
MySQL 서버에 설정된 격리수준에 따라 InnoDB 버퍼 풀, 언두 로그에 2개의 버전 중 상황에 맞는 버전의 레코드가 사용된다.

### 잠금 없는 일관된 읽기(Non-Locking Consistent Read)
InnoDB 스토리지 엔진은 MVCC 기술을 사용해 잠금을 걸지 않고 일기 작업을 수행하기 때문에 읽기 작업은 다른 트랜잭션이 가지고 있는 잠금을 기다리지 않고 바로 가능하다.
이러한 특징 때문에 UPDATE인 상태의 레코드를 다른 사용자가 SELECT 하더라도 이 UPDATE 트랜잭션이 SELECT 작업을 방해하지 않는다. 이를 잠금 없는 일관된 읽기라고 한다.

### 자동 데드락 감지
InnoDB 스토리지 엔진은 내무적으로 잠금이 교착 상태에 빠지지 않았는지 체크하기 위해 잠금 대기 목록을 그래프 형태로 관리한다.
InnoDB 스토리지 엔진은 데드락 감지 스레드를 가져 주기적으로 잠금 대기 그래프를 검사해 교착 상태의 트랜잭션들 중 하나를 강제 종료한다. 이때 기준은 트랜잭션 언두 로그 양이 적은 것 먼저 롤백된다.
동시 처리 스레드가 매우 많을 경우 데드락 감지 스레드는 더 많은 CPU 자원을 소모할 수도 있다.

### 자동화된 장애 복구
InnoDB에는 손실이나 장애로부터 데이터를 보호하기 위해 MySQL 서버가 시작될 때 완료되지 못한 트랜잭션이나 디스크에 일부만 기록된 데이터에 대한 복구 작업이 자동으로 진행된다.
자동 복구 작업이 불가능할 경우 MySQL 서버 설정 파일에 `innodb_force_recovery`를 설정해 MySQL 서버를 시작해야 한다.

### InnoDB 버퍼 풀
InnoDB 스토리지 엔진에서 가장 핵심적인 부분으로, 디스크의 데이터 파일이나 인덱스 정보를 메모리에 캐시해 두는 공간이다.
쓰기 작업을 지연시켜 일괄 작업을 처리할 수 있게 해주는 버퍼 역할도 한다.
InnoDB 버퍼 풀의 크기를 적절히 작은 값으로 설정해 조금씩 증가시키는 방법을 사용하는 것이 좋다.

### Double Write Buffer
InnoDB 스토리지 엔진의 리두 로그는 공간의 낭비를 막기 위해 변경된 내용만 기록하기 때문에 문제 발생시, 내용은 복구가 불가능할 수도 있다. → Partial-page라 한다.
이를 Double - Write로 해결한다

### 언두 로그
InnoDB 스토리지 엔진은 트랜잭션과 격리 수준을 보장하기 위해 DML로 변경되기 전 버전의 데이터를 별도로 백업한다. 이 백업된 데이터를 언두 로그라고 한다.
언두 로그는 백업해둔 이전의 데이터를 활용해 트랜잭션 롤백 시 데이터 복구를 하고, 변경중인 데이터에 다른 커넥션이 접근해 데이터를 조회할 경우 격리 수준에 맞게 변경중인 데이터가 아닌 백업 데이터를 읽어 반환한다.

### 체인지 버퍼
RDBMS의 레코드 변화시에는 데이터 파일을 변경할 뿐만 아니라 해당 테이블의 인덱스 또한 업데이트 해야 한다. 이때, 디스크를 읽는 작업이 필요하기 때문에 많은 자원을 소모할 수도 있다.
InnoDB는 변경해야 할 인덱스가 버퍼 풀에 있으면 바로 업데이트를 하지만 디스크로부터 읽어와야 할 경우 임시 공간에 둔다 이 공간을 체인지 버퍼라고 한다.

### 리두로그 및 로그 버퍼
리두 로그는 영속성과 가장 밀접하게 연관되어 있다.
대부분의 데이터베이스 서버는 데이터 변경 내용을 로그로 먼저 기록 → 쓰기 보다 읽기 성능을 고려(일반적으로 읽기보다 쓰기가 상대적으로 큰 비용 필요)
데이터베이스 서버는 쓰기 비용이 낮은 자료구조를 가진 리두 로그를 갖고 있으며 비정상 종료 시 리두 로그의 데이터를 사용해 복구한다.
데이터를 복구하거나 대용량 데이터를 한번에 적재하는 경우 리두 로그를 비활성화 해 데이터 적재 시간을 단축시킬 수도 있다.

```
mysql> ALTER INSTANCE DISABLE INNODB REDO_LOG;
```

데이터 적재 완료 후 반드시 리두 로그를 활성하 하자!

### 어댑티브 해시 인덱스
사용자가 수동으로 생성하는 인덱스가 아니라 InnoDB 스토리지 엔진에서 사용자가 자주 요청하는 데이터에 대해 자동으로 생성하는 인덱스이다.
어댑티브 해시 인덱스는 B-Tree의 검색 시간을 줄여주기 위한 기능이다.
어댑티브 해시 인덱스가 도움이 되는 경우

- 디스크 읽기가 많은 경우
- 특정 패턴의 쿼리가 많은 경우(조인, LIKE 패턴검색)
- 매우 큰 데이터를 가진 테이블 레코드를 폭넓게 읽는 경우

성능 향상에 도움이 많은 경우

- 디스크의 데이터가 InnoDB 버퍼 풀 크기와 비슷한 경우(디스크 읽기가 적음)
- 동등 조건 검색이 많은 경우
- 쿼리가 데이터 중에서 일부 데이터에만 집중된 경우

## MySQL 로그 파일
로그 파일을 이용하면 MySQL 서버의 깊은 내부 지식이 없이도 MySQL의 상태나 부하를 일으키는 원인을 쉽게 찾아 해결할 수 있다.

### 에러 로그 파일
MySQL이 실행되는 도중에 발생하는 에러나 경고 메시지가 출력되는 로그 파일.
위치는 MySQL 설정 파일(`my.cnf`)에서 `log_error`라는 파라미터 경로에 생성

### 제너럴 쿼리 로그 파일
쿼리 로그 파일의 경로는 `general_log_file`이란 파라미터에 설정돼 있다.
이 로그 파일을 검토해 MySQL 서버에서 실행되는 쿼리에는 어떤 것들이 있는지 검토할 수 있다.

### 슬로우 쿼리 로그
MySQL 서버의 쿼리  중 정기적인 점검을 위한 튜닝에서 어떤 쿼리가 문제의 쿼리인지 판단하기는 어려운데, 이때 슬로우 쿼리 로그가 도움이 된다.
`long_query_time` 시스템 변수에 설정된 시간 이상이 소요된 쿼리가 모두 기록된다. 즉, 슬로우 쿼리 로그 파일에 기록된 쿼리는 `long_query_time` 보다 시간이 오래 걸린 정상적인 쿼리이다.

# ch08. 인덱스
DBMS는 데이터베이스 테이블의 모든 테이블의 모든 데이터를 검색해서 원하는 결과를 가져오려면 시간이 오래 걸린다.   
컬럼의 값과 해당 레코드가 저장된 주소를 key-value로 삼아 인덱스를 만들어 정렬해 원하는 값을 빠르게 탐색할 수 있도록 한다.   
DBMS의 인덱스는 SortedList와 마찬가지로 저장되는 컬럼의 값을 이용해 항상 정렬된 상태를 유지하며, 데이터 파일은 ArrayList와 같이 저장된 순서대로 별도의 정렬 없이 저장해 둔다.   
DBMS의 인덱스는 데이터의 저장(INSERT, UPDATE, DELETE) 성능을 희생하고 데이터의 읽기(SELECT) 속도를 높이는 기능이다.   
테이블의 인덱스 추가 여부는 데이터의 저장 속도를 어디까지 희생할 수 있는지, 읽기 속도를 얼마나 더 빠르게 만들어야 하느냐에 따라 결정해야 한다.   

## B-Tree 인덱스
B-Tree(Balanced Tree)는 데이터베이스의 인덱싱 알고리즘 가운데 가장 일반적으로 사용되고, 가장 먼저 도입된 알고리즘이다.
B-Tree는 칼럼의 원래 값을 변형시키지 않고 인덱스 구조체 내에서는 항상 정렬된 상태로 유지한다.   
전문 검색과 같은 특수한 요건이 아닌경우, 대부분 인덱스는 거의 B-Tree를 사용할 정도로 일반적인 용도에 적합한 알고리즘이다.   

**Note:**      
💡 밸런스 트리(Balanced Tree)란?
트리의 노드가 한 방향으로 쏠리지 않도록, 노드 삽입 및 삭제 시 특정 규칙에 맞게 재 정렬되어 왼쪽과 오른쪽 자식 양쪽 수의 밸런스를 유지하는 트리이다. 
항상 양쪽 자식의 밸런스를 유지하므로, 무조건 O(logN)의 시간 복잡도를 가지게 된다.
다만 재정렬되는 작업으로 인해 노드 삽입 및 삭제 시 일반적인 트리보다 성능이 떨어지게 된다. 
그러므로 밸런스 트리는 삽입/삭제의 성능을 희생하고 탐색에 대한 성능을 높였다고 볼 수 있다.
{: .notice--info}   

![mysql-architecture]({{ site.baseurl }}/assets/images/study/algorithm/b_tree1.png)   
![mysql-architecture]({{ site.baseurl }}/assets/images/study/algorithm/b_tree2.png)   
![mysql-architecture]({{ site.baseurl }}/assets/images/study/algorithm/b_tree3.png)   
![mysql-architecture]({{ site.baseurl }}/assets/images/study/algorithm/b_tree4.png)   

### 구조 및 특성
B-Tree는  트리 구조의 최상위에 하나의 루트 노드가 존재하고 그 하위에 자식 노드가 붙어 있는 형태다.    
트리 구조의 가장 하위에 있는 노드를 리프 노드라 하고 중간 노드들을 브랜치 노드라고 한다.   
데이터베이스에서 인덱스와 실제 데이터가 저장된 데이터는 따로 관리되는데, 인덱스의 리프 노드는 항상 실제 데이터 레코드를 찾아가기 위한 주솟값을 가지고 있다.   

MyISAM은 세컨더리 인덱스가 물리적인 주소를 가지는 반면 InnoDB는 프라이머리 키를 주소처럼 사용하기 때문에 논리적인 주소를 가진다.   
그래서 InnoDB는 인덱스를 통해 레코드를 읽을 때는 데이터 파일을 바로 찾아가지 못한다.   
즉, 모든 세컨더리 인덱스 검색에서 데이터 레코드를 읽기 위해서는 반드시 프라이머리 키를 저장하고 있는 B-Tree를 다시 한번 검색해야 한다.   
![mysql-architecture]({{ site.baseurl }}/assets/images/study/architecture/real_mysql_b_tree.png)   

### B-Tree 인덱스 키
#### 인덱스 키 추가
B-Tree에 저장될 때는 저장될 키 값을 이용해 적절한 위치를 검색하고 저장될 위치가 결정되면 레코드의 키 값과 대상 레코드의 주소 정보를 B-Tree의 리프 노드에 저장한다.     
리프 노드가 꽉 차서 더는 저장할 수 없을 때는 리프 노드가 분리돼야 하는데, 이는 상위 브랜치 노드까지 처리의 범위가 넓어진다.   
이러한 작업으로 인해 B-Tree는 상대적으로 쓰기 작업에 비용이 많이 드는 것으로 알려져 있다.   

하지만 InnoDB의 경우 **키 추가 작업을 지연**시켜 나중에 처리할 수 있다.    
하지만 **프라이머리 키**나 **유니크 인덱스**의 경우 중복 체크가 필요하기 때문에 즉시 B-Tree에 추가하거나 삭제한다.      

#### 인덱스 키 삭제
B-Tree에서 키 값이 삭제되는 경우는 상당히 간단하다.    
해당 키 값이 저장된 B-Tree의 리프 노드를 찾아서 그냥 삭제 마크만 하면 작업이 완료된다.    
이렇게 마킹된 인덱스 키 공간은 방치하거나 재활용할 수 있다.   
InnoDB는 이러한 작업을 **버퍼링되어 지연 처리** 될 수도 있다.   

#### 인덱스 키 변경
인덱스의 키 값은 그 값에 따라 저장될 리프 노드의 위치가 결정되므로 B-Tree의 키 값이 변경되는 경우에는 단순히 인덱스상의 키 값만 변경하는 것은 불가능하다. 
B-Tree의 키 값 변경 작업은 먼저 키 값을 삭제한 후, 다시 새로운 키 값을 추가하는 작업으로 처리된다.
삭제와 추가 작업은 앞에서의 절차대로 처리된다.

#### 인덱스 키 검색
B-Tree 인덱스를 이용한 검색은 100% 일치 or 값의 앞부분만 일치하는 경우에 사용할 수 있다.
**인덱스 검색에서 중요한 사실은 인덱스의 키 값에 변형이 가해진 후 비교되는 경우에는 B-Tree의 빠른 검색 기능을 사용할 수 없다.**

**Note:**      
InnoDB 테이블에서 지원하는 레코드 잠금이나 넥스트 키락이 검색을 수행한 인덱스를 잠근 후 테이블의 레코드를 잠그는 방식으로 구현돼 있다.   
이러한 특성으로 인해 UPDATE, DELETE 문장이 실행될 때 테이블에 적절히 사용할 수 있는 인덱스가 없으면 불필요하게 많은 레코드를 잠근다.   
심지어 테이블의 모든 레코드를 잠글 수 있다.   
{: .notice--info}   

### B-Tree 인덱스 사용에 영향을 미치는 요소
B-Tree 인덱스는 인덱스를 구성하는 칼럼의 크기와 레코드의 건수, 그리고 유니크한 인덱스 키 값의 개수 등에 의해 검색이나 변경 작업의 성능이 영향을 받는다.

#### 인덱스 키 값의 크기
InnoDB 스토리지 엔진은 디스크에 데이터를 저장하는 가장 기본 단위를 페이지 또는 블록이라고 하며, 디스크의 모든 읽기 및 쓰기 작업의 최소 작업 단위가 된다.   
또한 페이지는 InnoDB 스토리지 엔진의 버퍼 풀에서 데이터를 버퍼링하는 기본 단위이기도 하다.   

B-Tree의 자식 노드는 인덱스의 페이지 크기와 키 값의 크기에 따라 결정된다. **(variable : innodb_page_size)**   

키 값의 크기가 클수록 한 페이지에 저장될 수 있는 키 값의 개수가 줄어들게 되고, SELECT 쿼리가 동시에 여러개의 레코드를 읽을때 한번 이상 읽어야 할 수 있다.   
따라서 키 값의 크기가 클수록 인덱스의 성능이 떨어지게 된다.   

#### B-Tree 깊이
B-Tree의 깊이는 인덱스 키 값의 크기에 따라서 깊이가 달라진다. 인덱스 키 값의 크기는 가능하면 작게 만드는 것이 좋다.

#### 선택도(기수성)
인덱스에서 선택도(Selectivity) 또는 기수성(Cardinality)은 거의 같은 의미로 사용되며, 기수성이란 모든 인덱스 키 값 가운데 유니크한 값의 갯수를 의미한다.   

```
선택도 = 카디널리티 / 전체 레코드 수
```

기수성이 높아야 인덱스의 효율이 좋아진다.   

#### 읽어야 하는 레코드의 건수
일반적인 DBMS의 옵티마이저에서는 인덱스를 통해 레코드 1견을 읽는 것이 테이블에서 직접 레코드 1건을 읽는 것보다 4-5배정도 비용이 더 많이 드는 작업인 것으로 예측한다.   
따라서 인덱스를 통해 읽어햐할 레코드가 많아진다면 **인덱스를 이용하지 않고** 테이블을 모두 읽고 필터링을 사용하는 방식으로 처리하는것이 효율적이다.   

### B-Tree 인덱스를 통한 데이터 읽기
#### 인덱스 레인지 스캔(Index Range Scan)
인덱스 레인지 스캔은 인덱스의 접근 방법 가운데 가장 대표적인 접근 방식이다.   
인덱스 레인지 스캔은 검색해야 할 인덱스의 범위가 결정됐을 때 사용하는 방식이다.   
```sql
SELECT * FROM employees WHERE emp_no BETWEEN 10001 AND 10010;
```

인덱스 레인지는 크게 3단계를 거친다.
1. 인덱스에서 조건을 만족하는 값이 저장된 위치를 찾는다. 이 과정을 인덱스 탐색이라고 한다.
2. 1번에서 탐색된 위치부터 필요한 만큼 인덱스를 차례대로 쭉 읽는다. 이 과정을 인덱스 스캔이라고 한다.
3. 2번에서 읽어 들인 인덱스 키와 레코드 주소를 이용해 레코드가 저장된 페이지를 가져오고, 최종 레코드를 읽어 온다.

쿼리가 필요로 하는 데이터에 따라 3번 과정은 필요하지 않을 수도 있다. 이를 커버링 인덱스라고 한다.   

```sql
SELECT emp_no FROM employees WHERE emp_no BETWEEN 10001 AND 10010;
```

**Note:**      
쿼리를 충족시키는데 필요한 모든 데이터를 가지고 있는 인덱스를 커버링 인덱스(Covering Index)라 한다.      
{: .notice--info}   

#### 인덱스 풀 스캔(Index Full Scan)
인덱스를 사용하지 않고 **전체 인덱스**를 읽어서 검색하는 방법을 의미한다.
이 방식은 쿼리의 조건절에 사용된 칼럼이 인덱스의 첫 번째 칼럼이 아닌 경우 인덱스 풀 스캔 방식이 사용된다.   

```sql
-- a, b, c 로 복합 Index가 걸려있을 경우
SELECT * FROM tb_sample WHERE b = 1 and c = 2;
```

일반적으로 인덱스를 사용한다 라는 것은 **인덱스 레인지 스캔**이나 **루스 인덱스 방식**으로 인덱스를 사용한다는 것을 의미한다.   
일반적으로 이 경우는 인덱스를 효율적으로 사용하지 못한 경우이다.   

#### 인덱스 루스 스캔 (Index Loose Scan)
인덱스 레인지 스캔과 비슷하게 ‘특정 인덱스 범위’만 스캔하는 방식과 비슷하지만 ‘중간중간 필요 없는 인덱스 키 값을 건너뛰고’ 다음으로 넘어가서 검색하는 방식을 의미한다.      
일반적으로 GROUP BY 또는 집합 함수 가운데 MAX(), MIN() 에 대해 최적화 하는 경우에 사용된다.   

```sql
-- (dept_no, emp_no) Index가 걸려있을 경우
SELECT dept_no, MIN(emp_no) 
FROM dept_emp 
GROUP BY dept_no;
```
#### 인덱스 스킵 스캔

데이터베이스 서베에서 인덱스의 핵심은 값이 정렬돼 있다는 것이며, 이로 인해 인덱스를 구성하는 칼럼의 순서가 매우 중요하다.

인덱스 스킵 스캔은 인덱스의 첫 번째 칼럼을 사용하지 않고 두 번째 칼럼부터 사용하는 방식이다.
이에 더해 첫 번째 컬럼의 값들을 이용하여 인덱스 레인지 스캔을 실행함으로써 인덱스의 첫 번째 칼럼을 사용하지 않고도 인덱스를 사용할 수 있다.
하지만 선행 컬럼의 유니크한 값이 적어야하며, 쿼리가 인덱스에 존재하는 컬럼으로만 처리가 가능하다. (커버링 인덱스)

```sql
-- (gender, birth_date) Index가 걸려있을 경우
-- 아래 쿼리는 인덱스 스캔이 불가능하다.
SELECT gender, birth_Date FROM employees WHERE birth_date > '2000-01-01';

-- DB 가 인덱스 스킵 스캔을 활용하여 처리하는 방식
SELECT gender, birth_Date FROM employees WHERE gender = 'M' AND birth_date > '2000-01-01';
SELECT gender, birth_Date FROM employees WHERE gender = 'F' AND birth_date > '2000-01-01';
```
#### 다중 칼럼(Multi-Colum) 인덱스

실제 서비스용 데이터베이스에서는 2개 이상의 칼럼을 포함하는 인덱스가 더 많이 사용된다. 두 개 이상의 칼럼으로 구성된 인덱스를 다중 칼럼 인덱스라고 하며, 또한 2개 이상의 칼럼이 연결됐다고 해서 **Concatenated Index** 라고도 한다.

다중 칼럼 인덱스는 인덱스의 첫 번째 칼럼을 기준으로 정렬되며, 첫 번째 칼럼의 값이 같은 경우 두 번째 칼럼의 값으로 정렬된다. 이러한 방식으로 인해 **인덱스 내에서 각 칼럼의 위치**가 상당히 중요하다.

#### B-Tree 인덱스의 정렬 및 스캔 방향
인덱스를 생성할 때 설정한 정렬 규칙에 따라서 인덱스의 키 값은 항상 오름차순이거나 내림차순으로 정렬되어 저장되지만, 인덱스를 어느 방향을 읽을지는 쿼리에 따라 옵티마이저가 실시간으로 만들어 내는 실행 계획에 따라 결정된다.

#### 인덱스의 정렬
MySQL 8.0 이상부터는 정렬 순서를 혼합한 인덱스를 생성할 수 있게 됐다.
인덱스 생성 시점에 오름차순 또는 내림차순으로 정렬이 결정되지만 쿼리가 그 인덱스를 사용하는 시점에 인덱스를 읽는 방향에 따라 오름차순 또는 내림차순 정렬 효과를 얻을 수 있다.   
내부적으로 InnoDB에서 인덱스 역순 스캔이 인덱스 정순 스캔에 비해 느리다.   
이는 InnoDB가 다음과 같은 특징이 존재하기 때문이다.
1. 페이지 잠금이 인덱스 정순 스캔에 적합한 구조
2. 페이지 내에서 인덱스 레코드가 단방향으로만 연결된 구조

#### B-Tree 인덱스의 가용성과 효율성
쿼리의 WHERE 조건이나 GROUP BY, 또는 ORDER BY 절이 어떤 경우에 인덱스를 사용할 수 있고 어떤 방식으로 사용할 수 있는지 식별할 수 있어야 한다.   
그래야만 쿼리의 조건을 최적화하거나, 역으로 쿼리에 맞게 인덱스를 최적으로 생성할 수 있다.

#### 비교 조건의 종류와 효율성
다중 칼럼 인덱스에서 각 칼럼의 순서와 그 칼럼에 사용된 조건이 동등 비교인지 아니면 크다 또는 작다 같은 범위 조건인지에 따라 각 인덱스 칼럼의 활용 형태가 달라지며, 그 효율 또한 달라진다.

```sql
SELECT * FROM dept_emp WHERE dept_no = 'd002' AND emp_no >= 10114;

-- INDEX (dept_no, emp_no) : dept_no = 'd002', emp_no >= 10114 둘 다 작업 범위 결정 조건
-- INDEX (emp_no, dept_no) : emp_no >= 10114 작업 범위 결정 조건, dept_no = 'd002' 필터링 조건
```

작업 범위 결정 조건은 많을 수록 쿼리 처리 성능을 높이지만 필터링 조건은 높이지는 못한다. (오히려 더 느릴 수 있다.) 

#### 인덱스의 가용성
B-Tree 인덱스의 특징은 왼쪽 값에 기준해서 오른쪽 값이 정렬돼 있다는 것이다. 여기서 왼쪽이란 하나의 칼럼 내에서뿐만 아니라 다중 칼럼 인덱스의 칼럼에 대해서도 함께 적용된다.
따라서 인덱스로 구성된 칼럼에 따라 인덱스의 효과를 얻을 수 있는지 아니면 얻을 수 없는지가 달라진다.

#### 가용성과 효율성 판단
B-Tree에서 인덱스는 다음 조건에서 작업 범위 결정 조건으로 사용할 수 없고 경우에 따라 체크 조건으로 인덱스를 사용할 수 있다.
1. NOT-EQUAL로 비교된 경우
   - WHERE column != 'N'
   - WHERE column NOT IN ('N','R')
   - WHERE column IS NOT NULL (MYSQL에서 IS NULL 은 인덱스 가능)
2. LIKE '%??' 형태로 문자열 패턴이 비교된 경우
3. 스토어드 함수나 다른 연산자로 인덱스 칼럼이 변형된 후 비교된 경우
   - WHERE SUBSTRING(column,1,1) = 'X'
   - WHERE DAYOFMONTH(column) = 1
4. NOT-DETERMINISTIC 속성의 스토어드 함수가 비교 조건에 사용된 경우 (비결정적 함수)
5. 데이터 타입이 서로 다른 비교
6. 문자열 데이터 타입의 콜레이션이 다른 경우

다중 칼럼으로 만들어진 인덱스는 어떤 조건에서 사용될 수 있고, 어떤 경우에 절대 사용할 수 없는지 보자.
```
INDEX ix_test (column_1, column_2, column_3 ..., column_n)
```
* 작업 범위 결정 조건으로 인덱스를 사용하지 못하는 경우
    1. column_1 칼럼에 대한 조건이 없는 경우
    2. column_1 칼럼의 비교 조건이 위의 인덱스 사용 불가 조건 중 하나인 경우
* 작업 범위 결정 조건으로 인덱스를 사용하는 경우(1<i<n)
    1. column_1~column_(i-1) 칼럼까지 동등 비교 형태로 비교
    2. column_i 칼럼에 대해 다음 연산자 중 하나로 비교
        * 동등비교
        * 크다 작다 형태
        * LIKE로 좌측 일치 패턴('??%')

### R-Tree 인덱스
MySQL에는 공간 인덱스라는 개념이 존재한다. 이는 R-Tree 인덱스 알고리즘을 이용하여 2차원의 데이터를 인덱싱하고 검색하는 목적의 인덱스다.    
기본적으로 B-Tree와 비슷하다. B-Tree는 인덱스를 구성하는 컬럼의 값이 1차원의 스칼라 값인 반면, R-Tree 인덱스는 2차원 공간 개념 값이라는 것이다.   
일반적으로 GPS 기준의 위도, 경도 좌표 등 좌표 시스템에 기반한 정보에 대해 사용된다.   

MySQL의 공간 확장에는 다음과 같이 크게 세 가지 기능이 포함돼 있다.
1. 공간 데이터를 저장할 수 있는 데이터 타입
2. 공간 데이터의 검색을 위한 공간 인덱스(R-Tree 알고리즘)
3. 공간 데이터의 연산 함수

#### 구조 및 특성
MySQL은 공간 정보의 저장 및 검색을 위해 여러 가지 기하학적 도형 정보를 관리할 수 있는 데이터 타입을 제공한다.   
이는 POINT, LINESTRING, POLYGON, GEOMETRY 등이 있다. GEOMETRY는 나머지 3개 타입의 슈퍼 타입이다.   
공간 정보의 검색을 위한 R-Tree 알고리즘을 이해하려면 MBR이라는 개념을 알고 있어야 한다. MBR은 Minimum Bounding Rectangle의 약자로, 
공간 데이터를 포함하는 가장 작은 사각형을 의미한다. 이 MBR을 이용하여 공간 데이터를 인덱싱하고 검색하는 것이 R-Tree 알고리즘이다.

### 전문 검색 인덱스
문서의 내용 전체를 인덱스화해서 특정 키워드가 포함된 문서를 검색하는 전문 검색에는 B-Tree을 사용할 수 없다.
문서 전체에 대한 분석과 검색을 위한 인덱싱 알고리즘을 전문 검색 인덱스라고 한다.

#### 인덱스 알고리즘
전문 검색에서는 문서 본문의 내용에서 사용자가 검색하게 될 키워드를 분석해 내고, 빠른 검색용으로 사용할 수 있게 이러한 키워드로 인덱스를 구축한다.
크게 어근 분석과 n-gram 알고리즘을 이용한 방법이 있다.

#### 어근 분석 알고리즘
MySQL 서버의 전문 검색 인덱스는 다음과 같은 두 가지 중요한 과정을 거쳐 색인 작업이 수행된다.
* 불용어 처리
* 어근 분석
  불용어 처리는 문서에서 검색에 사용할 수 없는 단어를 제거하는 과정이다. 어근 분석은 검색어로 선정된 단어의 뿌리인 원형을 찾는 작업이다.

#### n-gram 알고리즘
n-gram 알고리즘은 단순히 키워드를 검색해내기 위한 인덱싱 알고리즘이다.
n-gram은 본문을 무조건 몇 글자씩 잘라서 인덱싱하는 방법이다. 이 때 불용어와 동일하거나 불용어를 포함하는 경우 거른다.   

전문 검색의 경우 n-gram 알고리즘 말고도 Stopword 방법도 있다.   
[[MySQL] 전문 검색(FullText Search)](https://kjh8673a.github.io/til/2023-07-03-mySQL_FullTextSearch/)      

#### 불용어 변경 및 삭제
불용어 처리는 사용자에게 도움이 되기보다는 사용자를 더 혼란스럽게 하는 기능일 수도 있다. 
그래서 불용어 처리 자체를 완전히 무시하거나 MySQL 서버에 내장된 불용어 대신 사용자가 직접 불용어를 등록하는 방법을 권장한다.

#### 전문 검색 인덱스의 가용성
전문 검색 인덱스를 사용하려면 반드시 다음 두 조건을 갖춰야 한다.
1. 쿼리 문장이 전문 검색을 위한 문법을 사용 (MATCH ... AGAINST ...)
2. 테이블이 전문 검색 대상 칼럼에 대해서 **전문 인덱스** 보유

전문인덱스란 ? **FULLTEXT KEY** 를 통해 따로 만들어야 한다.   
```
CREATE TABLE tb_test (
    doc_id INT,
    doc_body TEXT,
    PRIMARY KEY (doc_id),
    FULLTEXT KEY fx_docbody (doc_body) WITH PARSER ngram
) ENGINE=MyISAM;
```

### 함수 기반 인덱스
mysql 8.0 부터 함수 기반 인덱스를 지원한다.  
- 칼럼의 값을 변형해서 만들어진 값에 대해 인덱스를 구축해야 할 때 사용
- **가상 칼럼**을 이용한 인덱스, **함수**를 이용한 인덱스로 구분할 수 있다.
- 인덱싱 값을 계산하는 과정의 차이만 있을 뿐, 실제 인덱스의 내부적인 구조 및 유지 관리 방법은 B-Tree 인덱스와 동일하다.

1. 가상컬럼을 이용한 인덱스
```sql
ALTER TABLE user
    ADD full_name VARCHAR(30) AS (CONCAT(first_name, ' ', last_name)) VIRTUAL,
    ADD INDEX idx_fullname(full_name);
```

2. 함수를 이용한 인덱스
```sql
CREATE TABLE (
    ...
    first_name VARCHAR(10),
    LAST_NAME VARCHAR(10),
    ...
    INDEX idx_fullname((CONCAT(first_name, ' ', last_name)))
);
```

### 멀티 밸류 인덱스
전문 검색 인덱스를 제외한 모든 인덱스는 레코드 1건이 1개의 인덱스 값을 가진다.
JSON타입을 지원하기 시작하면서 여러 개의 키 값을 가질 수 있는 인덱스도 필요하게 되었다.

### 클러스터링 인덱스
테이블의 레코드를 비슷한 것 (프라이머리 키 기준) 으로 물리적으로 묶어서 저장한다. (InnoDB 에서만 지원한다.)      
프라이머리 키 값이 비슷한 레코드끼리 묶어서 저장하는 것을 클러스터링 인덱스라고 표현한다.     
프라이머리 키 값으로 클러스터링된 테이블은 프라이머리 키 값 자체에 대한 의존도가 상당히 크기 대문에 신중히 프라이머리 키를 결정해야한다.      
만약 프라이머리 키가 없는 InnoDB 테이블은 자체적으로 대체할 칼럼을 선택한다.   

#### 세컨더리 인덱스에 미치는 영향
InnoDB에서는 클러스터링 키 값이 변경될 때마다 데이터 레코드의 주소가 변경되고 그때마다 해당 테이블의 모든 인덱스에 저장된 주솟값을 변경해야 할 것이다.
위에서 설명하고 있는 이유 때문에 InnoDB 테이블의 모든 세컨더리 인덱스는 해당 레코드가 저장된 주소가 아니라 프라이머리 키 값을 저장하도록 구현하였다.

#### 클러스터링 인덱스의 장점과 단점
해당 장단점은 Mysql에서 클러스터링 인덱스를 사용할 때의 장단점이다.   

장점
- 클러스터링 키로 검색할 때 처리 성능이 매우 빠름. 특히 범위 검색의 경우 매우 빠름
- 테이블의 모든 세컨더리 인덱스가 프라이머리 키를 가지고 있기 때문에 인덱스만으로 처리될 수 있는 경우가 많음. (커버링 인덱스)
단점
- 모든 세컨더리 인덱스가 프라이머리 키를 가지고 있기 때문에 프라이머리 키 값의 크기가 클 경우 전체적으로 인덱스의 크기가 커짐
- 세컨더리 인덱스를 통해 데이터를 조회할 때 프라이머리 키로 다시 한 번 검색해야해서 성능이 느림
- INSERT 할 때 프라이머리 키에 의해 레코드의 저장 위치가 결정되기 때문에 처리 성능이 느림
- 프라이머리 키를 변경할 때 레코드의 실제 위치을 변경하는 작업이 필요하기 때문에 처리 성능이 느림.

#### 클러스터링 테이블 사용 시 주의사항
1. 클러스터링 테이블의 경우 모든 세컨더리 인덱스가 프라이머리 키 값을 포함하기 때문에 크기에 민감하다.
2. 프라이머리 키는 Auto-Increment보다는 업무적인 칼럼으로 생성(가능한 경우)
   - InnoDB의 프라이머리 키는 클러스터링 키로 사용되며, 이 값에 의해 레코드의 위치가 결정. 따라서 읽기가 비교적 빠르다.
   - 프라이머리 키는 그 의미만큼이나 중요한 역할을 하기 때문에 대부분 검색에서 상당히 빈번하게 사용되는 것이 일반적이다.
   - 칼럼의 크기가 크더라도 업무적으로 해당 레코드를 대표할 수 있다면 그 칼럼을 프라이머리 키로 설정하는 것이 좋다.
3. 프라이머리 키는 반드시 명시할 것.
   - 프라이머리 키가 없으면 내부적으로 일련 번호 칼럼을 추가하는데, 이러면 보이지 않기 대문에 사용자가 전혀 접근할 수 가 없다.
4. Auto-increment 칼럼을 인조 식별자로 사용할 경우
   - 세컨더리 인덱스도 필요하고 프라이머리 키의 크기도 길다면 Auto-increment 칼럼을 추가하고, 이를 프라이머리 키로 설정하자
   - 위와 같이 프라이머리 키를 대체하기 위해 인위적으로 추가된 프라이머리 키를 인조 식별자라고 한다. 그리고 이것을 프라이머리 키로 설정하는 것이 성능 향상에 도움이 된다.

### 유니크 인덱스
유니크는 사실 인덱스라기 보다 제약 조건에 가깝다고 볼 수 있다.   
테이블이나 인덱스에 같은 값이 2개 이상 저장될 수 없음을 의미. (Null 은 값이 아니므로 가능)

#### 인덱스 읽기
유니크 인덱스가 빠르지 않다.   
읽어야 할 레코드 건수가 같다면 성능상의 차이는 미미하다.

#### 인덱스 쓰기
중복된 값이 있는지 없는지 체크하는 과정이 한 단계 더 필요하다. 그래서 유니크하지 않은 세컨더리 인덱스의 쓰기보다 느리다.   
또한 작업 자체를 중복 체크 때문에 버퍼링하지 못하므로 변경 작업이 더 느리게 작동.

#### 유니크 인덱스 사용 시 주의사항
유일성이 꼭 보장돼야 하는 칼럼에 대해선 유니크 인덱스를 생성하지만       
꼭 필요하지 않다면 유니크하지 않은 세컨더리 인덱스를 생성하는 방법도 한 번씩 고려해 보자.

### 외래키
InnoDB에서만 생성 가능하다.   
외래키 제약이 설정되면 자동으로 연관되는 테이블의 칼럼에 인덱스까지 생성한다.
외래키가 제거되지 않은 상태에서 자동으로 생성된 인덱스 삭제 할 수 없다.

- 테이블의 변경(쓰기 잠금)이 발생하는 경우에만 잠금 경합(잠금 대기)이 발생한다.
- 외래키와 연관되지 않은 칼럼의 변경은 최대한 잠금 경합(잠금 대기)을 발생시키지 않는다.

#### 자식 테이블의 변경이 대기하는 경우
부모 테이블의 해당 레코드가 쓰기 잠금이 걸려 있으면 해당 쓰기 잠금이 해제될 때까지 기다리게 되는 것   
자식 테이블의 외래키가 아닌 칼럼의 변경은 외래키로 인한 잠금 확장이 발생하지 않는다.    

#### 부모 테이블의 변경이 대기하는 경우
물리적인 외래키의 고려 사항은, 이러한 체크 작업이 아니라 이런 체크를 위해 연관 테이블에 읽기 잠금을 걸어야 하고 잠금이 다른 테이블로 확장되면 그만큼 전체적으로 쿼리의 동시 처리에 영향을 미치게 된다. 