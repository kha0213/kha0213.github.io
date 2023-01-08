---
title:  "JAVA 8 Lambda and Stream"

toc: true
toc_label: "go page"
toc_icon: "book"
toc_sticky : true

classes: wide

categories: 
  - java
tags:
  - java8
  - study
last_modified_at: 2023-01-03T00:10:00-00:00
---
# Java 8
자바 8에 들어서 굉장히 많은 변화가 있었다. 그 중 가장 큰 변화는 Java도 함수형 프로그래밍을 지원하게 되었다는 것이다.
기존에 익명 클래스로 작성해서 할 수 있었지만 코드 양이 많아 직관적이지 않고 코드가 불필요하게 길어졌는데 이를 람다와 스트림으로 깔끔하게 정리할 수 있었다.

람다식의 등장으로 자바는 객체지향 언어의 특징과 함께 함수형 언어의 특성을 갖추게 되었다.
지금부터 람다와 스트림 사용방법을 간단히 알아보자.

## Java Lambda 사용법
자바 8이상 에서는 메서드를 일급값으로 취급할 수 있다. 즉 함수도 값으로 취급할 수 있다.
예를 들어 인수로 (int x) -> x + 1 이라는 값을 넘기면 x라는 인수를 호출하면 x + 1 을 반환 하는 동작을 수행하도록 코드로 구현할 수 있다.    
간단한 예를 들어 알아보자.
```java
void param_fun() {
    int cal1 = cal(10, 5, (Integer x, Integer y) -> x + y); // 15
    int cal2 = cal(10, 5, (Integer x, Integer y) -> x - y); // 5
    int cal3 = cal(10, 5, (Integer x, Integer y) -> x * y); // 50
    int cal4 = cal(10, 5, (Integer x, Integer y) -> x / y); // 2
}

private int cal(int x, int y, BiFunction<Integer, Integer, Integer> biFunction) {
    return biFunction.apply(x, y);
}
```
이렇게 cal이라는 함수 구현하는 방법을 인수로 넘겨서 그 결과를 리턴받을 수 있다.   

람다 표현식의 특징을 알아보자.
* 익명 : 보통의 메서드와 달리 람다 표현식은 이름이 없으므로 "익명" 이다. 따라서 재사용이 안되고 간결하게 표현해야 한다.
* 함수 : 람다는 특정 클래스에 종속되지 않으므로 "함수"라고 부른다.
* 전달 : 람다 표현식을 메서드 인수로 전달하거나 변수에 할당할 수 있다.
* 간결성 : 구현 부분만 간결하게 적으면 추론에 의해 작동한다.

### 람다 표현식 작성방법
람다 표현식은 파라미터 리스트, 화살표, 람다 바디로 이루어진다.   
![lambda.example]({{ site.baseurl }}/assets/images/study/lambda.example.png)   

람다식은 익명 함수를 간단하게 나타낸 것이다. (물론 변수에도 할당가능하다.)   

```markdown
(매개변수, ...) -> { 실행문 }
```

화살표(->)를 기준으로 왼쪽에는 람다식을 실행하기 위한 매개변수가 위치하고, 오른쪽에는 매개변수를 이용한 실행 코드 혹은 실행 코드 블럭이 온다. 

1.  람다식의 매개변수 부분에는 타입을 생략 할 수 있다.
2.  람다식의 매개변수가 하나일 때는 괄호를 생략 할 수 있다.
3.  실행문이 한 줄인경우{} 을 생략할 수 있다. (이 때 세미콜론도 생략해야하며 return 문을 붙이지 말아햐 한다.
4.  람다식을 변수에 할당하려면 그 변수 타입은 Functional Interface여야 한다.

Ex) 예시를 통해 알아보자.
```java
int square(int x) {
    return x * x;
}
/*
    1. (int x) -> x * x	// type이 있으므로 괄호( ) 생략 불가
    2. (x) -> x * x	// type 생략 가능
    3. x -> x * x	// type이 없고, parameter가 1개이므로 괄호 ( ) 생략 가능
 */
    
int max(int a, int b) {
    return a > b ? a : b;
}
/*
    1. (int a, int b) -> { return a > b ? a : b; }
    2. (int a, int b) -> a > b ? a : b		// type이 있으므로 괄호( ) 생략 불가
    3. (a, b) -> a > b ? a : b			// parameter가 2개이므로 괄호( ) 생략 불가
*/
    
```



## Functional Interface (함수형 인터페이스)<br>
* Functional Interface란 '구현해야 할 추상 메서드가 하나만 정의된 인터페이스'를 가리킨다.   
* java 8 이후로 Interface 에 default 메서드로 구현하지 않아도 되는 메서드가 존재한다.   
* Functional Interface는 추상 메서드가 단 하나인 인터페이스를 말한다.   
* 추상 메서드가 하나여야만 람다식 적용이 가능하다.   

