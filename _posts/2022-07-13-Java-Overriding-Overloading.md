---
title:  "Java Overriding Overloading 차이"

categories:
  - java
tags:
  - tip
last_modified_at: 2022-07-12T00:40:00-00:00
---

## 개념 
### Java Overriding(재정의)이란?
메서드 시그니처(이름과 매개변수)가 슈퍼클래스와 자식클래스에서 같을 때 
이를 오버라이드(Overriding)라고 한다.

### Java Overrloading(다중정의)이란?
같은 메서드 이름을 가지는 메서드이나 다른 파라미터를 받을 때 다중정의라고 한다.   
대표적으로는 자바의 System.out.println 함수가 있다.
println(String a), println(int a), println() ... 등으로 
같은 메서드 이름이나 여러가지 매개변수를 받을 수 있고 그에 따라 실행되는 로직도 서로 다르다.    

```java
static class Parent {
    public void overload(int a) {
        System.out.println("overload a");
    }
    public void overload(int a, int b) {
        System.out.println("overload ab");
    }
    public void overload(String c) {
        System.out.println("overload c");
    }

    public void override(String a) {
        System.out.println("override a");
    }
}

static class Child extends Parent {
    @Override
    public void override(String a) {
        System.out.println("child override a");
    }
}
```
위의 코드에서 override 함수는 Parent 클래스의 함수를 Child 가 재정의했으니 overriding이다. 이런 식으로 overriding 하게 되면 일반적으로 부모의 로직 대신의 자식의 로직을 사용한다는 뜻이다.   
overload 함수는 매개변수로 int 하나를 받거나 int 2개를 받거나 String 변수를 받을 수 있다. 이렇게 메서드 이름은 같지만 서로 다른 매개변수를 가지는 것이 overloading이다.   

## 오버라이딩(Overriding) 와 오버로딩(Overrloading) 차이
오버로딩을 사용하면 호출할 메서드가 컴파일 타임에 결정되고. 오버라이딩하면 개체 유형에 따라 런타임에 메서드 호출이 결정된다.
코드를 보며 알아보자.
```java
static class Overload {
    public static void print(Object o) {
        System.out.println("this is Object parameter");
    }
    public static void print(String s) {
        System.out.println("this is String parameter");
    }
}

@Test
void overloadingTest() {
    Object a = "string";
    Overload.print(a); // this is Object parameter
}
```
위의 코드는 오버로딩을 구현한 코드이다. a가 가르키는 주소가 String 객체이라고 하더라고 
컴파일 시점엔 Object에 담았기 때문에 해당 메서드 출력은 this is Object parameter가 된다.

```java
static class Parent {
    public void print() {
        System.out.println("Parent print!!");
    }
}
static class Child extends Parent {
    @Override
    public void print() {
        System.out.println("Child print!!");
    }
}
@Test
void overridingTest() {
    Parent a = new Child();
    a.print(); // Child print!!
}
```
위 코드는 오버라이딩을 구현한 코드이다. a가 가르키는 주소가 Child 객체이고 
오버라이딩은 런타임 시점에 다형성이 구현되기 때문에 출력되는 결과는 Child print!! 가 된다. 
이로써 오버로딩과 오버라이딩 차이를 알 수 있다.

## 
<table>
    <tr>
        <th></th>
        <th>오버라이딩</th>
        <th>오버로딩</th>
    </tr>
    <tr>
        <td>접근 제어자</td>
        <td>부모 클래스의 메소드의 접근 제어자보다 같거나 더 넓은 범위의 접근 제어자를 자식 클래스의 메소드에서 설정할 수 있다</td>
        <td>모든 접근 제어자를 사용할 수 있다</td>
    </tr>
    <tr>
        <td>적용시점</td>
        <td>런타임 시점</td>
        <td>컴파일 시점</td>
    </tr>
    <tr>
        <td>리턴 타입</td>
        <td>동일해야 한다</td>
        <td>달라도 된다</td>
    </tr>
    <tr>
        <td>매개변수</td>
        <td>동일해야 한다</td>
        <td>달라야만 한다</td>
    </tr>
    <tr>
        <td>적용범위</td>
        <td><strong>상속관계</strong>에서 적용</td>
        <td><strong>같은 클래스</strong>에서 적용</td>
    </tr>
</table>
   
   
   
오버라이딩은 런타임에 동작하기 때문에 심각한 버그를 일으킬 수 있으므로 주의해서 사용해야한다.   
반면에 오버로딩은 컴파일 타임에 오류가 발생하고 수정하기 쉽다.   
(다만 상속관계를 오버로딩하는거에 주의하자!)
{: .notice--warning}

[https://dzone.com/articles/all-about-overriding-in-java](https://dzone.com/articles/all-about-overriding-in-java)      
[https://www.journaldev.com/32182/overriding-vs-overloading-in-java](https://www.journaldev.com/32182/overriding-vs-overloading-in-java)    