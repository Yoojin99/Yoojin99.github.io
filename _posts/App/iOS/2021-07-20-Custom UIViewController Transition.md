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

# 구현

구현이 과정이 복잡해 보일 수 있지만, 위에서 봤던 transitioning api 다이어그램의 구조를 그대로 가져간다.

Custom transition 예제가 굉장히 다양하기 때문에, 그 중 설명이 잘 되어 있는 예제 두 가지를 보면서 작동하는 원리와 custom transition을 구현하는 방법을 살펴볼 것이다.

## 1. [raywenderlich 예제](https://www.raywenderlich.com/322-custom-uiviewcontroller-transitions-getting-started)

시작하기 위한 초기 프로젝트도 사이트에서 제공하고 있다. 여기에서 바로 시작해보자.

### Animator 생성하기

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

2. UIKit이 뷰 계층과 애니메이션을 모두 관리하는 것을 간단하게 하기 위해 전체 전환을 컨테이너 뷰로 캡슐화한다. 우리는 이 컨테이너 뷰의 참조를 얻고 새로운 뷰의 최종 프레임이 어떻게 될 지를 결정하면 된다.
3. 스냅샷의 frame과 drawing을 구성해서 "from" 뷰에 있는 카드와 완전히 일치하고, 커버할 수 있도록 한다. 아까 우리가 스냅샷을 to 뷰가 전환이 일어난 이후의 뷰로 찍었었는데, 이를 Card에 맞춘 것이다.

그리고 아래의 코드를 `animateTransition(using:)`에 덧붙인다.

```swift
// 1
containerView.addSubview(toVC.view)
containerView.addSubview(snapshot)
toVC.view.isHidden = true

// 2
AnimationHelper.perspectiveTransform(for: containerView)
snapshot.layer.transform = AnimationHelper.yRotation(.pi / 2)
// 3
let duration = transitionDuration(using: transitionContext)
```

위에서 생성했던 `containerView`는 UIKit이 생성한 것으로, 오직 "from" 뷰만 포함하고 있다. 꼭 전환에 관련있는 다른 뷰들을 추가해야 한다.
이때 `addSubview(_:)`를 사용하면 새로운 뷰를 뷰 계층에 있는 모든 뷰들보다 가장 앞쪽에 추가한다는 것을 기억해야 한다.

1. "to" view를 뷰 계층에 추가하고 지금은 당장 화면에 나타낼 게 아니니 감춘다. 스냅샷을 가장 앞쪽에 둔다. 스냅샷은 그럼 보이지 않냐고 생각할 수 있는데, 이는 바로 밑에서 초기에 안보이게 설정한다.
2. 스냅샷을 y축을 기준으로 90도 회전에서 애니메이션을 시작하기 전에 스냅샷이 보이지 않게 설정한다.
3. 애니메이션이 실행될 시간을 불러온다.

*`AnimationHelper`는 따로 정의한 것으로, 뷰를 회전하는데 도움을 준다. 코드는 아래와 같다.*

```swift
struct AnimationHelper {
  static func yRotation(_ angle: Double) -> CATransform3D {
    return CATransform3DMakeRotation(CGFloat(angle), 0.0, 1.0, 0.0)
  }
  
  static func perspectiveTransform(for containerView: UIView) {
    var transform = CATransform3DIdentity
    transform.m34 = -0.002
    containerView.layer.sublayerTransform = transform
  }
}
```

이제 세팅은 완료되었고, 애니메이션을 보여줄 차례다. 아래의 코드를 이어서 추가한다.

```swift
// 1
UIView.animateKeyframes(
  withDuration: duration,
  delay: 0,
  options: .calculationModeCubic,
  animations: {
    // 2
    UIView.addKeyframe(withRelativeStartTime: 0.0, relativeDuration: 1/3) {
      fromVC.view.layer.transform = AnimationHelper.yRotation(-.pi / 2)
    }
    
    // 3
    UIView.addKeyframe(withRelativeStartTime: 1/3, relativeDuration: 1/3) {
      snapshot.layer.transform = AnimationHelper.yRotation(0.0)
    }
    
    // 4
    UIView.addKeyframe(withRelativeStartTime: 2/3, relativeDuration: 1/3) {
      snapshot.frame = finalFrame
      snapshot.layer.cornerRadius = 0
    }
},
  // 5
  completion: { _ in
    toVC.view.isHidden = false
    snapshot.removeFromSuperview()
    fromVC.view.layer.transform = CATransform3DIdentity
    transitionContext.completeTransition(!transitionContext.transitionWasCancelled)
})
```

