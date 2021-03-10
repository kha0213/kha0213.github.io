---
title:  "CSS_02.layout"

categories: 
  - study
tags:
  - css
  - study
last_modified_at: 2020-01-16T00:10:00-00:00
---

## 0.font   

font-size : 글자 사이즈 조절, px % 단위 rem단위는 사용자가 폰트크기 지정하는 것에 따라감.    
font-weight : 글자 두께   
line-height : 줄 높이   
text-align : 정렬, right, left, center로 정렬함, justify는 글 좌우 꽉차게 정렬(자간이 자동조절됨)   
   
**Note:** 폰트를 다운받지 않고 웹에서 구글 폰트에서 가져와서 쓸 수 있다.   
{: .notice--info} 
참조 : [https://www.codingfactory.net/10561](https://www.codingfactory.net/10561)

## 1.inline, block
display 에는 크게 세 가지 속성이 있다.   
* inline : default 값으로 해당부분이 차지하는 만큼만 박스로 쳐져 있다. 자동 줄바꿈 되지 않는다.   
* block : 해당부분 줄 전체에 박스가 쳐져있다. 자동 줄바꿈 된다. ex)h1, li   
* none : 박스가 생성되지 않는다. 따라서 공간을 차지하지 않는다.   

(display:none; 와 visibility:hidden; 은 서로 다르다    
none은 아예 박스가 없어져서 내용과 공간이 다 없어지고   
hidden은 내용이 보이지 않을 뿐이라 해당 공간은 남아있다)   
따라서 li를 왼쪽으로 길게 쓰려면 inline으로 바꾸어주거나 float:left를 주면 된다.   
</pre>

## 2.box model
화면구현에 각각의 요소에 box가 있다고 생각하자.   
border로 선을 줘서 박스 위치를 정하여 깔끔하게 보이도록하자.   
박스 앞뒤로 여백을 주는 명령어는 2개가 있다.   
padding : 박스 내부의 여백   
margin : 박스 외부의 여백   
일반적으로 width나 height는 margin과 border의 선을 제외하고 하여 계산이 어렵다.   

**Note:** Style에 box-sizing:border-box; 를 전체에다 주면 사이즈 계산이 쉽다.  
{: .notice--info} 

## 3.margin-collapsing
마진상쇄 라고 한다.   
top, bottom margin에서 일어나는데 크게 3가지 요소가 있다.   
1. 인접 형제 박스 간 상하 마진이 겹칠 때 : 큰 쪽의 마진값을 가진다.   
2. 빈 요소의 상하 마진이 겹칠 때   
   (빈 요소란 높이가 0인 블록 요소이다. 주로 height값을 정하지 않거나 inline에서 내부 값이 없을 때 발생한다.)   
3. 부모 박스와 첫 번째 자식 박스의 상단(or하단) 마진이 나란히 겹칠 때   
<br>
   
좋은 글이 있어 링크를 첨부한다.      
[마진 상쇄 설명 참조](https://velog.io/@raram2/CSS-%EB%A7%88%EC%A7%84-%EC%83%81%EC%87%84Margin-collapsing-%EC%9B%90%EB%A6%AC-%EC%99%84%EB%B2%BD-%EC%9D%B4%ED%95%B4)