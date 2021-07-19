---  
layout: post  
title: "[Swift] - Ordered Collection Diffing"  
subtitle: ""  
categories: app
tags: app-swift diffing
comments: true  
header-img: 
---  
  
> ``  

---

## Ordered Collection Diffing 

Swift 5.1에서 구현되었고, [wwdc19](https://developer.apple.com/videos/play/wwdc2019/723/) 에서 이 내용을 소개하고 있다.

Diffing은 콜렉션 사이의 차이를 적용할 수 있게 하는 API다. 

![image](https://user-images.githubusercontent.com/41438361/126088834-f8b87760-9027-4145-9e98-d4d8fc8c3ad5.png)

BIRD가 되고 싶은 BEAR가 있다. 곰은 새에게 없는 E, A가 있고, 새에는 곰에게 없는 I와 D가 있다. 

![image](https://user-images.githubusercontent.com/41438361/126088928-af8796a6-a8a1-45b7-9075-ad8646575d44.png)

그래서 E, A를 remove하고, I를 중간에, D를 끝에 추가해준다. 여기에는는 두 번의 삭제, 두 번의 삽입 연산이 일어난다.

이 작업을 Ordered Collection Diffing APi를 이용해서 굉장히 쉽게 할 수 있다.

![image](https://user-images.githubusercontent.com/41438361/126088995-13331cc6-0c7a-43aa-93e4-31224c217dad.png)

위 코드에서 `diff`는 collection difference type이다. 요소가 collection에서 삽입 혹은 삭제되었는지를 나타내는 삽입, 삭제 연산, 그리고 콜렉션에있는 요소의 offset으로 이루어진 콜렉션이다. 우리는 이 diff를 적용해서 bear를 bird 문자열로 바꿀 수 있다. 이 API는 문자열 뿐만 아니라, 모든 collection type에 적용할 수 있다. 

### difference

diff API는 모든 collection type에 적용할 수 있다고 했다. 위에서 언급했던 bear를 bird로 바꾸는 것을 해보자.

![image](https://user-images.githubusercontent.com/41438361/126089399-66cac59c-eb04-4413-ab65-02d2aadf0609.png)

우선 difference 메서드는 두 가지 방법으로 사용할 수 있다.

1. `difference(from:)`
2. `difference(from:by:)`

#### `difference(from:)`

주어진 콜렉션에서 변해야 하는 콜렉션 간의 차이를 리턴한다. 

![image](https://user-images.githubusercontent.com/41438361/126090648-bc60b748-1887-46de-ada4-13814a69409d.png)

정의는 위와 같고, `Element == C.Element`로 비교를 하는 부분 때문에 `Element`가 `Equatable`를 따르고 있어야 한다.

`other` 파라미터는 변하기 전의 콜렉션 상태를 의미한다.

이 함수는 요소의 이동에 대해서는 언급하지 않는다. 만약 이를 하고 싶다면, 결과로 나온 difference에 `inferringMoves()` 메서드를 호출하면 된다.

* 복잡도 : 최악의 경우 O(nm)이다. n은 변화된 콜렉션의 요소의 수, 그리고 m은 `other.count`이다. 콜렉션들이 많은 요소를 공유하고 있을 때 더 빠르게 동작하며, 특히 `Element`가 `Hashable`을 채택하고 있을 때 빠르게 작동한다는 것을 예상할 수 있다.

#### `difference(from:by:)`

위와 동일하지만, equivalence 테스트를 사용해서 결과를 예상한다.

![image](https://user-images.githubusercontent.com/41438361/126091540-22053b6e-ac24-4445-9cd8-a971b00c0c22.png)

`areEquivalent` 파라미터가 추가되었는데, 두 요소가 같은지 아닌지를 Bool 값으로 리턴하는 클로저다.

### `applying(_:)`

위에서 받은 차이를 collection에 적용한다.

![image](https://user-images.githubusercontent.com/41438361/126092282-ceaf6b1a-0569-4a4c-bea8-2bceb82d9b38.png)

차이를 적용할 수 있으면 차이가 적용된(바뀐) 인스턴스가 반환되고, 적용할 수 없으면 nil을 리턴한다.

* 복잡도 : O(n+c). n은 `self.count`, c는 변해야 하는 횟수이다.


## 사용

![image](https://user-images.githubusercontent.com/41438361/126091758-86cdb197-cf8d-4f86-8a2c-18418ff25759.png)

![image](https://user-images.githubusercontent.com/41438361/126091768-e469e606-5a93-44c4-9dc5-56621b963ebe.png)

`differnce(from:)` 메서드를 사용해서 bear와 bird의 차이를 `diff`로 받았다. 그리고 변화를 `applying`으로 적용했다.

diff는 차이를 리턴해준다고 했는데, 이 차이가 무엇일까? `diff`를 하나씩 출력해보면 아래와 같다.

![image](https://user-images.githubusercontent.com/41438361/126092609-736fd370-649d-42fd-8372-e264e7f56b98.png)

각 요소들(여기서는 글자)에 대한 연산이 나온다. 어떤 요소를 어디로 어떤 연산을 수행해서 바꿔라 하는 내용들이 들어가게 되는 것이다.

### CollectionView

이 ordered collection diffing은 collectionView에서 유용하게 사용될 수 있다.

두 상태간의 차이를 판단해서 적용하는 작업에서 에러가 발생하기도 쉽고, 복잡하기 때문에 diff를 적용해서 다양한 collection 타입에 차이(변화)를 적용할 수 있게 된 것이다. 이를 통해 많은 상태 관리 패턴이 좀 더 간단해진다.

위에서, `difference(from:)` 메서드를 사용해서 `Equatable`한 요소들을 가진 한 컬렉션에서 다른 컬렉션으로의 차이를 리턴했었다. 우리는 여기서
`switch`를 사용해서 `remove`, `insert` 작업을 처리할 수 있다.

```swift
let diff = newData.difference(from: data)

for change in diff {
  switch change {
  case let .remove(offset, element, associatedWith):
    // Do something with removal
  case let .insert(offset, element, associatedWith):
    // Do something with insertion
  }
}
```

`.remove`, `.insert` 케이스를 보면 associated value로 `offset`, `element`, 그리고 옵셔널 `associatedWith`이 있다.

* remove : `element`가 `offset`에서 제거되어야 한다.
* insert : `element`가 `offset`위치에 삽입되어야 한다.

`associatedWith`은 같은 시간에 변화의 쌍을 추적할 수 있게 하는 변화의 offset 값이다. 

### Diffing with UITableViews

위에서 CollectionView에 Diffing을 유용하게 사용할 수 있다고 했는데, UITableView에 적용해본다고 하자.

`UITableView`가 있고, 여러 cell들이 여기 포함되어 있을 것이다. 만약 새로운 데이터를 받았을 때 data source를 새로고침하고 `UITableViewCell`을 업데이트 할 것이다. 이때, 적절한 `UITableViewCell`을 삽입하고 삭제하기 위해 `performBatchUpdates(_:completion:)` 메서드와 diffing 메서드를 사용할 것이다. 

```swift
private func fetchNewData() {
    // Simulate a two second long network request
    networkQueue.asyncAfter(deadline: .now() + 2) { [weak self] in
      if #available(iOS 9999, *) {
        DispatchQueue.main.async {
          guard let self = self else { return }
          var deletedIndexPaths = [IndexPath]()
          var insertedIndexPaths = [IndexPath]()
          let newData = self.simulateNewData()
          let diff = newData.difference(from: self.data)

          // Gather the the index paths to be deleted and inserted via the diff
          for change in diff {
            switch change {
            case let .remove(offset, _, _):
              deletedIndexPaths.append(IndexPath(row: offset, section: 0))
            case let .insert(offset, _, _):
              insertedIndexPaths.append(IndexPath(row: offset, section: 0))
            }
          }

          self.data = newData

          self.tableView.performBatchUpdates({
            self.tableView.deleteRows(at: deletedIndexPaths, with: .fade)
            self.tableView.insertRows(at: insertedIndexPaths, with: .right)
          }, completion: { completed in
            self.refreshControl.endRefreshing()
            print("All done updating!")
          })
        }
      }
    }
  }
}
```

위와 같이 코딩하면 모든 cell, header, footer(변화가 없다고 해도)를 reload하는 `reloadData`를 호출하는 것보다 효율적이다. 또한, `reloadData`는 세부적인 컨트롤이나 변화를 애니메이션으로 보여주는 것을 허용하지 않는다.






* 출처
* https://developer.apple.com/videos/play/wwdc2019/723/
* https://github.com/apple/swift-evolution/blob/master/proposals/0240-ordered-collection-diffing.md
* https://zeddios.tistory.com/774
