---
title:  "CSS Selector"

categories: 
  - css
tags:
  - css
  - study
last_modified_at: 2020-01-15T00:10:00-00:00
---

# What is CSS
* CSS stands for Cascading Style Sheets
* CSS describes how HTML elements are to be displayed on screen, paper, or in other media
* CSS saves a lot of work. It can control the layout of multiple web pages all at once
* External stylesheets are stored in CSS files

## CSS Location
일반적으로 head 태그 안에 들어간다.
```html
<head>
    <meta charset="utf-8" />
    <title></title>
    <style>

        [이 부분이 CSS들어간다]

    </style>
</head>
<body>

		[이부분에 HTML내용이 들어간다]

</body>
```
## 0. CSS_cheat_sheet
![css_cheat_sheet](https://user-images.githubusercontent.com/56239469/72400530-e3803200-378c-11ea-930e-5b26bda37ccd.jpg)   
   
## 1.Selector_Declaration(선택자 정의)   
선택자란 html안의 원하는 요소을 선택하는 css의 문법이다.   
원하는 요소를 선택 후 속성을 변경 할 수 있다.
```html
EX) li{ color : red ; text-decoration:underline; }   
```
1. color,text-decoration : 속성
2. : 속성과 값의 연결연산자
3. red,underline : 속성의 value
4. ; 메소드 끝


## 2.ID selector(id 선택자)   
body에 여러개의 li가 있을 때 특정한 하나에 디자인 하고 싶을 때 id선언을 하자   
구현이란 li를 속성변환하고싶으면 아래와 같이 하면 된다.    

```html 
<li id="dodo">구현</li> body에 이렇게 하고   
style(CSS) 부분에 #dodo{color:red;} 하면 된다.   
```
   
**Note:** ID는 html안에서 하나만 주는 것이 올바른 방법이다. 
{: .notice--info}  

## 3.parent_child selector(부모 자식 선택자)

ul li{border:1px solid red;} 이러면 ul 밑의 ol만 적용되고   
ul>li{} 하면 ul바로 밑에있는 ol만 적용된다.   
ul,ol{}이러면 둘 다 적용된다.   

## 4.attribute selector(속성 선택자)

[안에 조건을 집어 넣는다.]   
=  : 속성안의 값이 특정 값을 단어로 포함		
^= : 속성안의 값이 특정 값으로 시작		
$= : 속성안의 값이 특정값으로 끝나는 경우		

ex) input[type='tel'] input 중 type이 tel		
ex) input[type^='te'] input 중 type이 te로 시작	
ex) input[type^='l'] input 중 type이 l로 끝	

## 5.pseudo class selector(가상 클래스 선택자)
사용법 뒤에 : 을 입력하고 메소드 입력한다.   
ex) table:hover{ }   
* link : 방문한 적이 없는 링크   
* visited : 방문한 적이 있는 링크   
* hover : 마우스를 롤오버 했을 때   
* active : 마우스를 클릭했을 때   
* focus : 탭 키를 눌러보면 focusing 된 상태 focus는 스타일 끝에 쓰자   

## 6. sibling selector(동위 선택자)

+,~ 두 개가 있다.   
h3 ~ div {   } 하면 h3과 같은 위치에 있는 div를 모두 말한다. (같은 위치라도 h3 이후에 있는 div만 적용)   
h3 + div {   } 하면 h3과 같은 위치에 있는 div중 바로 다음 한 개만 말한다.   

## 7. inheritance
[https://www.w3.org/TR/CSS21/propidx.html](https://www.w3.org/TR/CSS21/propidx.html)<br>
(상속 유무 사이트 정리)   

**Note:** 그냥 상속 되는지 안되는지 외우지말고 찾아보고, 생각해보자
{: .notice--info}

## 8. Cascading
[CSS]이다.   
Cascading은 우선순위에 대한 개념이다.   
같은 선택자 안에 여러 속성을 집어 넣어도 디테일한 것이 우선순위가 높다.
1. style attribute   
2. id selector   
3. class selector   
4. tag selector   
순으로 영향을 받는다. 생각해보자
더 포괄적인 개념이 우선순위가 낮고. 디테일한 것이 우선순위가 높다.
(적용이 적은 쪽이 우선순위가 높다)

★★우선순위를 무시하고 적용하고 싶을 때 !important 붙이면 무조건 우선순위가 제일이 된다.
