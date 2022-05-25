---  
layout: post  
title: "[iOS] - UIStackView dynamic height, 동적 높이 조절"  
subtitle: ""  
categories: app
tags: app-ios UIStackView
comments: true  
header-img: 

---  
  
> `UIStackView의 동적 높이의 동작에 대해 알아보자.`  

---

UIStackView는 참 사용하기 간단한 것 같으면서도 막상 사용하다보면 레이아웃과 관련해 이게 왜 안되지?? 하는 상황이 너무 자주 발생하는 것 같다. 분명 완벽하게 constraint를 설정한 것 같았는데 시뮬레이터로
실행해보면 이해할 수 없는 레이아웃이 보이는 경우도 있다. 이번에는 UIStackView의 높이가 동적으로 바뀌는 동작에 대해 자세히 정리해보려고 한다. 더불어 UIStackView의 내부 subview들(arrangedSubviews)을 
UIStackView에 추가하고 삭제하는 걸 보려고 한다.

# UIStackView

<img width="727" alt="image" src="https://user-images.githubusercontent.com/41438361/170193414-90ec86aa-73fa-4d42-8d63-54967588b0f9.png">

UIStackView는 **열이나 행으로 뷰들의 컬렉션을 배치하는 사용하기 쉽고 효율적인 인터페이스**다. 행으로 뷰들을 배열하면 가로로 일렬로 배열하고, 열로 뷰들을 배열하면 세로로 길게 배치될 것이다.

<img width="820" alt="image" src="https://user-images.githubusercontent.com/41438361/170195797-ef0fe086-d15b-42db-9449-7bde2104c6b0.png">

StackView는 auto layout과 함께 사용할 때 굉장히 유용하다. Autolayout을 사용하면 기기마다 다른 설정(화면 크기, 비율 등)에 동적으로 인터페이스를 적용할 수 있는데, stackView가 이런 이점을 활용하기 좋은 뷰라는 뜻이다.


## arrangedSubviews

StackView는 `arrangedSubviews` 프로퍼티에 있는 모든 뷰의 레이아웃을 관리한다. 

**`arrangedSubviews`는 stack view에 배열된 뷰들의 리스트다.**

<img width="758" alt="image" src="https://user-images.githubusercontent.com/41438361/170196565-53221687-0394-4c60-a1cf-df0571f8a5c1.png">

`arrangedSubviews`안에 있는 뷰들은 stack view에 설정된 축에 따라 배열 내의 순서대로 stackView에 배열된다. 

StackView의 더 자세한 레이아웃은 `axis`, `distribution`, `alignment`, `spacing` 등의 프로퍼티에 의해 결정된다.

<img width="498" alt="image" src="https://user-images.githubusercontent.com/41438361/170199150-c32249f3-e601-42cc-b591-68cee436e45a.png">

### arrangedSubviews vs subviews

UIStackView는 UIView를 상속한다. 그런데 모든 UIView는 기본적으로 `subviews`라는 바로 하위의 subview들의 리스트 프로퍼티를 가지고 있다.

공식문서에 의하면 `arrangedSubviews`는 `subviews`의 부분집합이라고 한다. 다만 정확히 뭐가 다른지는 나오지 않고 여기까지만 나와 있다. 하여간 `arrangedSubviews`가 `subviews`의 부분집합이기 때문에 다음의 것들이 가능하다.

* arranged subviews에 view 추가 - `addArrangedSubview(_:)` 메서드 호출 : 뷰가 준비되지 않았다면 뷰를 subview로 추가한다. 
* arranged subviews에서 view 제거 - `removeFromSuperview()` 메서드 호출 : 뷰를 `arrangedSubview` 배열에서 제거한다. `removeArrangedSubview(_:)`와 다르다는 것에 유의하자.

## StackView와 Auto Layout

**Stack view는 arranged view들의 위치와 크기를 정하는데 Auto Layout을 사용한다.** 

### 순서

위에서 stack view는 arranged subview 리스트의 순서대로 뷰를 배열한다고 했었다. 따라서 만약 가로로 배열하는 stack에서는 첫 번째 arranged view의 leading edge가 stack view의 leading edge와 같고, 마지막 arranged view의 trailing edgerk stack view의 trailing edge와 같다는 소리다.

