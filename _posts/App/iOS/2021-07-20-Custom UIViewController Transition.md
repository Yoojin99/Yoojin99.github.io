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

#### Block 1: Preparing UIViewControllerTransitioningDelegate

애니메이션은 아직 만들지 않고, custom transition을 위한 프로토콜을 채택한다.

FirstViewController+TransitioningDelegate.swift에 아래의 코드를 추가한다.

```swift
// B1 - 1
extension FirstViewController: UIViewControllerTransitioningDelegate {

    // B1 - 2
    func animationController(forPresented presented: UIViewController, presenting: UIViewController, source: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        return nil
    }

    // B1 - 3
    func animationController(forDismissed dismissed: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        return nil
    }
}
```

1. `UIViewControllerTransitioningDelegate`를 따라서 FirstVC가 custom transition 메서드를 구현한 delegate가 되도록 할 것이다.
2. Presenting을 위한 custom transition 메서드다. 전환을 애니메이팅할 객체를 리턴해야 한다. 현재는 nil인데, 이는 디폴트 전환을 사용하겠다는 뜻이다.
3. 2와 똑같은데, dismissing을 위한 점만 다르다.

`FirstViewController.swift`의 `presentSecondViewController(with:)` 메서드에서 아래의 코드를 추가한다.

```swift
func presentSecondViewController(with data: CellData) {
        let secondViewController = ...

        // B1 - 4
        secondViewController.transitioningDelegate = self
        ...
}
```

4. FirstVC를 SecondVC의 delegate로 설정한다. Presenting이 아닌 presented하는 뷰 컨트롤러의 delegate가 사용된다는 점을 기억하자.

이 상태로 실행하면 전과 기본 프로젝트 상태와 달라진게 없다.

#### Block2. Animator 클래스 생성, 애니메이션 없이 전환하기 기본 세팅하기

이제 애니메이션을 수행하는 Animator 클래스를 구현할 것이다.

먼저 FirstViewController.swift에 아래 코드를 추가한다.

```swift
class FirstViewController: UIViewController {

    // B2 - 5
    var selectedCell: CollectionViewCell?
    var selectedCellImageViewSnapshot: UIView?

    ...
```

5. `selectedCell`은 선택된 cell, `selectedCellImageViewSnapshot`은 선택된 cell의 이미지 뷰의 스냅샷이다.

스냅샷은 위에서도 말했듯이, 현재 렌더링된 뷰를 스크린샷 찍은 뷰라고 생각하면 된다. 하지만 원래 뷰처럼 자식 뷰들이 있는 것은 아니고, 단일한
하나의 뷰로 구성되어 있다.

이제 `FirstViewController+CollectionView.swift`의 `collectionView(_:didSelectItemAt:)`에서 아래의 두 줄을 추가한다.

```swift
func collectionView(_ collectionView: UICollectionView, didSelectItemAt indexPath: IndexPath) {
    // B2 - 6
    selectedCell = collectionView.cellForItem(at: indexPath) as? CollectionViewCell
    // B2 - 7
    selectedCellImageViewSnapshot = selectedCell?.locationImageView.snapshotView(afterScreenUpdates: false)
    ...
    
```

6. cell을 선택할 때 `selectedCell`에 할당한다.
7. 선택된 cell의 이미지 뷰의 스냅샷을 `selectedCellImageViewSnapshot`에 저장한다.

이제 `Animator.swift`에서 아래의 코드를 삽입한다.

```swift
import UIKit

// B2 - 8
final class Animator: NSObject, UIViewControllerAnimatedTransitioning {

    // B2 - 9
    static let duration: TimeInterval = 1.25

    private let type: PresentationType
    private let firstViewController: FirstViewController
    private let secondViewController: SecondViewController
    private let selectedCellImageViewSnapshot: UIView
    private let cellImageViewRect: CGRect

    // B2 - 10
    init?(type: PresentationType, firstViewController: FirstViewController, secondViewController: SecondViewController, selectedCellImageViewSnapshot: UIView) {
        self.type = type
        self.firstViewController = firstViewController
        self.secondViewController = secondViewController
        self.selectedCellImageViewSnapshot = selectedCellImageViewSnapshot

        guard let window = firstViewController.view.window ?? secondViewController.view.window,
            let selectedCell = firstViewController.selectedCell
            else { return nil }

        // B2 - 11
        self.cellImageViewRect = selectedCell.locationImageView.convert(selectedCell.locationImageView.bounds, to: window)
    }

    // B2 - 12
    func transitionDuration(using transitionContext: UIViewControllerContextTransitioning?) -> TimeInterval {
        return Self.duration
    }

    // B2 - 13
    func animateTransition(using transitionContext: UIViewControllerContextTransitioning) {
        // steps 18-20 will be here later.
    }
}

// B2 - 14
enum PresentationType {

    case present
    case dismiss

    var isPresenting: Bool {
        return self == .present
    }
}
```

