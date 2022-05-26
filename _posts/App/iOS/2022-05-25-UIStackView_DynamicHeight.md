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

**stack view bottom constraint가 없는 경우**

stackView에 bottom constraint를 주지 않은 상태에서 stack view에 버튼 하나를 만들었다. 버튼을 누르면 스택 뷰에 뷰가 추가되도록 설정했다.

```swift
private func setupStackView() {
	stackView = UIStackView()
	stackView.translatesAutoresizingMaskIntoConstraints = false

	stackView.axis = .vertical
	stackView.backgroundColor = UIColor.systemGreen

	view.addSubview(stackView)

	let button = createButton()
	stackView.addArrangedSubview(button)

        stackView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor).isActive = true
	stackView.leadingAnchor.constraint(equalTo: view.leadingAnchor).isActive = true
	stackView.trailingAnchor.constraint(equalTo: view.trailingAnchor).isActive = true
}

// subview 더하는 버튼
private func createButton() -> UIButton {
	let button = UIButton()
	button.translatesAutoresizingMaskIntoConstraints = false

	button.backgroundColor = UIColor.systemYellow
	button.setTitle("arranged subview 추가", for: .normal)

	button.addTarget(self, action: #selector(didTapAddButton), for: .touchUpInside)

	button.heightAnchor.constraint(equalToConstant: 50).isActive = true

	return button
}

// 버튼 터치 시 실행되는 arranged subview 추가 함수
@objc private func didTapAddButton() {
	stackView.addArrangedSubview(createSomeView())
	}

	// 내부에 추가할 뷰
	private func createSomeView() -> UIView {
	let view = UIView()
	view.translatesAutoresizingMaskIntoConstraints = false

	view.backgroundColor = UIColor.systemOrange
	view.layer.borderColor = UIColor.black.cgColor
	view.layer.borderWidth = 1

	view.heightAnchor.constraint(equalToConstant: 50).isActive = true

	return view
}
```

