---
title:  "Java의 String 클래스에 대해서"

toc: true
toc_label: "go page"
toc_icon: "book"
toc_sticky : true

classes: wide

categories:
  - java
tags:
  - String
last_modified_at: 2022-07-08T00:40:00-00:00
---

## 1. 자바에서 String 클래스란?
자바에서 문자열을 다루는 클래스의 대표가 String이다.   
자바를 해봤다면 당연히 써 봤을 것이나 여기서는 String 클래스에 대해 사용법에 대해 조금 더 자세하게 설명하고자 한다.   
String은 불변(immutable)의 객체로 값을 변경 할 수 없어 해시코드의 캐싱이 가능하고 쓰레드에 안전(Thread Safe)하다.   
```java
String a = "hello";
a = "world";
```
이렇게 사용하면 a의 값이 변경된 것이 아닌가? 라고 생각 할 수 있지만 결론은 아니다 새로운 String객체를 만들어 값을 할당하고 a가 가르키는 주소를 변경한 것이다. 
그러면 String 값이 변경될 때마다 새로운 객체가 생성이 될까? 이 답도 역시 아니다 java는 String Pool을 통해 리터럴 String을 관리한다.
자세히 알아보자

## String Pool
String 값을 생성하는 방법엔 크게 2 가지가 있다. 값을 그냥 할당하는 <strong>문자열 리터럴 생성 방식</strong>과 <strong>new 연산자</string>를 사용한 생성 방식이다.
```java
String a = "hello"; // 리터럴 생성 방식
String b = new String("hello"); // new 연산자 생성 방식
```
특수한 상황이 아니라면 a의 방법대로 생성을 해야한다.    
그 이유는 리터럴로 생성을 해야 String Pool 영역에 할당 되어 캐싱이 되기 때문이다.   

자바의 Heap메모리엔 String pool이 존재한다.     
![string-pool]({{ site.baseurl }}/assets/images/study/string-pool.png)       
1. 그림과 같이 처음 s1을 생성할 때에는 String pool이 비어 있었을 것이다. 그러면 pool에 "abc"라는
String 객체를 생성한 뒤 값을 리턴한다. 
2. s2를 생성할 때에는 new 연산자로 생성하여 pool에 관여하지 않고 heap의 다른 영역에 객체 생성이 된다.   
같은 "abc" 문자열이더라도 새로운 객체 주소를 가지게 되어 비효율적이다. 
3. s3를 생성할 때는 이미 String pool에 있던 "abc"를 가져온다.
4. s4를 생성할 때는 역시 새로운 String 객체를 생성한다. s2랑도 다르다.
5. String의 intern() 메서드를 쓰면 리터럴과 같이 String pool에 있는지 확인하고 객체를 가져온다.
6. a1은 new 연산자로 pool과 상관 없이 새로운 객체 주소를 가진다.
7. a2는 pool에 없으니 "def"를 만들어 pool에 등록하고 해당 객체를 가져온다.

😊StringTest.java
```java
public class StringTest {
    @Test
    void stringPool() {
        String s1 = "abc";
        String s2 = new String("abc");
        String s3 = "abc";
        String s4 = new String("abc");
        String s5 = new String("abc").intern();

        // comparing s1
        assertTrue(s1 != s2);
        assertTrue(s1 == s3);
        assertTrue(s1 != s4);
        assertTrue(s1 == s5);

        // comparing s2
        assertTrue(s2 != s3);
        assertTrue(s2 != s4);
        assertTrue(s2 != s5);

        // comparing s3
        assertTrue(s3 != s4);
        assertTrue(s3 == s5);

        // comparing s4
        assertTrue(s4 != s5);
    }
}
```
객체 주소를 확인하는 == 메서드로 비교해 본 결과 같은 pool에서 참조하는
s1과 s3과 s5는 주소가 같고 나머지는 다 다른것을 확인 할 수 있다.

* String pool을 사용하면 장점이 무엇일까?
문자열은 자바 코드에서 정말 널리 사용된다. 이것을 메모리에서 캐시하여 (해쉬기반으로) 메모리를 절약하고 성능을 향상 시킬 수 있다.   
이미 있는 문자열은 heap에 생성하므로 GC가 수행하는 작업도 줄어들게 된다.