<img width="582" alt="image" src="https://user-images.githubusercontent.com/41438361/170199956-d392d2c3-468f-4687-a823-c581fbc29d25.png">

그림처럼 `arrangedSubviews` 내의 뷰들의 순서대로 stackView에 배열된다.

### Distribution

UIStackView의 Distribution은 열거형이고, **stack view에 배치된 뷰들의 크기와 위치를 정의한다.**


# 코드

## UIStackView 생성

ViewController의 `viewDidLoad()` 메서드에서 아래의 코드가 실행되게 했다. stackView 자체에도 auto layout을 사용해서 화면에 꽉 찬 stack view를 만들었다.

```swift
stackView = UIStackView()
stackView.translatesAutoresizingMaskIntoConstraints = false

// subview들이 배열될 방향을 정해준다.
stackView.axis = .vertical
stackView.backgroundColor = UIColor.systemGreen

view.addSubview(stackView)

stackView.topAnchor.constraint(equalTo: view.topAnchor).isActive = true
stackView.leadingAnchor.constraint(equalTo: view.leadingAnchor).isActive = true
stackView.trailingAnchor.constraint(equalTo: view.trailingAnchor).isActive = true
stackView.bottomAnchor.constraint(equalTo: view.bottomAnchor).isActive = true
```

![image](https://user-images.githubusercontent.com/41438361/170203428-7ba96ce5-b1f2-41ca-8efe-7bf51c366bd1.png)

## Empty Stack view

현재 stack view에는 아무런 뷰도 배치되어 있지 않다. stack view는 arranged subview들의 크기와 위치를 정하는데 autolayout을 사용하니 아무런 뷰가 없을 때는 stack view의 높이가 0이 되지 않을까?
stackView의 bottomAnchor constraint를 없애고 실행해봤다.

![image](https://user-images.githubusercontent.com/41438361/170204522-71538efe-a626-4a3f-b34d-e20d5cc9ed69.png)

stack view의 top, leading, trailing constraint만 설정하고 stack view에 arranged subview가 없을 때는 내부 뷰가 없기 때문에 stackView가 줄어들어 화면에 나타나지 않는 것처럼 보인다.

## Arranged Subview를 가지고 있는 Stack view

위 상태에서 높이 100짜리 파란색 뷰를 스택 뷰에 추가했다.

```swift
stackView = UIStackView()
stackView.translatesAutoresizingMaskIntoConstraints = false

stackView.axis = .vertical
stackView.backgroundColor = UIColor.systemGreen

view.addSubview(stackView)

// 새로 높이 100짜리 뷰를 추가했다.
let tmpView = UIView()
tmpView.translatesAutoresizingMaskIntoConstraints = false
tmpView.backgroundColor = UIColor.systemBlue
tmpView.heightAnchor.constraint(equalToConstant: 100).isActive = true
stackView.addArrangedSubview(tmpView)

stackView.topAnchor.constraint(equalTo: view.topAnchor).isActive = true
stackView.leadingAnchor.constraint(equalTo: view.leadingAnchor).isActive = true
stackView.trailingAnchor.constraint(equalTo: view.trailingAnchor).isActive = true
```

![image](https://user-images.githubusercontent.com/41438361/170206667-b67c2bce-6554-48fd-adbd-6fc40cd9819f.png)
<img width="745" alt="image" src="https://user-images.githubusercontent.com/41438361/170206768-3e01bfa3-316a-45b6-9266-c4535ae60eac.png">

초록색이 stack view고, 파란색이 stack view애 arranged subview로 추가된 뷰다. stack view에 bottom constraint를 지정하지 않으면 auto layout으로 자동으로 내부 컨텐츠의 크기에 자신의 크기를 맞춘다.

## Subview expands

그렇다고 stack view의 constraint를 막 지정해도 되는 것은 아니다. stack view가 화면에 꽉 차도록 constraint를 설정하고, arranged view로 높이 100짜리 파란색 뷰를 추가하고 다시 실행했다.

```swift
stackView = UIStackView()
stackView.translatesAutoresizingMaskIntoConstraints = false

stackView.axis = .vertical
stackView.backgroundColor = UIColor.systemGreen

view.addSubview(stackView)

let tmpView = UIView()
tmpView.translatesAutoresizingMaskIntoConstraints = false
tmpView.backgroundColor = UIColor.systemBlue
tmpView.heightAnchor.constraint(equalToConstant: 100).isActive = true
stackView.addArrangedSubview(tmpView)

stackView.topAnchor.constraint(equalTo: view.topAnchor).isActive = true
stackView.leadingAnchor.constraint(equalTo: view.leadingAnchor).isActive = true
stackView.trailingAnchor.constraint(equalTo: view.trailingAnchor).isActive = true

// 위 코드에서 이 부분이 추가됐다
stackView.bottomAnchor.constraint(equalTo: view.bottomAnchor).isActive = true
```

![image](https://user-images.githubusercontent.com/41438361/170209734-9cb7c62a-ed88-4d05-b5ae-298bcee6bfe9.png)

**Stack view의 arranged subview가 높이 100으로 설정된 constraint를 깨고 stack view를 완전히 채운 것을 볼 수 있다.** 왜 이런 현상이 나타났을까?

* stack view는 내부 배열된 뷰의 크기에 맞게 자신의 크기를 조정한다.
* 현재 stack view의 constraint는 화면을 꽉 채우게 설정되어 있다.

=> 내부 배치된 뷰의 크기에 맞추려면 stack view의 높이가 100이 되어야 한다 vs 근데 stack view의 constraint는 화면을 꽉 채우게 되어 있다

가 충돌해서 이런 현상이 발생하는 것이다. 위 코드를 실행하면 콘솔에 아래와 같이 나온다.

```
2022-05-25 16:42:45.238151+0900 Test[88134:19339134] [LayoutConstraints] Unable to simultaneously satisfy constraints.
	Probably at least one of the constraints in the following list is one you don't want. 
	Try this: 
		(1) look at each constraint and try to figure out which you don't expect; 
		(2) find the code that added the unwanted constraint or constraints and fix it. 
(
    "<NSLayoutConstraint:0x6000006ced00 UIView:0x7fa5fe006d90.height == 100   (active)>",
    "<NSLayoutConstraint:0x6000006cec10 V:|-(0)-[UIStackView:0x7fa5fe006200]   (active, names: '|':UIView:0x7fa5fe004380 )>",
    "<NSLayoutConstraint:0x6000006fff20 UIStackView:0x7fa5fe006200.bottom == UIView:0x7fa5fe004380.bottom   (active)>",
    "<NSLayoutConstraint:0x6000006d6120 'UISV-canvas-connection' UIStackView:0x7fa5fe006200.top == UIView:0x7fa5fe006d90.top   (active)>",
    "<NSLayoutConstraint:0x6000006d60d0 'UISV-canvas-connection' V:[UIView:0x7fa5fe006d90]-(0)-|   (active, names: '|':UIStackView:0x7fa5fe006200 )>",
    "<NSLayoutConstraint:0x6000006d6440 'UIView-Encapsulated-Layout-Height' UIView:0x7fa5fe004380.height == 926   (active)>"
)

Will attempt to recover by breaking constraint 
<NSLayoutConstraint:0x6000006ced00 UIView:0x7fa5fe006d90.height == 100   (active)>
```

로그를 보면 UIView의 height == 100 constraint를 깼다는 걸 확인할 수 있다. 위의 조건이 상충되면서 내부 arranged subview를 stack view의 크기에 맞춘 것이다. 이 현상과 관련된 [stackOverflow](https://stackoverflow.com/questions/53532797/why-uistackview-resize-arranged-subviews) 글도 있으니 참고하면 될 것 같다.

즉 stack view는 

* **높이 제약이 있다 => subview를 stack view의 높이에 맞게 조정한다.**
* **높이 제약이 없다 => subview의 높이를 이용해서 stack view의 높이를 조정한다.**

## Dynamic Stack view height

이제 stack view가 내부 컨텐츠에 맞게 자신의 크기를 조정할 수 있다는 점을 알았다. 그렇다면 stack view에 subview들이 계속 추가되거나 삭제되면 stack view의 높이도 자동으로 늘어나거나 줄어들까?

## Arranged Subview 추가하기


* 참조
* https://developer.apple.com/documentation/uikit/uistackview
* https://stackoverflow.com/questions/53532797/why-uistackview-resize-arranged-subviews
