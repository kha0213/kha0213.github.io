---
title: "What is Thymeleaf"

toc: true
toc_label: "go page"
toc_icon: "book"
toc_sticky : true

classes: wide

categories:
  - html
tags:
  - html
  - thymeleaf
last_modified_at: 2021-10-18T00:40:00-00:00
---

# what is Thymeleaf
Thymeleaf는 현대의 server-side Java 템플릿 엔진이다. 쉽게 말해 Java 진영의 서버에서 html을 동적으로 만드는 작업을 쉽게 한다.    
natural 템플릿 엔진 특성을 가지고 있어. 서버가 작동하지 않을 때에도 일반 html을 볼 수 있다. (협업이 용이하다.)
특히 Spring framework와 잘 호환되게 만들어져있고 편의성 있는 기능이 많이 들어가 있다.   
[공식문서 : https://www.thymeleaf.org/](https://www.thymeleaf.org/)   

## Thymeleaf Basic Grammer
타임리프를 사용하려면 최상단 html에 다음과 같이 작성해야 한다.
```markdown
<html xmlns:th="http://www.thymeleaf.org">
```   

간단한 표현:
* 변수 표현식: ${...}
* 선택 변수 표현식: *{...}
* 메시지 표현식: #{...}
* 링크 URL 표현식: @{...}
* 조각 표현식: ~{...}
* 리터럴
* 텍스트: 'one text', 'Another one!',…
* 숫자: 0, 34, 3.0, 12.3,…
* 불린: true, false
* 널: null
* 리터럴 토큰: one, sometext, main,…
* 문자 연산:
* 문자 합치기: +
* 리터럴 대체: |The name is ${name}|
* 산술 연산:
* Binary operators: +, -, *, /, %
* Minus sign (unary operator): -
* 불린 연산:
* Binary operators: and, or
* Boolean negation (unary operator): !, not
* 비교와 동등:
* 비교: >, <, >=, <= (gt, lt, ge, le)
* 동등 연산: ==, != (eq, ne)
* 조건 연산:
* If-then: (if) ? (then)
* If-then-else: (if) ? (then) : (else)
* Default: (value) ?: (defaultvalue)
* 특별한 토큰:
* No-Operation: 

### 1. text, utext
