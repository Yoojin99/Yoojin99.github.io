---  
layout: post  
title: "[iOS] - visibleCells in nested UICollectionView"  
subtitle: ""  
categories: app
tags: app-ios UICollectionView visibleCells
comments: true  
header-img: 

---  
  
> `UICollectionView에서 스크린에 보여지는 cell들만 확인하는 방법이 있을까?`  

---

## visibleCells

UICollectionView를 사용하다 보면 UICollectionView가 출력하고 있는 cell들 중에서 화면에 띄워져 있는 cell들만 알고 싶은 경우도 생길 것이다.

UICollectionView의 property를 찾아보던 중 `visibleCells`를 찾을 수 있었다.

![image](https://user-images.githubusercontent.com/41438361/148900191-d11512e0-bd12-4b35-9b47-aa114ac3395d.png)

UICollectionView의 인스턴스 프로퍼티인 `visibleCells`는 현재 collectionView에 의해 출력된 cell들 중 보여지는 cell들의 배열이다. 만약 보여지는 cell이 없다면 빈 배열이 return 된다고 한다. 

### 예시

우선 UICollectionView를 가지는 UIViewController를 아래와 같이 만들어줬다. 그리고 하나의 cell 크기는 크게 하고, 데이터의 수는 적당히 많게 설정해서 모든 cell들이 한 번에 화면에 노출되지 않은 경우를 설정했다.

![image](https://user-images.githubusercontent.com/41438361/149457990-895aa970-2801-4a23-8fce-a4ebd499e59d.png)

그리고 ScrollViewDelegate를 이용해서 스크롤 할 때마다 현재 visibleCell이 무엇인지 출력했다.

딱 cell의 모서리를 포함하여 모서리가 안보일 때는 visibleCell에 포함이 되지 않았고, 

![image](https://user-images.githubusercontent.com/41438361/149459259-3de65d79-b7b9-447f-bb69-963db3361bfe.png)

모서리가 조금이라도 화면에 출력되면 visibleCell에 포함되는 것을 확인할 수 있었다.

![image](https://user-images.githubusercontent.com/41438361/149459378-1ee95b4a-6480-4726-9629-3e1c35f1cd62.png)

그런데 이 visibleCell이 잘 동작하지 않을 수도 있다. [StackOverflow](https://stackoverflow.com/questions/38281883/check-whether-cell-at-indexpath-is-visible-on-screen-uicollectionview)의 글에 의하면, UICollectionView가 여러 개 중첩되어 있을 때 이런 현상이 발생한다.

중첩된 UICollectionView는 스크롤 되지 않을 필요가 있고, 따라서 contentOffset이 제공되지 않으며 iOS는 모든 cell들이 항상 화면에서 보여지고 있다고 여긴다. 이 경우는 visibleCell을 찾기 위해서
cell이 실제로 화면의 bound 내에 위치하는지 확인할 필요가 있다.

그 경우에는 아래의 코드를 사용하면 된다.

```swift
let cellRect = cell.contentView.convert(cell.contentView.bounds, to: UIScreen.main.coordinateSpace)
if UIScreen.main.bounds.intersects(cellRect) {
    print("cell is visible")
}
```

위 코드를 잠시 살펴보면, `cellRect`로 cell의 bound 사각형을 구하고, `intersects`로 스크린과 이 사각형이 겹치는지 확인한다는 것을 예상할 수 있다.
그런데 `convert(_:to:)`와 `intersects`가 무슨 함수인지 정확하게 알아봐야 할 것 같다.

## convert

`convert(_:to:)`는 UIView의 인스턴스 메서드로, 뷰의 좌표계의 특정 point를 `to` 뷰의 좌표계로 바꿔진 point를 리턴해준다. 
`convert(_:from:)`이라는 메서드도 있는데, 이는 반대로 `from` 뷰 내의 point를 수신자(메서드 호출자)뷰의 좌표계로 바꿔준다.

![image](https://user-images.githubusercontent.com/41438361/149460758-39ec7711-8e61-4c64-80c6-5d247b0c374f.png)

만약 `to` 가 nil이면, 윈도우 기반 좌표계로 변환이 된다. 그렇지 않다면 뷰와 `to` 뷰는 모두 같은 `UIWindow` 내에 속해야 한다.

그렇데 한 뷰의 좌표계에 있는 point를 다른 뷰의 좌표계의 point로 바꾼다는 게 무슨 말일까? 그 전에 이를 이해하기 위한 배경 지식인 Frame과 Bound에 대해 알아보자.

### Frame & Bound

frame 사각형은 **부모 뷰의 좌표계**안에서 뷰의 **위치와 크기**를 나타낸다. 여기서 중요한 것은 부모 뷰를 기준으로 뷰의 위치와 크기를 나타낸다는 것이다. 
frame의 프로퍼티를 조정하면 `center` 프로퍼티의 point가 변하고, bounds의 size가 변하게 된다. frame을 조정하면 `draw(:_)` 메서드를 호출하지 않고 자동으로 뷰를 다시 그린다.

bound 사각형은 **뷰 자신의 좌표계**안에서 뷰의 **위치와 크기**를 나타낸다. Default bound의 origin은 (0,0) 이고, size는 frame의 사이즈와 같다. 
bound의 size를 늘리거나 줄이면 center를 중심으로 크기가 늘어나거나 줄어든다. 또 bound의 size를 조정하면 frame의 size도 조정된다. bounds를 조정하면 `draw(:_)` 메서드를 호출하지 않고 자동으로 뷰를 다시 그린다.

즉 frame과 bound는 거의 비슷한데, 가장 중요한 차이는 frame은 부모 뷰를 기준으로, bound는 자신을 기준으로 뷰의 위치와 크기를 나타낸다는 것이다.

테스트로 아래와 같이 노란색 뷰 하나를 만들어서 UIViewController의 view에 그대로 붙여봤다.

```swift
let firstView: UIView = {
    let view = UIView(frame: CGRect(x: 0, y: 0, width: 100, height: 100))
    view.backgroundColor = .systemYellow

    return view
}()
```

결과는 아래와 같이 나왔다.

![image](https://user-images.githubusercontent.com/41438361/149710718-7404158b-a4af-4747-a961-da058bf67e15.png)

이번에는 아래와 같이 frame의 origin을 조정해서 노란색 뷰를 이동시켜 봤다.

```swift
let firstView: UIView = {
    let view = UIView(frame: CGRect(x: 100, y: 200, width: 100, height: 100))
    view.backgroundColor = .systemYellow

    return view
}()
```

노란색 뷰의 superView는 UIViewController의 view이고, 이 뷰를 기준으로 x축으로 100만큼, y축으로 200만큼 이동한 것이다. 

![image](https://user-images.githubusercontent.com/41438361/149711250-b884f9a9-3df1-410f-badd-ec87fd800ae6.png)

이번에는 45도 회전시켜보고 frame을 출력해봤다.

```swift
let firstView: UIView = {
    let view = UIView(frame: CGRect(x: 100, y: 200, width: 100, height: 100))
    view.backgroundColor = .systemYellow
    view.transform = .init(rotationAngle: 45)

    return view
}()
```

![image](https://user-images.githubusercontent.com/41438361/149711502-109cc4a1-f11f-41d0-9ef7-0c03de2afa9d.png)

![image](https://user-images.githubusercontent.com/41438361/149711529-4515eb0b-f339-40bb-8ea4-63e3154f3b15.png)

나는 분명 UIView의 frame을 `CGRect(x: 100, y: 200, width: 100, height: 100)`로 설정했는데 왜 회전시키니 origin을 비롯하여 size까지 frame이 바뀌었을까? 이는 frame이라는 것이 뷰 영역을 감싸는 사각형을 의미하기 때문이다.

그래서 뷰를 위와 같이 회전시키면 실제로 frame은 아래와 같이 회전된 노란색 뷰를 감싸는 사각형이 되므로 frame이 변하게 된다.

![image](https://user-images.githubusercontent.com/41438361/149711815-448629a9-99db-43a7-a4e6-c7688eb392d4.png)

이번에는 bounds를 확인했다. 회전시키지 않은 노란색 뷰의 bound를 출력하면 아래와 같다.

![image](https://user-images.githubusercontent.com/41438361/149711978-5765d3df-8944-42ae-89b1-13ac0539559a.png)

![image](https://user-images.githubusercontent.com/41438361/149711995-6a296744-a21b-43a9-966a-7d3b26bde852.png)

노란색 뷰의 frame의 origin은 부모 뷰를 기준으로 (100,200)인데, bounds의 origin은 자기 자신을 기준으로 하니 (0,0)이 나오는 걸 확인할 수 있다. 이 bounds의 origin은 scrollView 등에서
bounds의 origin을 바꾸는 등 여러 곳에서 조작하면 유용하다. 이는 뒤에서 보기로 하고, 회전을 시키면 bounds가 어떻게 출력되는지 보겠다.

![image](https://user-images.githubusercontent.com/41438361/149712987-566e8bca-dffa-4dbd-afe3-8244784d4749.png)

![image](https://user-images.githubusercontent.com/41438361/149713041-44c3132c-04eb-4114-8d30-13e073c9961c.png)

view를 회전시켰는데, frame과는 다르게 bounds는 origin과 size 모두 변하지 않는다! 

이번에는 짤간색 뷰를 생성해서 노란색 뷰에 addSubview를 해봤다.

```swift
let secondView: UIView = {
    let view = UIView(frame: CGRect(x: 0, y: 0, width: 100, height: 100))
    view.backgroundColor = .systemRed

    return view
}()
```

빨간 뷰의 부모 뷰는 노란색 뷰이고, 이 노란색 뷰를 기준으로 origin이 (0,0)이니 노란 뷰의 왼쪽 상단 모서리와 빨간 뷰의 왼쪽 상단 모서리가 겹치는 걸 볼 수 있다.

![image](https://user-images.githubusercontent.com/41438361/149713412-c921c8c8-6b12-4646-96c2-bee305de6397.png)

빨간 뷰의 frame의 origin을 (50, 50)으로 변경하고 출력하니 아래와 같았다. 부모 뷰인 노란뷰를 기준으로 x축으로 50, y축으로 50만큼 떨어져서 빨간 뷰가 나타난 것을 볼 수 있다.

![image](https://user-images.githubusercontent.com/41438361/149713618-6590ecfc-e1b1-48a9-9b89-6b8d87f4840f.png)

그렇다면 노란 뷰의 bound를 조정하면 어떻게 될까?

![image](https://user-images.githubusercontent.com/41438361/149713910-84c84ecf-f252-4969-a901-fda19be2292a.png)

노란 뷰의 bound의 origin을 (50,50)으로 조정했는데 빨간 뷰의 위치가 바뀌었다! 정확히 말하면 바뀐 것이 아니라 바뀐 것처럼 나오는 것이다.

더 알아보기 위해 이번에는 초록뷰를 추가해서 아래와 같이 만들어줬다. 초록뷰의 부모뷰는 빨간 뷰로 설정했다.

![image](https://user-images.githubusercontent.com/41438361/149714211-44a9bf23-641c-436e-8723-ac8f9865f08e.png)

이 상태에서 빨간 뷰의 bounds의 origin을 (50,50)으로 설정했다.

![image](https://user-images.githubusercontent.com/41438361/149714361-c8e5fe6f-d811-4838-b7fe-7da8397b5b92.png)

그랬더니 엉뚱하게 이번에는 또 초록색 뷰가 움직인다. 정확히는 움직인게 아니라 움직인 것처럼 보였다. 이 상황의 원인을 이해할 수 있게 해주는 [StackOverflowd 글](https://stackoverflow.com/questions/7872552/whats-the-meaning-of-view-bounds-origin)을 참고하면 이해하기 쉽다.

bounds 사각형은 frame 사각형과 같이 view의 보여질 수 있는 부분을 나타낸다. Bounds의 origin 값을 바꿔 뷰의 다른 부분을 보여줄 수 있다. 그래서 예를 들어 위쪽 상황에서 첫 번째 상황을 보면, 나는 노란 뷰의 bound의 origin을 (50,50)으로 바꿔줬고, 이는 노란 뷰 내부에서 보여질 수 있는 부분을 이동시킨 것인데 (50,50)만큼 이동시킨 것이다. 이는 내가 카메라 맨이라고 생각하고, 화면 상 어떻게 보여지는 지를 생각하면 이해하기 쉽다. (0,0) 위치에 서 있는 사람이 있고 나는 그 정면에서 카메라로 사람을 찍고 있다고 해보자. 그런데 내가 예를 들어 오른쪽으로 3미터 이동하면 실제로 사람은 가만히 있지만 화면 상에는 그 사람이 왼쪽으로 삼미터 정도 이동한 것처럼 보이는 것과 똑같다.

![image](https://user-images.githubusercontent.com/41438361/149718646-cc54e20c-3701-42df-948f-f569db51182a.png) 

그러면 두 번째 상황에서 처럼 노란 뷰 안에 짤간 뷰 안에 초록 뷰가 있는데 빨간 뷰의 bounds의 origin을 (50,50)으로 설정한 경우는 어떻게 될까?

![image](https://user-images.githubusercontent.com/41438361/149719275-459f434d-2a75-4847-9f5a-1dceecc32abe.png)

앞에서 잠깐 언급한 bounds의 origin을 바꾸는 상황에 대해 알아보자. 아래와 같은 이미지가 있고, 이를 화면에 띄워서 보고 싶다고 해보자.

![image](https://user-images.githubusercontent.com/41438361/149721068-9089a159-b462-4100-ab53-7a763b12d5a5.png)

하지만 이 이미지는 너무 커서 스마트폰 화면으로 다 볼 수 없다. 그래서 UIScrollView를 이용해서 이 이미지의 부분만 화면에 보여지고, 스크롤 해서 이미지의 다른 부분을 볼 수 있게 만들었다.

아무것도 설정하지 않고(bounds는 default (0,0)임) 화면에 띄우면 아래와 같이 이미지의 왼쪽 상단을 (0,0)을 기준으로 잡아 이미지를 보여주고 있다.

![image](https://user-images.githubusercontent.com/41438361/149721241-dcd23913-4ae5-40f5-8963-57c3072debfd.png)

만약 scollview의 bound를 조정한다면? bound.origin을 (500,500)으로 조정해보았다.

![image](https://user-images.githubusercontent.com/41438361/149721795-21516f2e-e9fb-4d0c-bd9a-2536eb6c97e0.png)

그러자 이미지가 보여지는 위치가 달라졌다. 즉 화면이 카메란데, (500,500) 만큼 이동해서 아래와 같이 이미지가 보여지는 부분이 달라지는 것이다. scrollView를 스크롤하게 되면 scrollView의 origin이 변하면서 이미지의 또 다른 부분들을 보여준다. 이렇게 bounds의 origin을 조작하는 일은 scrollView에서 흔하다.

![image](https://user-images.githubusercontent.com/41438361/149722354-d48cdd22-9181-4972-becb-80261f4e210f.png)


### convert

이제 frame과 bounds에 대해서는 알아봤으니, 다시 view.convert로 돌아와보자. 