### 1. Function<T,R>
자바 8에선 Function 객체를 이용하여 익명함수를 쉽게 구현할 수 있다.   
new 연산자에서 추상 메소드를 Override해서 쓸 수 있지만 람다를 사용하면 코드의 가독성이 좋아지고 깔끔하게 해결된다.   
Function<T,R> 객체에서는 T가 입력 값 R이 결과값(리턴 타입)이다.   
```java
// 일반적인 Function 함수
Function<String,Integer> toInt = new Function<String, Integer>() {
    @Override
    public Integer apply(String value) {
        return Integer.parseInt(value);
    }
};

// 람다를 이용한 Function 함수
final Function<String,Integer> toIntRamda = 
	value -> Integer.parseInt(value);
```

### 2. Consumer<T>
Function과 비슷하지만 리턴 타입이 void이면 Consumer 객체로 만들면 된다.   
```java
final Consumer<String> printR = new Consumer<String>() {
    @Override
    public void accept(String value) {
       System.out.println("printR:::"+value);
    }
};
       
// 람다
final Consumer<String> printR2 = 
	value -> System.out.println("printR2:::"+value);
```

### 3. Predicate<T>
결과값(리턴 타입)이 항상 boolean인 Function을 말한다. 
Function<T,Boolean>과도 같으나 boolean이 자주 사용되고 객체가 아닌 기본자료형을 사용 할 때 자주 쓰인다.   
```java
    public static void main(String[] args) {
        List<Integer> numbers = 
			Arrays.asList(-5, -4, -3, -2, -1, 0, 1, 2, 3, 4, 5, 6);

        Predicate<Integer> isPositive = n -> n > 0;
        List<Integer> positiveNumbers = 
			filter(numbers, isPositive);

        Predicate<Integer> lessThan3 = n -> n < 3;
        List<Integer> lessThan3Numbers = 
			filter(numbers, lessThan3);
        
        // Predicate 부분을 바로 만들어 넘겨도 된다.
        List<Integer> zeroNumber = filter(numbers, i -> i == 0);
    }
    
    // T로 모든 객체에 대응 할 수 있게 만들어보았다.
    private static <T> List<T> filter(List<T> list, Predicate<T> filter) {
        List<T> result = new ArrayList<>();
        for(T input : list) {
            if(filter.test(input)) {
                result.add(input);
            }
        }
        return result;
    }    
```

### 4. Supplier<T>
리턴타입이 T인 Function을 말한다. 인자가 없는게 특징이다.   
```java
Supplier<String> supplier = () -> "supplier method!";
```

* * *
## boxing and unboxing
자바의 데이터 타입은 기본형(int, double, byte, char) 또는 참조형 (Integer, Double, Byte, Object, List...) 이 있다.   
제네릭 파라미터 <T> 에서 T는 참조형 만 사용할 수 있다.   
java에서 기본형을 참조형으로 변환하는 기능을 boxing 이라고 한다. (참조형을 기본형으로 변환은 unboxing)   
자동으로 boxing과 unboxing이 이루어지는 것을 autoboxing 이라고 한다.  
하지만 이런 변환은 비용이 소모되어 함수형 인터페이스는 여러 종류를 지원한다.

ex) IntPredicate, LongSupplier...
```java
// autoboxing이 일어난다.
Predicate<Integer> integerPredicate = a -> a > 0;
// 기본형 변수로 일어나지 않는다.
IntPredicate intPredicate = b -> b > 0;
```

다음은 이런 함수형 인터페이스를 설명한다.   

