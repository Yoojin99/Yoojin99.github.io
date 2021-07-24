---  
layout: post  
title: "[iOS] - Custom UIViewController Transition, 커스텀 뷰컨트롤러 이동 구현하기 튜토리얼"  
subtitle: ""  
categories: app
tags: app-ios
comments: true  
header-img: 

---  
  
> `커스텀으로 UIViewController가 전환되도록 구현해보자.`  

---

[이 글](https://yoojin99.github.io/app/Custom-UIViewController-Transition/)에서는 custom uiviewcontroller transition을 구현한
설명이 잘 되어 있는 예제 두 개를 정리했다. 

여기에서는 아래와 같은 custom transition을 구현해보려고 한다. 

전체 프로젝트 코드는 [여기](https://github.com/Yoojin99/iOS-UIViewController-Custom-Transition/tree/main/Complete)에서 확인할 수 있다.

## 0. 시작

[Github](https://github.com/Yoojin99/iOS-UIViewController-Custom-Transition) 에 시작접이 되는 프로젝트를 올려놓았다.

`Complete` 폴더는 완성된 프로젝트의 코드이고, 그 외를 제외한 나머지가 starter 프로젝트에 포함된다고 생각하면 된다.

ViewController는 총 두 개로, 기본이 `ViewController`, 애니메이션을 수행하면서 띄울 뷰 컨트롤러가 `PopupViewController`다.

프로젝트를 실행시키면 "Go" 버튼이 있는 `ViewController`가 나오고, 이 버튼을 누르면 팝업이 띄워져 있는 형태의 `PopupViewController`가 나온다.
그리고 이 팝업의 "아니오"나 "알겠습니다" 버튼을 누르면 팝업이 닫힌 `PopupViewController`가 나오고, 여기에서 다시 "Back" 버튼을 누르면 `PopupViewController`가 닫히면서
`ViewController`가 나온다. 이때 뷰 컨트롤러를 present 하고 dismiss할 때는 기본 애니메이션이 실행된다. Present할 때는 `PopupViewController`가
아래에서 위로 올라오고, dismiss할 때 위에서 아래로 내려간다.

![c1](https://user-images.githubusercontent.com/41438361/126753132-a291af84-f04a-420e-af5f-5d44d4c25cb5.gif)

편의상 **`ViewController`를 from view, `PopupViewController`를 to view라고 하겠다.**

## 1. `UIViewControllerTransitioningDelegate` 프로토콜 채택

A ViewController에서 B ViewController로 custom transition을 할 때, 
