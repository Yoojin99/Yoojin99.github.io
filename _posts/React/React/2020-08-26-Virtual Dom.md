---
title:  "React - Virtual Dom"
excerpt: ""

toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
header:
  teaser: 
  
  
categories:
  - react
  
tags:
  - react-react
  - 공부
  
last_modified_at: 
---

MVC, MVVM 의 공통점에는 Model이 있다. 양방향 바인딩을 한다. 한쪽에서 변경되면 다른 쪽에서 변경되는 거.

데이터가 바뀌면 기존의 뷰를 날려버리고 새로 만들어버리면 어떨까라는 생각으로 만들었다. 하지만 성능의 문제가 있는데,

Virtual DOM이 여기서 나왔다. 정말 변화가 필요한 곳에만 업데이트 하는 것이다. 변화가 생긴 부분을 가상의 DOM에 그린 후
바뀐 부분을 알아내고 그 특정 부분에만 패치를 하는 것이다.