![인터페이스 ](https://user-images.githubusercontent.com/56239469/104201429-1a0a8e80-546d-11eb-9581-e4c0552e131f.png)   
![functionalInterface]({{ site.baseurl }}/assets/images/study/lambda/functionalInterface.png)   

# Stream

Java 8에서 `Stream`API는 선언적 방식으로 데이터를 처리하는 방법으로 도입되었습니다. 스트림은 하나 이상의 작업을 수행할 수 있는 일련의 요소를 나타냅니다.

`Stream`API 사용에는 세 가지 주요 부분이 있습니다 .

1.  **생성 작업** : 처리하고자 하는 데이터로 스트림을 생성합니다. List나 Array아니면 생성하는 함수로 생성합니다.

2.  **중간 작업** : 데이터에 대해 수행되는 작업입니다. 데이터를 입력으로 사용하고 새 스트림을 출력으로 생성합니다. 중간 작업의 몇 가지 예에는 `filter`, `map`및 가 `sorted`있습니다.

3.  **종단 작업** : 결과를 생성하는 최종 작업입니다. 스트림을 소비하고 스트림이 아닌 결과를 생성합니다. 터미널 작업의 몇 가지 예에는 `forEach`, `reduce`및 가 `count, collect 등이 `있습니다.

![stream01]({{ site.baseurl }}/assets/images/study/lambda/stream01.jpeg)


## 생성연산

### 컬렉션(Collection)으로 생성

기본적으로 컬렉션 구현 클래스의 `stream` 메서드를 이용하여 스트림을 생성할 수 있습니다.
```java
List<String> list =
    List.of("a1","a2","b1","b2","c2","c1");
Stream<String> stream2 = list.stream();
```
### 

배열(Array)로 생성

Arrays.stream 메서드를 사용하여 배열로 스트림을 생성할 수 있습니다.
```java
String[] array = new String[]{"a1","a2","b1","b2","c2","c1"};

Stream<String> stream3 = Arrays.stream(array);
```

### 자료들로 부터 스트림 생성

Stream.of 메서드를 사용하여 스트림을 생성할 수 있습니다.

```java
Stream<String> stream1 = Stream.of("a1","a2","b1","b2","c2","c1");
```

### 병렬 스트림 생성

스트림을 생성할 때 parallelStream() 로 생성하거나 기존 스트림에 parallel() 메서드를 추가하면 병렬 스트림으로 생성할 수 있습니다.

### 기본 타입에 특화된 스트림 생성

오토 박싱과 언박싱의 비효율적인 측면을 줄이기 위해 기본 타입에 특화된 스트림을 사용할 수 있습니다.   
자바에서는 기본적으로 IntStream, LongStream, DoubleStream이 제공됩니다.

기존 스트림의 mapToInt(), mapToDouble() 등을 통해 기본 타입 스트림으로 변화할 수 있습니다.

```
int[] intArray = new int[]{1, 2, 3, 4, 5, 6, 7, 8};
IntStream intStream = IntStream.of(intArray);

```

## 중간 연산

### filter : 스트림 내 필터링

스트림 내 요소들을 조건에 맞게 필터링할 수 있습니다. 메서드의 인자는 Predicate&lt;T&gt; 입니다.

### map : 특정 형태로 변환

스트림 내 요소를 원하는 특정 형태로 변환할 수 있습니다.

### sorted : 정렬

스트림 내 요소를 정렬할 수 있습니다.

### distinct : 중복제거

스트림 내의 요소의 중복을 제거합니다. 기본형 타입의 경우 값(value)으로 비교하지만 객체의 경우 Object.equals 메서드로 비교합니다.

### flatmap : 단일 원소 스트림으로 변환

중첩된 구조를 한 단계 없애고 단일 원소 스트림으로 만들어줍니다.

## 종단 연산

### 결과 모으기 (Collect)

- Collectors 패키지를 이용하면 쉽게 스트림 내 결과를 모을 수 있다.
- List, set, map, String(join) 등등..

### 순회 (forEach, forEachOrdered)

- forEach : 스트림 내 요소를 순서에 상관없이 실행
- forEachOrdered : 스트림 내 요소를 정렬된 순서로 실행

### 요소 검사 (anyMatch, allMatch, noneMatch)

- anyMatch : 스트림 요소 중 하나라도 만족하면 true
- allMath : 스트림 요소가 모두 만족해야 true
- noneMatch : 스트림 요소 중 만족하는 것이 없어서 true

### 단일 요소 반환 (findAny, findFirst)

- findAny : 스트림 중 단일 요소 아무거나 반환
- findFirst : 스트림 중 가장 첫 번째 요소 반환 (sort와 같이 쓰면 좋다.)

### 계산하기: 최솟값, 총합, 평균 등

IntStream 의 summaryStatistics 기능을 사용하면 min, max, count, average, sum 등의 정보를 알 수 있다.
IntStream.of(1,2,3,4,5,6,7,8)
.summaryStatistics()

### 결과 합치기 : reduce

![reduce]({{ site.baseurl }}/assets/images/study/lambda/reduce.jpeg)

reduce 모든 스트림 요소를 처리하여 결과를 구할 수 있습니다.
```java
List<Integer> list = List.of(1,2,3,4,5,6,7,8,9,10);

Optional<Integer> reduce = list.stream()

reduce((num1, num2) -> num1 + num2);

int sum = reduce.get();
```

## Reference
[https://hbase.tistory.com/78](https://hbase.tistory.com/78)   
[https://madplay.github.io/post/java-streams-intermediate-operations](https://madplay.github.io/post/java-streams-intermediate-operations)   

