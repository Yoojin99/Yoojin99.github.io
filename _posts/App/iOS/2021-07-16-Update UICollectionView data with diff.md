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

## Edit distance(편집 거리)

이런 연산들을 직접 수행하는 것은 지루하며 에러를 발생시킬 위험이 있다. 우리는 알고리즘을 이용해서 이런 연산을 추상화시킬 수 있다. 그 중 하나는
두 문자열 사이의 거리(한 문자열에서 한 글자씩 수정해서 다른 문자열로 만들 수 있기까지의 단계 수)를 알기 위해 동적 프로그래밍을 사용하는 [Wagner-Fischer 알고리즘](https://en.wikipedia.org/wiki/Wagner%E2%80%93Fischer_algorithm)이다.

Edit distance는 한 문자열이 다른 문자열로 바뀌기 위해 바뀌어야 하는 단계 수를 의미한다. 문자열은 글자들의 collection이니, 이를 아이템의 collection으로 동작하게 일반화시킬 수 있다. 글자를 비교하는 대신, 아이템이 `Hashable`을 따르게 할 수 있다.

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

그리고 desintation collection의 다음 글자로 넘어간다. 여기에서 "k"와 "a"는 같지 않다. 우리는 왼쪽, 위쪽, 그리고 대각선 방향의 왼-위쪽 에서의 최소값을 가져온다. 그리고 1을 증가시킨다.

![image](https://user-images.githubusercontent.com/41438361/125893859-d3f33c37-9e59-46b4-b0a4-baf69119d563.png)

현재 위쪽은 2i, 왼쪽 위는 1i, 왼쪽은 0의 값을 가지고 있다. 그래서 최솟값은 왼쪽의 0이 되는 것이다. 우리는 이 값을 가져올 것이다. 
여기서 우리는 **왼쪽, 즉 수평 방향**에서 값을 가져왔다. 그래서 우리는 1 insertion 으로 값을 증가시킨다. 수평방향의 값을 가져왔으니 insertion이고, 값에서 1을 증가시키므로 1i가 되는 것이다.

**"k"에서 "kat" 👉 2 insertions**

이어서, "t"와 "k"는 같지 않기 때문에, 왼쪽에서 값을 가져온다. "k"에서 "kat"이 되기 위해 우리는 2번의 insertion이 필요하듯이, 이런 과정이 
일어난다고 생각하면 된다.

![image](https://user-images.githubusercontent.com/41438361/125894211-02717581-5bad-41f9-942c-f39554e097a3.png)

**Bottom right value**

다음 행으로 계속 이동하고, 그리고 우리는 edit distance를 나타내는 우측 하단 값을 갖게 될 것이다. 여기서 1 substitution은 우리가 "kit"에서 "kat"으로 가기 위해
1번의 대체 연산을 수행해야 한다는 것을 의미한다.

![image](https://user-images.githubusercontent.com/41438361/125894473-9453c1d9-7197-46b1-a7cf-2440b635d4bc.png)

이제 우리가 인덱스 1을 업데이트 해야 한다는 것을 알 수 있다.

**다른 예제**

아직도 헷갈리니 다른 예제를 가지고 와봤다.

AUERY에서 GARUEY 로 바꾸는 것을 해보자. 아래처럼 초기 행렬을 만들었다. 편의상 i, d는 표시하지 않았다. 

![image](https://user-images.githubusercontent.com/41438361/125907232-5b90c51c-4f6b-4da8-b198-dda1350d943a.png)

위에서도 봤지만, 처음 1d, 2d, 3d,.. 1s, 2s, 3s,,, 는 빈문자열 "" 기준으로 source에서 "", ""에서 dest 문자열이 되기 위해 어떤 연산이 일어나는지를 생각해보면 왜 축이 저 값을 기본으로 설정된 상태에서 시작하는 지 알 수 있다. 

1. 초기 0에서 시작하자. 0은 빈 문자열인 ""을 의미한다.
2. A와 G는 같지 않다. 따라서 왼, 위, 왼위 중 최솟값을 선택한 후 1을 증가시켜야 한다. 최솟값은 왼-위의 0이고, 대각선 방향이므로 1s가 된다.
3. 다음으로, source의 A와 dest의 A를 비교한다. 둘은 같으므로 대각선 왼쪽 위의 값을 그대로 가져온다. 따라서 1s가 될 것이다.
4. 위에서 배웠던 알고리즘을 통해 첫번째 행을 완성한다.
5. 쭉쭉 다음 행까지 완성한다.

![image](https://user-images.githubusercontent.com/41438361/125908776-51d7a11a-e6ab-4328-9eb0-55f9d3ec9656.png)

이렇게 모든 칸을 채웠다. 결론적으로 3번 연산을 해서 AUERY -> GARUEY로 만들 수 있는 것을 알 수 있다. 더 자세히 보기 위해, 아래와 같이
발생하는 연산들을 표시했다. 

![image](https://user-images.githubusercontent.com/41438361/125914388-78cf57ec-745d-4660-a86d-f34e278358da.png)

빨간색은 delete, 파란색은 insert, 초록색은 substitute 로 생각하면 된다. 그리고 최종적으로 따라가는 루트를 보라색으로 체크했는데, 글씨가 개판이라 다시 적으면 아래와 같다.

1. 0 -> 1i
2. 1i -> 1i
3. 1i -> 2i
4. 2i -> 2i
5. 2i -> 2i
6. 2i -> 2i 1d
7. 2i 1d -> 2i 1d

그리고 실제로 2번의 insert, 1번의 delete로 문장이 바뀔 수 있는 지를 확인해 보니 G insert, R insert, R delete 로 문자열이 바뀌는 것을
확인할 수 있었다.

## DeepDiff

이 알고리즘은 2개의 문자 사이의 변화를 보여주는데, 문자열은 글자의 collection으로 생각할 수 있다고 했다. 이 개념을 아이템의 collection을 만드는 데 적용하기 위해 일반화할 수 있다.

![1_w5n7s2u_eXRN_F9DdwIdIA](https://user-images.githubusercontent.com/41438361/125924878-cca85dbb-0c5d-447d-ba5c-afd25f620fbb.gif)

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

또한 성능은 collection의 크기와 아이템이 얼마나 복잡한지에 크게 영향을 받는다. `Equatable`을 얼마나 깊게, 복잡하게 수행할지에 따라서도 성능이 영향을 받게 된다.

Wagner 알고리즘 위키 페이지를 보면, 성능을 향상시키기 위해 알 수 있는 것들에 대한 힌트가 있다.

> 알고리즘이 특정 시간에 이전 행과 현재의 행에 대한 정보만을 요구하므로 우리는 O(mn)대신에 O(m)을 채택해서 알고리즘이 더 적은 공간을 사용하게 할 수 있다.

한 번에 한 행만을 연산하기 때문에 전체 행렬을 저장하는 것은 비효율적이기 때문에, 대신에 우리는 2개의 배열을 가지고 연산하면 된다. 이는 메모리에 접근하는 횟수 또한 줄여줄 것이다.

**Change**

`Change`의 각 case는 상호 배타적이기 때문에, 열거형으로 나타내기 좋은 예시다.

```swift
public enum Change<T> {
  case insert(Insert<T>)
  case delete(Delete<T>)
  case replace(Replace<T>)
  case move(Move<T>)
}
```

* `insert` : 아이템이 해당 인덱스에 삽입됨
* `delete` : 아이템이 해당 인덱스에서 삭제됨
* `replace` : 해당 인덱스의 아이템이 다른 아이템에 의해 대체됨
* `move` : 해당 인덱스의 아이템이 다른 인덱스로 옮겨짐

**2개의 행만 사용하기**

말했듯이, 2개의 행만 가지고 연산을 하면 된다. 행의 각 칸은 변화의 collection이다. 여기서 diff는 문자열을 포함한 `Hashable` 아이템을 가지는
모든 collection을 수용하는 제네릭 함수다.

```swift
public func diff<T: Hashable>(old: Array<T>, new: Array<T>) -> [Change<T>] {
  let previousRow = Row<T>()
  previousRow.seed(with: new)
  let currentRow = Row<T>()
  …
}
```

관심사를 분리하는 것이 좋기 때문에, 각 행은 각각의 상태를 관리하고 있어야 한다. Slot의 배열을 가지는 `Row` 객체를 정의하는 것에서 시작한다.

```swift
class Row<T> {
  /// Each slot is a collection of Change
  var slots: [[Change<T>]] = []
}
```

이로 인해 `previousRow`, `currentRow`는 아래와 같은 구조를 가지게 된다. `Change`를 담는 배열들의 배열이 `slots`이 되는 것이다. 이 `Change`에는 위에서 봤던 1i, 1d 이런 것들이 될 것이다. 

![image](https://user-images.githubusercontent.com/41438361/125916301-efc869ea-ee68-4f45-9294-e9d021c6f65f.png)


행, 열마다 반복했던 알고리즘을 생각해서 2 loop을 사용한다.

```swift

old.enumerated().forEach { indexInOld, oldItem in
  new.enumerated().forEach { index, item in
    
  }
}
```

그리고 old, new 배열에 있는 아이템을 비교하고, `Row`객체에 있는 slot을 업데이트 하면 된다.

**`Hashable` vs `Equatable`**

우리는 객체가 복잡하면 Equatable 메서드가 시간을 잡아먹기 때문에 동일성을 체크할 때 신중히 해야 한다. `Hashable`이 `Equatable`을 따르고, 두개의 같은 객체가 같은 해시 값을 갖고 있는 것을 알 것이다. 그래서 만약 같은 해시 값을 갖지 않으면, equatable 하지 않다라고 할 수 있는 것이다. 역은 성립하지 않지만, 이는 `Equatable` 함수를 호출하는 횟수를 줄이기에 충분하다.

```swift
private func isEqual<T: Hashable>(oldItem: T, newItem: T) -> Bool {
  // Same items must have same hashValue
  if oldItem.hashValue != newItem.hashValue {
    return false
  } else {
    // Different hashValue does not always mean different items
    return oldItem == newItem
  }
}
```

**Move는?**

하지만 여기서 insertion, deletion, replacement에 대한 단계만 업데이트 했지, 이동은 없는 것을 알았을 것이다. 이동은 같은 아이템을
삭제하고 삽입하는 것과 같다. `MoveReducer`를 보면 매우 효율적으로 구현된 것은 아니지만, 힌트를 준다.

**`UICollectionView`의 `IndexPath`언급하기**

`DeepDiff`에 의해 반환된 changes 배열을 가지고, `UICollectionView`가 업데이트를 수행하기 위한 적절한 `IndexPath`의 집합을 알려줄 수 있다.

`Change`에서 `IndexPath`로 바꾸는 것은 자명하다. `UICollectionView` [익스텐션](https://github.com/onmyway133/DeepDiff/blob/master/Sources/iOS/UICollectionView%2BExtensions.swift)을 봐도 된다. 

한 가지를 유의하지 않으면 앞서 봤던 것과 비슷한 `NSInternalInconsistencyException`을 받을 것이다. 바로 `reloadItems`를 `performBatchUpdates` 밖에서 호출해야 한다는 것이다. 이는 이 알고리즘으로 인해 리턴된 `Replace` 단계는 collection이 업데이트 된 이후 상태의 `IndexPath`를 포함하지만, `UICollectionView`는 이 상태 이전의 것을 기대하기 때문이다.

그 외에는 꽤 명료하다. Changes가 얼마나 빠르고 유익한지 알면 놀랄 것이다.

## 사용법

```swift
let oldItems = self.items // 변화하기 이전의 데이터 ("kit")
let items = DataSet.generateItems() // 이렇게 변해야 함 ("kat")
let changes = diff(old: oldItems, new: items) 

let exception = tryBlock {
  // collectionView의 extension을 이용한다.
  self.collectionView.reload(changes: changes, updateData: {
    self.items = items
  })
}
```

사용하려면 위의 코드대로 사용하면 된다. 더 자세한 내용을 보자면, `changes`는 `Changes` 구조체 타입으로, 아래와 같이 구성되어 있다.

Insert, Delete, Replace, Move의 각 연산마다 item(old + new), index가 정의되어 있다.

```swift
public struct Insert<T> {
  public let item: T
  public let index: Int
}

public struct Delete<T> {
  public let item: T
  public let index: Int
}

public struct Replace<T> {
  public let oldItem: T
  public let newItem: T
  public let index: Int
}

public struct Move<T> {
  public let item: T
  public let fromIndex: Int
  public let toIndex: Int
}

/// The computed changes from diff
///
/// - insert: Insert an item at index
/// - delete: Delete an item from index
/// - replace: Replace an item at index with another item
/// - move: Move the same item from this index to another index
public enum Change<T> {
  case insert(Insert<T>)
  case delete(Delete<T>)
  case replace(Replace<T>)
  case move(Move<T>)
}
```

**UICollectionView+Extensions.swift**

```swift
func reload<T: DiffAware>(
  changes: [Change<T>],
  section: Int = 0,
  updateData: () -> Void,
  completion: ((Bool) -> Void)? = nil) {
  
  let changesWithIndexPath = IndexPathConverter().convert(changes: changes, section: section)

  performBatchUpdates({
    updateData()
    insideUpdate(changesWithIndexPath: changesWithIndexPath)
  }, completion: { finished in
    completion?(finished)
  })

  // reloadRows needs to be called outside the batch
  outsideUpdate(changesWithIndexPath: changesWithIndexPath)
}
```

`reload`를 좀 더 자세히 보면, change의 연산들마다 이미 설정되어 있는 `index`로 `indexPath`를 생서하고, 이를 이용해 `performBatchUpdates`에서 연산을 수행한다.

## Where to go from here

이 가이드가 끝나면, 직접 `IndexPath`를 연산해서 `UiCollectionView`를 업데이트 하는 방법을 알았을 것이다. 이런 예외로 고생하면서, 라이브러리의 도움을 받는 것이 얼마나 도움이 되는지를 알았을 것이다. 알고리즘을 알았으니 순수 Swift 언어로 구현할 수 있을 것이다. `Hashable`과 `Equatable`이 어떻게 사용되는지도 봤다.

현재 `DeepDiff`는 선형 시간 안에 수행되고 더 빠른 `Heckel` 알고리즘을 사용한다. [여기](https://github.com/onmyway133/DeepDiff/tree/master/Example/Benchmark)에서 벤치마크를 확인할 수 있다.

![image](https://user-images.githubusercontent.com/41438361/125905172-5642a105-1fe7-401e-a7d7-baa2ba4de813.png)

[IGListKit](https://github.com/instagram/IGListKit) 또한 `Heckel` 알고리즘을 채용하지만, Objective C++로 구현되어 있으며 최적화가 많이 되었다. 다음 글에서는 `Heckel` 알고리즘에 대해, 또 순수 Swift로 구현하는지를 볼 것이다. 

