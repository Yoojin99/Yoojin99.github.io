---  
layout: post  
title: "[iOS] - sizeToFit"  
subtitle: ""  
categories: app
tags: app-ios sizeToFit
comments: true  
header-img: 

---  
  
> `sizeToFit()` 메서드는 무엇을 하고 어쩔 때 쓸까?  

---

## 정의

`sizeToFit()`은 UIView의 인스턴스 메서드로, **뷰의 사이즈를 재조정하고 이동시켜서 하위 뷰들을 딱 맞게 감싸도록 한다.**

현재의 뷰의 사이즈를 재조정해서 가장 적합한 크기의 공간을 사용하게 하고 싶으면 이 메서드를 호출하면 된다. 특정 UIKit 뷰들은 그들 스스로 내부의 필요에 따라 자기의 사이즈를 재조정한다.
어떤 경우에 만약 뷰가 상위 뷰를 가지고 있지 않다면, 스스로를 화면 경계에 맞춰 사이즈를 조정할 것이다. 그러므로 만약 주어진 뷰를 상위 뷰에 맞게 사이즈를 조정하고 싶으면 이 메서드를 호출하기 전에
상위 뷰에 더해야 한다.

| 그렇다면 이 메서드와 부모 뷰에 더하는 메서드를 호출하는 순서가 뷰의 사이즈에 영향을 끼친다는 것일까?

테스트를 위해 UIViewController에서 UILabel을 생성했다.

```swift
label = UILabel(frame: CGRect(x: 100, y: 100, width: 200, height: 100))
        
label.layer.borderWidth = 2
label.layer.borderColor = UIColor.black.cgColor

view.addSubview(label)

label.text = "This is sizeToFit test"
```

화면 상에서는 아래와 같이 나온다.

<img width="290" alt="image" src="https://user-images.githubusercontent.com/41438361/154435903-5a3698d7-42bc-4262-b6ac-092c0a7eb753.png">

그렇다면 `sizeToFit()` 메서드를 사용하면 어떻게 달라질까? 위 코드에서 `sizeToFit()`을 사용했을 때 올바르게 사용하는 방법은 label의 텍스트를 설정한다면 호출하는 것이었다.
label의 text가 설정되기 전에 `sizeToFit()`을 호출해버리면 라벨의 텍스트가 아예 없는 상태에서 내부의 사이즈에 완전히 맞게 UILabel의 사이즈가 조정되기 때문에 너비 0, 높이 0의 사이즈를 갖게 된다. 즉 sizeToFit은 호출 이후 내부 내용이 변경될때마다 크기가 동적으로 변하는 것이 아니라
호출 시점에서의 내부 내용에 따라 크기가 설정된다.
label.text를 설정하고 나서 `sizeToFit()`을 호출하면 아래와 같이 나온다.

<img width="227" alt="image" src="https://user-images.githubusercontent.com/41438361/154436497-44231c6a-6912-4f08-bb45-1f8dc8ca1a3c.png">

그렇다면 autolayout을 사용할 때 `sizeToFit()`을 쓰게 되면 어떻게 될까? 결론만 보면 무의미할 가능성이 높다고 할 수 있다. `sizeToFit`은 뷰가 딱 적절한 사이즈를 가지게 크기를 재조정해주는데, 뷰가
autolayout constraints로 제약이 주어졌다면 constraints를 명확하게 사용할 필요는 무엇이며 constraints를 정의했는데 `sizeToFit()`을 호출할 이유는 무엇일까?

실제 코드 상에서도 아래와 같이 constraint를 설정하고 `sizeToFit()`을 호출하거나 `sizeToFit()`을 호출하고 constraint를 설정하면 뷰는 constraint에 따라 레이아웃이 정해진다.

```swift
label.sizeToFit() // useless
        
view.addSubview(label)

label.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 100).isActive = true
label.topAnchor.constraint(equalTo: view.topAnchor, constant: 100).isActive = true
label.widthAnchor.constraint(equalToConstant: 200).isActive = true
label.heightAnchor.constraint(equalToConstant: 100).isActive = true

label.text = "This is sizeToFit test"
label.sizeToFit() // useless
```

<img width="292" alt="image" src="https://user-images.githubusercontent.com/41438361/154437787-93a50ef9-cb79-4d1f-8835-e8e04f0b6a71.png">

다시 본론으로 돌아오자.

이 메서드를 override해서는 안된다. 만약 뷰의 default 사이즈 정보를 바꾸고 싶다면 대신 `sizeThatFits(_:)` 메서드를 오버라이딩하자. 이 메서드는 필요한 연산을 수행하고 그 결과를 
메서드에 반환해서 변화를 적게 만든다.