![Simulator Screen Recording - iPhone 12 Pro Max - 2022-05-26 at 10 57 58](https://user-images.githubusercontent.com/41438361/170399740-36604c17-0983-4222-855c-57dbd89cf365.gif)

실행하면 처음에는 버튼 하나의 크기에 맞는 stackview가 있다. 그리고 버튼을 클릭하면 stack view에 뷰들이 새로 추가된다.

<img width="367" alt="image" src="https://user-images.githubusercontent.com/41438361/170400556-52fc5123-4b07-4964-9c5a-58304b7f4dcd.png">

따로 레이아웃을 업데이트를 시켜주거나 레이아웃 관련 함수를 호출하지 않아도 auto layout이 자동으로 stack view의 크기를 내부 컨텐츠의 크기에 맞춘다.

**stack view bottom constraint가 있는 경우**

stack view의 bottom constraint를 뷰 컨트롤러의 바닥과 일치하게 설정했다.

![Simulator Screen Recording - iPhone 12 Pro Max - 2022-05-26 at 11 07 42](https://user-images.githubusercontent.com/41438361/170400814-680487cc-65d9-4b27-939e-1eb8fb5cfad3.gif)

초기 상태는 위에서도 봤듯이 버튼 하나만 있는데, stack view의 bottom constraint를 설정함으로써 내부 버튼 높이와 stack view를 갖게 만들지 않고 내부 버튼의 높이 constraint가 깨지게 되어 확장된 상태로 시작한다.

그리고 버튼을 계속 누르면 subview들이 계속 추가되고, 버튼이 크기가 줄어드는 것을 확인할 수 있다.

## Arranged Subview 제거하기

```swift
	private func setupStackView() {
	stackView = UIStackView()
	stackView.translatesAutoresizingMaskIntoConstraints = false

	stackView.axis = .vertical
	stackView.backgroundColor = UIColor.systemGreen

	view.addSubview(stackView)

	let button = createButton()
	stackView.addArrangedSubview(button)

	// 처음에 10개의 뷰를 추가했다.
	for _ in 1...10 {
	    stackView.addArrangedSubview(createSomeView())
	}

	stackView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor).isActive = true
	stackView.leadingAnchor.constraint(equalTo: view.leadingAnchor).isActive = true
	stackView.trailingAnchor.constraint(equalTo: view.trailingAnchor).isActive = true
}

// subview 더하는 버튼
private func createButton() -> UIButton {
	let button = UIButton()
	button.translatesAutoresizingMaskIntoConstraints = false

	button.backgroundColor = UIColor.systemYellow
	button.setTitle("arranged subview 추가", for: .normal)

	button.addTarget(self, action: #selector(didTapRemoveButton), for: .touchUpInside)

	button.heightAnchor.constraint(equalToConstant: 50).isActive = true

	return button
}

@objc private func didTapRemoveButton() {
	guard let view = stackView.arrangedSubviews.last else {
	    return
	}

	stackView.removeArrangedSubview(view)
	view.removeFromSuperview()
}

// 내부에 추가할 뷰
private func createSomeView() -> UIView {
	let view = UIView()
	view.translatesAutoresizingMaskIntoConstraints = false

	view.backgroundColor = UIColor.systemOrange
	view.layer.borderColor = UIColor.black.cgColor
	view.layer.borderWidth = 1

	view.heightAnchor.constraint(equalToConstant: 50).isActive = true

	return view
}
```

![Simulator Screen Recording - iPhone 12 Pro Max - 2022-05-26 at 12 50 24](https://user-images.githubusercontent.com/41438361/170412307-b014679d-6c42-4db1-b24d-896f34b8f8ee.gif)

삭제하는 것도 별반 다르지 않다. bottom constraint를 지정하지 않으면 자동으로 뷰가 삭제될때마다 내부 컨텐츠에 맞게 stack view의 크기가 맞춰진다.

<img width="498" alt="image" src="https://user-images.githubusercontent.com/41438361/170412488-57581fd5-5569-4d03-897c-87ffabcb4009.png">

bottom constraint가 있으면 위에서도 봤지만 내부에 배치된 뷰의 높이 constraint가 깨지게 된다.

![Simulator Screen Recording - iPhone 12 Pro Max - 2022-05-26 at 12 55 08](https://user-images.githubusercontent.com/41438361/170412762-c65fc5c5-9d1d-4047-94ff-479ec9fd2925.gif)

### removeFromSuperView와 removeArrangedSubview

위 코드에서는 arranged subviews에서 뷰를 제거할 때 `removeArrangedSubview`와 `removeFromSuperview`를 동시에 호출하고 있다. 이 둘을 같이 호출해야 하는 이유는 무엇일까?

사실 `removeFromSuperview`만 하면 충분하다. 공식문서에서는 뷰의 `removeFromSuperview` 메서드를 호출하면 stack view가 해당 뷰를 arrangedSubview 배열에서 제거한다고 했기 때문에, 굳이 `removeArrangedSubview`를 같이 쓰는 것은 불필요하다.

관련된 stackoverflow 글이 있으니 참고하는 것도 좋다.

https://stackoverflow.com/questions/37525706/uistackview-is-it-really-necessary-to-call-both-removefromsuperview-and-remove

## Stack view를 다른 뷰 안에 넣었을 때 확장, 축소하기

stack view를 특정 뷰(컨테이너 뷰라고 부르겠다) 안에 넣는다고 해보자. 뷰의 구성은 아래와 같이 구성할 것이다.

<img width="452" alt="image" src="https://user-images.githubusercontent.com/41438361/170415030-c90f2b15-8ed4-43d1-bf82-1ccdeb9ae6d0.png">

결론부터 얘기해서 스택 뷰에 맞게 컨테이너 뷰의 높이를 조정하려면 

1. 컨테이너 뷰에 bottom constriant를 두지 않는다. (컨테이너 뷰의 높이가 모호한 상태)
2. 스택 뷰에 bottom constraint를 두지 않는다. (stack view의 높이가 내부 컨텐츠에 맞게 조정)
3. 스택 뷰의 height constraint가 컨테이너 뷰의 height와 같도록 한다. (내부 컨텐츠에 맞게 조정된 stack view의 높이와 컨테이너 뷰의 높이와 같게 조정함으로써 컨테이너 뷰의 높이를 맞춘다.)

혹은

1. 컨테이너 뷰에 bottom constriant를 두지 않는다. (컨테이너 뷰의 높이가 모호한 상태)
2. 스택 뷰에 bottom constraint를 컨테이너 뷰의 bottom constraint와 같게 한다. (stack view의 높이는 여전히 동적으로 조절된다. 컨테이너 뷰의 높이가 모호하기 때문에 스택 뷰의 높이를 명시적으로 지정하지 않은 상태이기 때문이다.)

```swift
private func setupViews() {
	view.backgroundColor = .white
	setupContainerView()
	setupStackView()
}

// 파란색 컨테이너 뷰 생성
private func setupContainerView() {
	containerView = UIView()
	containerView.translatesAutoresizingMaskIntoConstraints = false

	containerView.backgroundColor = UIColor.systemBlue

	view.addSubview(containerView)
	
	// 컨테이너 뷰에 bottom constraint를 두지 않는다.
	containerView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor).isActive = true
	containerView.leadingAnchor.constraint(equalTo: view.leadingAnchor).isActive = true
	containerView.trailingAnchor.constraint(equalTo: view.trailingAnchor).isActive = true
}

// 주황색 스택 뷰 생성
private func setupStackView() {
	stackView = UIStackView()
	stackView.translatesAutoresizingMaskIntoConstraints = false

	containerView.addSubview(stackView)

	stackView.axis = .vertical
	stackView.backgroundColor = UIColor.systemOrange
	
	// 내부에 추가하는 뷰 
	let view = createSomeView()
	stackView.addArrangedSubview(view)
	
	// stack view에 bottom constraint를 두지 않는다. 외부 컨테이너 뷰의 높이를 스택 뷰의 높이와 같게 설정한다.
	stackView.topAnchor.constraint(equalTo: containerView.topAnchor).isActive = true
	stackView.leadingAnchor.constraint(equalTo: containerView.leadingAnchor).isActive = true
	stackView.trailingAnchor.constraint(equalTo: containerView.trailingAnchor).isActive = true
	stackView.heightAnchor.constraint(equalTo: containerView.heightAnchor).isActive = true
}
```

<img width="518" alt="image" src="https://user-images.githubusercontent.com/41438361/170416752-568b0dd0-7f7e-4315-b2e7-e9e232e1f45e.png">

### Stack view 높이 동적으로 조정, 변경

위에서 해본 것과 똑같다. 스택 뷰가 컨테이너 뷰에 포함됐느냐 아니냐의 차이만 있을 뿐이다.

```swift
private func setupViews() {
	view.backgroundColor = .white
	setupContainerView()
	setupStackView()
}

private func setupContainerView() {
	containerView = UIView()
	containerView.translatesAutoresizingMaskIntoConstraints = false

	containerView.backgroundColor = UIColor.systemBlue

	view.addSubview(containerView)

	containerView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor).isActive = true
	containerView.leadingAnchor.constraint(equalTo: view.leadingAnchor).isActive = true
	containerView.trailingAnchor.constraint(equalTo: view.trailingAnchor).isActive = true
}

private func setupStackView() {
	stackView = UIStackView()
	stackView.translatesAutoresizingMaskIntoConstraints = false

	containerView.addSubview(stackView)

	stackView.axis = .vertical
	stackView.backgroundColor = UIColor.systemOrange

	let button = createButton()
	stackView.addArrangedSubview(button)

	for _ in 1...10 {
	    stackView.addArrangedSubview(createSomeView())
	}

	stackView.topAnchor.constraint(equalTo: containerView.topAnchor).isActive = true
	stackView.leadingAnchor.constraint(equalTo: containerView.leadingAnchor).isActive = true
	stackView.trailingAnchor.constraint(equalTo: containerView.trailingAnchor).isActive = true
	stackView.bottomAnchor.constraint(equalTo: containerView.bottomAnchor).isActive = true
}

// subview 더하는 버튼
private func createButton() -> UIButton {
	let button = UIButton()
	button.translatesAutoresizingMaskIntoConstraints = false

	button.backgroundColor = UIColor.systemYellow
	button.setTitle("arranged subview 삭제", for: .normal)

	button.addTarget(self, action: #selector(didTapRemoveButton), for: .touchUpInside)

	button.heightAnchor.constraint(equalToConstant: 50).isActive = true

	return button
}

@objc private func didTapRemoveButton() {
	guard let view = stackView.arrangedSubviews.last else {
	    return
	}

	view.removeFromSuperview()
}

// 내부에 추가할 뷰
private func createSomeView() -> UIView {
	let view = UIView()
	view.translatesAutoresizingMaskIntoConstraints = false

	view.backgroundColor = UIColor.systemRed
	view.layer.borderColor = UIColor.black.cgColor
	view.layer.borderWidth = 1

	view.heightAnchor.constraint(equalToConstant: 50).isActive = true

	return view
}
```

![Simulator Screen Recording - iPhone 12 Pro Max - 2022-05-26 at 13 48 11](https://user-images.githubusercontent.com/41438361/170418110-fd67ef17-c89a-49d1-be9e-3fdc80bb0132.gif)

<img width="471" alt="image" src="https://user-images.githubusercontent.com/41438361/170418273-ce44ab4b-d676-4caa-8c52-c906d6a4a5ba.png">

아래에 UIViewController의 전체 코드를 첨부한다.

```swift
import UIKit

class ViewController: UIViewController {
    
    private var containerView: UIView!
    private var stackView: UIStackView!
    
    // MARK: - override
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
        setupViews()
    }
}

// MARK: - setup
extension ViewController {
    private func setupViews() {
        view.backgroundColor = .white
        setupContainerView()
        setupStackView()
    }
    
    private func setupContainerView() {
        containerView = UIView()
        containerView.translatesAutoresizingMaskIntoConstraints = false
        
        containerView.backgroundColor = UIColor.systemBlue
        
        view.addSubview(containerView)
        
        containerView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor).isActive = true
        containerView.leadingAnchor.constraint(equalTo: view.leadingAnchor).isActive = true
        containerView.trailingAnchor.constraint(equalTo: view.trailingAnchor).isActive = true
    }
}

// MARK: - private
extension ViewController {
    private func setupStackView() {
        stackView = UIStackView()
        stackView.translatesAutoresizingMaskIntoConstraints = false
        
        containerView.addSubview(stackView)
        
        stackView.axis = .vertical
        stackView.backgroundColor = UIColor.systemOrange
        
        let button = createButton()
        stackView.addArrangedSubview(button)
        
        for _ in 1...10 {
            stackView.addArrangedSubview(createSomeView())
        }
        
        stackView.topAnchor.constraint(equalTo: containerView.topAnchor).isActive = true
        stackView.leadingAnchor.constraint(equalTo: containerView.leadingAnchor).isActive = true
        stackView.trailingAnchor.constraint(equalTo: containerView.trailingAnchor).isActive = true
        stackView.bottomAnchor.constraint(equalTo: containerView.bottomAnchor).isActive = true
    }
    
    private func createButton() -> UIButton {
        let button = UIButton()
        button.translatesAutoresizingMaskIntoConstraints = false
        
        button.backgroundColor = UIColor.systemYellow
        button.setTitle("arranged subview 삭제", for: .normal)
        
        button.addTarget(self, action: #selector(didTapRemoveButton), for: .touchUpInside)
        
        button.heightAnchor.constraint(equalToConstant: 50).isActive = true
        
        return button
    }
    
    @objc private func didTapRemoveButton() {
        guard let view = stackView.arrangedSubviews.last else {
            return
        }

        view.removeFromSuperview()
    }
    
    private func createSomeView() -> UIView {
        let view = UIView()
        view.translatesAutoresizingMaskIntoConstraints = false
        
        view.backgroundColor = UIColor.systemRed
        view.layer.borderColor = UIColor.black.cgColor
        view.layer.borderWidth = 1
        
        view.heightAnchor.constraint(equalToConstant: 50).isActive = true
        
        return view
    }
}
```

* 참조
* https://developer.apple.com/documentation/uikit/uistackview
* https://stackoverflow.com/questions/53532797/why-uistackview-resize-arranged-subviews
