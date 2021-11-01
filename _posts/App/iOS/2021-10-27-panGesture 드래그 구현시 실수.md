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

https://user-images.githubusercontent.com/41438361/139093612-4f1792f2-f9ac-4450-97bd-3ebca6d509ef.mp4

실제로는 아래로 굉장히 조금 드래그 했는데, 위와 같이 굉장히 아래로 뷰가 빠르게 내려갔다.

위에서 봤던 메서드는 gestureRecognizer가 이벤트를 감지했을 때마다 실행되는 메서드라 실행될 때마다 `bottomSheetViewTopConstraint.constant`가 드래그 된 위치로 업데이트 되었고, `translation.y`는 아래로 드래그 할 수록
점점 증가하니 이 `bottomSheetViewTopConstraint.constant`가 결국 드래그 시간이 지날수록 엄청나게 커지게 되었다.

즉 내가 처음 예상했던 동작은

![image](https://user-images.githubusercontent.com/41438361/139091606-50e5f748-8c35-4f95-a87d-6423453877a1.png)

로, 예를 들어 사용자가 아래로 50만큼 드래그 하면 뷰도 50 만큼 내려가는 건데, 실제로는 아래와 같이 동작하는 것이다.

![image](https://user-images.githubusercontent.com/41438361/139092430-08d980e3-9810-42db-9545-5f6487e07099.png)

저 `viewPanned` 함수가 gesture가 감지될때마다 실행되기 때문에, 예를 들어 처음 constant가 0이었고, 사용자가 아래로 드래그를 시작했다면 1만큼 아래로 드래그했을 때 `viewPanned` 함수가 실행되고,
constant는 사용자가 아래로 내린 1이 더해져서 1이 된다. 그리고 사용자는 아직 아래로 드래그 하는 중이어서 또 1만큼 내려갔다면, `viewPanned` 함수가 또 실행되고, constant는 아까 업데이트 된 1에, 사용자가
처음 드래그를 시작해서 아래로 내린 2만큼이 더해져서 3이 된다. 따라서 이런 식으로 반복적으로 함수의 호출이 일어나면서 값이 업데이트가 되었기 때문에 내가 50만큼 내린다고 뷰가 50만큼 내려가는 게 아니고, 드래그 하는 시간이 길 수록 더 빨리 내려가게 된 것이다.

그래서 외부에 처음 드래그를 시작했을 때 constant를 저장할 변수를 하나 생성하고, `viewPanned` 메서드를 아래와 같이 수정했다.

```swift
@objc private func viewPanned(_ panGestureRecognizer: UIPanGestureRecognizer) {
    let translation = panGestureRecognizer.translation(in: self.view)

    switch panGestureRecognizer.state {
        case .began:
            // 드래그가 시작되었을 때의 constant를 외부에 따로 저장한다.
            bottomSheetPanStartTopConstant = bottomSheetViewTopConstraint.constant
        case .changed:
            if translation.y > 0 {
                // constant를 드래그가 시작되었을 때 저장했던 constant에 드래그 한 만큼의 값을 더해서 업데이트한다.
                bottomSheetViewTopConstraint.constant = bottomSheetPanStartTopConstant + translation.y
            }
        default:
            break
        }
}
```

이제 이러면 외부에 드래그가 시작했을 때의 constant가 저장되어 있고, 이 메서드가 반복적으로 실행된다 해도 `.began` 상태일 때의 코드는 오직 드래그가 시작된 처음, 한 번만 실행될테니 `bottomSheetPanStartTopConstant`는 값이 고정된다. 그리고 `changed` 상태일 때, 즉 드래그가 진행중일 때 외부에 저장했던 constant에 내가 드래그 한 만큼을 더한 값을 constant로 정해주면 된다.

그러면 아래와 같이 아주 살짝 드래그해도 뷰가 확 내려가는 일이 없다.

![Simulator Screen Recording - iPhone 13 Pro Max - 2021-10-28 at 00 14 06](https://user-images.githubusercontent.com/41438361/139095057-694ebb35-6dac-4599-bad2-4621d3b64967.gif)

gesture가 발생할 때마다 함수가 반복적으로 실행되는 건 당연한데 이 기본적인 내용을 생각을 못해서 시간을 썼나 싶다... ^ㅡ^ 그래도 이제 안 헷갈리면 되겠지

+ 추가

물론 아래와 같이 `setTranslation`을 통해서 코드를 좀 더 간단하게 만들어서 같은 동작을 구현할 수도 있지만, 나는 panning을 시작한 처음 기준점에서
내린 높이에 따라 다른 동작을 구현했기 때문에 이를 여기서는 사용하지 않았다.

```swift
@objc private func viewPanned(_ panGestureRecognizer: UIPanGestureRecognizer) {
    let translation = panGestureRecognizer.translation(in: self.view)

    switch panGestureRecognizer.state {
        case .changed:
            if translation.y > 0 {
                bottomSheetViewTopConstraint.constant += translation.y
            }
        default:
            break
    }
    
    panGestureRecognizer.setTranslation(.zero, in: self.view)
}
```
