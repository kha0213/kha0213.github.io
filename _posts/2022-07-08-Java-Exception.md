---
title:  "Java Exception"

toc: true
toc_label: "go page"
toc_icon: "book"
toc_sticky : true

classes: wide

categories:
  - java
tags:
  - exception
last_modified_at: 2022-07-08T00:40:00-00:00
---

## 1. 자바에서 예외처리
예외는 프로그램의 주요 로직에서 비정상적인 일이 발생했을 때 작업을 지속할 수 있게 해주는 것을 말한다. 간단히 예를 들어보자.

```java
    public int divide(int a, int b) {
        int result = a / b;
        return result;
    }
```
a, b 두 개의 인자를 받아 나눈 몫을 전달하는 간단한 함수이다. 그러나 여기서 b의 인자로 0을 넘겨주면 어떻게 될까?   
0 으로 나누는 것은 수학적으로 불가능하다 이와 같이 비정상적인 상황에 JVM은 예외를 발생시킨다.   

```java
    public int divide(int a, int b) {
        if( b == 0 ) return 0; // 예외경우
        int result = a / b;
        return result;
    }
```
혹시 0으로 나누는 것과 같이 간단한 예외는 if문으로 위와같이 분기 처리 할 수 있다고 생각할지 모른다. 그럼 다음 예제를 확인해보자.

```java
readFile {
    open the file; 
    determine its size;
    allocate that much memory;
    read the file into memory;
    close the file;
}
```
readFile은 파일을 열고, 파일 크기를 가져오고 메모리의 크기를 늘리고 메모리에 파일 정보를 할당하고 파일을 닫는 간단한 메서드로 구성되어있다.
여기서도 비정상적인 일이 발생할 경우는 굉장히 많다.
1. 파일을 찾을 수 없다면?
2. 만약 권한 때문에 파일을 열 수 없다면?
3. 파일 길이를 결정 할 수 없다면?
4. 메모리의 크기를 늘릴 수 없거나 메모리에 파일 정보를 할당 할 수 없다면?
5. 파일이 실행중이어서 닫을 수 없다면?
과 같이 비정상적인 일이 생길 여지는 아주 많다.

java는 예외가 생길 때 처리할 수 있는 방법이 크게 2가지가 있다.

![exception1]({{ site.baseurl }}/assets/images/study/exception1.PNG)   
1. 예외를 처리 한 다음에 리턴하는 것이다.
```java
    void A() {
        B();
    }

    void B() {
        try {
            int b = 1 / 0; // 0으로 나눌 수 없어서 ArithmeticException 에러 발생
        } catch (ArithmeticException e) {
            // 에러 처리 후 정상 흐름 리턴     
        }
    }
```
0으로 나누는 과정에서 ArithmeticException이 발생하지만 메서드 B에서 예외를 처리하여 정상흐름으로 되돌린 뒤 리턴한다.   

2. 예외를 호출한 곳으로 던지는 것이다.    
![exception2]({{ site.baseurl }}/assets/images/study/exception2.PNG)   
```java
    void A() {
        B(); // 여기서 에러 발생!!
    }

    void B() throws ArithmeticException {
        int b = 1 / 0;
    // ArithmeticException 에러 발생한 것을 호출한 곳으로 던진
    }
```   
에러난 메서드 B를 호출한 메서드 A로 에러를 던지는 것이다. 그러면 A 메서드에서 다시 try catch를 걸어주거나 다시 throws로 던지면 된다.   

## 2. Checked Exception UnChecked Exception
![Exception-Class]({{ site.baseurl }}/assets/images/study/Exception-Class.png)   
java는 에러 난 상황을 크게 오류(Error)와 예외(Exception)으로 구분한다. 예외(Exception)는 개발자가 처리 할 수 있는 에러이고 오류(Error)는 시스템에서 발생하여 개발자가 처리 할 수 없는 에러이다.   
오류(Error)로 예를 들면 Out of memory가 있다. Jvm의 메모리가 부족한 경우 발생하는 에러는 개발자가 코드로 할 수 없는 부분이다. (그래서 Throwable 뒤의 동작은 자바가 보증하지 않는다.)
개발자가 처리할 수 있는 최상의 예외 클래스는 Exception 클래스이다

* java Exception은 크게 두 종류로 Checked Exception 과 UnChecked Exception 으로 나눌 수 있다.
Exception의 자식클래스 중 RuntimeException과 그 자식클래스는 UnChecked Exception이고 나머지는 Checked Exception이다.     
Checked Exception은 명시적으로 throws를 작성해야 하는 것이고 UnChecked Exception은 throws를 생략 가능한 것이다.   

