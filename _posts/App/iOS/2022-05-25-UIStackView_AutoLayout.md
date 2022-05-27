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

UIStackView의 Distribution은 열거형이고, **stack view에 배치된 뷰들의 크기와 위치를 정의한다.** 이 distribution은 다섯 가지가 있다.

* fill
* fillEqually
* fillProportionally
* equalSpacing
* equalCentering

#### fill

Arranged view의 크기를 재조정해서 stack view의 축의 방향으로 arranged view의 크기를 맞게 채운다. 

* arranged view의 크기 < stack view의 높이/너비 : arranged view의 compresseion resistance priority에 따라 뷰의 크기를 줄인다.
* arranged view의 크기 > stack view의 높이/너비 : arranged view의 hugging priority에 따라 뷰의 크기를 늘린다.

만약 뷰들의 크기를 재조정하는데 constraint나 priority에서 모호함이 있다면 stack view는 `arrangedSubviews` 배열의 뷰들의 순서에 따라 배치된 뷰들의 크기를 재조정한다.

높이 200짜리 stackview에 높이 50짜리 초록색 view를 arranged subview로 추가했다. 높이 50짜리 뷰의 크기가 재조정되어 stack view의 높이 200에 맞게 조정되었다.

![image](https://user-images.githubusercontent.com/41438361/170420736-90ea3625-dfd1-4617-8019-8133c9f96d32.png)

<img width="933" alt="image" src="https://user-images.githubusercontent.com/41438361/170420981-7e873421-7f27-4ed5-94f8-bbd50cfaeb8b.png">

이번에는 높이 50짜리 초록색 뷰 하나와, 높이 50짜리 파란색 뷰 하나를 높이 200짜리 stackView에 차례로 arranged subview로 추가해봤다. 

```swift
let v1 = createSomeView(color: UIColor.systemGreen)
let v2 = createSomeView(color: UIColor.systemBlue)

v1.tag = 1
v2.tag = 2

stackView.addArrangedSubview(v1)
stackView.addArrangedSubview(v2)
```

두 뷰를 높이를 합쳐도 stack view의 높이보다 작으니 두 뷰의 hugging priority에 따라 뷰의 크기를 늘릴 것이다. 그런데 따로 hugging priority를 변경하지 않았으므로 두 뷰의 hugging priority는 현재 같은 상태다. 따라서 priority에 모호함이 있는 상태이므로 stack view의 `arrangedSubviews` 배열의 인덱스에 따라 뷰의 크기가 재조정된다.

![image](https://user-images.githubusercontent.com/41438361/170421286-9a4bb81d-f493-4f11-ad9b-f667abf26b57.png)


#### fillEqually

fill과 같은데, arranged view들이 같은 크기로 크기가 재조정되는 것이다.

![image](https://user-images.githubusercontent.com/41438361/170432841-067eee9f-ab8c-4fbd-981d-f1574dcc3b9a.png)

#### fillProportionally

subview들이 intrinsic content size에 비율에 따라 크기가 재조정된다.

#### equalSpacing

뷰 간의 공간을 띄워 스택 뷰를 채우는 건데, 뷰 사이사이 공간을 모두 같은 크기로 설정하는 것이다. 주황색이 stack view이고, 초록색, 파란색이 arranged subview로 추가된 뷰다.

<img width="583" alt="image" src="https://user-images.githubusercontent.com/41438361/170433397-d8c0c1c3-4816-4e4b-9a3e-86c8e41370aa.png">

### Stack view positioning, sizing

Stack view가 auto layout을 직접적으로 사용하지 않고서도 컨텐츠를 배치할 수 있게 하지만, stack view를 위치시키기 위해 auto layout을 사용하긴 해야 한다. 일반적으로 stack view의 최소 두 edge를 특정 edge에 고정시켜야 한다는 것이다. 만약 **두 edge 외에 추가적인 constraint를 지정하지 않는다면 시스템에서 stack view의 크기를 컨텐츠에 맞게 조정**한다.

* 스택의 axis 방향 : 축 방향의 크기는 내부에 배치된 뷰들의 크기 + 뷰 사이의 space와 같다.
* 스택의 axis에 수직인 방향 : 가장 큰 arranged view의 크기와 같다.

만약 stack view읜 `isLayoutMarginsRelativeArrangement` 프로퍼티가 true라면, stack view의 사이즈는 마진을 완전히 채우게 되어있다.

**Stack view는 arranged view의 `intrinsicContentSize` 프로퍼티를 사용해서 스택의 축 방향으로의 크기를 계산한다.** 이게 무슨 말인지 보자.

아래 코드에서는 stack view의 위, 왼쪽 edge를 고정시키고, 내부에 `intrinsicContentSize`가 (100, 50)인 파란색 UIView를 추가했다.

```swift
class IntrinsicContentSizeView: UIView {
	let myIntrinsicContentSize: CGSize
	
	// intrinsicContentSize를 override 해서 내가 설정한 intrinsicContentSize를 반환한다.
	override var intrinsicContentSize: CGSize {
	    myIntrinsicContentSize
	}

	init(myIntrinsicContentSize: CGSize) {
	    self.myIntrinsicContentSize = myIntrinsicContentSize

	    super.init(frame: .zero)
	}

	required init?(coder: NSCoder) {
	    fatalError("init(coder:) has not been implemented")
	}
}

private func setupStackView() {
	stackView = UIStackView()
	stackView.translatesAutoresizingMaskIntoConstraints = false

	view.addSubview(stackView)

	stackView.axis = .vertical
	stackView.backgroundColor = UIColor.systemOrange

	let subview = IntrinsicContentSizeView(myIntrinsicContentSize: CGSize(width: 100, height: 50))
	subview.backgroundColor = UIColor.systemBlue
	stackView.addArrangedSubview(subview)

	stackView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor).isActive = true
	stackView.leadingAnchor.constraint(equalTo: view.leadingAnchor).isActive = true
}
```

![image](https://user-images.githubusercontent.com/41438361/170436475-fc03a8a0-f1ab-4519-9396-b53c582c8dd3.png)
<img width="419" alt="image" src="https://user-images.githubusercontent.com/41438361/170436525-bc82a68c-a1de-4df9-978c-3c10ab4a27b5.png">

보면 stack view의 크기가 내부 설정한 뷰 (100x50)에 맞춰서 설정된 것을 확인할 수 있다. 여기에서 stack view에 layoutMargin을 설정하고 실행하면 아래와 같이 나오는데, 아까와 같다.

```swift
stackView.layoutMargins = UIEdgeInsets(top: 10, left: 10, bottom: 10, right: 10)
```

![image](https://user-images.githubusercontent.com/41438361/170437660-01245002-d259-4942-bb93-52707ccf0903.png)


이제 stack view의 `isLayoutMarginsRelativeArrangement` 프로퍼티를 true로 설정하면 아래와 같이 나온다. 

```swift
stackView.layoutMargins = UIEdgeInsets(top: 10, left: 10, bottom: 10, right: 10)
stackView.isLayoutMarginsRelativeArrangement = true
```

![image](https://user-images.githubusercontent.com/41438361/170437875-f9371288-4781-48c1-b00f-809304a4ff27.png)

마진을 상하좌우로 10씩 줬는데, stack view가 마진을 채워서 크기가 설정된 것을 확인할 수 있다.

### Stack view 컨텐츠 동적으로 수정하기

**Stack view는 1. `arrangedSubviews` 배열에 뷰가 추가되거나, 삭제되거나, 삽입될 때, 2. arranged subview 중 하나라도 `isHidden` 프로퍼티가 변경될 때 자동으로 레이아웃을 업데이트한다.**

```swift
// Appears to remove the first arranged view from the stack.
// The view is still inside the stack, it's just no longer visible, and no longer contributes to the layout.
let firstView = stackView.arrangedSubviews[0]
firstView.isHidden = true
```

**Stack view의 어떤 프로퍼티라도 변하면 자동으로 이에 반응해서 업데이트한다.**

```swift
// Toggle between a vertical and horizontal stack
if stackView.axis == .Horizontal {
    stackView.axis = .Vertical
}
else {
    stackView.axis = .Horizontal
}
```

이런 변화들을 애니메이션 블럭 안에 넣어서 애니메이션으로 보여줄 수도 있다.

```swift
// Animates removing the first item in the stack.
UIView.animateWithDuration(0.25) { () -> Void in
    let firstView = stackView.arrangedSubviews[0]
    firstView.isHidden = true
}
```

근데 좀 웃긴건 `isHidden` 프로퍼티를 toggle 시키면 애니메이션이 좀 웃기게 보인다는 것이다.

![Simulator Screen Recording - iPhone 12 Pro Max - 2022-05-27 at 11 11 18](https://user-images.githubusercontent.com/41438361/170614838-f46494b3-b5e2-46ac-b8c9-8f4acf3af74e.gif)

그런데 arranged subview 가 여러개일 때는 굉장히 매끄럽게 동작했다.

![Simulator Screen Recording - iPhone 12 Pro Max - 2022-05-27 at 11 48 48](https://user-images.githubusercontent.com/41438361/170619010-22253829-137b-4c4d-8f9e-363518078175.gif)


이걸 좀 매끄럽게 나타나게 하고 싶어서 `addArrangedSubview`를 `animateWithDuration` 내에서 호출했는데, 애니메이션이 적용되지 않았다. [관련 글](https://stackoverflow.com/questions/32513377/should-i-enclose-addarrangedsubview-to-the-animation-block-or-not)을 찾아보니, **subview를 추가하는 것은 animateWithDuration 함수에서 애니메이션을 적용할 수 없다고 한다.**

그래서 아래와 같이 `arrangedSubview`에 추가할 뷰의 alpha 값을 처음에 0으로 설정하고, 애니메이션 블럭 안에서 alpha 값을 1로 설정해줘서 애니메이션이 매끄럽게 나타나게 하는 방법으로 구현해야 한다고 한다.

```swift
someSubview.alpha = 0.0
horizontalStackView.addArrangedSubview(someSubview)

UIView.animateWithDuration(0.25) { () -> Void in
    someSubview.alpha = 1.0
}
```

![Simulator Screen Recording - iPhone 12 Pro Max - 2022-05-27 at 11 40 12](https://user-images.githubusercontent.com/41438361/170617997-2fadd06b-51da-4093-9c42-655d136d3b29.gif)

이건 애니메이션이 매끄럽게 나오기는 하는데 색이 노란색이었다가 파란색으로 바뀌는 듯해서 별로 맘에 들지 않는다.

결론적으로는 `isHidden` 프로퍼티를 toggle하는 방법으로 애니메이션을 설정하는 것이 좋아보인다.

### Intrinsic size & priority

Stack view는 내부에 배치된 뷰들의 크기를 계산할 때 intrinsic content size를 이용한다고 했다. 만약 stack view의 크기에 맞게 내부 뷰들의 크기를 조정해야 하는 경우에서 어떤 뷰의 크기를 조정해야 할 지 결정하기 위해서는 우선순위를 따져야 한다. 이 우선순위를 content hugging priority나 compresseion resistance priority를 통해 설정할 수 있는데, auto layout을 사용할 때 주의해야 할 점이 있다.

[관련 stackOverflow 글](https://stackoverflow.com/questions/62824788/uistackviews-subviews-have-intrinsic-size-yet-still-ignore-content-hugging-pri)

Content Hugging priority는 Constraint 우선순위와 같지 않다. 그리고 constraint는 intrinsic Content size를 결정하지 않는다. Content Huggin priority/Content Compression resistance priority는 intrinsic content size에 기반하고 있다. 그래서 우선순위를 설정할 때 우리가 할 수 있는 방법은 두 가지가 있다.

1. Constraint에 우선순위 부여하기
2. 뷰에 intrinsic content size를 주고 hugging priority를 설정하기

**첫 번째 방법**

```swift
let view1 = UIView()
shelfContentView.addArrangedSubview(view1)
view1.backgroundColor = .systemRed

let wc1 = view1.widthAnchor.constraint(equalToConstant: 100)
wc1.isActive = true
wc1.priority = .defaultHigh

let view2 = UIView()
shelfContentView.addArrangedSubview(view2)
view2.backgroundColor = .systemPurple

let wc2 = view2.widthAnchor.constraint(equalToConstant: 100)
wc2.isActive = true
wc2.priority = .defaultLow

let view3 = UIView( )
shelfContentView.addArrangedSubview(view3)
view3.backgroundColor = .systemBlue

let wc3 = view3.widthAnchor.constraint(equalToConstant: 100)
wc3.isActive = true
wc3.priority = .defaultHigh
```

**두 번째 방법**

```swift
class IntrinsicView: UIView {
    var myIntrinsicSize: CGSize = .zero
    override var intrinsicContentSize: CGSize {
        return myIntrinsicSize
    }
}
        
let view1 = IntrinsicView()
shelfContentView.addArrangedSubview(view1)
view1.backgroundColor = .systemRed

view1.myIntrinsicSize = CGSize(width: 100, height: 0)
view1.setContentHuggingPriority(.defaultHigh, for: .horizontal)

let view2 = IntrinsicView()
shelfContentView.addArrangedSubview(view2)
view2.backgroundColor = .systemPurple

view2.myIntrinsicSize = CGSize(width: 100, height: 0)
view2.setContentHuggingPriority(.defaultLow, for: .horizontal)

let view3 = IntrinsicView( )
shelfContentView.addArrangedSubview(view3)
view3.backgroundColor = .systemBlue

view3.myIntrinsicSize = CGSize(width: 100, height: 0)
view3.setContentHuggingPriority(.defaultHigh, for: .horizontal)
```

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

**Stack view의 arranged subview가 높이 100으로 설정된 constraint를 깨고 stack view를 완전히 채운 것을 볼 수 있다.** 위에서 봤듯이, stack view는 auto layout을 사용해서 arranged view들의 크기와 위치를 조정한다. Stack view는 `distribution` 프로퍼티가 `fillEqually`일 때를 제외하고 arranged view들의 크기를 뷰들의 `intrinsicContentSize`를 사용해서 계산한다. 이 상황에서 두 경우로 나눠서 stack view는 동작한다.

* **stack view의 두 edge가 고정된 상태(stack view의 높이나 너비가 모호할 수 있는 상태) : arranged view들의 크기에 맞게 stack view의 크기를 재조정한다.**
* **stack view에 높이/너비에 대한 constraint가 추가적으로 주어진 상태 : arranged view들을 늘리거나 줄여서 stack view의 크기에 맞춘다.**

위 두 조건에서 현재는 아래의 상황에 해당한다. stack view의 top, bottom constraint를 설정함으로써 높이에 대한 constraint가 설정된 상태다. 이 상태에서 높이보다 작은 높이를 가진 view가 arranged subview가 추가되었으니 subview의 크기가 stack view의 크기에 맞게 재조정된 것이다.

콘솔 로그를 보면 더 명확해진다.

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

## Dynamic Stack view height

stack view에 subview들이 계속 추가되거나 삭제되면 stack view의 높이도 자동으로 늘어나거나 줄어들까?

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

공식문서에서는 관련해서 아래와 같이 나와있다.

<img width="739" alt="image" src="https://user-images.githubusercontent.com/41438361/170606627-38b48f4f-10a9-444c-a183-08ba8e1a00ab.png">

마지막 항목을 보면, `arrangedSubviews` 배열에서 뷰를 제거한다 해도 subview로 제거하는 것은 아니라고 한다. 그래서 stack view가 해당 뷰의 크기와 위치를 관리하지는 않지만 뷰가 여전히 뷰 계층에 있기 때문에 화면에 보일 수도 있다는 것이다.

관련된 stackoverflow 글이 있으니 참고하는 것도 좋다.

https://stackoverflow.com/questions/37525706/uistackview-is-it-really-necessary-to-call-both-removefromsuperview-and-remove

참고로 `arrangedSubviews`은 `subviews` 배열의 부분집합이기 때문에, 이들의 순서는 독립적으로 존재한다.

* `arrangedSubviews` 내의 순서 : stack view에 노출될 순서를 결정한다.
* `subviews` 내의 순서 : subview들의 Z 축에서의 순서를 정의한다. 만약 두 뷰가 겹친다면, 더 높은 인덱스를 가진 subview가 낮은 인덱스를 가진 뷰 위에 노출된다.

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
