---
title:  "Java Optimizing"

toc: true
toc_label: "go page"
toc_icon: "book"
toc_sticky : true

classes: wide

categories: 
  - java
tags:
  - java
last_modified_at: 2025-01-13T00:08:00-00:00
---

# Java 성능 최적화 기법

Java 애플리케이션은 코드 작성, JVM 설정, 데이터 구조 선택 및 시스템 설계 전반에 걸친 다양한 기법을 통해 성능 최적화가 가능하다. 


---

## **1. 코드 수준 최적화**

### **1.1 알고리즘 및 데이터 구조 최적화**

- 적절한 알고리즘과 데이터 구조를 선택해야 한다.
    - 예: 빠른 조회를 위해 `HashMap`, 순차적인 데이터 접근을 위해 `ArrayList`, 잦은 삽입/삭제를 위해 `LinkedList`를 사용.

#### **코드 예시**:
```java
import java.util.HashMap;

public class Example {
    public static void main(String[] args) {
        HashMap<String, Integer> map = new HashMap<>();
        map.put("apple", 1);
        map.put("banana", 2);

        if (map.containsKey("apple")) {
            System.out.println("Apple exists with value: " + map.get("apple"));
        }
    }
}
```

- 루프를 최적화해야 한다.
    - 중첩된 루프를 가능한 피하고, 간결하고 효율적인 루프를 작성해야 한다.

#### **코드 예시**:
```java
for (int i = 0; i < items.size(); i++) { 
    processItem(items.get(i)); 
}

// 향상된 for 루프 사용 (가독성과 효율성 향상):
for (Item item : items) { 
    processItem(item); 
}
```

### **1.2 객체 생성 최소화**

- 불필요한 객체 생성을 피하고, 재사용 가능한 객체를 활용해야 한다.

#### **코드 예시**:
```java
// 비효율적인 방식: 매번 새로운 String 객체 생성
String str = new String("Hello");

// 효율적인 방식: String 풀(pool) 사용
String str = "Hello";

// 루프 내 문자열 연결 시 StringBuilder 사용
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 10; i++) {
    sb.append(i);
}
String result = sb.toString();
```

### **1.3 컬렉션 최적화**
요구사항에 맞는 적절한 컬렉션을 선택해야 한다.
1. List : List는 순서가 있는 데이터 구조이며, 중복을 허용한다. 각 요소는 인덱스를 통해 접근할 수 있다.
    1. **ArrayList** : 읽기와 순차적 삽입이 빈번한 경우에 유용하다.
        * 내부적으로 배열을 사용한다.
        * 동적 크기 조정을 지원한다.
    2. LinkedList : 순차적이 아닌 삽입/삭제 작업이 다수 필요한 경우. 큐(queue)나 스택(stack)처럼 작동하는 데이터 구조.
        * 내부적으로 이중 연결 리스트로 구현된다.
    3. CopyOnWriteArrayList : 읽기가 빈번하고, 쓰기가 드문 경우 사용한다.
        * 멀티스레드 환경에서 안전한 ArrayList이다.
    4. Stack : LIFO 구조가 필요한 경우. (Vector는 성능상 사용하지 않는게 좋다.)
2. Map : key-value pair 형태로 저장하기 위한 인터페이스이다. 키는 고유하고, 하나의 키에 하나의 값이 매핑된다.
    1. **HashMap** : 빠른 검색, 삽입, 삭제가 필요한 경우. 
        * 키와 값을 저장하기 위해 해시 테이블을 사용한다.
        * null 키와 null 값을 허용합니다.
        * TreeMap
    2. LinkedHashMap : 데이터의 순서를 유지하면서 해시 기반 성능이 필요한 경우.
        * HashMap과 유사하지만, 입력된 순서를 유지한다.
    3. TreeMap : 키의 정렬 순서가 필요한 경우.
        * 키를 정렬된 순서로 저장한다.
        * 내부적으로 Red-Black Tree 구조를 사용한다.
    4. ConcurrentHashMap :
        * 멀티스레드 환경에서 안전한 HashMap이다. 
3. Set : Set은 고유한 값만 저장할 수 있는 데이터 구조다. 데이터의 중복을 허용하지 않는다.
    1. HashSet : 순서에 상관없이 중복 없는 데이터가 필요할 때
        * 내부적으로 HashMap을 사용하여 구현됨.
        * 저장된 요소의 순서를 보장하지 않음.
    2. LinkedHashSet : 중복을 방지하면서 데이터의 입력 순서를 유지하고 싶을 때
    3. TreeSet : 중복을 방지하면서 정렬된 데이터를 유지하고 싶을 때
        * 데이터가 정렬된 상태로 저장됨. 
        * 내부적으로 TreeMap을 사용하여 구현됨. 
    4. EnumSet : 특정 열거형(enum)의 집합을 저장하고 싶을 때 (enum을 저장할 때 성능적으로 가장 빠름)


