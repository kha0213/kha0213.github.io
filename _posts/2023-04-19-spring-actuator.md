---
title:  "Spring Boot Actuator 소개"

toc: true
toc_label: "go page"
toc_icon: "book"
toc_sticky : true

classes: wide

categories: 
  - spring
tags:
  - actuator
  - spring boot
  - Production ready
last_modified_at: 2023-04-19T00:08:00-00:00
---
## Production ready
어떠한 서비스를 운영하기 위해서는 해당 서버의 자원은 어느정도 쓰이고 있는지, 에러가 발생하는 상황인지, 발생했다면 
그 에러를 추적할 수 있는지, 서버는 현재 살아있는지, 로그 정보는 정상적으로 설정되어있는지, 커넥션 풀은 어느정도 사용되고 있는 지 등의 정보를 알아야 한다.   

운영환경에서 서비스하기 위해 필요한 기능들을 **Production ready** (프로덕션 준비 기능) 라고 한다.   

## Actuator란?
Spring Boot Actuator는 이러한 프로덕션 준비 기능을 매우 편리하게 사용할 수 있는   
Spring Boot 애플리케이션의 모니터링 및 관리를 지원하는 라이브러리이다.   
HTTP 엔드포인트 또는 JMX를 사용하여 애플리케이션을 관리하고 모니터링하도록 선택할 수 있다.    
감시나 상태 및 메트릭 수집을 애플리케이션에 자동으로 적용할 수도 있다.   

Actuator는 실행 중인 애플리케이션의 내부를 볼 수 있게 하고, 어느 정도까지는 애플리케이션의 작동 방법을 제어할 수 있게 한다. 
예를 들면, 다음과 같다.

- 애플리케이션의 환경정보, OS나 git, 구성 속성들 정보
- 애플리케이션 다양한 로깅 레벨(logging level) 조회 및 변경
- 애플리케이션이 사용 중인 메모리, 쓰레드, 커넥션 개수 실시간 모니터링
- 지정된 엔드포인트가 받은 요청 횟수, 엔드포인트에 대한 요청과 응답 정
- 애플리케이션의 적용중인 프로그램의 상태 점검

## Actuator 적용
**spring-boot-actuator** 의존성만 주입 받으면 기본적인 기능이 활성화된다.

```groovy
# build.gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
}
```

스프링 부트 어플리케이션을 실행한 다음 **/actuator** 로 요청을 날려보면 다음과 같은 응답이 온다. 

```json
{
    "_links": {
        "self": {
            "href": "http://localhost:8080/actuator",
            "templated": false
        },
        "health": {
            "href": "http://localhost:8080/actuator/health",
            "templated": false
        },
        "info": {
            "href": "http://localhost:8080/actuator/info",
            "templated": false
        }
    }
}
```

Actuator에 기본적인 /health, /info 외에는 엔드포인트가 노출이 안 되어 있는 모습이다.
엔드포인트에는 민감한 정보가 포함될 수 있으므로 언제 노출 할지 신중하게 고려해야 한다.
프로퍼티를 설정하여 엔드포인트를 노출 할 수 있고 더 세부적인 다양한 정보들을 알 수 있다.

```properties
# application.properties
management.endpoints.web.exposure.exclude=beans,env
management.endpoints.web.exposure.include=*
```

**Note:** exclude 속성이 include 보다 우선한다.       
{: .notice--info}    

## Actuator 기능
Actuator는 엔드포인트로 다음과 같은 기능을 제공한다.

|                 |                 |                                                 |
|-----------------|-----------------|-------------------------------------------------|
| HTTP 메서드        | 경로              | 설명                                              |
| GET             | /auditevents    | 호출된 audit 이벤트 리포트를 생성한다.                        |
| GET             | /beans          | 스프링 컨테이너에 등록된 스프링 빈을 보여준다.                      |
| GET             | /conditions     | condition의 조건과 성공 또는 실패했던 자동 구성 조건의 내역을 생성한다.   |
| GET             | /configprop     | 모든 구성 속성들을 현재 값과 같이 알려준다.                       |
| GET,POST,DELETE | /env            | 스프링 애플리케이션에 사용할 수 있는 모든 속성 근원과 이 근원들의 속성을 알려준다. |
| GET             | /env/{toMatch}  | 특정 환경 속성의 값을 알려준다.                              |
| GET             | /health         | 애플리케이션의 상태 점검 정보를 반환한다.                         |
| GET             | /heapdump       | 힙 덤프를 다운로드한다.                                   |
| GET             | /httptrace      | 요청에 대한 추적 기록을 생성한다.                             |
| GET             | /info           | 애플리케이션에 관련 정보를 반환한다.                            |
| GET             | /loggers        | 애플리케이션의 로깅 정보를 알려준다.                            |
| GET,POST        | /loggers/{name} | 로깅 레벨을 반환한다. 로깅 레벨을 변경할 수 있다.                   |
| GET             | /mappings       | 모든 HTTP 매핑과 이 매핑들을 처리하는 핸들러 메서드들의 내역을 제공한다.     |
| GET             | /metrics        | 모든 메트릭 리스트를 반환한다.                               |
| GET             | /metrics/{name} | 지정된 메트릭의 값을 반환한다.                               |
| GET             | /scheduledtasks | 스케줄링된 모든 태스크의 내역을 제공한다.                         |
| GET             | /threaddump     | 모든 애플리케이션 스레드의 내역을 반환한다.                        |
| POST            | /shutdown       | 애플리케이션을 종료한다. 이 기능은 기본으로 비활성화 되어 있다.            |

