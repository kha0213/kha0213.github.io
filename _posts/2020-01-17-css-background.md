---
title:  "CSS Background"

categories: 
  - css
tags:
  - css
  - study
last_modified_at: 2020-01-17T14:10:00-00:00
---

## 1.background   
```css
background-(명령어) 가 온다. 명령어를 생략하고 작성   
```
* color:red; 		가장 기본적인 명령어. 배경 색을 지정한다.   
* image:url(); 	배경 이미지 지정한다. ()안에 ''로 파일경로 + 이름   
* repeat: 		반복 설정한다. x축만, y축만 or 반복없이 지정가능.   
* position:right center; no-repeat일때만 가능. 배경 이미지 위치를 지정한다.   
* attachment:fixed; 배경을 고정한다.(default값은 scroll)   
* size:500px 100px; (width와 height)   
* size:cover; 	화면에 이미지를 꽉차게 한다(이미지가 짤릴 수 있음)   
* size:contain; 	화면에 모든 이미지가 들어간다(박스안에 여유공간 생김)   


## 2.background 축약형

위의 메소드를 일일히 하는것이 아니라 한 번에 지정할 수 있다   
   
```css
border:1px solid red;
background: red url('1.png') no-repeat fixed center;   
```

## 3.background만 투명하게 할 때

부모 container에 배경이 있고 그 안의 itmes에 내용이 있다고 할 때 
container에 opacity를 주면 자식인 items까지 투명해진다. 
배경만 투명하게 하고싶다면 다음과 같이 하자   
1. container::after를 만든다. (after은 가상요소이다)   
2. 원본의 이미지를 after로 옮긴다.   
3. after를 position을 absolute로 weith와 height를 100%로 top,left를 0으로 한다.   
   (원본과 동일한 크기와 위치를 갖게 함)   
4. content:"";도 추가한다.(이미지만 가지고 있기 위해)   
5. z-index 설정으로 원본보다 가상요소를 뒤에 놔둔다.(자동으로 원본 자식보다도 뒤로가진다.)   
6. 마지막으로 가상요소에 opacity를 설정한다.   
 
**Note:** [참고] 배경화면만 투명하게   
[https://codingbroker.tistory.com/58](https://codingbroker.tistory.com/58)   
{: .notice--info} 

