---  
layout: post  
title: "[iOS] - UIPanGestureRecognizer"  
subtitle: ""  
categories: app
tags: app-ios panGestureRecognizer
comments: true  
header-img: 

---  
  
> `UIPanGestureRecognizer 정리`  

---

UIPanGesuture는 panning 제스처를 해석하는 추상적인 gesture recognizer다.

![image](https://user-images.githubusercontent.com/41438361/139604688-87b1a9a9-e262-49c5-bdef-444100159db8.png)

Panning은 쉽게 드래그하는 제스처라고 생각하면 된다. 

## 개요

`UIPanGestureRecognizer`는 `UIGestureRecognizer`의 구상클래스(concrete class)이다. 이 클래스의 클라이언트는 제스처의 현재 전환(`translation(in:)`
)과 이동의 속도(`velocity(in:)`)를 `UIPanGestureRecognizer`에 쿼리할 수 있다. 전환과 속도 값을 위해 view의 좌표 시스템을 사용할 수도 있다. 

사용자는 panning할 때 꼭 하나 이상의 손가락을 뷰에 누르고 있어야 한다. Panning gesture는 연속적이다. 사용자가 최소 손가락 개수(`minimumNumberOfTouches`) 이상을
사용해서 pan임을 알 수 있을 정도의 거리를 움직였을 때 시작한다(`UIGestureRecognizer.State.began`). 또 사용자가 이 손가락들을 누른 상태에서 움직이면
변한다(`UIGestureRecognizer.State.changed`). 사용자가 모든 손가락을 뗐을 때 종료된다(`UIGestureRecognizer.State.ended`).

공식문서를 더 살펴보니 아래와 같은 주제들이 있었다.

1. Gesture Recognizer 구성하기
2. Gesture의 위치, 속도 추적하기
3. Scroll Event 추적하기

### Gesture Recognizer 구성하기

**maximumNumberOfTouches** 

Int 값으로, gesture 탐지에 사용될 view에 터치한 최대 손가락의 개수다. 기본값은 `NSUIntegerMax`다.

예를 들어 이 값을 2라고 설정했는데, 3개의 손가락 이상을 사용해서 panning 하면 제스처로 인식을 하지 않는다는 소리가 될 것이다.

**minimumNumberOfTouches**

Int 값으로, gesture 탐지에 사용될 view에 터치한 최소 손가락의 개수다. 기본값은 1이다.

이 값도 예를 들어 2라고 설정했는데, 1개의 손가락을 사용해서 panning 하면 제스처로 인식하지 않는다는 말이 될 것이다.

### Gesture의 위치, 속도 추적하기

**func translation(in: UIView) -> CGPoint**

특정 뷰의 좌표계 내에서 pan gesture를 해석한다.

인자로 받는 뷰의 좌표계를 사용해서 pan gesture가 수행되어야 하는 위치를 계산한다. 만약 view의 위치를 사용자의 손가락 아래로 조정하고 싶다면,
이 뷰의 부모 뷰의 좌표계에 translation을 요청하면 된다.

이 함수가 리턴하는 값은 지정된 부모 뷰의 좌표계에서의 새로운 좌표이다. 이 좌표의 x, y 값은 시간에 걸쳐 리포트된다. 이전에 translation이 report 되었을 때의 값의
델타 값이 아니다. 따라서 gesture가 처음 인식되었을 때 translation 값을 뷰의 상태에 적용하고, 핸들러가 호출될떄마다 값을 이으면 안된다. 

![image](https://user-images.githubusercontent.com/41438361/139605563-24e31da5-bbf0-45c2-b144-d301b26b9acf.png)

위의 그림처럼 (0,0) 좌표에서 panning 해서 2초에 걸쳐 노란색 좌표에 도달했다고 해보자. 그리고 시작점에서 중간 지점까지는 1초, 또 마지막 점까지
1초가 걸렸다. 이때 `translation`을 하면 2초에서의 translation이 1초에서 report된 (1,2)에서의 델타 값인 (1,2)가 아닌 (2,4)가 되는 것이다.
무조건 시작점을 기준으로 생각하면 된다.

**func setTranslation(\_:in:) -> CGPoint**

특정 뷰의 좌표계에서의 translation 값을 설정한다.

첫 번째 인자에는 새로운 translation 값을 넣고, 다음 인자에는 translation이 발생해야 하는 좌표계를 가진 뷰를 설정한다. 

![image](https://user-images.githubusercontent.com/41438361/139605878-8b62e80a-88b3-4f65-b2e4-91839e0d9aff.png)

아까 봤던 예제인데, 만약 1초에서 translation을 통해 (1,2) 값을 받아오고 바로 `setTranslation(.zero, in: view)`를 했으면 2초에서
translation 값을 받아왔을 때 (1,2)가 된다. 그 이유는 `setTranslation(.zero, in: view)`을 통해 기준점이 다시 (0,0)이 되었고, 2초에서
이 기준점에 대해서 (1,2) 만큼 움직여서 이런 결과가 나오게 되는 것이다. 

참고로 translation 값을 바꾸면 pan의 속도도 초기화시킨다.

**func velocity(in:)**

특정 뷰의 좌표계에서 pan gesture의 속도를 해석한다.

이 속도는 수평과 수직 컴포넌트로 쪼개질 수 있다. 즉 x, y값을 비교해서 x축으로 속도가 빠른지, y축으로 속도가 빠른지를 비교하여 pan gesture의 
방향도 결정할 수 있는 것이다. 

그리고 이 속도 값을 이용하면 사용자가 적게 움직였지만 빠르게 아래로 swipe했을 때 뷰가 내려가는 이런 동작들도 구현할 수 있다.