8. Animator는 애니메이션을 구현할 클래스다. `UIViewControllerAnimatedTransitioning`을 따른다. 
9. 애니메이션에 필요한 프로퍼티들이다.
10. 설정했던 프로퍼티에 값을 할당하는 커스텀 init이다. 중요한 것은 무언가가 잘못 작동하면, 꼭 nil을 반환해야 한다. 이렇게 하면 뭔가 잘못 돌아갈 때 기본 present/dismiss 애니메이션을 사용하게 되어서 막히지 않고 실행을 계속할 수 있다.
11. 윈도우의 프레임에 상대적인 cell의 프레임을 구한다. 이는 transition container view에서 애니메이션을 보여주기 위해 필수적인 과정이며, 콜렉션뷰의 셀을 적절한 좌표계의 위치로 전환해야 한다.
12. `UIViewControllerAnimatedTransitioning` 프로토콜의 요구되는 메서드이다. 애니메이팅하고 싶은 시간을 리턴하면 된다. 
13. 12번과 마찬가지로 요구되는 메서드이다. 모든 transition 로직과 애니메이션 구현 코드가 여기 들어갈 것이다.
14. `enum`을 정의해 화면이 해제되었는지 출력되었는지를 확인한다. Animator가 어떤 애니메이션을 사용할지를 결정하는데 사용된다.

`FirstViewController.swift`에 아래 코드를 추가한다.

```swift
// B2 - 5
...

// B2 - 15
var animator: Animator?

override func viewDidLoad() { }
...
```

그리고 `FirstViewController+TransitioningDelegate.swift`를 열고 아래의 코드로 두 메서드를 수정한다.

```swift
// B1 - 2
func animationController(forPresented presented: UIViewController, presenting: UIViewController, source: UIViewController) -> UIViewControllerAnimatedTransitioning? {
    // B2 - 16
    guard let firstViewController = presenting as? FirstViewController,
        let secondViewController = presented as? SecondViewController,
        let selectedCellImageViewSnapshot = selectedCellImageViewSnapshot
        else { return nil }

    animator = Animator(type: .present, firstViewController: firstViewController, secondViewController: secondViewController, selectedCellImageViewSnapshot: selectedCellImageViewSnapshot)
    return animator
}

// B1 - 3
func animationController(forDismissed dismissed: UIViewController) -> UIViewControllerAnimatedTransitioning? {
    // B2 - 17
    guard let secondViewController = dismissed as? SecondViewController,
        let selectedCellImageViewSnapshot = selectedCellImageViewSnapshot
        else { return nil }

    animator = Animator(type: .dismiss, firstViewController: self, secondViewController: secondViewController, selectedCellImageViewSnapshot: selectedCellImageViewSnapshot)
    return animator
}
```

16. `Animator`의 인스턴스를 초기화하기 위한 프로퍼티들을 준비한다. 만약 준비하는 도중에 문제가 생기면 default 애니메이션을 보여준다. 
17. 16번과 똑같지만, dismiss하는 것에서만 차이가 난다.

그리고 `Animator.swift`를 열고 `animateTransition(using:)` 메서드에 아래 코드를 넣는다.

```swift
// B2 - 13
func animateTransition(using transitionContext: UIViewControllerContextTransitioning) {
    // B2 - 18
    let containerView = transitionContext.containerView

    // B2 - 19
    guard let toView = secondViewController.view
        else {
            transitionContext.completeTransition(false)
            return
    }

    containerView.addSubview(toView)

    // B2 - 20
    transitionContext.completeTransition(true)
}
```

18. `containerView`(Transition Container View)는 transition에 의해 제공되는 인스턴스다. 이 컨테이너를 애니메이션을 보여줄때 사용하는 뷰라고 생각하면 된다. 기본적으로 1st, 2nd 뷰 컨트롤러가 애니메이션을 보여주기 위해 두 뷰 사이에 붙여지는 뷰이다.
19. 2nd 화면을 출력하기 위해, 이를 `containerView`에 자식 뷰로 추가해야 한다. 만약 여기서 실패하면 `completeTransition`을 `false`로 넘겨서 전환이 일어나지 않게 한다.
20. 끝에는 `completeTransition`을 불러서 `transitionContext`가 전환이 끝났을을 알게 한다. 

이제 실행시키면 아무런 사용자 지정 애니메이션 없이 default 전환이 일어나는 것을 확인할 수 있다. 지금까지 한 작업이 애니메이션을 보여주기 위해 틀을 닦은 것이다.

