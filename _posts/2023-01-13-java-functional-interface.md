---
title:  "JAVA 8 Lambda and Stream"

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
람다에도 여러 줄로 실행이 될 때 {} 를 사용할 수 있지만 그러면 마지막에 세미콜론(;)을 붙여야한다.

Ex)
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