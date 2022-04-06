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
해당 방법 같이 변환하거나 enum을 통해 변환하는 것이 가장 안전한 싱글톤 패턴 생성 방법입니다.

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
구체적으로 어떤 인스턴스를 만들지는 서브 클래스가 정한다.


## Reference
[백기선-코딩으로 학습하는 GoF의 디자인패턴](https://www.inflearn.com/course/%EB%94%94%EC%9E%90%EC%9D%B8-%ED%8C%A8%ED%84%B4/dashboard)     



