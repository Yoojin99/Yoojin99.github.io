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

## 1. `UIViewControllerTransitioningDelegate` 프로토콜 채택, delegate 등록

A ViewController에서 B ViewController로 custom transition을 할 때, B ViewController의 transitioning delegate를 쓰게 된다.

우리는 `ViewController`에서 `PopupVIewController`로 전환을 시도하므로, `ViewController`가 `UIViewControllerTransitioningDelegate`를 따르도록 한 다음에 `PopupViewController`의 transitioning delegate에 `ViewController`를 할당해야 한다.

먼저, ViewController.swift 파일의 맨 아래에 아래와 같이 extension을 추가한다.

```swift
extension ViewController: UIViewControllerTransitioningDelegate {
    func animationController(forPresented presented: UIViewController, presenting: UIViewController, source: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        return nil
    }
}
```

이 `animationController(~~)` 메서드는 애니메이션을 수행하는 animation controller를 리턴한다. 위에서는 presenting 할 때 특정 animation controller를 반환해서 이 컨트롤러를 사용해서 애니메이션을 수행하겠다는 소리가 된다. nil을 리턴하고 있으니 transition 할 때는 default 애니메이션을 이용하게 된다. 

from view가 `UIViewControllerTransitioningDelegate`를 따르니 to view의 transitioning delegate를 from view로 설정해준다.

ViewController.swift의 `vc`를 선언한 부분 밑에 아래와 같이 코드를 작성한다.

```swift
@objc private func touchGoBtn() {
        ...
        let vc = PopupViewController(viewModel: model)
        vc.modalPresentationStyle = .fullScreen
        // new
        vc.transitioningDelegate = self
        
        present(vc, animated: true, completion: nil)
    }
```

## 2. Animation Controller 만들기

Animator라는 폴더 안에 PopupAnimator.swift라는 파일을 생성했다. 이 애니메이터는 바로 위에서 구현했던 `animationController()`메서드에서 리턴할 애니메이션 컨트롤러가 될 것이다.











