---  
layout: post  
title: "[iOS] - Diff 프레임워크로 UICollectionView 데이터 업데이트하기"  
subtitle: ""  
categories: app
tags: app-ios
comments: true  
header-img: 

---  
  
> ``  

---

![image](https://user-images.githubusercontent.com/41438361/125879044-d8a992a5-08f9-4b78-8d4d-e37e829c5573.png)

## 친숙한 애들

iOS나 tvOS에서 `UITableView`, `UICollectionView`, macOS에선 `NSCollectionVIew`과 같은 클래스를 사용하지 않고 table view, collection view
를 사용하는 앱을 상상하기 어렵다. 대부분의 시간을 백엔드에서 데이터를 불러오고 캐싱한 다음, 리스트나 그리드 형식으로 데이터를 보여주기 위해 필터링한다.
그리고 이후 데이터가 바뀌었을 때, 새로 추가되거나 삭제된 몇몇 아이템들을 반영하기 위해 인터페이스를 업데이트한다.

여기에서 당신이 많이 사용했을 법한 `reloadData` 함수가 사용된다. 이 메서드를 사용하면 전체 리스트가 완전히 새로운 컨텐츠로 새로고침된다. 컨텐츠를 
다시 불러오기 위해 빠른 방법을 이용하는 경우에는 괜찮지만, `UITableView`가 cell의 사이즈를 다시 검증하게 하기 때문에 성능을 저하시키게 될 수 있다. 
데이터에 생기는 변화들을 사용자에게 알려야 하고, 사용자에게 현재 어떤 일이 일어나고 있는지 명확히 알려주고 싶은 것이라면 직접 행을 추가하고 삭제하는 방법이
선호된다.

만약 안드로이드 개발자라면 `notifyDataSetChanged`를 호출하는 것 대신에 제공된 `DiffUtil`을 이용해 변화를 계산하게 해서 `RecyclerView`를 업데이트하는 것을
더 쉽게 할 수 있다는 것을 알 것이다. 하지만 iOS는 이런 사치를 부릴 수 없기 때문에 어떻게 이를 할 수 있는지를 배워야 한다.

이 가이드는 `UICollectionView`를 예로 들고 있지만, `UITableView`도 똑같이 동작한다. 

## Drag and Drop

`UICollectionView`가 어떻게 동작하는지를 보자. 한 collection에서 다른 collection으로 아이템을 이동할 수 있는 앱을 상상해보자.

이를 iOS 11의 [drag and drop API](https://developer.apple.com/design/human-interface-guidelines/ios/user-interaction/drag-and-drop/)을
이용해서 구현할 수 있다.

![image](https://user-images.githubusercontent.com/41438361/125881836-2b32996e-3731-4bb8-bd46-86136e2c8f75.png)

꼭 `UICollectionView`에 업데이트 하는 메서드를 호출하기 전에 데이터가 변했다는 것을 보장해야 한다. 그리고 데이터가 변경된 것을 반영하기 위해 `deleteItems`와
`insertItems`를 호출한다. `UICollectionView`는 이를 멋진 애니메이션과 함꼐 보여준다.

```swift
func collectionView(_ collectionView: UICollectionView, performDropWith coordinator: UICollectionViewDropCoordinator) {
  let destinationIndexPath = coordinator.destinationIndexPath
  let sourceIndexPath = coordinator.items.last?.dragItem.localObject as! IndexPath
  // remove
  sourceItems.remove(at: sourceIndexPath.item)
  sourceCollectionView.deleteItems(at: [sourceIndexPath])
  // insert
  destinationItems.insert(draggedItem, at: destinationIndexPath.item)
  destinationCollectionView.insertItems(at: [destinationIndexPath])
}
```

이건 단순히 collection에서 하나의 아이템을 없애고 추가하는 예제였다. 하지만 실제 프로젝트에서 데이터는 더 복잡하고, 데이터에서 일어나는 변화또한 간단하지 않다.
만약 백엔드 응답으로 인해 많은 아이템들의 삽입과 삭제가 일어난다면 호출해야 하는 정확한 `IndexPath`를 계산해야 하는데, 이는 쉬운 일이 아니다. 아래와
같은 크래시를 많이 겪게 될 것이다.

**NSInternalInconsistencyException**

> Terminating app due to uncaught exception ‘NSInternalInconsistencyException’,
> reason: ‘Invalid update: invalid number of items in section 0.
> The number of items contained in an existing section after the update (213)
> must be equal to the number of items contained in that section before
> the update (154), plus or minus the number of items inserted or
> deleted from that section (40 inserted, 0 deleted) and plus
> or minus the number of items moved into or out of
> that section (0 moved in, 0 moved out).’

사용자가 각각 다른 데이터를 가지고 있기 때문에 위와 같은 에러가 무작위로 나타났다. 이 메세지가 굉장히 서술적임에도 불구하고 원인을 찾는데 시간이 오래 걸릴 것이다.

## IndexPath의 게임

몇 예제를 통해 `IndexPath`에 대한 지식을 다시 점검해보자. 6개의 아이템을 가진 collection으로, 몇 update 연산을 수행하고 IndexPath가 어떻게 될 지 알아보자.

`items = [“a”, “b”, “c”, “d”, “e”, “f”]`

### index vs offset

나아가기 앞서, 위에서 말한 `index`는 `offset`임을 말한다는 것을 짚고 넘어가겠다. enumerated 함수를 한 번 봤다면, index 대신 offset으로 명명하고 있는 것을 알 수 있다.

```swift
Array(0..<10).enumerated().forEach { tuple in
  print(tuple)
}
```

![image](https://user-images.githubusercontent.com/41438361/125883100-32831c96-bc46-4b22-828f-a0c2312071b6.png)

아래의 몇 예제들은 collection, `IndexPath`에 간단한 변화를 일으킨다. 우리가 `UICollectionView`의 업데이트 메서드를 호출하기 전에 아이템
배열(data source가 될 배열이다)을 업데이트 해야 하는 것을 기억해라.

**1. 끝에 3개의 아이템을 추가한다.**

```swift
items.append(contentsOf: [“g”, “h”, “i”])
// a, b, c, d, e, f, g, h, i
let indexPaths = Array(6…8).map { IndexPath(item: $0, section: 0) }
collectionView.insertItems(at: indexPaths)
```

**2. 끝의 3개의 아이템을 삭제한다.**

```swift
items.removeLast()
items.removeLast()
items.removeLast()
// a, b, c
let indexPaths = Array(3…5).map { IndexPath(item: $0, section: 0) }
collectionView.deleteItems(at: indexPaths)
```

**3. index 2의 아이템을 업데이트한다.**

```swift
items[2] = “👻”
// a, b, 👻, d, e, f
let indexPath = IndexPath(item: 2, section: 0)
collectionView.reloadItems(at: [indexPath])
```

**4. 아이템 "c"를 맨 뒤로 옮긴다.**

```swift
items.remove(at: 2)
items.append(“c”)
// a, b, d, e, f, c
collectionView.moveItem(
  at: IndexPath(item: 2, section: 0),
  to: IndexPath(item: 5, section :0)
)
```

**5. 앞의 세 개의 아이템을 삭제하고, 끝에 3개의 아이템을 추가한다.**

여러 다른 연산을 수행할 때는, `performBatchUpdates`를 사용해야 한다.

여러 변화를 collection view에서 일으킬 때 각각을 애니메이션으로 보여주지 않고 하나의 애니메이션으로 보여주고 싶을 때 이 메서드를 사용한다. 
아이템을 추가, 삭제, reload, 이동할 때나 하나 이상의 cell과 관련된 layout 파라미터를 바꾸기 위해서 이 메서드를 사용한다.

```swift
items.removeFirst()
items.removeFirst()
items.removeFirst()
items.append(contentsOf: [“g”, “h”, “i”])
// d, e, f, g, h, i
collectionView.performBatchUpdates({
  let deleteIndexPaths = Array(0…2).map { IndexPath(item: $0, section: 0) }
  collectionView.deleteItems(at: deleteIndexPaths)
  let insertIndexPaths = Array(3…5).map { IndexPath(item: $0, section: 0) }
  collectionView.insertItems(at: insertIndexPaths)
}, completion: nil)
```

**6. 끝에 3개의 아이템을 추가하고, 앞의 3개 아이템을 삭제하기**

```swift
items.append(contentsOf: [“g”, “h”, “i”])
items.removeFirst()
items.removeFirst()
items.removeFirst()
// d, e, f, g, h, i
collectionView.performBatchUpdates({
  let insertIndexPaths = Array(6…8).map { IndexPath(item: $0, section: 0) }
  collectionView.insertItems(at: insertIndexPaths)
  let deleteIndexPaths = Array(0…2).map { IndexPath(item: $0, section: 0) }
  collectionView.deleteItems(at: deleteIndexPaths)
}, completion: nil)
```

위 6번의 경우 실행하면 크래시가 난다.

> Terminating app due to uncaught exception
> ‘NSInternalInconsistencyException’,
> reason: ‘attempt to insert item 6 into section 0,
> but there are only 6 items in section 0 after the update’

### performBatchUpdates

이는 `performBatchUpdates`가 동작하는 방법 때문이다. 공식 문서에는 아래와 같이 나와있다.

> Deletes are processed before inserts in batch operations. This means the indexes for the deletions are processed relative to the indexes of the collection view’s state before the batch operation, and the indexes for the insertions are processed relative to the indexes of the state after all the deletions in the batch operation.

우리가 어떻게 insert와 delete를 호출하든, `performBatchUpdates`는 항상 삭제 연산을 먼저 수행한다. 그래서 삭제가 먼저 이루어질 경우 올바른 
`IndexPath`를 사용해서 `deleteItems`, `insertItems`를 호출해야 한다.

```swift
items.append(contentsOf: [“g”, “h”, “i”])
items.removeFirst()
items.removeFirst()
items.removeFirst()
// d, e, f, g, h, i
collectionView.performBatchUpdates({
  let deleteIndexPaths = Array(0…2).map { IndexPath(item: $0, section: 0) }
  collectionView.deleteItems(at: deleteIndexPaths)
  let insertIndexPaths = Array(3…5).map { IndexPath(item: $0, section: 0) }
  collectionView.insertItems(at: insertIndexPaths)
}, completion: nil)
```

### Operation

`UICollectionView`에는 많은 연산들이 있고, 전체 section을 업데이트하는 연산도 여기 포함된다. [Ordering of Operations and Index Paths](https://developer.apple.com/library/archive/documentation/UserExperience/Conceptual/TableView_iPhone/ManageInsertDeleteRow/ManageInsertDeleteRow.html)
를 보자.

```swift
insertItems(at indexPaths: [IndexPath])
deleteItems(at indexPaths: [IndexPath])
reloadItems(at indexPaths: [IndexPath])
moveItem(at indexPath: IndexPath, to newIndexPath: IndexPath)

performBatchUpdates(_ updates, completion)

insertSections(_ sections: IndexSet)
deleteSections(_ sections: IndexSet)
reloadSections(_ sections: IndexSet)
moveSection(_ section: Int, toSection newSection: Int)
```

![image](https://user-images.githubusercontent.com/41438361/125892147-192d9077-13c6-4bc1-98dd-1cdf955ecf65.png)

## Edit distance

이런 연산들을 직접 수행하는 것은 지루하며 에러를 발생시킬 위험이 있다. 우리는 알고리즘을 이용해서 이런 연산을 추상화시킬 수 있다. 그 중 하나는
두 문자열 사이의 거리(한 문자열에서 한 글자씩 수정해서 다른 문자열로 만들 수 있기까지의 단계 수)를 알기 위해 동적 프로그래밍을 사용하는 [Wagner-Fischer 알고리즘](https://en.wikipedia.org/wiki/Wagner%E2%80%93Fischer_algorithm)이다.

Edit distance는 한 문자열이 다른 문자열로 바뀌기 위해 바뀌어야 하는 단계 수를 의미한다. 문자열은 글자들의 collection이니, 이를 아이템의 collection으로
동작하게 일반화시킬 수 있다. 글자를 비교하는 대신, 아이템이 `Hashable`을 따르게 할 수 있다.

### "kit" to "kat"

"kit"이라는 단어를 "kat"으로 어떻게 바꿔야 할까? 어떤 종류의 연산을 해야 할까? "그냥 i를 a로 바꿔"라고 할 수 있겠지만, 이 예시는 알고리즘을 이해할 수 있게
도와줄 것이다.

![image](https://user-images.githubusercontent.com/41438361/125893242-dac38ef0-59af-45e1-9505-473225775b8c.png)

**Deletions**

"kit"을 빈 문자열 ""로 만들기 위해 세 번의 삭제 연산을 수행한다.

![image](https://user-images.githubusercontent.com/41438361/125893221-baad869c-c55c-48d1-a86e-4db3ebda3e8a.png)

“kit” -> “” 👉 3 deletions
“ki” -> “” 👉 2 deletions
“k” -> “” 👉 1 deletion

**Insertions**

빈 문자열 ""에서 "kat"을 만들기 위해, 세 번의 삽입 연산이 필요하다.

![image](https://user-images.githubusercontent.com/41438361/125893292-5da9b0ae-d4d8-4c20-813e-64f3a2003eac.png)

“” -> “k” 👉 1 insertion
“” -> “ka” 👉 2 insertions
“” -> “kat” 👉 3 insertions

**만약 같다면, 왼쪽 상단의 값을 가져온다**

우리가 source 문자열에서 빈 문자열로, 다시 destination 문자열까지 변환했던 알고리즘을 생각해보자. 우리는 업데이트하는 횟수를 최소화할 것이다.
수평 방향으로 가는 것은 삽입을, 수직으로 이동하는 것은 삭제를 의미하며 대각선으로 이동하는 것은 대체하는 것을 의미한다.

이 방법으로 우리는 행, 열마다 반복하는 행렬을 만들 수 있다. 먼저, source collection의 글자 "k"는 destination collection의 단어 "k"와 같으므로
우리는 왼쪽 위에 있는 값을 가져올 수 있고, 이는 0 substitution를 의미한다.

![image](https://user-images.githubusercontent.com/41438361/125893662-3ac170ea-fda3-4d09-a414-f570fb51f7a5.png)


**같지 않을 때**

그리고 desintation collection의 다음 글자로 넘어간다. 여기에서 "k"와 "a"는 같지 않다. 우리는 왼쪽, 위쪽, 그리고 대각선 방향의 왼-위쪽 에서의 최소값을 가져온다.
그리고 1을 증가시킨다.

![image](https://user-images.githubusercontent.com/41438361/125893859-d3f33c37-9e59-46b4-b0a4-baf69119d563.png)

여기서 우리는 왼쪽, 즉 수평 방향에서 값을 가져온다. 그래서 우리는 1 insertion으로 값을 증가시킨다.

**"k"에서 "kat" 👉 2 insertions**

이어서, "t"와 "k"는 같지 않기 때문에, 왼쪽에서 값을 가져온다. "k"에서 "kat"이 되기 위해 우리는 2번의 insertion이 필요하듯이, 이런 과정이 
일어난다고 생각하면 된다.

![image](https://user-images.githubusercontent.com/41438361/125894211-02717581-5bad-41f9-942c-f39554e097a3.png)

**Bottom right value**

다음 행으로 계속 이동하고, 그리고 우리는 edit distance를 나타내는 우측 하단 값을 갖게 될 것이다. 여기서 1 substitution은 우리가 "kit"에서 "kat"으로 가기 위해
1번의 대체 연산을 수행해야 한다는 것을 의미한다.

![image](https://user-images.githubusercontent.com/41438361/125894473-9453c1d9-7197-46b1-a7cf-2440b635d4bc.png)

이제 우리가 인덱스 1을 업데이트 해야 한다는 것을 알 수 있다.

## DeepDiff

이 알고리즘은 2개의 문자 사이의 변화를 보여주는데, 문자열은 글자의 collection으로 생각할 수 있다고 했다. 이 개념을 아이템의 collection을 만드는 데 적용하기 위ㅐ
일반화할 수 있다.

DeepDiff를 구현한 것은 [깃헙](https://github.com/onmyway133/DeepDiff)에서 확인할 수 있다. `old`와 `new` 배열이 주어지면, 변하기 위해
필요한 연산을 계산한다. 이런 변화는 변화의 타입(`insert`, `delete`, `replace`, `move`), 그리고 변하는 `index`로 구성되어 있다.

```swift
let old = Array(“abc”)
let new = Array(“bcd”)
let changes = diff(old: old, new: new)
// Delete “a” at index 0
// Insert “d” at index 2
```

### 복잡도

우리는 `m`과 `n`이 각각 source와 destination collection의 길이를 의미하는 행렬 안에서 반복할 것이다. 그래서 이 알고리즘의 복잡도는 O(mn)이 된다.

또한 성능은 collection의 크기와 아이템이 얼마나 복잡한지에 크게 영향을 받는다.




ddddddsss
dddddsss