어떤 일들이 일어나는지 명확하게 이해하기 위해서, 아래의 두 가지 작업을 해보는 것도 좋다.

1. `animateTransition(using:)` 메서드에서 `addSubview`나 `completeTransition` 호출을 주석처리 해본다.
  * 우선 `addSubview`를 주석처리해서 `toView`를 자식 뷰로 추가를 안했더니 cell을 터치하고 나서 화면이 까맣게 변했다.
  * `completeTransition`을 주석처리하니 second view를 presenting 하는 것은 됐으나 dismissing 하는 게 동작하지 않았다.
2. `Animator`의 `init`이 무조건 `nil`을 리턴하게 해본다. 이 경우는 `init`이 실패했을 때 어떻게 동작하는지 확인하기 위함인데, 전환 과정에서 멈추지 않고 기본 애니메이션을 출력하는 것을 볼 수 있다.

#### Block3: 애니메이션 구현하기

이제 셀의 이미지를 두번째 뷰 컨트롤러의 이미지로 전환하는 첫번째 애니메이션을 구현할 것이다.

`Animator.swift`에서 step 20에서 했던 부분을 지우고 아래의 코드를 추가한다.

```swift
func animateTransition(using transitionContext: UIViewControllerContextTransitioning) {
    ...
    containerView.addSubview(toView)

    // B2 - 20
    // deleted line: transitionContext.completeTransition(true)
    // B3 - 21
    guard let selectedCell = firstViewController.selectedCell,
        let window = firstViewController.view.window ?? secondViewController.view.window,
        let cellImageSnapshot = selectedCell.locationImageView.snapshotView(afterScreenUpdates: true),
        let controllerImageSnapshot = secondViewController.locationImageView.snapshotView(afterScreenUpdates: true)
        else {
            transitionContext.completeTransition(true)
            return
    }

    let isPresenting = type.isPresenting

    // B3 - 22
    let imageViewSnapshot: UIView

    if isPresenting {
        imageViewSnapshot = cellImageSnapshot
    } else {
        imageViewSnapshot = controllerImageSnapshot
    }

    // B3 - 23
    toView.alpha = 0

    // B3 - 24
    [imageViewSnapshot].forEach { containerView.addSubview($0) }

    // B3 - 25
    let controllerImageViewRect = secondViewController.locationImageView.convert(secondViewController.locationImageView.bounds, to: window)

    // B3 - 26
    [imageViewSnapshot].forEach {
        $0.frame = isPresenting ? cellImageViewRect : controllerImageViewRect
    }

    // B3 - 27
    UIView.animateKeyframes(withDuration: Self.duration, delay: 0, options: .calculationModeCubic, animations: {
        UIView.addKeyframe(withRelativeStartTime: 0, relativeDuration: 1) {
            // B3 - 28
            imageViewSnapshot.frame = isPresenting ? controllerImageViewRect : self.cellImageViewRect
        }
    }, completion: { _ in
        // B3 - 29
        imageViewSnapshot.removeFromSuperview()

        // B3 - 30
        toView.alpha = 1

        // B3 - 31
        transitionContext.completeTransition(true)
    })
}
```

21. `selectedCell`과 `window`가 `nil`이 아님을 보장해야 한다. Window에는 현재 출력되고 있는 화면을 보여주고 있다. 그래서 presenting할 때에는 FirstVC의 window가 될 것이고, dismissing할 때는 SecondVC의 window가 될 것이다. `cellImageSnapshot`은 선택된 cell의 이미지의 스냅샷이고, `controllerImageSnapshot`은 Second VC의 이미지의 스냅샷이다.
22. `imageViewSnapshot`은 이미지의 transition을 애니메이션으로 보여주기 위해 사용되는 뷰이다. **Presenting할 때는 cell의 이미지, dismissing할 때는 VC의 이미지를 애니메이션에 사용할 것이다.**
23. 출력되어야 하는 뷰는 투명하지 않으면 우리의 애니메이션 뷰를 덮기 때문에 투명해야 한다. 
24. 우리가 애니메이팅 할 첫번째 뷰인 `imageViewSnapshot`을 container view에 추가한다. 왜 배열로 감싸서 `forEach`를 했는지 궁금할텐데, 이는 뒤에서 다른 뷰들을 배열에 더 추가하기 때문이다.
25. SecondVC에 있는 이미지 뷰의 프레임을 가져온다. 이를 cell의 프레임에서 전환할때 사용할 것이다.
26. 애니메이션이 적용될 스냅샷을 초기 프레임에 할당한다. 이를 하지 않으면, 우리가 이 새로운 뷰를 뷰 계층에 추가한 이후에 왼쪽 상단 구석에서 끝날 것이다. 스냅샷은 완전히 새로운 뷰이고, cell이나 SecondVC의 이미지가 아님을 이해하는 것이 중요하다. 
27. 마침내 애니메이션을 하는 부분이다! 우리가 하나의 애니메이션만 하고 있는데 `UIView.animate`를 하지 않는 이유가 궁금할 것이다. 이는 이후에 시작 지점과 애니메이션을 보여줄 시간을 다르게 잡는 것을 추가할 것이기 때문이다. `UIView.animateKeyframes`에 대한 설명은 바로 밑에 추가해뒀다.
28. 애니메이션은 이제 간단한 작업들만 남았다. 지금 presenting하고 있다면, cell의 프레임이 `imageViewSnapshot`에 할당되어쓸 것이고, 이 프레임을 SecondVC의 이미지의 프레임으로 바꾼다. Dismissing하는 경우에는 반대가 될 것이다.
29. 완료되었다면 전환에 사용했던 모든 뷰들을 지워야 한다. 여기서는 `imageViewSnapshot`을 지운다.
30. 23번째 단계에서, `toView`를 투명하게 만든 것을 기억할 것이다. 이제 전환이 끝났으니 다시 불투명하게 만들어주면 된다.
31. 마지막으로 전환을 마친다.

