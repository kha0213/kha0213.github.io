---
title:  "CSS Filter, Transform"

categories: 
  - css
tags:
  - css
  - study
last_modified_at: 2020-01-18T00:10:00-00:00
---

## 0.wepkit,moz,ms,o
최신기술의 경우 모든 웹 브라우저에서 동작하지 않는 경우가 있다.   
브라우저의 호환성을 위해 접두사를 밴더 프리픽스라고 하며   
-webkit- : 구글, 사파리 브라우저에 적용   
-moz- : 파이어폭스 브라우저에 적용   
-ms- : 익스플로러에 적용, 보통 생략합니다.   
-o- : 오페라 브라우저에 적용   

## 1.filter
**Note:** filter : 필터효과를 적용한다   
{: .notice--info}   
아래에 여러가지 명령어가 있다.    
* blur(px) : 이미지를 흐리게 한다   
* brightness(%) : 이미지의 밝기를 변경한다   
* contrast(%) : 이미지의 대비를 변경한다   
* drop-shadow(x-offset y-offset blur color) : 이미지에 그림자를 추가한다   
* grayscale(%) : 이미지에  흑백효과를 적용한다   
* hue-rotate(deg) : 이미지에 색상환의 각도를 지정   
* invert(%) : 이미지 반전   
* opacity(%) : 이미지에 투명도 적용   

## 2.transform
10번 설명을 읽는 것보다 1번 실행화면을 보는 것이 빠를 때도 있다.   
아래 예제를 보자.   

**Note:** [참고] transform예제   
[https://www.opentutorials.org/course/2418/13684](https://www.opentutorials.org/course/2418/13684)   
{: .notice--info} 


포토샵으로 변경하는것 같은 변화를 줄 때 많이 이용한다.   
띄어쓰기로 여러개의 transform을 한 번에 이용 할 수 있다.   
이 이외에도 여러가지 명령어가 있다.   

* translate(x,y)	요소의 위치를 X축으로 x만큼, Y축으로 y만큼 이동시킨다.	  
* scale(x,y)	요소의 크기를 X축으로 x배, Y축으로 y배 확대 또는 축소 시킨다.   
* skew(x-angle,y-angle)	요소를 X축으로 x 각도만큼, Y축으로 y 각도만큼 기울인다.	  
* rotate(angle)	요소를 angle만큼 회전시킨다.   

