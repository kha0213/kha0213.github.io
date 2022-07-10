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

## StringUtils
java에서 String은 정말 많이 다루다 보니 이를 쉽게 활용할 수 있는 StringUtils 클래스가 많이 있다 대표적으로 
springframework에서 기본적으로 포함된 StringUtils와 apache.commons.lang3에서 만든 StringUtils를 살펴보자.      

[참고: apache.commons.lang3의 유용한 패키지들](https://recordsoflife.tistory.com/474)   
apache.commons.lang3 에선 java에서 사용하기 편한 다양한 Utils 클래스를 제공한다.
{: .notice--info}   
     
### org.apache.commons.lang3.StringUtils
기본적으로 null-safe 한 연산을 해주기 때문에 아주 유용하고 기본 String에는 없는 유용한 메서드들이 있다. (최근에 repeat(java 11) 같은 메서드들도 String에 기본으로 들어가 있긴 하지만 그래도 유용하다.)      
😊아래는 개인적으로 자주 쓰는 메서드이다. 
```java
@Test
void stringUtilTest() {
    // null 이거나 "" 인 경우 true
    assertTrue(StringUtils.isEmpty(""));
    // 인자가 null일 경우 ""로 바꿔준다.
    assertEquals("", StringUtils.defaultString(null));
    // 인자의 string을 뒤집어 준다.
    assertEquals("edcba", StringUtils.reverse("abcde"));
    // 인자 String만큼 반복해준다.
    assertEquals("*****", StringUtils.repeat("*", 5));
    // 앞에 해당 char만큼 붙여준다. (숫자나 날짜 계산에 편하다)
    assertEquals("0010", StringUtils.leftPad("10", 4, '0'));
    // 알파벳만 있는지 검사해준다.
    assertTrue(StringUtils.isAlpha("abcdddee"));
}
```
전체 메서드이다. 특히 주석에 예제까지 굉장히 잘 달아놔서 공식문서 없이 그냥 사용해도 될 수준으로 잘 되어있으니 꼭 사용해보자.   

* Is Empty/Is Blank - 문자열에 텍스트가 포함되어 있는지 확인합니다.
* Trim/Strip - 앞 뒤에 공백 제거
* Equals/Compare - 두 문자열을 비교합니다.
* starts With - 문자열의 앞부분이 해당 단어로 시작하는지 확인합니다.
* endsWith - 문자열이 뒷부분이 해당 당어로 끝나는지 확인합니다.
* IndexOf/LastIndexOf/Contains - 문자열의 Index에 대한 검색입니다.
* IndexOfAny/LastIndexOfAny/IndexOfAnyBut/LastIndexOfAnyBut - 문자열 인덱스 중 어느 것이라도 포함되는지 확인합니다.
* ContainsOnly/ContainsNone/ContainsAny - 문자열에 이러한 문자만 포함/포함하지 않음/
* Substring/Left/Right/Mid - 하위 문자열 추출입니다.
* Substring Before/SubstringAfter/SubstringBetween - 다른 문자열에 대한 하위 문자열 추출
* Split/Join - 문자열을 하위 문자열 배열로 분할하거나 그 반대로 분할합니다.
* Remove/Delete - 문자열의 일부를 제거합니다.
* Replace/Overlay - 문자열을 검색하고 한 문자열을 다른 문자열로 바꿉니다.
* Chomp/Cop - 문자열의 마지막 부분을 제거합니다.
* AdpendIfMissing - 없는 경우 문자열 끝에 해당 단어를 추가합니다.
* PrependIfMissing - 없는 경우 문자열의 시작 부분에 해당 단어를 추가합니다.
* LeftPad/RightPad/Center/Repeat - 문자열 패드
* UpperCase/LowerCase/SwapCase/Capitalize/Uncapitalize - 문자열의 대소문자를 변경합니다.
* CountMatches - 다른 문자열에서 한 문자열의 발생 횟수를 카운트합니다.
* IsAlpha/IsNumeric/IsWhitespace/IsAsciiPrintable - 문자열의 문자를 확인합니다.
* DefaultString - null 입력 문자열로부터 보호합니다.
* Rotate - 문자열 회전(원형 이동)
* Reverse/ReverseDelimited - 문자열을 역순으로 바꿉니다.
* Abbreviate - 줄임말 또는 다른 주어진 문자열을 사용하여 문자열을 생략합니다.
* Difference - 문자열과 해당 차이에 대한 보고서를 비교합니다.
* Levenshtein Distance - 한 문자열을 다른 문자열로 변경하는 데 필요한 변경 횟수

이 이외에도 apache.commons.lang3에서는 NumberUtils, ArrayUtils 등 정말 유용한 메서드들을 많이 정의해놓았다.   
{: .notice--info}

### org.springframework.util.StringUtils
기본적인 springframework 에서 가지고 있는 StringUtils이다. 위의 apache보단 기능이 부족하지만 그래도 어차피 사용하는 spring에만 의존적이기 때문에 유용할 경우가 많다.
<pre>    
Mainly for internal use within the framework; 
consider Apache's Commons Lang for a more 
comprehensive suite of String utilities.
</pre>
😊위와같이 문서에서도 Apache의 StringUtils가 더 기능이 많다고 나와있다.
```java
@Test
void springStringUtils() {
    // apache의 isEmpty와 비슷하다 null 이다 "" 이면 false 이다.
    assertFalse(StringUtils.hasLength(""));
    // apache의 isBlank와 비슷하다 null, "", "  " 이면 false이다.
    assertFalse(StringUtils.hasText(""));

    // collection의 string을 join한다.
    List<String> list = List.of("apple", "banana", "lemon");
    assertEquals("apple|banana|lemon", StringUtils.collectionToDelimitedString(list, "|"));
    // 사실상 String의 join을 더 자주 쓰긴 한다.
    assertEquals("apple|banana|lemon", String.join("|", list));

    // 파일 이름을 출력한다 (확장자 포함)
    assertEquals("test.txt",StringUtils.getFilename("D:/app/test.txt"));
    // 파일 확장자만 출력한다.
    assertEquals("txt",StringUtils.getFilenameExtension("D:/app/test.txt"));
}
```

## Reference
[https://jeong-pro.tistory.com/85](https://jeong-pro.tistory.com/85)   
[https://www.baeldung.com/java-string-pool](https://www.baeldung.com/java-string-pool)   
[https://www.javamadesoeasy.com/2015/05/string-pool-string-literal-pool-string.html](https://www.javamadesoeasy.com/2015/05/string-pool-string-literal-pool-string.html)   
[https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/StringUtils.html](https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/StringUtils.html)      
[https://docs.oracle.com/javase/8/docs/api/java/lang/String.html](https://docs.oracle.com/javase/8/docs/api/java/lang/String.html)   
      




   




