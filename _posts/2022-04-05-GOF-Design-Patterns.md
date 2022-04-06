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

## 3. Abstract Factory (추상 팩토리 패턴)
추상 팩토리 패턴은 팩토리 메서드 패턴과 유사하지만, 명확한 차이점이 있다.

* 팩토리 메서드 패턴
조건에 따른 객체 생성을 팩토리 클래스로 위임하여, 팩토르 클래스에서 객체를 생성하는 패턴
* 추상 팩토리 패턴
팩토리 하위의 컴포넌트들을 추상화 시켜서 구체 클래스를 지정하지 않고도 팩토리 클래스가 결정될 때 하위 컴포넌트들의 구체 클래스도 결정되게 하는 패턴

코드로 알아보자.
```java
public interface IButton {
    void paint();
}
```
```java
public interface IGUIFactory {
    IButton createButton();
}
```
```java
public class WinButton implements IButton {
    @Override
    public void paint() {
        System.out.println("WinButton paint!!");
    }
}
```
```java
public class WinFactory implements IGUIFactory {
    @Override
    public IButton createButton() {
        return new WinButton();
    }
}
```
```java
public class Program {
    public static void main(String[] args) {
        // DI로도 가능하다.
        IGUIFactory factory = new WinFactory();

        // 부속품 만드는 메서드는 추상화되어 클라이언트는 구체 클래스를 몰라도 가능하다.
        IButton button = factory.createButton();
        button.paint();
    }
}
```
[참고: (위키백과) 추상 팩토리 패턴](https://ko.wikipedia.org/wiki/%EC%B6%94%EC%83%81_%ED%8C%A9%ED%86%A0%EB%A6%AC_%ED%8C%A8%ED%84%B4)   
추상 팩토리 패턴 예제이다. IGUIFactory라는 인터페이스 팩토리에 해당하는 구체 클래스를 new로 생성하였지만 이후에 DI로 이것을 주입받으면 구체 클래스를 클라이언트 코드에서 작성하지 않아도 된다.   
이후 다른 OSXFactory나 OSXButton 등이 나오더라도 다 추상화 된 IButton을 구현체, IGUIFactory의 구현체 이므로 클라이언트 코드 변경 없이 Factory와 Button을 쉽게 변경 할 수 있다.

## 4. Builder Pattern (빌더 패턴)
빌더 패턴이란 복합 객체의 생성 과정과 표현 방법을 분리하여 동일한 생성 절차에서 서로 다른 표현 결과를 만들 수 있게 하는 패턴이다.   
코드로 확인해보자.

```java
@Data
public class User {
    private int id;
    private String name;
    private int height;
    private int weight;
}
```
해당 User 객체를 생성할 때 setter를 사용하게 된다면 불변객체를 만들 수 없다. 
따라서 생성자에서 객체를 만들어야 하는데 키 또는 몸무게 값 중 하나만 받는 상황이라면 생성자도 여의치 않다.
ex)
```java
public User(int id, String name, int height) {
    this.id = id;
    this.name = name;
    this.height = height;
}

// Java 에서 파라미터 타입이 완전히 같으면 생성자를 2개 만들 수 없다. (변수 명 상관없이)
//public User(int id, String name, int weight) {
//    this.id = id;
//    this.name = name;
//    this.weight = weight;
}
```
위와 같은 상황이라면 키 또는 몸무게 중 하나만 받는 생성자를 2개 만들 수 없다. 이와 같은 경우 빌더 패턴을 사용하면 의미도 명확해진다.
```java
public interface UserBuilder {
    UserBuilder id(int id);
    UserBuilder name(String name);
    UserBuilder height(int height);
    UserBuilder weight(int weight);
    User build();
}
```
```java
public class DefaultUserBuilder implements UserBuilder {
    private User user = new User();

    @Override
    public UserBuilder id(int id) {
        user.setId(id);
        return this;
    }

    @Override
    public UserBuilder name(String name) {
        user.setName(name);
        return this;
    }

    @Override
    public UserBuilder height(int height) {
        user.setHeight(height);
        return this;
    }

    @Override
    public UserBuilder weight(int weight) {
        user.setWeight(weight);
        return this;
    }

    @Override
    public User build() {
        return user;
    }
}
```
```java
public class Main {
    public static void main(String[] args) {
        UserBuilder builder = new DefaultUserBuilder();
        User 롱롱이 = builder.id(1)
                .height(160)
                .weight(70)
                .name("롱롱이")
                .build();
        System.out.println("롱롱이 = " + 롱롱이);
    }
}
```
해당 방식으로 Builder을 만들게 되면 원하는 필드만 입력받는 것 뿐 아니라 build 할 때 객체가 생성이 되어 안전하다. 그리고 키나 몸무게에 유효성 체크를 하여 Exception 처리를 할 수도 있다. 