1. 표준 UIView keyframe 애니메이션을 사용한다. 애니메이션이 실행되는 시간은 전환이 일언는 시간과 정확히 일치해야 한다.
2. "from" 뷰를 y축으로 90도 회전해서 화면에서 안보이게 한다.
3. 90도로 회전되어 있던 스냅샷을 다시 원래대로 돌려놔서 화면에서 보이게 한다.
4. 스냅샷의 프레임을 화면 전체 크기가 되게 한다.
5. 스냅샷은 이제 "to" 뷰와 완전히 일치하기 때문에(크기나 위치나) 실제의 "to" view를 노출해도 괜찮다. 스냅샷은 이제 더 이상 필요가 없기 때문에 뷰 계층에서 제거한다. 그리고 "from" 뷰를 원래의 상태로 되돌려놓는다. 그렇지 않으면 다시 원래의 "from" 뷰로 되돌아갈 때 감춰져 있을 것이다. `completeTransition(_:)`을 호출해서 UIKit에게 애니메이션이 끝났다는 것을 알릴 수 있다. 이를 통해 최종 상태가 이제 변하지 않을 것이고 "from" 뷰를 컨테이너에서 없애게 된다.

여기까지의 작업을 통해 animation controller를 이제 사용할 수 있게 된 것이다.

### Animator와 ViewController 연결하기

UIKit은 transitioning delegate가 전환에 필요한 animation controller를 내놓기를 기대하고 있다. 이를 위해 먼저 이 delegate에 `UIViewControllerTransitioningDelegate`를 따르는 객체를 할당해야 한다. 이 예제에서는 `CardViewController`가 transitioning delegate의 역할을 할 것이다. `CardViewController`가 이 프로토콜을 따르게 아래와 같이 코드를 작성한다.

```swift
extension CardViewController: UIViewControllerTransitioningDelegate {
  func animationController(forPresented presented: UIViewController,
                           presenting: UIViewController,
                           source: UIViewController)
    -> UIViewControllerAnimatedTransitioning? {
    return FlipPresentAnimationController(originFrame: cardView.frame)
  }
}
```

이 메서드에서 card의 프레임으로 우리가 아까 만든 animation controller를 초기화해서 반환한다.
이제 `CardViewController`를 transitioning delegate로 설정해주면 된다. `prepare(for:sender:)` 메서드에서 아래와 같이 코드를 작성한다.

```swift
destinationViewController.transitioningDelegate = self
```

여기서 중요한 것은 **화면에 표시될(to) 뷰 컨트롤러의 transitioning delegate를 사용하는 것이지, 화면에 다른 뷰 컨트롤러를 표시하는(from)뷰 컨트롤러의 transitioning delegate를 사용하는 것이 아니라는 것이다.**

프로젝트를 실행하면 이제 아래와 같이 잘 나온다.

