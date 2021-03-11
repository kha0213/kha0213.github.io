---
title:  "CSS Media Query"

categories: 
  - css
tags:
  - css
  - study
last_modified_at: 2020-01-21T00:10:00-00:00
---

# 1. what is media_query

출력 장치의 여러 특징 가운데 일부를 참조하여 CSS 코드를 분기 처리함으로써 하나의 HTML 소스가 여러가지 뷰를 갖도록 구현할 수 있는 명세이다. 일반적으로 뷰포트 해상도에 따라 CSS 코드를 분기한다.
ex)출력 디바이스의 width에 따라 구현하는 화면을 달리할 수 있다. 즉 하나의 웹사이트를 
웹용, 모바일용, 태블릿용으로 쉽게 구분하여 사용 할 수 있다.

## 2. How to use media_query

### (1).CSS 내부에서 분기하는 방법

CSS내부에서 @media와 같은 명령어로 실행한다.   

**중요:** 꼭 style시트 맨 위에 있어야 한다.!!!!
{: .notice--warning}   

```css
@media only all and (조건문) {실행문}   
```

### (2).CSS 외부에서 분기하는 방법

별로 권장하지 않는 방법이다. link를 따로 두어 media에 원하는 대상을 집어넣는다.   
```css
link rel="stylesheet" 
type="text/css" 
media="all and (조건A)" 
href="A.css"
```
보통 CSS는 하나의 파일로 병합하고 내부에서 조건분기하는 방식이 좋다.   

## 3. 미디어 유형과 특성

### (1) 미디어 유형

* only (default값) 일반적으로 작성하지 않는다.
* all 은 모든 장치에 사용
* print 인쇄 결과물 및 출력 미리보기 화면에 표시중인 문서에 적용
* screen 화면 대상
* speech 음성 합성장치 대상

### (2) 미디어 특성

* 미디어 특성 표현식은 괄호로 감싸져야하고 이후에 나오는 논리연산자도 사용 가능하다.   

| 속성 							| 설명 																|
|:------------------------------|:------------------------------------------------------------------|
|width							| 뷰포트 너비. (보통 너비로 모바일인지 웹인지 태블릿화면인지 나뉜다)	                |
|height							| 뷰포트 높이.	                                                        |
|aspect-ratio					| 뷰포트 가로세로비.	                                                    |
|orientation					| 뷰포트 방향.	                                                        |
|resolution						| 출력 장치의 해상도.	                                                |
|scan							| 출력 장치의 스캔 방법.	                                                |
|grid							| 격자와 비트맵 중 출력 장치가 사용하는 화면.	                                |
|update							| 출력 장치가 화면을 업데이트하는 주기.                                       |
|overflow-block					| 블록 축의 내용이 넘쳤을 때 출력 장치가 처리하는 방법.                          |
|overflow-inline				| 인라인 축의 내용이 넘쳤을 때 스크롤의 가능 여부.	                            |
|color							| 출력 장치의 색상 채널별 비트 수, 흑백일 땐 0.	                            |
|color-gamut					| 사용자 에이전트와 출력 장치가 지원하는 색상의 대략적인 범위.                      |
|color-index					| 출력 장치의 색상 룩업 테이블(LUT) 항목 수, 존재하지 않을 땐 0.	                |
|display-mode					| 웹 앱 매니페스트의 display 항목이 정의한 애플리케이션의 표시 모드.               |
|monochrome						| 출력 장치의, 모노크롬 프레임 버퍼의 픽셀 당 비트 수. 단색을 사용하지 않으면 0.	    |
|inverted-colors				| 사용자 에이전트나 운영체제가 색을 반전 중인지 여부.                            |
|pointer						| 포인팅 장치가 주요 입력 수단인지 여부와, 그렇다면 그 정확도.                     |
|hover							| 주요 입력 수단으로 요소 위에 호버할 수 있는지 여부.                            |
|any-pointer					| 입력 수단 중 포인팅 장치의 존재 여부와, 존재한다면 그 정확도.                    |
|any-hover						| 어떤 입력 수단으로 요소 위에 호버할 수 있는지 여부.                            |
|light-level					| 환경의 광도.                                                         |
|prefers-reduced-motion			| 사용자가 줄어든 움직임을 선호함.                                          |
|prefers-reduced-transparency	| 사용자가 줄어든 투명도를 선호함.                                          |
|prefers-contrast				| 사용자가 시스템에 고대비/저대비를 요청했는지 여부.                             |
|prefers-color-scheme			| 사용자가 밝은 테마나 어두운 테마를 선호하는지 여부.                            |
|scripting						| 스크립트(JavaScript 등)를 사용할 수 있는지 여부.	                        |

## 4.논리연산자

media_query에 논리연산자를 조합하여 복잡한 쿼리를 만들 수 있다.   
여러 media_query는 쉼표로 구분하여 하나의 규칙으로 만든다.   
다른 언어의 논리연산자와 작동방식이 같아 이해가 쉽다.   
1. and : 둘 다 참이어야 실행된다.
2. not : 거짓인 경우 not으로 참이되어 실행된다.
3. only : default값으로 보통 사용하지 않는다.
4. ,(쉼표) : or와 같은 동작을 한다. 하나면 참이어도 미디어쿼리는 동작한다.

**Note:** 참고 사이트
[https://developer.mozilla.org/ko/docs/Web/Guide/CSS/Media_queries](https://developer.mozilla.org/ko/docs/Web/Guide/CSS/Media_queries)   
[https://offbyone.tistory.com/121](https://offbyone.tistory.com/121)   
[https://naradesign.github.io/article/media-query.html](https://naradesign.github.io/article/media-query.html)   
{: .notice--info}   