아래에 더 많은 기능이 표시되어 있다.   
[https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints)   

Actuator의 기능 중 몇 가지에 대해 더 자세히 알아보자.   

### 1. health
health 엔드포인트는 애플리케이션의 상태를 확인할 수 있는 엔드포인트이다.   

[GET] http://localhost:8080/actuator/health
```json
{
    "status": "UP"
}
```

서버 상태가 정상이면 **UP**, 비정상이면 **DOWN** 이다.
health에 대해 더 상세한 정보를 확인하고 싶으면 다음과 같은 프로퍼티를 추가하면 된다.

```properties
# application.properties
management.endpoint.health.show-components=always
```

```json
{
    "status": "UP",
    "components": {
        "db": {
            "status": "UP"
        },
        "diskSpace": {
            "status": "UP"
        },
        "ping": {
            "status": "UP"
        }
    }
}
```

**Note:** 만약 이것보다 더 자세한 정보를 원하면   
management.endpoint.health.show-details=always 프로퍼티를 추가하면 된다.   
{: .notice--info}   

해당 정보 뿐만 아니라 HealthIndicators 를 구현한 빈을 등록하면 해당 빈의 상태도 확인할 수 있다.
스프링부트에서는 다음과 같은 HealthIndicators 를 기본적으로 제공한다.
* CasandraHealthIndicator : Cassandra 연결 상태를 확인한다.
* CouchbaseHealthIndicator : Couchbase 연결 상태를 확인한다.
* DataSourceHealthIndicator : 데이터베이스 연결 상태를 확인한다.
* DiskSpaceHealthIndicator : 디스크 용량을 확인한다.
* ElasticsearchHealthIndicator : Elasticsearch 연결 상태를 확인한다.
* HazelcastHealthIndicator : Hazelcast 연결 상태를 확인한다.
* InfluxDbHealthIndicator : InfluxDB 연결 상태를 확인한다.
* JmsHealthIndicator : JMS 연결 상태를 확인한다.
* LdapHealthIndicator : LDAP 연결 상태를 확인한다.
* MailHealthIndicator : Mail 연결 상태를 확인한다.
* MongoHealthIndicator : MongoDB 연결 상태를 확인한다.
* Neo4jHealthIndicator : Neo4j 연결 상태 를 확인한다.
* PingHealthIndicator : ping 명령어를 사용해서 특정 호스트에 연결 가능한지 확인한다.
* RabbitHealthIndicator : RabbitMQ 연결 상태를 확인한다.
* RedisHealthIndicator : Redis 연결 상태를 확인한다.

## 2. info

[https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints.info](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints.info)   

info 엔드포인트는 애플리케이션의 기본 정보를 노출한다.

확인 할 수 있는 정보는 다음과 같다.

* java : 자바 런타임 정보
* os: OS 정보
* env : Environment 에서 info. 로 시작하는 정보
* build : 빌드 정보, META-INF/build-info.properties 파일이 필요하다. 
* git : git 정보, git.properties 파일이 필요하다.

각각의 정보를 활성화 하기 위해서는 **management.info.<id>.enabled=true** 프로퍼티를 추가하면 된다.

```properties
management.info.java.enabled=true
management.info.os.enabled=true
management.info.env.enabled=true
```

build 정보를 활성화 하기 위해서는 META-INF/build-info.properties 파일이 필요하다.   

다음과 같이 build.gradle 파일에 추가하면 간단히 만들 수 있다.
```groovy
// build.gradle
springBoot {
    buildInfo()
}
```

git 정보를 활성화 하기 위해서는 git.properties 파일이 필요하다.

다음과 같이 build.gradle 파일에 추가하면 간단히 만들 수 있다. (물론 해당 프로젝트가 git으로 관리되고 있어야 한다.)   
이러면 build 폴더 안에 resources/main/git.properties 파일이 생성된다.
```groovy
// build.gradle
plugins {
    id "com.gorylenko.gradle-git-properties" version "2.4.1" //git info
}
```

## 3. logger
[https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints.loggerhttps://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.loggers]([https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints.loggerhttps://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.loggers)   

logger 엔드포인트는 로거의 레벨을 확인하고 변경할 수 있다.

**/actuator/loggers** 로 조회하면 모든 로깅레벨을 확인할 수 있다.

다음과 같은 패턴으로 특정 로거를 조회할 수 있다.  
```markdown
[GET] http://localhost:8080/actuator/loggers/{로거이름}
```

예를 들면
```
[GET] http://localhost:8080/actuator/loggers/org.springframework.web.util
```

```json
{
    "effectiveLevel": "INFO"
}
```

## Actuator와 보안
Actuator는 HTTP 엔드포인트로 제공되기 때문에 보안에 대한 고려가 필요하다.   
Actuator로 접속 가능한 포트를 분리하고 내부망을 사용하는 것이 안전하다.   
application.properties에 다음과 같이 설정하면 Actuator에 사용되는 포트를 분리할 수 있다.   

```properties
# application.properties
management.server.port=9292
```

## Reference
[https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html)    
[https://techblog.woowahan.com/9232/](https://techblog.woowahan.com/9232/)      
   

