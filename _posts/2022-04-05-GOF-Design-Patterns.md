---
title:  "Design Patterns"

toc: true
toc_label: "go page"
toc_icon: "book"
toc_sticky : true

classes: wide

categories:
  - java
tags:
  - Design_Patterns
last_modified_at: 2022-04-05T00:40:00-00:00
---

## 1. Singleton Patterns (싱글톤 패턴)
인스턴스를 오직 한 개만 제공하는 패턴이다.   
가장 많이 본 것 같다. 시스템 런타임, 환경 세팅 등 인스턴스가 여러 개 일 때 문제가 생길 수 있다. 이럴 때 인스턴스를 오직 한 개만 만들어 제공하는 클래스가 필요하다.   
또한 성능을 위해 하나만 생성하는게 유리한 경우도 많다. (JpaFactory 등등)

* 구현
```java
public class Setting {
    private static Setting instance;

    private Setting(){}

    public static Setting getInstance() {
        if ( instance == null ) {
            instance = new Setting();
        }
        return instance;
    }
}
```
기본 생성자를 private로 두고 static 필드를 만들어 리턴한다. 단 이 부분은 멀티 쓰레드 일 경우 객체가 2개 생길 우려가 있다.


* Eager-Loading
```java
public class Setting {
    private static Setting instance = new Setting();

    private Setting(){}

    public static Setting getInstance() {
        return instance;
    }
}
```
이와 같은 방법은 Thread-Safe 하나 객체를 먼저 초기화 하니 사용 안하는 객체라면 메모리 부담이 될 수 있다.

```java
public class Setting {
    private static volatile Setting instance = null;

    private Setting(){}

    public static Setting getInstance() {
        if (instance == null) {
            synchronized (Setting.class) {
                if( instance == null )
                    instance = new Setting();
            }
        }
        return instance;
    }
}
```
getInstance에 synchronized를 거는 것은 매우 비효율적인 코딩이다. (최초 시도 이후에 Instance만 리턴하면 되는 경우에서도 계속 synchronize로 lock을 걸기 때문이다. )   
따라서 null일 경우만 synchronized 하면 된다. 다만 이 경우는 instance 변수를 volatile 선언 해야 한다.

```java
public class Setting {
    private Setting(){}

    public static class SettingBundle {
        private static final Setting INSTANCE = new Setting();
    }

    public static Setting getInstance() {
        return SettingBundle.INSTANCE;
    }
}
```
해당 방법 같이 변환하거나 enum을 통해 변환하는 것이 가장 안전한 싱글톤 패턴 생성 방법이다.

* 사용 예
```java
package java.lang;
public class Runtime {
    private static final Runtime currentRuntime = new Runtime();

    public static Runtime getRuntime() {
        return currentRuntime;
    }

    private Runtime() {}
```
java.lang.Runtime 클래스는 시스템 정보를 알려주는 클래스여서 싱글톤으로 구현된 걸 확인 할 수 있다.
그 외에도 스프링의 Bean들은 기본 설정이 Singleton 이어서 쉽게 싱글톤 패턴을 확인 할 수 있다.

## 2. Factory method (팩토리 메서드 패턴)
팩토리 메소드 패턴(Factory Method Pattern)은 객체를 생성하기 위해 인터페이스를 만든다.
구체적으로 어떤 클래스의 인스턴스를 만들지를 서브클래스에서 결정하도록 한다. 
팩토리 메소드를 이용하면 인스턴스를 만드는 일을 서브클래스로 미룰 수 있다.

😊Pizza.java
```java
public abstract class Pizza {
    public abstract void order();

    public enum PizzaType {
        HamMushroom, Deluxe, Seafood
    }

    public static Pizza PizzaFactory(PizzaType pizzaType) {
        switch (pizzaType) {
            case HamMushroom:
                return new HamMushroomPizza();
            case Deluxe:
                return new DeluxePizza();
            case Seafood:
                return new SeafoodPizza();
        }
        throw new IllegalStateException("The pizza type is not recognized");
    }
}
```
😊SeafoodPizza.java
```java
public class SeafoodPizza extends Pizza {
    @Override
    public void order() {
        System.out.println("SeafoodPizza order!!!");
    }
}
```
😊HamMushroomPizza.java
```java
public class HamMushroomPizza extends Pizza {
    @Override
    public void order() {
        System.out.println("HamMushroomPizza order!!!");
    }
}
```

[참고: (위키백과)팩토리메서드 패턴](https://ko.wikipedia.org/wiki/%ED%8C%A9%ED%86%A0%EB%A6%AC_%EB%A9%94%EC%84%9C%EB%93%9C_%ED%8C%A8%ED%84%B4)   
위키백과의 팩토리메서드 패턴 예제이다.   
추상클래스인 Pizza의 PizzaFactory 메서드로 Pizza의 Concrete 클래스를 생성하게 하였다. 이렇게 하면 다른 피자가 추가되었을 시 기존 피자에서 enum만 추가하고 Pizza를 상속받는 클래스를 만들기만 하면 된다. (기존의 코드의 수정없이 기능이 추가 가능하다.)

🤚Question
* 팩토리 메소드 패턴을 적용했을 때의 장점과 단점
  - 장점은 기존 코드를 건드리지 않고 새로운 기능을 추가하기 쉽다.
  - Product와 Factory 사이에 느슨한 결합을 통해 OCP원칙을 잘 지킬 수 있다.
  - 단점은 클래스가 늘어난다.
* 확장에는 열려있고 변경에 닫혀있는 객체지향 원칙을 설명하시오 (Open Close Principle)
  - 확장에 열려있고 변경에 닫혀있다는 말은 기존 코드를 변경하지 않고 새로운 기능을 추가할 수 있다는 말이다. 
  - 위의 피자 예처럼 새로운 피자 하위 타입을 추가할 때 기존 코드를 변경하지 않아도 된다.
* 자바 8에 추가된 default 메서드에 대해 설명하세요.
  - java 1.8 부터 인터페이스 접근제한자가 기본 public 이 되었고 default 를 명시하면 인터페이스에서 구현 코드를 작성할 수 있다. 또한 인터페이스에서 static 메서드도 생성이 가능해서 간단한 기능이 있는 유틸리티 클래스로써의 기능도 가능하다.
  - 추가로 java 1.9 부터 인터페이스에 private static 메서드가 가능해짐에 따라 원하는 기능만 public static으로 할 수 있어 더욱 모듈화가 강화되었다.

## Reference
[백기선-코딩으로 학습하는 GoF의 디자인패턴](https://www.inflearn.com/course/%EB%94%94%EC%9E%90%EC%9D%B8-%ED%8C%A8%ED%84%B4/dashboard)     