> *UIView.animateKeyframes?*
> 
> ![image](https://user-images.githubusercontent.com/41438361/126596490-ddb74787-9227-4df2-a320-754cabd91e8a.png)
>
> 현재 뷰에 적용할 수 있는 keyframe을 기반으로 하는 애니메이션을 설정하는데 사용되는 animation block 객체를 생성한다.
> 
> * duration : 전체 애니메이션을 실행할 시간으로, 초단위이다. 만약 0이하의 값으로 설정했다면 애니메이션 없이 변화가 바로 적용될 것이다.
> * delay : 애니메이션을 시작하기 전까지 기다릴 시간으로, 초단위이다.
> * options : 어떻게 애니메이션을 수행하고 싶은지를 나타내는 옵션이다.
> * animations : 뷰에 일어날 변화를 포함하는 블럭 객체다. 일반적으로, 이 블럭 안에서 
> `addKeyframe(withRelativeStartTime:relativeDuration:animations)` 메서드를 한 번 이상 호출한다. 만약 애니메이션을 실행하는 총 시간 동안 변화를 보여주고 싶다면 뷰의 값을 직접 바꿔도 된다. 이 블럭은 파라미터와 리턴 값이 없다. 파라미터로 nil을 전달하지 마라.
> * completion : 애니메이션이 종료되었을 때 수행할 블럭 객체다. 이 블럭은 반환 값이 없고 애니메이션이 completion 핸들러가 호출되기 전에 끝났는지 아닌지에 대한 걸 나타내는 bool 값을 인자로 받고 있다. 만약 애니메이션의 기간이 0초이면, 이 블록은 다음 run loop cycle의 초반에 실행된다. nil을 인자로 넘겨줄 수 있다.
>
> 더 자세한 내용은 [애플의 공식문서](https://developer.apple.com/documentation/uikit/uiview/1622552-animatekeyframes)를 참조하면 된다.

다시 전환하는 걸로 돌아가서, 위의 코드까지 작성하고 실행하면 전환이 잘 일어나지만 문제점들이 보인다.

![transition](https://user-images.githubusercontent.com/41438361/126597367-e560bb39-4f53-4609-b9d7-dbc2d54fbb65.gif)

1. 이미지가 presenting할 때 늘려지고, dismissing할 때 찌그러진다.
2. 애니메이션이 끝나면, 한 이미지에서 다른 이미지로 즉각적으로 전환이 일어나는 것을 볼 수 있다.

Cell과 SecondVC의 이미지 모두 `contentMode = .scaleAspectFill`을 사용하고 있다. 그래서 주어진 사각형을 완전히 채우는 이미지가 보여지는 것이다. 

여기서 위에 언급한 두가지 문제점이 생기는 이유는 무엇일까? 이는 두 이미지 뷰 프레임이 다른 비율을 가지기 때문이다. Cell의 이미지는 정사각형이지만, 뷰 컨트롤러의 이미지는 직사각형이다.

이미지 뷰의 비율을 직접 조정해서 이런 문제가 일어나지 않는 것처럼 보이게 할 수도 있지만, 이렇게 수정하는 것은 바보 같은 짓에 가깝다. 

이제 이슈가 우리가 전환에 사용하는 이미지들의 비율이 다르다는 것을 알았는데, 어떻게 해결해야 할까?

![aribnbTransition](https://user-images.githubusercontent.com/41438361/126599047-770bfcfa-22a5-4c7f-9a60-b6a83c14795f.gif)

위는 에어비엔비의 앱 실행화면이다. 보면, fade와 함께 이미지의 전환이 일어난다. 따라서 아래와 같은 방법으로 해결해볼 수 있다.

1. 전환할 때 2개의 이미지가 필요하다.
2. 두 이미지의 프레임을 같은 시간에 바꾼다.
3. Presenting할 때에는 cell의 이미지를 점점 없어지게 하고 controller의 이미지를 점점 나타나게 한다. Dismissing할 때는 반대가 될 것이다.

#### Block4: 늘어남/찌그러짐을 방지하기 위해 cell/controller 사이의 이미지 전환을 자연스럽게 만들기

`Animator.swift`에서 `selectedCellImageViewSnapshot`을 `var` 로 수정한다.

그리고 22단계에서 했던 코드를 지우고 아래의 코드를 추가한다.

```swift
// B4 - 33
if isPresenting {
    selectedCellImageViewSnapshot = cellImageSnapshot
}
```

33. Presenting할 때, `selectedCellImageViewSnapshot`에 `cellImageSnapshot`을 할당한다. 이는 `selectedCellImageViewSnapshot`을 찍는 순간에, view가 업데이트 되지 않는 이슈때문에 다시 snapshot을 찍기 위함이다. 

이제, 대부분의 수정은 `animateTransition(using:)`에서 일어난다. 이제 `animateTransition(using:)` 메서드를 아래의 코드로 수정한다.

```swift
func animateTransition(using transitionContext: UIViewControllerContextTransitioning) {

    ...

    // B3 - 22 - deleted step
    // B4 - 33
    if isPresenting {
        selectedCellImageViewSnapshot = cellImageSnapshot
    }

    // B3 - 23
    toView.alpha = 0

    // B3 - 24 - deleted step
    // B4 - 34
    [selectedCellImageViewSnapshot, controllerImageSnapshot].forEach { containerView.addSubview($0) }

    // B3 - 25
    let controllerImageViewRect = secondViewController.locationImageView.convert(secondViewController.locationImageView.bounds, to: window)

    // B4 - 35
    [selectedCellImageViewSnapshot, controllerImageSnapshot].forEach {
        $0.frame = isPresenting ? cellImageViewRect : controllerImageViewRect
    }

    // B4 - 36
    controllerImageSnapshot.alpha = isPresenting ? 0 : 1

    // B4 - 37
    selectedCellImageViewSnapshot.alpha = isPresenting ? 1 : 0

    // B3 - 27
    UIView.animateKeyframes(withDuration: Self.duration, delay: 0, options: .calculationModeCubic, animations: {

        UIView.addKeyframe(withRelativeStartTime: 0, relativeDuration: 1) {
            // B3 - 28
            // B4 - 38
            self.selectedCellImageViewSnapshot.frame = isPresenting ? controllerImageViewRect : self.cellImageViewRect
            controllerImageSnapshot.frame = isPresenting ? controllerImageViewRect : self.cellImageViewRect
        }

        // B4 - 39
        UIView.addKeyframe(withRelativeStartTime: 0, relativeDuration: 0.6) {
            self.selectedCellImageViewSnapshot.alpha = isPresenting ? 0 : 1
            controllerImageSnapshot.alpha = isPresenting ? 1 : 0
        }
    }, completion: { _ in
        // B3 - 29
        // B4 - 39.1
        self.selectedCellImageViewSnapshot.removeFromSuperview()
        controllerImageSnapshot.removeFromSuperview()

        // B3 - 30
        toView.alpha = 1

        // B3 - 31
        transitionContext.completeTransition(true)
    })
}
```

34. 24단계에서 했던 코드를 삭제했다. 대신, 2개의 스냅샷을 추가했다. Cell의 이미지와 컨트롤러의 이미지이다. 
35. 26단계에서 했던 것과 비슷하지만, 이제 2개의 새로운 뷰가 있다. 이 두 뷰에 모두 초기 프레임을 할당할 것이다. 
36, 37. 만약 presenting하고 있는 중이라면, 컨트롤러의 이미지가 투명해야 하고 cell의 이미지가 불투명해야 한다. Dismissing하는 경우에는 반대다.
38. 28단계에서 했던 코드를 삭제했다. 28단계에서 했던 것과 똑같이 추가한다. 두 이미지들의 프레임을 전환 후 최종 상태의 프레임으로 설정해서 애니메이션이 일어나게 한다.
39. 두 이미지의 알파값을 동시에 애니메이션으로 수정한다.
39.1. 두 스냅샷 이미지를 뷰에서 없앤다.

이제 실행하면 전환이 일어날때 매끄럽게 이미지의 변환이 일어나는 것을 볼 수 있다.

![transition2](https://user-images.githubusercontent.com/41438361/126608895-a60ff356-7ae9-44e8-8fc3-9c4d8b02db38.gif)

이제 이미지가 늘어나고 줄어드는 이슈는 해결했다. 그래서 여기까지 우리는 셀의 이미지 하나, 컨트롤러의 이미지 하나 이렇게 두 개의 이미지를 애니메이션 작업을 한 것이다. Presenting할 때 셀의 이미지를 fade out하는 동시에 controller의 이미지를 fade in 시켰다. 하지만 전환이 일어날때 배경, 버튼, 텍스트는 포함되지 않았다. 이는 위에서 지금까지 했던 것과 비슷한 로직으로 적용할 것이다.

#### Block 5: Fading하는 배경 추가하기

이제 애니메이션에 하얀색 배경을 추가할 것이다.

```swift
func animateTransition(using transitionContext: UIViewControllerContextTransitioning) {

    ...

    let isPresenting = type.isPresenting

    // B5 - 40
    let backgroundView: UIView
    let fadeView = UIView(frame: containerView.bounds)
    fadeView.backgroundColor = secondViewController.view.backgroundColor

    // B4 - 33
    if isPresenting {
        selectedCellImageViewSnapshot = cellImageSnapshot

        // B5 - 41
        backgroundView = UIView(frame: containerView.bounds)
        backgroundView.addSubview(fadeView)
        fadeView.alpha = 0
    } else {
        backgroundView = firstViewController.view.snapshotView(afterScreenUpdates: true) ?? fadeView
        backgroundView.addSubview(fadeView)
    }

    // B3 - 23
    toView.alpha = 0

    // B4 - 34
    // B5 - 42
    [backgroundView, selectedCellImageViewSnapshot, controllerImageSnapshot].forEach { containerView.addSubview($0) }

    // B3 - 25
    let controllerImageViewRect = secondViewController.locationImageView.convert(secondViewController.locationImageView.bounds, to: window)

    // B4 - 35
    [selectedCellImageViewSnapshot, controllerImageSnapshot].forEach {
        $0.frame = isPresenting ? cellImageViewRect : controllerImageViewRect
    }

    // B4 - 36
    controllerImageSnapshot.alpha = isPresenting ? 0 : 1

    // B4 - 37
    selectedCellImageViewSnapshot.alpha = isPresenting ? 1 : 0

    // B3 - 27
    UIView.animateKeyframes(withDuration: Self.duration, delay: 0, options: .calculationModeCubic, animations: {

        UIView.addKeyframe(withRelativeStartTime: 0, relativeDuration: 1) {
            // B4 - 38
            self.selectedCellImageViewSnapshot.frame = isPresenting ? controllerImageViewRect : self.cellImageViewRect
            controllerImageSnapshot.frame = isPresenting ? controllerImageViewRect : self.cellImageViewRect

            // B5 - 43
            fadeView.alpha = isPresenting ? 1 : 0
        }

        // B4 - 39
        UIView.addKeyframe(withRelativeStartTime: 0, relativeDuration: 0.6) {
            self.selectedCellImageViewSnapshot.alpha = isPresenting ? 0 : 1
            controllerImageSnapshot.alpha = isPresenting ? 1 : 0
        }
    }, completion: { _ in
        // B4 - 39.1
        self.selectedCellImageViewSnapshot.removeFromSuperview()
        controllerImageSnapshot.removeFromSuperview()

        // B5 - 44
        backgroundView.removeFromSuperview()

        // B3 - 30
        ...
    })
}
```

40. 배경과 fade 뷰를 생성한다.
41. 만약 presenting하는 경우, SecondVC에는 아무것도 없기 때문에 하얀색일 것이다. Dismiss하는 경우, 배경은 collection view의 배경의 스냅샷이 외고, fadeView를 위에 올려놓는다. 그래서 애니메이션할 때 fade할 것이다. 
42. 배경 뷰를 배열의 가장 첫번째에 둔다. 이는 transition container view 뷰들 중 가장 뒤에 위치한다는 의미이므로 중요하다.
43. Fade 뷰에 fade 애니메이션을 적용하기 위해 알파값을 바꾼다.
44. 배경을 수퍼 뷰에서 제거한다.

이제 실행시키면 애니메이션이 실행될 때 배경이 잘 보이는 것을 확인할 수 있다. 

![transition3](https://user-images.githubusercontent.com/41438361/126612109-caed7bcb-ba92-488b-85c8-b7b2eab7339c.gif)

이제 텍스트를 애니메이션에 추가할 것이다.

#### Block6: 라벨의 전환을 애니메이션으로 보여주기

라벨은 이미지와 함께 이동하면서 커질것이다.

```swift
...

private let cellImageViewRect: CGRect

// B6 - 45
private let cellLabelRect: CGRect

// B2 - 10
init?(type: PresentationType, firstViewController: FirstViewController, secondViewController: SecondViewController, selectedCellImageViewSnapshot: UIView) {
  
...
```

45. 새로운 `cellLabelRect` 프로퍼티를 생성한다.

```swift
// B2 - 10
init?(type: PresentationType, firstViewController: FirstViewController, secondViewController: SecondViewController, selectedCellImageViewSnapshot: UIView) {
    ...

    // B6 - 46
    self.cellLabelRect = selectedCell.locationLabel.convert(selectedCell.locationLabel.bounds, to: window)
}
```

46. 라벨의 프레임을 윈도우의 프레임으로 변환해서 최근 선언한 프로퍼티에 할당한다.

```swift
// B2 - 13
func animateTransition(using transitionContext: UIViewControllerContextTransitioning) {
    ...

    // B3 - 21
    guard
        let selectedCell = firstViewController.selectedCell,
        ...
        // B6 - 47
        let cellLabelSnapshot = selectedCell.locationLabel.snapshotView(afterScreenUpdates: true)
        else {
            transitionContext.completeTransition(true)
            return
    }

    ...

    // B3 - 23
    toView.alpha = 0

    // B4 - 34
    // B5 - 42
    // B6 - 48
    [backgroundView, selectedCellImageViewSnapshot, controllerImageSnapshot, cellLabelSnapshot].forEach { containerView.addSubview($0) }

    // B3 - 25
    let controllerImageViewRect = secondViewController.locationImageView.convert(secondViewController.locationImageView.bounds, to: window)
    // B6 - 49
    let controllerLabelRect = secondViewController.locationLabel.convert(secondViewController.locationLabel.bounds, to: window)

    ...

    // B6 - 50
    cellLabelSnapshot.frame = isPresenting ? cellLabelRect : controllerLabelRect

    // B3 - 27
    UIView.animateKeyframes(withDuration: Self.duration, delay: 0, options: .calculationModeCubic, animations: {

        UIView.addKeyframe(withRelativeStartTime: 0, relativeDuration: 1) {
            ...

            // B6 - 51
            cellLabelSnapshot.frame = isPresenting ? controllerLabelRect : self.cellLabelRect
        }

        ...
    }, completion: { _ in
        ...

        // B6 - 52
        cellLabelSnapshot.removeFromSuperview()

        ...
    })
}
```

47. 셀의 라벨의 스냅샷을 찍어둔다. 이 스냅샷 뷰는 나중에 전환을 애니메이션을 통해 보여줄 때 사용된다. 
48. `cellLabelSnapshot`을 4번째 요소로 추가해서, 라벨을 animation 컨테이너에 추가한다.
49. SecondVC의 라벨의 사각형을 가져온다. 이는 우리의 셀 포지션에서 라벨을 어떤 포지션으로 전환해야 하는지 알기 위해 필요하다. Dismiss할 때는 반대로 작용한다. 라벨의 초기 포지션을 SecondVC에 잡는다. 
50. 라벨의 초기 포지션을 잡는다. 위에서 말한대로, presenting할 때는 셀의 라벨 사각형의 포지션에서 시작하고, dismissing할 때는 컨트롤러의 라벨 사각형의 포지션으로 잡는다.
51. 이전 단계에서, 시작 포지션을 잡았다. 그리고 이제 끝 포지션을 잡는다. 애니메이션 블럭안에 있기 때문에, 프레임의 변화를 애니메이션으로 보여줄 것이다. 비슷하게, presenting할 때는 컨트롤러의 라벨 사각형을 잡고, 아닐 때는 셀의 라벨 사각형을 잡는다.
52. 항상 그랬듯이, 사용했던 스냅샷을 제거한다.

이제 프로젝트를 다시 실행하면 아래와 같이 라벨이 잘 변환되는 것을 확인할 수 있다.

![transition4](https://user-images.githubusercontent.com/41438361/126614458-cdab2b37-8253-44d0-80dc-8828d3043e3f.gif)

위에서 했던 것처럼 더 많은 것을 확실히 이해하기 위해, 50번째 단계에서 한 내용을 주석 처리하고 실행해보자. 

![transition5](https://user-images.githubusercontent.com/41438361/126614712-8f8c206d-02c0-491c-929b-fd9e11fd0e0a.gif)

주석처리를 하고 실행하면, 라벨의 초기 프레임을 잡아주지 않았기 때문에 왼쪽 상단에서 시작하는 것을 확인할 수 있다.

51번째 단계를 주석처리하고 실행하면, 라벨이 최종적으로 어떤 포지션으로 변환될지 애니메이션에서 포함시키지 않았기 때문에 전환될때 라벨은 애니메이션이 일어나지 않는다.

#### Block7: 닫기 버튼 애니메이션 처리하기

우리는 닫기 버튼이 나타나는 것을 처리할 것이다. 포지션도 움직이지 않을 것인데, SecondVC에서만 닫기 버튼이 보이기 때문이다.

```swift
func animateTransition(using transitionContext: UIViewControllerContextTransitioning) {
    ...

    // B3 - 21
    guard
        let selectedCell = firstViewController.selectedCell,
        ...
        // B7 - 53
        let closeButtonSnapshot = secondViewController.closeButton.snapshotView(afterScreenUpdates: true)
        else {
            transitionContext.completeTransition(true)
            return
    }

    ...

    // B6 - 48
    // B7 - 54
    [backgroundView, selectedCellImageViewSnapshot, controllerImageSnapshot, cellLabelSnapshot, closeButtonSnapshot].forEach { containerView.addSubview($0) }

    ...
    // B6 - 49
    let controllerLabelRect = secondViewController.locationLabel.convert(secondViewController.locationLabel.bounds, to: window)
    // B7 - 55
    let closeButtonRect = secondViewController.closeButton.convert(secondViewController.closeButton.bounds, to: window)

    ...

    // B6 - 50
    cellLabelSnapshot.frame = isPresenting ? cellLabelRect : controllerLabelRect

    // B7 - 56
    closeButtonSnapshot.frame = closeButtonRect
    closeButtonSnapshot.alpha = isPresenting ? 0 : 1

    // B3 - 27
    UIView.animateKeyframes(withDuration: Self.duration, delay: 0, options: .calculationModeCubic, animations: {
        ...
        
        // B4 - 39
        UIView.addKeyframe(withRelativeStartTime: 0, relativeDuration: 0.6) {
            self.selectedCellImageViewSnapshot.alpha = isPresenting ? 0 : 1
            controllerImageSnapshot.alpha = isPresenting ? 1 : 0
        }

        // B7 - 57
        UIView.addKeyframe(withRelativeStartTime: isPresenting ? 0.7 : 0, relativeDuration: 0.3) {
            closeButtonSnapshot.alpha = isPresenting ? 1 : 0
        }
    }, completion: { _ in
        ...
        // B6 - 52
        cellLabelSnapshot.removeFromSuperview()
        // B7 - 58
        closeButtonSnapshot.removeFromSuperview()
    })
}
```

53. 닫기 버튼의 스냅샷 뷰를 만든다. 이 뷰는 애니메이션에 사용될 것이다.
54. Transition Container View에 `closeButtonSnapshot`을 추가한다.
55. 우리는 FirstVC에 닫기 버튼이 없기 때문에 SecondVC에서의 닫기 버튼의 프레임만 가지고 있으면 된다. 그래서 이것은 두번째 화면에서만 가지고 있으면 되는 모든게 된다. 여기서는 단순히 fade in/out만 시키면 된다. 스냅샷을 만들고, 애니메이션을 적용하는 개념은 항상 같다.
56. 이전 단계에서 가져온 프레임을 애니메이션을 통해 닫기 버튼을 보여주기 위해 할당한다. 초기 알파 값을 설정한다. 처음에는 0으로 설정해서 fade in이 될 수 있게 한다. Dismissing할 때는 초기 값을 1로 설정하면 fade out이 될 것이다.
57. 닫기 버튼을 fade in/fade out하기 위해 keyframe 애니메이션을 추가한다. `isPresenting ? 0.7 : 0`이 코드를 확인할 수 있을 것이다. 이는 presenting할 때는 마지막 0.3초 동안 fade in을 해서 이 닫기 버튼을 이미지가 아직 전환되지 않았는데 빨리 보여지는 것을 방지한다. 닫기 버튼이 화면이 아직 준비되지 않았는데 보여지는 것은 예쁘지 않다.
58. 항상 그랬듯이 사용했던 스냅샷을 없앤다.

![transition6](https://user-images.githubusercontent.com/41438361/126616307-cfc4d674-24a9-4715-a0d4-29f3d62d8543.gif)

이제 닫기 버튼이 적절한 시점에 예쁘게 잘 나오는 것을 확인할 수 있다.



dddsss




* 출처
  * https://www.raywenderlich.com/322-custom-uiviewcontroller-transitions-getting-started
  * https://medium.com/@tungfam/custom-uiviewcontroller-transitions-in-swift-d1677e5aa0bf
