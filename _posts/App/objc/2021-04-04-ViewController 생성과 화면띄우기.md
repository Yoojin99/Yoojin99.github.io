---  
layout: post  
title: "ViewController 생성과 화면 띄우기, presentViewController와 pushViewController"  
subtitle: ""  
categories: app
tags: app-objc
comments: true  
header-img: 
---  
  
> `ViewController를 생성해서 화면에 띄워보자. 아래에서 위로, 오른쪽에서 왼쪽으로 나타날 수 있다.`  

---

먼저 내가 현재 실습중인 프로젝트에서 생각하는 동작은 다음과 같다.

![image](https://user-images.githubusercontent.com/41438361/113504110-e4a7a580-9570-11eb-9992-77cd4dc848b2.png)

처음 실행하면 로고와 함께 메인 화면이 나오고, 화면의 아래에서 위로 영화 목록 화면이 노출되며, 이 화면에서 특정 영화를 클릭하면 해당 영화의 상세 영화 페이지가 뜨게 했다.
즉 영화목록 화면은 등장할 때 아래에서 위로, 영화 상세 페이지는 오른쪽에서 왼쪽으로 뜨면서 노출되어야 하는 것이다.





