---
title:  "React - LifeCycle API"
excerpt: ""

toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
header:
  teaser: 
  
  
categories:
  - React
  
tags:
  - 공부
  - React
  
last_modified_at: 
---

## LifeCycle API

생명주기.

Component가

1. 나타날 때
2. 업데이트 될 때
3. 사라질 때

중간중간에서 사용하고 싶을 때 사용한다.

종류가 많다.

constructor -> render -> 등등,,, 하나하나가 다 함수이다.

## Mounting

브라우저 상에 나타날 때.

constructor : 만들어질 때 가장 처음 됨. state

getDerivedStateFromProps

render 

componentDidMount : 외부 라이브러리에서 특정 DOM에 그려달라, API 요청같은거 할 때? 우리가 만든 컴포넌트가 브라우저에 나타난 시점에 어떤걸 하냐.

## Updating

getDerivedStateFromProps

shouldComponentUpdate : 렌더링 작업이 불필요해질 때(업데이트 된 것만 바뀌어야 한다.) Virtual DOM에 그리는 것도 아끼기 위해. true나 false 값 반환.true이면 렌더링 하고 false이면 렌더링 안한다.

render

getSnapshotBeforeUpdate : 브라우저에 반영되기 직전에 호출되는 거.

componentDidUpdate : 작업을 마치고 업데이트 되었을 떄 호출. 이전의 상태와 지금의 상태가 바뀌었을 때 사용 등등

## Unmounting 

componentWillUnmount : 설정한 리스너 없애기

브라우저 상에서 사라질 때.