## StringBuilder StringBuffer
java에서 문자열을 여러 번 더할 때 StringBuilder나 StringBuffer을 쓰라는 말을 들어 본 적이 있을 것이다. 
String 그리고 StringBuilder, StringBuffer의 차이에 대해 알아보자.

StringBuilder와 StringBuffer는 둘 다 가변클래스(mutable)로 문자열의 연산이 자주 발생할 때 사용하면 효율적이다.   
문자열의 크기가 매우 커져서 내부 버퍼가 오버플로되면 자동으로 버퍼 크기가 커지기 때문에 사용하기 편하다.    
StringBuilder는 쓰레드로부터 안전하지 않고 StringBuffer은 내부에 synchronized를 가지고 있어 쓰레드로부터 안전하다.   
(당연히 synchronized없는 StringBuilder가 성능상으로는 더 좋다)   

😊StringBuilder.java
```java
// 가변 클래스 이지만 내부에 synchronized 가 없어 쓰레드로부터 안전하지 않다.
// 성능은 StringBuffer보다 좋다.
@Override
public int compareTo(StringBuilder another) {
    return super.compareTo(another);
}

@Override
public StringBuilder append(Object obj) {
    return append(String.valueOf(obj));
}
```

😊StringBuffer.java
```java
// 가변 클래스 이고 내부에 synchronized 가 있어 쓰레드로부터 안전하다.
// 성능은 StringBuilder보다 느리다
@Override
public synchronized int compareTo(StringBuffer another) {
    return super.compareTo(another);
}
@Override
public synchronized StringBuffer append(Object obj) {
    toStringCache = null;
    super.append(String.valueOf(obj));
    return this;
}
```
synchronized를 제외하고 나머지 사용법은 StringBuffer와 StringBuilder가 거의 동일하다. 사용법을 알아보자.

* String과 StringBuffer, StringBuilder를 비교해 보자
<table>
    <colgroup>
        <col width="16%">
        <col width="28%">
        <col width="28%">
        <col width="28%">
    </colgroup>
    <tr>
        <th></th>
        <th>String</th>
        <th>StringBuffer</th>
        <th>StringBuilder</th>
    </tr>
    <tr>
        <td>클래스 타입</td>
        <td>불변(immutable)</td>
        <td>가변(mutable)</td>
        <td>가변(mutable)</td>
    </tr>
    <tr>
        <td>멀티쓰레드</td>
        <td>안전(thread safe)</td>
        <td>안전(thread safe)</td>
        <td>안전하지 않다. (non thread safe)</td>
    </tr>
    <tr>
        <td>런타임 시 문자열 처리 성능</td>
        <td>느림</td>
        <td>빠름</td>
        <td>매우 빠름</td>
    </tr>
    <tr>
        <td>컴파일 시 문자열 처리 성능</td>
        <td><strong>매우 빠름(StringBuilder사용)</strong></td>
        <td>빠름</td>
        <td>매우 빠름</td>
    </tr>
</table>

## String 메서드 소개




## StringUtils
java에서 String은 정말 많이 다루다 보니 이를 쉽게 활용할 수 있는 StringUtils 클래스가 많이 있다 대표적으로 
springframework에서 기본적으로 포함된 StringUtils와 apache.commons.lang3에서 만든 StringUtils를 살펴보자.      

[참고: apache.commons.lang3의 유용한 패키지들](https://recordsoflife.tistory.com/474)   
apache.commons.lang3 에선 java에서 사용하기 편한 다양한 Utils 클래스를 제공한다.
{: .notice--info}   
     
### org.apache.commons.lang3.StringUtils


### org.springframework.util.StringUtils
## Reference
[https://jeong-pro.tistory.com/85](https://jeong-pro.tistory.com/85)   
[https://www.baeldung.com/java-string-pool](https://www.baeldung.com/java-string-pool)   
[https://www.javamadesoeasy.com/2015/05/string-pool-string-literal-pool-string.html](https://www.javamadesoeasy.com/2015/05/string-pool-string-literal-pool-string.html)   
[https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/StringUtils.html](https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/StringUtils.html)      
[https://docs.oracle.com/javase/8/docs/api/java/lang/String.html](https://docs.oracle.com/javase/8/docs/api/java/lang/String.html)   
      




   




