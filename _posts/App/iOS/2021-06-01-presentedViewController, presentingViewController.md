---  
layout: post  
title: "[iOS] - presentedViewController, presentingViewController의 차이와 사용"  
subtitle: ""  
categories: app
tags: app-ios 
comments: true  
header-img: img/dev/app/ios/XcodeImg.png

---  
  
> `presentedViewController, presentingViewController의 차이에 대해 보도록 하자.`  

---

핵심만 먼저 말하면 presentedViewController는 지금 ViewController가 **띄우는** ViewController,

presentingViewController는 지금 ViewController를 **띄우는** ViewController다.

즉 ViewController A, B, C가 있다고 하고 A가 `present`로 B를 띄우고, B에서 `present`로 C를 띄운다고 하면

B의 presentedViewController는 C, presentingViewController는 A가 된다.

이제 가장 큰 차이를 알았으니 개념에 대해 좀 더 자세히 보도록 하겠다. 

**presentedViewController**

애플의 [공식문서](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621407-presentedviewcontroller)에 따르면 

presentedViewController는 인스턴스 프로퍼티로, 이(현재) ViewController에 의해 present된 View Controller를 의미한다. 혹은 뷰 컨트롤러 계층에서의
조상 중 한명이 된다.

만약 뷰 컨트롤러를 `present(_:animated:completion:)` 메서드를 이용해서 명시적으로나 암묵적으로나 modal처럼 뷰 컨트롤러를 띄우면, 이 메서드를 호출한 뷰컨트롤러의
이 프로퍼티는 present한 view controller가 된다. 만약 현재 뷰 컨트롤러가 다른 뷰 컨트롤러를 modal로 띄우지 않는다면, 이 프로퍼티의 값은 nil이 된다.

**presentingViewController**

얘도 마찬가지로 인스턴스 프로퍼티로, 이 뷰 컨트롤러를 띄운 뷰 컨트롤러를 의미한다.

위의 애와 반대로, 만약 뷰 컨트롤러를 `present` 메서드를 이용해서 present했을 때, 보여진 뷰 컨트롤러의 이 프로퍼티는 이 뷰 컨트롤러를 띄운 뷰 컨트롤러로 맞춰진다.

만약 뷰 컨트롤러가 modal로 띄워지지 않았지만 조상 중 하나는 띄워졌을 때, 이 프로퍼티는 조상을 띄운 뷰 컨트롤러를 포함한다. 만약 현재 뷰 컨트롤러나 이 조상들중 하나도 modal로 띄워지지 않았다면,
이 프로퍼티의 값은 nil이 된다.

### 내가 이용한 상황

나는 A가 B 뷰컨트롤러를 띄웠는지(즉 B가 화면에 올라와 있던 상태였는지)를 판단해서 앱이 background에서 foreground로 올 때
B가 맨 위의 화면에 노출이 되어 있던 상태였으면 B의 특정 component를 다시 로드하고, 노출이 되어있지 않은 상태였다면 B 컨트롤러 자체를
`present`로 띄우려고 했다. 처음에는 이런 프로퍼티들이 있는지 몰라 막무가내로 A컨트롤러를 dismiss 하고 B 컨트롤러를 present했었다. 
B컨트롤러가 이미 띄워져 있는 상태에서 바로 `present` 메서드로 B 컨트롤러를 띄우면 오류가 발생했기 때문에 위와 같은 방법으로 했었다.

아래와 같이 B 컨트롤러가 띄워져있는지 검사해서 띄워져 있지 않은 경우에만 `present` 메서드로 B 컨트롤러를 띄웠다.

```swift
if controllerA.presentedViewController != nil {
    controllerA.dismiss(animated: false, completion: nil) // controllerB가 띄워져 있었다면 B를 다시 화면에서 없애고 
}

controllerA.present(ControllerB(), animated: false, completion: nil)
```

참고로 위에서 controllerA.dismiss 이렇게 사용했는데 처음에 이부분도 헷갈렸다. 나는 컨트롤러 인스턴스에 dismiss 메서드를 호출하면
해당 컨트롤러가 화면에서 없어지는 줄 알았는데, 개념을 보니 조금 달랐다. 참고로 해당 [공식 문서](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621505-dismiss)다.

`dismiss(animated:completion)` 메서드는 **뷰 컨트롤러에 의해 modal하게 띄워진 뷰 컨트롤러를 버리는** 메서드다.

다른 