* Checked Exception 과 UnChecked Exception 차이
<table>
    <colgroup>
        <col width="10%">
        <col width="10%">
        <col width="10%">
    </colgroup>
    <tr>
        <th></th>
        <th>Checked Exception</th>
        <th>UnChecked Exception</th>
    </tr>
    <tr>
        <td>구분</td>
        <td>Exception의 하위 클래스 중 <br/>RuntimeException을 제외한 모든 예외</td>
        <td>RuntimeException 의 하위 예외</td>
    </tr>
    <tr>
        <td>처리여부</td>
        <td>throws 또는 try로 반드시 예외 처리 방법을 명시해야 한다. </td>
        <td>try는 가능하고 throws는 생략 가능하다.</td>
    </tr>
    <tr>
        <td>스프링 트랜잭션에서 처리</td>
        <td>예외나도 커밋함</td>
        <td>예외나면 롤백</td>
    </tr>
</table>

Checked Exception과 Unchecked Exception은 언제 사용하면 좋을까?
```java
public class UserController {
    public void join(User user) throws SQLException {
        userService.save(user);
    }
}

public class UserService {
    private final DataSource ds;

    public void save(User user) throws SQLException {
        Connection conn = DataSourceUtils.getConnection(dataSource);
        PreparedStatement pstmt = conn.prepareStatement("insert into Member values (?, ?)");
        pstmt.setString(user.getName());
        pstmt.executeUpdate();
    }
}
```
간단히 Controller에서 데이터를 받아 Service에서 유저를 등록하는 로직이다.   
여기서는 DB와 연결하기위해 Connection을 사용함으로써 Checked Exception인 SQLException이 발생할 수 있어 throws를 명시하였다.   
하지만 SQLException이 나는 경우를 생각해보면 Sql문법이 오류나거나 DB연결이 안되거나 하는 예외가 발생할 수 있다. 이 예외는 Service나 Controller에서 처리 할 수 없는 예외이다.   
(DB연결이 정보가 틀려서 예외인 경우 정보를 바르게 입력해야지 개발자의 코드로 어떻게 할 수 있는 부분이 아니다.)   
따라서 여기서 throws SQLException을 해봤자 의미 없는 코드의 나열이고 결합도만 높아져 클린 코드라 할 수 없다.   
이 예제에서는 SQLException이 어차피 컨트롤러나 서비스에서 처리할 수 없는데 Checked Exception이어서 명시적으로 처리 하는게 나쁜 예시라 할 수 있다.   
이에 스프링에서는 예외변환기를 통해 Checked Exception을 UnChecked Exception으로 바꿔준다.   

## 3. Spring ExceptionTranslator
스프링은 위와같은 불필요한 CheckedException 을 UnCheckedException으로 변환해주어서 결합도를 약화시켰다.
대표적으로 
SQLErrorCodeSQLExceptionTranslator 클래스를 확인해보면 

😊SQLErrorCodeSQLExceptionTranslator.java
```java
if (Arrays.binarySearch(sqlErrorCodes.getBadSqlGrammarCodes(), errorCode) >= 0) {
    logTranslation(task, sql, sqlEx, false);
    return new BadSqlGrammarException(task, (sql != null ? sql : ""), sqlEx);
}
else if (Arrays.binarySearch(sqlErrorCodes.getInvalidResultSetAccessCodes(), errorCode) >= 0) {
    logTranslation(task, sql, sqlEx, false);
    return new InvalidResultSetAccessException(task, (sql != null ? sql : ""), sqlEx);
}
else if (Arrays.binarySearch(sqlErrorCodes.getDuplicateKeyCodes(), errorCode) >= 0) {
    logTranslation(task, sql, sqlEx, false);
    return new DuplicateKeyException(buildMessage(task, sql, sqlEx), sqlEx);
}
else if (Arrays.binarySearch(sqlErrorCodes.getDataIntegrityViolationCodes(), errorCode) >= 0) {
    logTranslation(task, sql, sqlEx, false);
    return new DataIntegrityViolationException(buildMessage(task, sql, sqlEx), sqlEx);
}
```
위 처럼 SQLException의 에러코드가 있을 때 UnCheckedException인 DataAcceptException의 하위예외로 변환해주는 것을 알 수 있다. 





## Reference
[https://docs.oracle.com/javase/tutorial/essential/exceptions/index.html](https://docs.oracle.com/javase/tutorial/essential/exceptions/index.html)   
[https://docs.oracle.com/javase/tutorial/essential/exceptions/runtime.html](https://docs.oracle.com/javase/tutorial/essential/exceptions/runtime.html)      




   




