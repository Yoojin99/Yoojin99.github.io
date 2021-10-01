---  
layout: post  
title: "[iOS] - UIStackView Scroll하기"  
subtitle: ""  
categories: app
tags: app-ios uistackview scroll
comments: true  
header-img: 

---  
  
> `UIStackView가 화면에 나타나 수 있느 범위르 넘어섰을 때 스크롤할 수 있게 만들어보자.`  

---

UIStackView는 내부에 뷰들을 넣을 때마다 그 크기가 커진다. 그렇다면 UIStackView에 넣은 뷰들이 화면에 다 출력될 수 없을 때
이를 스크롤을 해서 다른 뷰들을 볼 수 있게 하는 방법도 미리 생각을 해놔야 한다.

관련해서 [StackOverflow](https://stackoverflow.com/questions/31668970/is-it-possible-for-uistackview-to-scroll)에 이에 대한 내용이 있어서 이를 정리해봤다.

먼저 그 전에 UIStackView에 대해 봐야 한다. UIStackView는 UIView를 상속한 클래스다.

![image](https://user-images.githubusercontent.com/41438361/135565142-6ab56fa6-d267-442e-b1a8-0bb56ef847cf.png)

UIStackView는 단독으로 있을 때 스크롤할 수 없다. 따라서 내재된 뷰를 스크롤하고, 확대/축소할 수 있게 하는 UIScrollView의 도움이 필요하다. 참고로 UIScrollView도
UIView를 상속한 클래스이다. 

![image](https://user-images.githubusercontent.com/41438361/135565276-4b5cb9a3-a0eb-4602-93ff-f2e2cc1fb2ad.png)

UIKit의 클래스들 중 UIScrollView를 상속한 클래스들은 자동으로 컨텐츠 크기가 프레임을 벗어나게 되면 컨텐츠를 스크롤 할 수 있게 해주지만, UIStackView는 UIScrollView와는 별개의 클래스이므로 UIStackView를 스크롤하려면 UIScrollView 안에 넣어서 스크롤할 수 있게 해야 한다.

그림으로 표현하면 아래와 같다. 

<img width="790" alt="image" src="https://user-images.githubusercontent.com/41438361/135565881-cf51d5d1-0b8d-4a21-9d26-d8b4d68f4cbe.png">

만약 scrollview의 크기를 기기 화면의 크기와 같게 설정했을 때, 그 안에 있는 컨텐츠의 크기가 scrollView의 프레임보다 크기 때문에 화면을 터치, 스크롤해서 컨텐츠를
이동할 수 있다.

이제 구체적인 방법을 보겠다.

### 1. `UIScrollView`를 생성하고 제약을 설정한다.

```swift
private let scrollView: UIScrollView = {
        let view: UIScrollView = UIScrollView()
        view.translatesAutoresizingMaskIntoConstraints = false
        view.backgroundColor = .white
        return view
    }()

private func setScrollViewConstraint() {
        NSLayoutConstraint.activate([
            scrollView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            scrollView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            scrollView.topAnchor.constraint(equalTo: view.topAnchor),
            scrollView.bottomAnchor.constraint(equalTo: view.bottomAnchor),
        ])
    }
```

### 2. `UIScrollView`에 `UIStackView`를 하위 뷰로 추가한다.

```swift
private let stackView: UIStackView = {
        let view: UIStackView = UIStackView()
        view.translatesAutoresizingMaskIntoConstraints = false
        view.axis = .vertical
        view.spacing = 5
        return view
    }()

private func setUpView() {
        view.addSubview(scrollView)
        scrollView.addSubview(stackView)
    }
```

### 3. `UIStackView`의 leading, trailing, top, bottom constraint를 `UIScrollView`와 같게 설정한다. 

```swift
private func setStackViewConstraint() {
        NSLayoutConstraint.activate([
            stackView.leadingAnchor.constraint(equalTo: scrollView.leadingAnchor),
            stackView.trailingAnchor.constraint(equalTo: scrollView.trailingAnchor),
            stackView.topAnchor.constraint(equalTo: scrollView.topAnchor),
            stackView.bottomAnchor.constraint(equalTo: scrollView.bottomAnchor),
        ])
    }
```

### 4. `UIStackView`와 `UIScrollView`의 width를 같게 설정한다.

여기에서 중요한 것은 UIStackView의 axis가 `vertical`일 때는 width를, `horizontal`일 때는 height를 같게 설정해야 한다는 것이다.

```swift
            stackView.widthAnchor.constraint(equalTo: scrollView.widthAnchor),
```

### 5. 뷰들을 `UIStackView`에 추가한다.

```swift
class SubView: UIView {
    init() {
        super.init(frame: .zero)
        translatesAutoresizingMaskIntoConstraints = false
        backgroundColor = .systemGreen
        widthAnchor.constraint(equalToConstant: 100).isActive = true
        heightAnchor.constraint(equalToConstant: 100).isActive = true
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```

그러면 이제 UIStackView의 axis에 따라 세로 방향, 가로 방향으로 스크롤되는 뷰를 만들 수 있다.

**Vertical일 때**

![image](https://user-images.githubusercontent.com/41438361/135568032-6828d977-de53-430a-b7c7-a7bc36b1b065.png)


**Horizontal일 때**

![image](https://user-images.githubusercontent.com/41438361/135568425-c52b862e-f1d4-41db-8da0-047e42ecb464.png)