#### **코드 예시**:
```java
// 비효율적: 초기 용량 미설정
ArrayList<Integer> list = new ArrayList<>();

// 효율적: 초기 용량 설정으로 리사이징 비용 감소
ArrayList<Integer> list = new ArrayList<>(100);
```

### **1.4 불필요한 동기화 제거**

- 동기화는 필요한 최소한의 코드 블록에서만 사용해야 한다.
- 동시성 제어가 필요한 경우 `ConcurrentHashMap`과 같은 효율적인 동시성 컬렉션을 사용해야 한다.

#### **코드 예시**:
```java
// 비효율적: 메서드 전체 동기화
public synchronized void addItem(String item) {
    list.add(item);
}

// 효율적: 필요한 부분만 동기화
public void addItem(String item) {
    synchronized (list) {
        list.add(item);
    }
}
```

---

## **2. 메모리 관리 최적화**

### **2.1 Garbage Collection 튜닝**

- JVM 힙 크기를 적절히 설정해야 합니다.
    - `-Xms` 옵션으로 초기 힙 크기를 설정하고, `-Xmx` 옵션으로 최대 힙 크기를 설정.

#### **JVM 옵션 예시**:
```bash
java -Xms512m -Xmx1024m -XX:+UseG1GC -jar MyApp.jar
```

### **2.2 메모리 누수 방지**

- 불필요한 참조를 유지하지 않아야 하며, 필요 시 약한 참조(`WeakReference`)를 사용해야 한다.

#### **코드 예시**:
```java
import java.lang.ref.WeakReference;

public class WeakReferenceExample {
    public static void main(String[] args) {
        String strong = new String("StrongReference");
        WeakReference<String> weak = new WeakReference<>(strong);

        strong = null;
        System.gc();

        if (weak.get() == null) {
            System.out.println("The object has been garbage collected.");
        }
    }
}
```

---

## **3. CPU 성능 최적화**

### **3.1 병렬 처리**

- 멀티코어 프로세서를 활용하기 위해 `ForkJoinPool`이나 `ParallelStream`을 사용할 수 있다.

#### **코드 예시**:
```java
import java.util.stream.IntStream;

public class ParallelStreamExample {
    public static void main(String[] args) {
        IntStream.range(1, 1000)
                 .parallel()
                 .forEach(i -> System.out.println(Thread.currentThread().getName() + " - " + i));
    }
}
```

### **3.2 스레드 풀 사용**

- 직접 스레드를 생성하지 말고, `ExecutorService`를 사용하여 스레드 풀을 관리해야 힌다.

#### **코드 예시**:
```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ThreadPoolExample {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(5);

        for (int i = 0; i < 10; i++) {
            executor.submit(() -> {
                System.out.println("Thread: " + Thread.currentThread().getName());
            });
        }

        executor.shutdown();
    }
}
```

---

## **4. I/O 최적화**

### **4.1 버퍼링된 I/O 사용**

- 파일 I/O 성능을 높이기 위해 `BufferedReader`와 `BufferedWriter`와 같은 버퍼링된 스트림을 사용해야 한다.

#### **코드 예시**:
```java
import java.io.*;

public class BufferedIOExample {
    public static void main(String[] args) throws IOException {
        try (BufferedReader reader = new BufferedReader(new FileReader("input.txt"));
             BufferedWriter writer = new BufferedWriter(new FileWriter("output.txt"))) {

            String line;
            while ((line = reader.readLine()) != null) {
                writer.write(line);
                writer.newLine();
            }
        }
    }
}
```

---

## **5. 데이터베이스 최적화**

### **5.1 쿼리 최적화**

- 필요 없는 데이터를 조회하지 않도록 해야 한다.
- 인덱스를 적절히 활용해야 한다.

#### **SQL 예시**:
```sql
-- 비효율적인 쿼리
SELECT * FROM users WHERE LOWER(username) = 'john';

-- 효율적인 쿼리 (인덱스를 사용할 수 있음)
SELECT id, username FROM users WHERE username = 'john';
```

---

## **6. 캐싱**

### **6.1 애플리케이션 캐싱**

- `Ehcache`, `Caffeine`과 같은 인메모리 캐시를 사용하여 성능을 개선할 수 있다.

#### **코드 예시**:
```java
import com.github.benmanes.caffeine.cache.Cache;
import com.github.benmanes.caffeine.cache.Caffeine;

public class CaffeineExample {
    public static void main(String[] args) {
        Cache<String, String> cache = Caffeine.newBuilder()
                                               .maximumSize(100)
                                               .build();

        cache.put("key", "value");
        System.out.println(cache.getIfPresent("key"));
    }
}
```

---

## **7. 프로파일링 및 모니터링**

### **7.1 프로파일링 도구**

- VisualVM, JProfiler, YourKit과 같은 도구를 사용하여 병목 구간을 분석할 수 있다.

