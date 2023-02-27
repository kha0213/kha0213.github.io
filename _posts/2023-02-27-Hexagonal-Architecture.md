---
title:  "헥사고날 아키텍쳐 (Hexagonal Architecture)"

toc: true
toc_label: "go page"
toc_icon: "book"
toc_sticky : true

classes: wide

categories: 
  - architecture
tags:
  - hexagonal
last_modified_at: 2023-02-27T00:10:00-00:00
---

# 헥사고날 아키텍처란?
헥사고날 아키텍처는 소프트웨어 설계에 사용되는 아키텍처 패턴 중 하나이며 사전적 의미로는 ”육각형 건축물”을 뜻한다.
이 아키텍처는 구성 요소를 모든 수준에서 쉽게 교환할 수 있게 하고 테스트 자동화를 용이하게 한다. 
포트와 어댑터를 통해 여러 소프트웨어 환경에 쉽게 연결할 수 있도록, 느슨하게 결합된 응용 프로그램 구성요소를 만드는 것을 목표로 한다.
헥사고날 아키텍처에서 사용하는 포트(Port)와 어댑터(Adapter)의 이름을 따서 포트 앤드 어댑터 아키텍처(Ports and adapters architecture - 엘리스테어 콕번)라고도 불린다.

## 헥사고날 아키텍처의 장점 
1. 아키텍처 확장이 용이하다.
2. SOLID 원칙을 쉽게 적용할 수 있다.
3. 모듈 일부를 배포하는 게 용이하다.
4. 테스트를 위해 모듈을 가짜로 바꿀 수 있으므로 테스트가 더 안정적이고 쉽다.
5. 더 큰 비즈니스적 가치를 갖고 더 오래 지속되는 도메인 모델에 큰 관심을 둔다.

## 헥사고날 아키텍쳐 설명
![hexagonal]({{ site.baseurl }}/assets/images/study/architecture/hexagonal.png)   

이름에서 알 수 있듯, 육각형 구조의 코어를 가지고 여러 외부 어댑터들로 이루어져 있다.

- 핵심적으로 보아야 하는 것은 코어에서 외부로 향하는 의존성이 없다는 것이다. 이는 외부 어댑터들이 목적에 맞게 포트와 잘 커뮤니케이션 되도록 구현되었다면 쉽게 교체가 가능하다는 뜻이다.
- 즉, 외부와 도메인 로직의 결합성(의존성)을 제거하여 변경할 이유의 수를 줄일 수 있고, 이는 유지보수성을 높인다.

이 아키텍처에서 비즈니스 로직은 외부 시스템이 애플리케이션과 상호 작용할 수 있도록 하는 인터페이스인 애플리케이션 포트로 둘러싸인 중앙에 있다. 
이러한 포트는 수신(예: REST API, 메시징 큐) 또는 송신(예: 데이터베이스, 이메일 서비스)일 수 있다. 
어댑터는 포트와 애플리케이션 코어 간의 요청과 응답을 변환하는 역할을 한다.
Port에 정의된 인터페이스로만 요청과 응답을 하고 내부 구현을 public이 아니게 감춰져 있기 때문에 변경 사항에 있어서 확인할 범위가 대폭 축소된다.


## GPT가 설명하는 헥사고날
요즘 핫한 GPT로 검색해보니 헥사고날의 정의에 대해 정말 자세하게 설명하고 있다..

he hexagonal architecture, also known as ports and adapters architecture or clean architecture, is a software architecture pattern that aims to create software that is independent of frameworks, databases, or any external dependencies. The architecture is built around the idea of dividing a software system into several layers or "ports," which interact with each other via "adapters."
In this architecture, the business logic is at the center, surrounded by the application ports, which are the interfaces that allow external systems to interact with the application. These ports can be incoming (e.g., REST API, messaging queue) or outgoing (e.g., database, email service). The adapters are responsible for translating the requests and responses between the ports and the application core.
The hexagonal architecture promotes modularity, testability, and maintainability. By keeping the business logic separate from the implementation details, it becomes easier to modify or replace the implementation without affecting the rest of the system. This architecture also encourages the use of dependency injection to manage the dependencies between the different components of the system.
Overall, the hexagonal architecture is a flexible and adaptable architecture that can be applied to a wide range of software systems, especially those that need to be scalable and maintainable over time.



## Reference
[https://mesh.dev/20210910-dev-notes-007-hexagonal-architecture/](https://mesh.dev/20210910-dev-notes-007-hexagonal-architecture/)    
[https://devlos.tistory.com/55](https://devlos.tistory.com/55)    

