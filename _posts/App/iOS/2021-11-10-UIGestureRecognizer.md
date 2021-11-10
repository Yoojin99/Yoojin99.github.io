---  
layout: post  
title: "[iOS] - UIGestureRecognizer"  
subtitle: ""  
categories: app
tags: app-ios UIGestureRecognizer
comments: true  
header-img: 

---  
  
> `UIGestureRecognizer 정리`  

---

사용자가 앱을 사용하기 위해서는 단순히 UI만 존재하는 것이 아니라 UI와 상호작용을 할 수 있어야 한다. 버튼을 터치한다던가, 스와이프를 한다던가,
드래그하는 동작과 같은 상호작용을 통해 사용자는 앱을 조작할 수 있다. 여기에 사용할 수 있는 것이 바로 `UIGestureRecognizer`이다.

UIGestureRecognizer는 이름에서부터 짐작할 수 있듯이, UI에 사용자가 하는 gesture를 recognize 해주는 역할을 하는 클래스인 것 같다.

공식 문서의 설명을 보면 UIGestureRecognizer는 concrete gesture recognizer들을 위한 기본이 되는 클래스다.

![image](https://user-images.githubusercontent.com/41438361/141068289-f867e832-fd37-4fa3-b587-7e79f05f84a3.png)

Gesture recognizer 객체나 gesture recognizer는 

* 연속적인 터치(아니면 다른 입력)을 인식하기 위한 로직
* 인식한 이후 반응하는 것

이 두 가지를 분리한다.

Gesture recognizer 중 하나가 이런 일반적인 제스쳐들이나 제스쳐의 변화를 감지하면 각각 target 객체에 action message를 보낸다.

사람이 할 수 있는 제스쳐가 굉장히 다양하기 때문에, `UIGestureRecognizer`의 concrete class(구현 클래스)도 굉장히 많다.

* UITapGestureRecognizer
* UIPinchGestureRecognizer
* UIRotationGestureRecognizer
* UISwipeGestureRecognizer
* UIPanGestureRecognizer
* UIScreenEdgePanGestureRecognizer
* UILongPressGestureRecognizer
* UIHoverGestureRecognizer

`UIGestureRecognizer` 클래스는 모든 concrete gesture recognizer를 구성할 수 있는 일반적인 행위들을 정의한다. `UIGestureRecognizerDelegate` 프로토콜을 채택한 객체인
delegate와 상호작용해서 특정 행위들을 커스터마이징하는 것도 가능하다.

Gesture recognizer는 특정 뷰와 그 뷰의 자식 뷰들에 hit-test 된 터치들에 대해 동작한다. 따라서 뷰와 꼭 연관되어 있어야 한다. 이런 연관성을 만들어주기 위해
꼭 UIView의 `addGestureRecognizer(_:)` 메서드를 수행해야 한다. Gesture Recognizer는 뷰의 respoonder chain에 참여하진 않는다.



* 참고
* https://developer.apple.com/documentation/uikit/uigesturerecognizer
