---  
layout: post  
title: "[iOS] - Custom UIViewController Transition, 커스텀 뷰컨트롤러 이동 구현하기"  
subtitle: ""  
categories: app
tags: app-ios
comments: true  
header-img: 

---  
  
> `커스텀으로 UIViewController가 전환되도록 구현해보자.`  

---

iOS에는 기본적으로 멋진 view controller transition을 제공한다. Push, pop, cover vertically와 같은 전환이 있지만 이를 커스텀으로 만들 수 있다.

## Transitioning API

들어가기 전에 transitioning API를 알고 있어야 한다. Transitioning API는 프로토콜의 collection이다. 아래 그림은
API의 메인 컴포넌트들이다.

![image](https://user-images.githubusercontent.com/41438361/126275121-adf0cd89-1b09-469d-8bf0-079c53d448a1.png)

### Transitioning Delegate

모든 view controller들은 [`UIViewControllerTransitioningDelegate`](https://developer.apple.com/documentation/uikit/uiviewcontrollertransitioningdelegate)를 따르는 객체인 `transitioningDelegate`를 가질 수 있다.

뷰 컨트롤러를 present하거나 dismiss할때마다, UIKit은 이 `transitioningDelegate`에 사용할 애니메이션 컨트롤러를 요청한다. 기본으로 주어지는
전환 애니메이션(push, pop 등등)을 커스텀 애니메이션으로 바꾸기 위해서는 `UIViewControllerTransitioningDelegate`를 꼭 구현해야 하고 적절한 애니메이션 컨트롤러를 리턴해야 한다.

#### UIViewControllerTransitioningDelegate

뷰 컨트롤러 간의 고정된 수나 interactive한 전환을 관리하기 위해 사용되는 객체들을 제공하는 메서드들의 집합인 프로토콜이다. iOS 7.0부터 이용이 가능하다.

위에서 커스텀 전환 애니메이션을 사용하기 위해서 이 프로토콜을 구현해야 한다고 했는데, 정확하게 말하면 뷰 컨트롤러를 custom modal presentation style 타입으로 출력하고 싶으면, 이 출력하고 싶은 뷰 컨트롤러의 `modalPresentationStyle`을 `custom`으로 설정하고, `transitioningDelegate` 프로퍼티에 이 프로토콜을 따르는 객체를 할당하면 된다. 이 뷰 컨트롤러를 출력할 때, UIKit이 뷰 컨트롤러를 특정 위치로 애니메이팅할 때 사용해야 하는 객체들을 transitioning delegate에 요청한다. 

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


### Animation Controller

다시 원래의 transitioning API로 돌아가서, transitioning delegate가 반환하는 애니메이션 컨트롤러는 `UIViewControllerAnimatedTransitioning`을 구현한 객체다. 이 애니메이션 컨트롤러들이 전환을 애니메이션으로 보여주는 것을 구현하는 무거운 작업을 한다고 생각하면 된다.

#### UIViewControllerAnimatedTransitioning

[이 프로토콜](https://developer.apple.com/documentation/uikit/uiviewcontrolleranimatedtransitioning)은 커스텀 뷰 컨트롤러 전환을 위한 애니메이션을 구현하기 위한 메서드들로 이루어져 있다. 마찬가지로 iOS 7.0부터 이용 가능하다.

여기에 있는 메서드들은 특정 시간에 뷰 컨트롤러를 스크린에 노출시키거나 스크린에서 없애는 전환을 위한 애니메이션을 생성하는 animator object(위 `UIViewControllerTransitioningDelegate` 에서 봤었다)를 정의하게 해준다. 이 프로토콜을 사용해서 생성한 애니메이션은 **꼭 상호작용이 들어가면 안된다.** Interactive 전환을 생성하기 위해서는 animator와 애니메이션의 타이밍을 조절하는 다른 객체를 섞어야 한다.

Animator object에서, 전환이 일어날 시간을 명시하기 위해 `transitionDuration(using:)` 메서드를 구현하고, 스스로 애니메이션을 생성하기 위해 `animateTransition(using:)` 메서드를 구현하면 된다. 전환에 포함된 객체들에 대한 정보는 context 객체의 형태로 `animateTransition(using)` 메서드에 전달된다. 이 context 객체에 포함된 정보를 이용해서 특정 시간에 target 뷰 컨트롤러를
화면에서 띄우거나 해제하면 된다.

Animator 객체를 transitioning delegate(위에서 본 `UIViewControllerTransitioningDelegate`를 구현한 객체)를 이용해서 생성한다. 뷰 컨트롤러를 present할 때, presentation 스타일을 `UIModalPresentationStyle.custom`로 설정하고 transitioning delegate를 뷰 컨트롤러의 `transitioningDelegate` 프로퍼티에 할당하면 된다. 뷰 컨트롤러는 animator 객체를 transitioning delegate에서 받고 애니메이션을 수행하기 위해 사용한다. 뷰 컨트롤러를 표시하고 해제하기 위해 별개의 animator 객체를 사용할 수 있다.

위에서도 잠깐 언급했지만, 사용자 상호작용이 들어간 뷰 컨트롤러 전환을 만드려면 animaator 객체와 **interactive animator 객체(`UIViewControllerInteractiveTransitioning` 프로토콜을 구현한 객체)**를 함께 써야 한다.

![image](https://user-images.githubusercontent.com/41438361/126282877-c2eb34f1-b789-42bb-a017-f4821f71ddfa.png)

메서드는 위의 메서드들이 있고, Required로 표시된 `animateTransition`, `transitionDuration`은 구현해야 한다.

### Transitioning Context

Transitioning context 객체는 `UIViewControllerContextTransitioning`을 구현하고 전환 과정에서 핵심 역할을 담당한다. 위에서 context 객체를 언급하면서 잠깐 얘기했지만, 이 context 객체는 전환하는 과정에 관련있는 뷰와 뷰 컨트롤러에 대한 정보를 캠슐화한다.

다이어그램에서 볼 수 있듯이, 이 프로토콜을 직접 구현하지는 않는다. UIKit이 전환 context를 생성해서 구성하고 전환이 일어날 때마다 animation controller에 전달한다.

#### UIViewControllerContextTransitioning

[이 프로토콜](https://developer.apple.com/documentation/uikit/uiviewcontrollercontexttransitioning)은 뷰 컨트롤러 간 전환 애니메이션에 문맥적인 정보를 제공하는 메서드로 이루어져 있다.

![image](https://user-images.githubusercontent.com/41438361/126284291-19879af3-8d49-4c7d-b6b9-7469e6c70cf8.png)

위에서 봤듯이, 이 프로토콜을 직접구현하면 안된다. 전환 도중에, 전환에 포함된 animator 객체들이 UIKit에서 완전히 구성된 context 객체를 받을 것이다. `UIViewControllerAnimatedTransitioning`이나 `UIViewControllerInteractiveTransitioning` 프로토콜을 채택한 custom animator 객체들은 제공된 객체에서 필요한 정보들을 꼭 받아야 한다.

Context 객체는 전환에 포함된 뷰와 뷰 컨트롤러에 대한 정보를 포함하고 있다. 또한 전환을 어떻게 수행해야 할지에 대한 디테일들도 포함하고 있다. 상호작용이 있는 전환의 경우, interactive animator 객체는 애니메이션의 진행도를 report하기 위해 이 프로토콜의 메서드를 사용한다. 애니메이션이 시작되면, interactive animator 객체는 context 객체를 가리키는 포인터를 저장해야 한다. 사용자 상호작용에 기반해서, animator 객체는 애니메이션이 끝났는지를 report하기 위해 `updateInteractiveTransition(_:)`, `finishInteractiveTransition()`이나 `cancelInteractiveTrnasition()` 메서드를 호출한다. 

![image](https://user-images.githubusercontent.com/41438361/126291047-0676038e-8607-4da8-bacd-667a466cca23.png)

중요한 점은, custom animator 객체를 정의할 때, 애니메이션을 생성해야 할지 말지를 결정하기 위해 `isAnimated()` 메서드에 의해 반환되는 값을 항상 체크해야 한다는 점이다. 그리고 전환 애니메이션을 생성하면, `completeTransition(_:)` 메서드를 호출해서 UIKit이 애니메이션이 모두 끝났다는 것을 알게 해야 한다.

### Transitioning Process

뷰 컨트롤러를 presenting하는 전환은 아래의 단계로 진행된다. Dismiss하는 경우도 거의 똑같다. Dismiss하는 경우에는 UIKit이 "from" 뷰 컨트롤러(해제되는 뷰 컨트롤러)의 transitioning delegate를 요청하게 된다.

1. 전환을 하는 코드를 실행한다.
2. UIKit이 "to" 뷰 컨트롤러(보여져야 하는 뷰 컨트롤러)의 transitioning delegate가 있는지 확인한다. 만약 없다면, UIKit은 디폴트 전환을 사용한다.
3. UIKit은 위에서 찾은 transitioning delegate에게 animation controller를 요구하는데, `animationController(forPresented:presenting:source:)` 메서드를 통해 요구한다. 만약 이 메서드가 nil을 리턴하면, 전환은 기본 애니메이션을 사용할 것이다.
4. UIKit이 transitioning context를 구성한다.
5. UIKit이 `transitionDuration(using:)`을 호출해서 애니메이션으로 보여줄 시간이 얼마나 될지 animation controller에게 물어본다.
6. UIKit이 전환에 사용되는 애니메이션을 보여주기 위해 animation controller의 `animateTransition(using:)`을 호출한다.
7. 마지막으로, animation controller가 애니메이션이 끝났을을 나타내기 위해 transitioning context의 `completeTransition(_:)`을 호출한다.

## 구현

구현이 과정이 복잡해 보일 수 있지만, 위에서 봤던 transitioning api 다이어그램의 구조를 그대로 가져간다.

Custom transition 예제가 굉장히 다양하기 때문에, 그 중 설명이 잘 되어 있는 예제 두 가지를 보면서 작동하는 원리와 custom transition을 구현하는 방법을 살펴볼 것이다.

### 1. [raywenderlich 예제](https://www.raywenderlich.com/322-custom-uiviewcontroller-transitions-getting-started)

시작하기 위한 초기 프로젝트도 사이트에서 제공하고 있다. 여기에서 바로 시작해보자.

#### Animator 생성하기

애니메이션을 수행하는 animatation controller를 생성할 것이다. Animation Controllers 폴더에 FlipPresentAnimationController.swift를 생성했다. 

위에서 언급했듯이, animation controller들은 `UIViewControllerAnimatedTransitioning` 프로토콜을 채택해야 한다. 

![image](https://user-images.githubusercontent.com/41438361/126294725-c5a1c775-d874-412c-ad3a-778feaf6ad90.png)

`transitionDuration(using:)`은 애니메이션을 수행할 시간을 설정할 수 있고, `animateTransition(using:)`을 사용해서 실제 애니메이션을 동작하는 부분을 구현한다.

`animateTransition(using:)`에 추가할 코드들을 하나씩 살펴보자.

```swift
// 1
guard let fromVC = transitionContext.viewController(forKey: .from),
  let toVC = transitionContext.viewController(forKey: .to),
  let snapshot = toVC.view.snapshotView(afterScreenUpdates: true)
  else {
    return
}

// 2
let containerView = transitionContext.containerView
let finalFrame = transitionContext.finalFrame(for: toVC)

// 3
snapshot.frame = originFrame
snapshot.layer.cornerRadius = CardViewController.cardCornerRadius
snapshot.layer.masksToBounds = true
```

1. 사라져야 하는 뷰 컨트롤러와, 나타나야 하는 뷰 컨트롤러 모두에 대한 참조를 획득한다. 또한 전환 이후에 화면이 어떻게 보일지에 대한 snapshot을 만든다.

여기서 snapshot이 뭘까? 스냅샷은 말 그래도 스크린 샷이다. [공식문서](https://developer.apple.com/documentation/uikit/uiview/1622531-snapshotview)에 따르면, `snapshotView(afterScreenUpdates:)` 메서드는 현재 뷰에 있는 컨텐츠를 기반으로 snapshot을 리턴한다. `afterScreenUpdates` 파라미터는 스냅샷이 변화가 일어난 이후에 찍어져야 하는지 아닌지를 나타내는 bool 값이다. 만약 false를 전달하면 변화를 적용하지 않은 현재 상태에서의 화면의 snapshot을 찍는다.

2. UIKit





* 출처
  * https://www.raywenderlich.com/322-custom-uiviewcontroller-transitions-getting-started
  * https://medium.com/@tungfam/custom-uiviewcontroller-transitions-in-swift-d1677e5aa0bf