* 빌더 패턴의 장점
  1. 불완전한 객체 생성 방지
  2. 객체 유효성 체크
  3. 코드의 명확성 증가
* 빌더 패턴의 단점
  1. 코드량 증가 및 클래스 증가

😊 How to Use : 생성자로 의미가 전달이 불분명하거나 불완전한 객체가 사용될 우려가 있는 곳에 빌더 패턴을 적용하자.

## Prototype Patterns (프로토 타입 패턴)
프로토타입 패턴은 생성할 객체들의 타입이 프로토타입인 인스턴스로부터 결정되도록 하며, 
인스턴스는 새 객체를 만들기 위해 자신을 복제(clone)하게 된다.

😊GithubRepository.java
```java
@Data
public class GithubRepository {

    private String user;

    private String name;
}
```

😊GithubIssue.java  
```java
@Data
@EqualsAndHashCode
public class GithubIssue implements Cloneable {

    private int id;

    private String title;

    private GithubRepository repository;

    public GithubIssue(GithubRepository repository) {
        this.repository = repository;
    }

    public String getUrl() {
        return String.format("https://github.com/%s/%s/issues/%d",
                repository.getUser(),
                repository.getName(),
                this.getId());
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```
여기서 GithubIssue 객체가 프로토타입으로 clone을 사용하게 되면 얕은 복사 Shallow Copy 하게 된다.    
GithubIssue 인스턴스는 다른 인스턴스가 생성되고 id, title 등 value 타입은 새로 생성되나 reference 타입은 주소값이 같은 주소값을 가르키기 때문에 오류가 생길 수 있다. 따라서 내부 값들을 불변 객체로 채우거나 깊은 복사로 copy 메서드를 오버라이드 해야한다.
     
다음은 실제로 Java에서 사용되는 프로토타입 패턴의 예를 알아보자. 
😊 ArrayList.java
```java
public ArrayList(Collection<? extends E> c) {
    Object[] a = c.toArray();
    if ((size = a.length) != 0) {
        if (c.getClass() == ArrayList.class) {
            elementData = a;
        } else {
            elementData = Arrays.copyOf(a, size, Object[].class);
        }
    } else {
        // replace with empty array.
        elementData = EMPTY_ELEMENTDATA;
    }
}

public Object clone() {
    try {
        ArrayList<?> v = (ArrayList<?>) super.clone();
        v.elementData = Arrays.copyOf(elementData, size);
        v.modCount = 0;
        return v;
    } catch (CloneNotSupportedException e) {
        // this shouldn't happen, since we are Cloneable
        throw new InternalError(e);
    }
}
```
Java의 ArrayList 클래스는 생성자에 Collection 상속 개체를 받거나 clone 메서드를 호출하면 리스트를 복제하여 새로운 리스트에 담아 리턴한다.   


* 프로토 타입 패턴의 장점
  - 객체를 생성할 때 고유의 비용이 주어진 프로그램에서 이 비용없이 새로운 객체를 만들 수 있게 한다.

😊 How to Use : 객체를 생성하는 데 비용이 많이 들고, 비슷한 객체가 있는 경우 해당 패턴을 사용하자.
## Reference
[백기선-코딩으로 학습하는 GoF의 디자인패턴](https://www.inflearn.com/course/%EB%94%94%EC%9E%90%EC%9D%B8-%ED%8C%A8%ED%84%B4/dashboard)     