![r1](https://user-images.githubusercontent.com/41438361/126452422-6cc55847-cd74-4e89-9299-2f139eecded7.gif)

### View Controller 해제하기

이제 새로운 뷰 컨트롤러를 띄우는 것을 해봤으니 해제하는 것도 해볼 것이다. `FlipDissmissAnimationController`를 생성한다.
그리고 아래와 같이 코드를 작성한다.

```swift
class FlipDismissAnimationController: NSObject, UIViewControllerAnimatedTransitioning {
  
  private let destinationFrame: CGRect
  
  init(destinationFrame: CGRect) {
    self.destinationFrame = destinationFrame
  }
  
  func transitionDuration(using transitionContext: UIViewControllerContextTransitioning?) -> TimeInterval {
    return 0.6
  }
  
  func animateTransition(using transitionContext: UIViewControllerContextTransitioning) {
    
  }
}
```

이 animation controller는 presenting할 때의 애니메이션을 정반대로 수행해야 한다. 이를 위해 아래의 작업들을 해야 한다.

* 현재 띄워진 뷰를 카드의 사이즈로 줄여야 한다. `destinationFrame` 이 이 값을 가진다.
* 원래의 카드를 보여주기 위해 view를 뒤집어야 한다.

`animateTransition(using:)`에 아래의 코드를 추가한다.

```swift
// 1
guard let fromVC = transitionContext.viewController(forKey: .from),
  let toVC = transitionContext.viewController(forKey: .to),
  let snapshot = fromVC.view.snapshotView(afterScreenUpdates: false)
  else {
    return
}

snapshot.layer.cornerRadius = CardViewController.cardCornerRadius
snapshot.layer.masksToBounds = true

// 2
let containerView = transitionContext.containerView
containerView.insertSubview(toVC.view, at: 0)
containerView.addSubview(snapshot)
fromVC.view.isHidden = true

// 3
AnimationHelper.perspectiveTransform(for: containerView)
toVC.view.layer.transform = AnimationHelper.yRotation(-.pi / 2)
let duration = transitionDuration(using: transitionContext)
```

앞서 했던 작업과 비슷하지만, 몇가지 차이가 있다.

1. 이번에는 "from" view의 스냅샷을 찍는다.
2. 레이어링 하는 것은 중요하다. 뒤에서 앞쪽으로 뷰들이 정렬되어 있는 순서는 "to", "from", 스냅샷 순이다. 
3. "to" 뷰를 회전에서 스냅샷을 회전할 때 즉시 나타나지 않게 한다.

아래의 코드를 추가한다.

```swift
UIView.animateKeyframes(
  withDuration: duration,
  delay: 0,
  options: .calculationModeCubic,
  animations: {
    // 1
    UIView.addKeyframe(withRelativeStartTime: 0.0, relativeDuration: 1/3) {
      snapshot.frame = self.destinationFrame
    }
    
    UIView.addKeyframe(withRelativeStartTime: 1/3, relativeDuration: 1/3) {
      snapshot.layer.transform = AnimationHelper.yRotation(.pi / 2)
    }
    
    UIView.addKeyframe(withRelativeStartTime: 2/3, relativeDuration: 1/3) {
      toVC.view.layer.transform = AnimationHelper.yRotation(0.0)
    }
},
  // 2
  completion: { _ in
    fromVC.view.isHidden = false
    snapshot.removeFromSuperview()
    if transitionContext.transitionWasCancelled {
      toVC.view.removeFromSuperview()
    }
    transitionContext.completeTransition(!transitionContext.transitionWasCancelled)
})
```

위 작업들은 presenting 할 때와 완전히 정반대로 작동한다.

1. 스냅샷 뷰를 줄인다음에, 90도로 회전한다. 그리고 "to" 뷰를 원래의 포지션으로 다시 회전시킨다.
2. 스냅샷을 지우고 "from" 뷰의 상태를 원래대로 돌려서 뷰 계층에 가했던 변화를 처리한다. 만약 전환이 취소되었다면 전환 전에 뷰 계층에 추가한 것들을 모두 지워야 한다.

이 dismissing하는 애니메이션은 transitioning delegate에서 처리하니 아래의 코드를 CardViewController 쪽에 추가해준다.

```swift
func animationController(forDismissed dismissed: UIViewController)
  -> UIViewControllerAnimatedTransitioning? {
  guard let _ = dismissed as? RevealViewController else {
    return nil
  }
  return FlipDismissAnimationController(destinationFrame: cardView.frame)
}

```

사실 여기까지 했을 때 예제 사이트에서는 dismiss transition이 정상적으로 작동할 것이라고 했지만 실제로 실행시켰을 때는 애니메이션 후 화면이 까맣게 나왔다. 그래서 다음 예제를 보려고 한다. 실행에서 오류가 있었음에도 이 예제를 글로 썼던 이유는 transitioning 을 위한 기본 구조 설명이 잘 되어 있고, 어떻게 동작하는지 감을 잡을 수 있게 해주는 예제이기 때문이다.

## 2. [Tung Fam](https://medium.com/@tungfam/custom-uiviewcontroller-transitions-in-swift-d1677e5aa0bf) 예제

이 예제는 끝까지 했을 때 가이드에서 나온대로 예상하는 동작이 정상적으로 실행된다. 또한 이 튜토리얼이 설명이 굉장히 자세하기 때문에 글을 직접 읽고 따라하는 것이 많은 도움이 될 것 같다. 이 프로젝트의 전체 코드는 [여기](https://github.com/tungfam/CustomTransitionTutorial)에서 확인이 가능하다. 

### 전제

2개의 뷰가 있다. 첫번째 뷰는 FirstVC, 두번째 뷰는 SecondVC다. 두 뷰 사이의 전환은 1초라고 가정한다. 이 시간동안 시스템에 의해 transition container view라는 새로운 뷰가 제공된다. 그래서 이 뷰는 1초동안 살아있어서 전환 애니메이션을 수행한다. 이 transition container view에 이미지, 라벨, 아이콘 등등을 추가하고 앱에서 일반적으로 애니메이션하는 방법으로 애니메이팅을 할 것이다.

라벨이나 다른 요소들을 애니메이팅하기 위해서는 처음 포지션(frame)과 최종 포지션이 필요하다. 또한 알파 값을 적절히 조정해서 애니메이션을 부드럽게 보여줄 것이다.

### Let’s code 👩‍💻👨‍💻

#### 





dddsssfd







* 출처
  * https://www.raywenderlich.com/322-custom-uiviewcontroller-transitions-getting-started
  * https://medium.com/@tungfam/custom-uiviewcontroller-transitions-in-swift-d1677e5aa0bf
