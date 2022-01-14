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

그렇데 한 뷰의 좌표계에 있는 point를 다른 뷰의 좌표계의 point로 바꾼다는 게 무슨 말일까?


