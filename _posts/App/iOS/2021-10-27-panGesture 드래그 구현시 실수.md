---  
layout: post  
title: "[iOS] - panGestureRecognizer를 이용해서 뷰 드래그 구현 삽질 내용"  
subtitle: ""  
categories: app
tags: app-ios panGestureRecognizer drag
comments: true  
header-img: 

---  
  
> `panGestureRecognizer를 이용해서 뷰를 드래그 하는 것을 구현할 때 삽질했던 내용을 정리했다.`  

---

`UIPanGestureRecognizer`를 이용해서 뷰를 드래그 할 수 있게 구현하고 있었는데, 정말 기본적인 내용에서 시간을 써버리면서 삽질했던 내용이 있어서 기록 겸 정리할 겸 삽질했던 내용을 적어보려고 한다.
아래와 같은 화면에서 뷰를 아래로 드래그 하면 뷰가 아래로 드래그 되는 것을 구현하고 싶었다.

![image](https://user-images.githubusercontent.com/41438361/139088477-f05a317a-4847-4cad-b319-0b01fe1069e3.png)


1. `UIPanGestureRecognizer` 생성

우선 드래그 제스처를 인식하고 이를 이용하기 위해 `UIPanGestureRecognizer`를 아래와 같이 생성했다. 터치 할 때 즉각적으로 드래그가 일어나게 하기 위해 `delaysTouchesBegan`와 `delaysTouchesEnded` 프로퍼티를
false 로 설정했다.

```swift
let viewPan = UIPanGestureRecognizer(target: self, action: #selector(viewPanned(_:)))
viewPan.delaysTouchesBegan = false
viewPan.delaysTouchesEnded = false
view.addGestureRecognizer(viewPan)
```

2. `viewPanned` 메서드 구현

드래그 이벤트가 발생할 때 실행될 메서드를 구현했다. 아래 코드는 내가 초기에 작성했던 메서드다.

```swift
@objc private func viewPanned(_ panGestureRecognizer: UIPanGestureRecognizer) {
    let translation = panGestureRecognizer.translation(in: self.view)

    switch panGestureRecognizer.state {
        // 드래그가 시작되어서 지금 진행중일 때
        case .changed:
            // 드래그 방향이 아래쪽이면
            if translation.y > 0 {
                // 현재 하얀 뷰의 topConstraint의 constant에 아래로 드래그 한 만큼을 더한다.
                bottomSheetViewTopConstraint.constant += translation.y
            }
        default:
            break
        }
}
```

이 메서드는 gestureRecognizer가 이벤트를 감지했을 때마다 실행되는 메서드라 실행될 때마다 `bottomSheetViewTopConstraint.constant`가 드래그 된 위치로 업데이트 되었고, `translation.y`는 아래로 드래그 할 수록
점점 증가하니 이 `bottomSheetViewTopConstraint.constant`가 결국 드래그 시간이 지날수록 엄청나게 커지게 되었다.

즉 내가 처음 예상했던 동작은

![image](https://user-images.githubusercontent.com/41438361/139091606-50e5f748-8c35-4f95-a87d-6423453877a1.png)

로, 예를 들어 사용자가 아래로 50만큼 드래그 하면 뷰도 50 만큼 내려가는 건데, 실제로는 아래와 같이 동작하는 것이다.

