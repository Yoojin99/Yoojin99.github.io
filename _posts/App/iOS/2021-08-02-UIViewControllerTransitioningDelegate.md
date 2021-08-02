---  
layout: post  
title: "[iOS] - UIViewControllerTransitioningDelegate"  
subtitle: ""  
categories: app
tags: app-ios
comments: true  
header-img: 

---  
  
> `Custom UIViewController Transition을 가능하게 하는 UIViewControllerTransitioningDelegate에 대해 알아보자.`  

---

## UIViewControllerTransitioningDelegate

[이 프로토콜은](https://developer.apple.com/documentation/uikit/uiviewcontrollertransitioningdelegate) 뷰 컨트롤러 간의 고정된
길이나 상호작용 가능한 transition을 관리하는 객체를 제공하는 메서드들을 담고 있다.

iOS 7.0부터 사용 가능하다.

Custom modal presentation type을 사용해서 뷰 컨트롤러를 출력하고 싶으면, `modalPresentationStyle` 프로퍼티를 `custom`으로 설정하고
`transitioningDelgate` 프로퍼티에 이 프로토콜을 따르를 객체를 할당한다. 뷰 컨트롤러를 출력할 때, UIKit은 뷰 컨트롤러를 제 위치로 애니메이팅할 때
transitioning delegate에 사용할 객체를 요구한다.

Transitioning delegate 객체를 구현할 때, 뷰 컨트롤러가 나타나는지, 아니면 해제되는지에 따라 다른 animator 객체를 반환할 수 있다. 모든 전환은
`UIViewControllerAnimatedTransitioning` 프로토콜을 따르는 객체인 **transition animator** 객체를 사용해서 기본적인 애니메이션을 구현한다.  이 transition animator 객체는 유한한 시간동안 애니메이션들을 수행한다. 

만약 애니메이션의 타이밍을 조절할 때 터치나 다른 사용자 interaction을 사용하고 싶으면 `UIViewControllerInteractiveTransitioning` 프로토콜을 따르는 객체인 **interactive animator** 객체를 제공해서 애니메이션의 진행도를 업데이트할 수 있다. 

Custom modal transition 스타일에 대해서는 animator 객체에 추가로 `UIPresentationController` 객체를 제공할 수 있다. 시스템은 이 presentation controller를 뷰 컨트롤러를 출력하기 전에 생성하고 뷰 컨트롤러가 해제될 때까지 이 객체에 대한 참조를 가지고 있는다. 이 presentation controller 객체가 animator 객체의 생명주기를 넘어서서 존재하기 때문에, presentation controller 객체를 사용해서 어려운 presentation이나 dismissal 하는 처리를 하도록 할 수 있다. 예를 들어 custom transition 스타일이 뷰 컨트롤러의 컨텐츠의 배경으로 별도의 그림자 뷰를 출력한다면, presentation controller는 이 그림자 뷰를 만들고 적절한 때에 나타내고 숨길 수 있다.

위에서 총 3가지의 객체에 대해 얘기했다. 

1. Transition animator object 
2. Interactive animator obejct
3. Custom Presentation controller

프로토콜의 메서드도 이에 맞게 각각의 객체들을 가져오는 메서드들로 이루어져 있다.

![image](https://user-images.githubusercontent.com/41438361/126279764-5523d31c-03a9-45de-9324-3edf0091415a.png)
![image](https://user-images.githubusercontent.com/41438361/126279782-048fdf1c-509e-404c-b063-6a16d41289ce.png)
![image](https://user-images.githubusercontent.com/41438361/126279805-ce9274e1-7ff9-40f2-bc20-2629f3ca9911.png)

### 1. Getting the Transition Animator Objects

#### animationController(forPresented:presenting:source:)

뷰 컨트롤러를 띄울 때 delegate에 사용할 **transition animator 객체**를 요청한다.

```swift
optional func animationController(forPresented presented: UIViewController, 
                       presenting: UIViewController, 
                           source: UIViewController) -> UIViewControllerAnimatedTransitioning?
```

* presented: 화면에 띄워지는 뷰 컨트롤러 객체다.
* presenting: 뷰 컨트롤러를 띄우는 뷰 컨트롤러다. Window의 루트 뷰 컨트롤러가 될 수도 있고, 현재 context를 정의하는 부모 뷰 컨트롤러일 수도 있고, 가장 위에 띄워진 뷰 컨트롤러일 수도 있다. source에 있는 것과 같거나 같지 않을 수도 있다.
* source: `present(_:animated:completion:)` 메서드가 호출된 뷰 컨트롤러다.

**리턴 값**

뷰 컨트롤러를 띄울 때 사용되는 animator 객체를 리턴하거나, custom transition을 이용해서 뷰 컨트롤러를 띄우기 싶지 않을 경우 `nil`을 리턴한다.
여기서 리턴하는 객체는 상호작용이 가능하지 않은 고정된 길이의 애니메이션을 구생하는 객체여야 한다.

#### animationController(forDismissed:)

위와는 반대로, 뷰 컨트롤러를 해제할 때 사용할 **transition animator 객체**를 반환한다.

```swift
optional func animationController(forDismissed dismissed: UIViewController) -> UIViewControllerAnimatedTransitioning?
```

* dismissed: 해제될 뷰 컨트롤러 객체다.

나머지는 `animationController(forPresented:presenting:source:)`와 비슷하다. 해제할 때 사용할 애니메이터 객체를 리턴하거나, 애니메이션을
적용하지 않고 해제하려면 `nil`을 리턴한다.

### 2. Getting the Interactive Animator Objects

#### interactionControllerForPresentation(using:)

이것도 위에서 봤던 것처럼 animator 객체를 요청하는데, 다른 점은 **interactive** animator 객체를 요청한다는 것이다.

```swift
optional func interactionControllerForPresentation(using animator: UIViewControllerAnimatedTransitioning) -> UIViewControllerInteractiveTransitioning?
```

위에서는 리턴 타입으로 `UIViewControllerAnimatedTransitioning?` 타입을 반환했었는데, 여기에서는 리턴타입으로 `UIViewControllerInteractiveTransitioning?`을 리턴하는 것을 보아
확실히 다르다는 것을 알 수 있다.

* animator: `animationController(forPresented:presenting:source:)`에 의해 리턴된 transition animator 객체다.

여기서도 마찬가지로, 상호작용이 가능한 interactive transition을 하고 싶지 않으면 `nil`을 리턴하고, 그렇지 않으면 transition의 타이밍을 관리하는
상호작용 animator 객체를 반환한다.

중요한 것은 이 메서드를 구현하면, 파라미터로 넘겨줄 `animator`를 받기 위해 `animationController(forPresented:presenting:source:)` 메서드를 구현해서 custom transition animator 객체를 반환해야 한다는 것이다.

#### interactionControllerForDismissal(using:)

뷰 컨트롤러를 해제할 때 사용할 interactive animator 객체를 받는 메서드다.

```swift
optional func interactionControllerForDismissal(using animator: UIViewControllerAnimatedTransitioning) -> UIViewControllerInteractiveTransitioning?
```

* animator: `animationController(forDismissed:)` 메서드에서 리턴된 애니메이터 객체다.

여기서도 animator에 `animationController(forDismissed:)`에서 반환된 객체를 넣어주므로 이 메서드를 구현하면 꼭 `animationController(forDismissed:)`를 메서드를 구현해야 한다.

### 3. Getting the Custom Presentation Controller

#### presentationController(forPresented:presenting:source:)

뷰 컨트롤러를 출력할 때 뷰 계층을 관리할 때 사용하는 custom presentation controller를 delegate에 요청한다.

UIKit은 새로운 ViewController를 화면에 그릴 때 UITransitionView를 만들고, 그 위에 ViewController를 그리는데, 이 transitionView를 관리하는 controller라고 생각하면 된다.

```swift
optional func presentationController(forPresented presented: UIViewController, 
                          presenting: UIViewController?, 
                              source: UIViewController) -> UIPresentationController?
```

* presented: 띄워지는 뷰 컨트롤러다
* presenting: 띄워지는 뷰 컨트롤러다. 이 파라미터는 뷰 컨트롤러를 띄우는 뷰 컨트롤러가 나중에 결정될 것이라는 것을 나타내기 위해 `nil`이 될 수 있다.
* source: `present(_:animated:completion:)` 메서드를 호출한 뷰 컨트롤러다.

**리턴 값**

Modal presentation을 관리하기 위한 custom presentation controller를 반환한다.

**기타**

`UIModealPresentationStyle.custom` presentation 스타일을 사용해서 뷰 컨트롤러를 띄운다면, 시스템은 이 메서드를 호출하고 커스텀 스타일을 관리하는 presentation controller를 요청한다.
이 메서드를 구현하면, presentation process를 관리하기 위한 custom presentation controller 객체를 생성해서 반환하게 한다.



dddddssss


ddddsss
