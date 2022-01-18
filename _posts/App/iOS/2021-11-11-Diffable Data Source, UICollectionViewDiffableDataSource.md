---  
layout: post  
title: "[iOS] - Diffable Data Source, UICollectionViewDiffableDataSource"  
subtitle: ""  
categories: app
tags: app-ios UICollectionViewDiffableDataSource
comments: true  
header-img: 

---  
  
> `Diffable DataSource는 무엇인지, 그리고 UICollectionViewDiffableDataSource는 어떻게 사용하는지`  

---

# Diffable Datasource

Diffable에 관한 내용은 이전에 여러 번 정리해둔 것이 있다. (https://yoojin99.github.io/app/Ordered-Collection-Diffing/)

Ordered collection diffing은 값을 비교할 수 있는 요소들을 가진 컬렉션에 적용된 변화를 파악하고, 변화를 적용하기 쉽게 해준다. 그렇다면 Diffable DataSource는 대략적으로 Datasource에 diffable한 기능을 더한 것이라고 생각해볼 수 있다.

이 Diffable Datasource를 다룬 [WWDC19](https://developer.apple.com/videos/play/wwdc2019/220/)영상이 있다.

## WWDC19 - Advances in UI Data Sources

이 WWDC19 세션에서는 아래의 네 가지를 얘기하고 있다.

1. 현재의 state-of-the-art(최신식). 지금(이 발표가 있던 2019년) data source와 어떻게 상호작용하는가?
2. 새로운 접근법. iOS, tvOS, Mac에 새로 도입한 접근법을 얘기한다.
3. Demo. 새로운 API로 구현한 데모들을 본다.
4. Consideration. 이 API를 최대로 활용하는 방법에 대한 고찰

순서대로 이 발표를 정리해보겠다.

### 1. Current state-of-the-art

UITableView와 collection view들은 UI data source와 어떻게 상호작용할까?

![image](https://user-images.githubusercontent.com/41438361/149882174-406c9918-9a58-458a-9f38-28f5637d58c3.png)

위의 코드와 같이 섹션의 수, 섹션 안에 아이템의 수, 그리고 cell을 요청한다. 직관적인 방법으로, 이를 약 10년 정도 사용했다. 이 방법은 적은 수의 메서드를 작성하고, 빠르게 반복할 수 있으면서도 유연한 방법이다. 왜냐하면 data source에 제공할 데이터 구조가 정해져 있지 않기 때문이다. 일차원 배열이 될 수도 있고, 만약 섹션이 여러개고 아이템도 여러 개라면 아마 이차원 배열이 될 것이다. 

물론 간단하고 직관적이지만, 일차원-이차원 배열을 사용할 때보다 앱은 더 복잡해졌고, 더 복잡해지고 있다. 앱은 더 많은 기능을 수행하고, data source들은 앱 내의 복잡한 controller에 의해 뒷받침된다. 이 controller들은 또 다양한 작업들을 한다. Core Data와 상호작용할 수 있고, 웹 서비스와 통신하는 등 굉장히 많은 작업들을 한다.

그리고 이렇게 data를 불러오기 위해 힘들고 많은 작업을 하는 controller layer와 UI layer 간의 소통을 보면, UI layer는 Controller layer에 "출력할 섹션이나 cell을 줘"라고 요청한다. 

![image](https://user-images.githubusercontent.com/41438361/149883250-a70219fb-4b58-441b-b973-1c71ac32ecb7.png)

위의 상황은 직관적이고, 간단하지만 더 복잡한 상황들이 생기게 된다. 예를 들어 controller가 웹 서비스 요청을 하고 응답을 받았다고 해보자. 이 응답을 받으면 controller layer는 "내 뭔가가 바뀌었다"라고 알려야 한다. 그럼 이제 UI layer가 결정을 해야 하는데, controller가 알린 변화를 UI layer에 업데이트 해야 할 것이다. 이 업데이트에는 table view와 collection view에 적용되어야 할 변화들이 있을 것이다.

![image](https://user-images.githubusercontent.com/41438361/149884025-a963763d-4227-42a1-b2e6-999cbc41084a.png)

이는 복잡한 작업인데, batch update를 적절히 수행하고, backing store를 변경하는 등의 작업 등등이 필요하다. 물론 잘 해결할 수도 있겠지만 이런 때에 아래와 같은 에러를 보게 된다.

![image](https://user-images.githubusercontent.com/41438361/149888409-ca441bde-9a4c-4984-990b-a4ae5d8a02e0.png)

이 문제의 해결방법은 그냥 `reloadData`를 호출하는 건데, 이는 애니메이션을 보여주지 않는다. 그리고 이는 사용성을 떨어뜨린다.

그렇다면 여기서의 문제는 무엇일까? 문제의 모든 답을 가지고 있는 주체는 누구인가? 여기서 문제는 **data source가 시간에 따라 바뀌는 자신만의 truth를 가진다는 것이다. UI도 자신만의 truth를 가지고 있다.** 그리고 UILayerCode는 이를 완화시켜서 모든 것이 동기화 되어 있도록 하는 책임이 있다. 하지만 위에서 에러가 뜨는 걸 봤듯이 이게 어려울 때도 있다. 그래서 현재의 접근법은 에러를 발생시키는 경향이 있다. 그리고 이는 **중앙화된 truth라는 개념이 없기 때문이다.**

![image](https://user-images.githubusercontent.com/41438361/149888780-29775015-2784-4f14-ad87-84a1193f2050.png)

### 2. A new approach

위에서 최신식의 접근법을 봤고, 발생할 수 있는 문제들을 봤다. 여기에서 새로운 접근법인 **DiffableDataSource**가 등장하게 된다.

DiffableDataSource는 `performBatchUpdates`가 없기 때문에 이와 관련된 크래시, 복잡성, 그리고 다루지 않고 싶었던 것들도 함께 버려졌다. 대신, `apply`가 등장했다. Apply는 간단하고, 자동적이며 혼란이 없는 diffing이다.

![image](https://user-images.githubusercontent.com/41438361/149889722-847420f5-09b2-4e75-942d-8608fb682954.png)

그리고 이를 **Snapshot**이라 부르는 새로운 구조로 이를 수행한다. Snapshot은 굉장히 간단한 아이디어로, **현재 UI 상태의 truth이다.**
또 snapshot은 IndexPath 대신 모두 유일한 섹션 identifier들과 item identifier들의 연결 또는 컬렉션이다. 그래서 IndexPath로 업데이트하지 않고, identifier로 업데이트하게 된다.

![image](https://user-images.githubusercontent.com/41438361/149890402-f062c7e1-a40d-404e-9478-6737ab8c4f1b.png)

어떤 일이 일어나는 지를 시각화해봤다. 처음에 FOO, BAR, BIF가 화면에 있고, 이는 앱 내의 identifier들이다.

![image](https://user-images.githubusercontent.com/41438361/149890570-17804be0-ca99-4ceb-af1d-186a46aa9e86.png)

그리고 이제 우리의 controller가 변했다. 그리고 아래와 같이 적용하고 싶은 새로운 Snapshot이 생겼다. 어떻게 이 새로운 truth를 현재의 Snapshot에 가져올 수 있을까? 보면 새로운 스냅샷은 BAR, FOO, BAZ로 되어 있고 몇몇은 순서가 바뀌어 있는 걸 볼 수 있다.

![image](https://user-images.githubusercontent.com/41438361/149890902-20e5b589-df6e-41f6-b27c-2d0a7ae6a49e.png)

개념적으로 Apply는 현재의 state를 알고 있고 새로운 state에 있는 걸 알고 있다. 그리고 단순히 그 차이를 적용해서 아래와 같이 변경사항을 적용한다.

![image](https://user-images.githubusercontent.com/41438361/149891266-37cc9370-992c-4efc-80d4-e1313a9def3c.png)

그럼 이를 어떻게 사용할까?

* iOS, TVoS : UICollectionViewDiffableDataSource, UITableViewDiffableDataSource
* Mac : NSCollectionViewDiffableDataSource

그리고 모든 플랫폼에는 현재의 UIState NSDiffableData SourceSnapshot를 책임지는 Snapshot 클래스가 있다.

### 3. Demos

이제 코드를 보자. [프로젝트는 여기에서 받을 수 있다.](https://developer.apple.com/documentation/uikit/views_and_controls/collection_views/implementing_modern_collection_views)

이 데모에서는 DiffableDataSource로 세 개의 다른 예시를 구현했다. 세 개의 예시를 하나씩 차례로 볼 텐데, 모두 세 단계로 이루어져 있다. 변화를 주고 싶을 때, Collection view나 UITableview에 full data source와 함께 새로운 데이터를 넣고 싶다면 Snapshot을 생성하면 된다. Update cycle에 출력하고 싶은 아이템들에 대한 설명과 함께 Snapshot을 생성한다. 그리고 Snapshot을 적용해 자동으로 UI에 수정사항이 적용되도록 한다. DiffableDataSource는 UI 요소에 변화를 일으키는 모든 diffing과 변화들을 관리한다.

#### 1. Mountain Search

이 예시는 상단의 검색창에 산 이름을 검색하면 검색하는 단어에 따라 애니메이션으로 밑에 검색 결과가 나오는 예시다. 이 작업은 자동적이며 애니메이션과 함께 일어난다. 

![diff1](https://user-images.githubusercontent.com/41438361/141413978-c327cc7b-0602-4ac6-855d-4b915175c585.gif)




### 4. Considerations

### 1. 새로운 NSDiffableDataSourceSnapshot 생성

새로운 NSDiffableDataSourceSnapshot을 생성한다. 얘는 초기에는 비어있고 아무것도 가지고 있지 않다. 그래서 우리가 원하는 섹션과 아이템을 채우는 것은 우리가 해야 한다. 그리고 여기에서는 출력할 섹션이 하나밖에 없다. 여기서는 main이라는 섹션을 추가한다. 그리고 우리가 이 업데이트에서 출력하고 싶은 아이템들의 identifier들을 추가한다. 보통 여기에서는 identifier들의 배열을 전달하지만, Swift에서는 편의상 내가 생성한 나만의 타입들을 전달할 수도 있다. Struct나 enum과 같은 값 타입이더라도, hashable하게 만들면 나만의 객체를 전달할 수도 있다.

Snapshot은 제네릭 클래스이기 때문에, SectionIdentifierType과 ItemIdentifierType으로 인수를 받게 되어있다. 그래서 SectionIdentifierType을 먼저 보면, 이걸 위한 enum을 선언해도 된다. Enum은 자동으로 hashable하기 때문에, 그냥 enum을 선언해서 사용하면 된다.

그리고 mountaintype도 보면, mountain도 struct로 선언했다. Struct type도 hashable하기 때문에, identifier로 DiffableDataSource에
전달하기 보다는 그 자체를 전달하는 것이 좋다. 여기서 중ㅇ한 것은 각 mountain은 각각의 hash 값으로 uniquely identifiable하기 때문에
mountain을 자동으로 unique identifier로 생성하는 것이 가능해진다.


## 2. DiffableDataSource에게 snapshot으로 변화를 애니메이션으로 표현하는 것을 처리하게 한다.

이 예제에서는 collectionview로 작업을 하고 있기 때문에, UICollectionViewDiffableDataSource를 section과 item type으로 생성한다.
그리고 collectionView에 pointer를 제공한다. 이러면 DiffableDataSource는 이 포인터를 가지고 자동으로 collectionView의 데이터 소스와 자기자신을 엮을 것이다. 

DiffableDataSource는 자동으로 이전 업데이트와 다음 업데이트 사이에 무엇이 바뀌었는지를 찾아낸다. 

## 3. 

![image](https://user-images.githubusercontent.com/41438361/141416669-3a105685-d6be-4407-a2ed-1ad2d33933d5.png)

CollectionView에 콜백을 통해 데이터를 출력하고 싶은 적절한 타입의 cell을 요청한다. 그리고 이 cell을 생성하고 리턴한다. 그래서 이건 cellForItemAt IndexPath 코드를 우리가 data source를 생성할 때 전달하는 클로저안에 심는 것이랑 같은 작업이다. 

그래서 정리하자면 세 단계는 아래와 같이 정리할 수 있다. 

1. 스냅샷 생성
2. 스냅샷 구성
3. Apply

이제는 performBatchUpdates를 더 이상 호출하지 않고 apply 메서드를 호출하면 된다. 

스냅샷을 만드는데는 두 가지 방법이 있다.

1. 빈 스냅샷 생성 -> 스냅샷 section과 item의 타입을 지정해서 생성한다.
2. 현재 상태의 스냅샷에서 하나를 생성 -> 변화가 작은 것일 때 이를 사용하면 좋다.

![image](https://user-images.githubusercontent.com/41438361/141419884-dbe0eab9-afef-4ae3-8e9f-13c6cfb291c3.png)




이번에는 collection view에 cell을 제공하고 데이터를 관리하는데 사용할 수 있는 `UICollectionViewDiffableDataSource`에 대해 알아보려고 한다.
얘는 iOS 13이상부터 사용 가능하다.

```swift
@MainActor class UICollectionViewDiffableDataSource<SectionIdentifierType, ItemIdentifierType> : NSObject where SectionIdentifierType : Hashable, ItemIdentifierType : Hashable
```

보니 Hashable한 SectionIdentifierType, ItemIdentifierType을 가지고 있다. Hashable한 성질을 이용해서 diffing을 계산하는 것 같다.

*Diffable data source* 객체는 collection view 객체와 같이 동작하는 특별한 타입의 data source다. Collection view의 데이터와 UI를
간단하고, 효율적인 방법으로 관리하기 위해 필요한 행위들을 제공한다. 또한 `UICollectionViewDataSource` 프로토콜을 채택하고 있어서 이 프로토콜의 메서드들도
사용이 가능하다.

Collection view를 데이터로 채우려면 아래와 같이 하면 된다.

1. Diffable data source를 collection view에 연결한다.
2. Collection view의 cell들을 구성할 cell provider를 구현한다.
3. 데이터의 현재 상태를 생성한다.
4. 데이터를 UI에 출력한다.

Diffable data source를 collection view에 연결하려면, diffable data source를 `init(collectionView: cellProvider: )` 이니셜라이저를
사용해서 생성하고, 이 데이터 소스에 연결하고 싶은 collection view에 전달하면 된다. 또한 UI에 cell을 구성하는 cell provider에도 전달한다.

```swift
dataSource = UICollectionViewDiffableDataSource<Int, UUID>(collectionView: collectionView) {
    (collectionView: UICollectionView, indexPath: IndexPath, itemIdentifier: UUID) -> UICollectionViewCell? in
    // Configure and return cell.
}
```

그리고 스냅샷을 만들고 적용해서 데이터의 현재 상태를 생성하고 데이터를 출력한다. 더 자세한 내용은 `NSDiffableDataSourceSnapshot`을 참고하면 된다.

**Diffable data source로 collection view를 구성하고 나서 dataSource를 변경하면 안 된다. 만약 collection view가 내가 초기에 설정한 데이터 소스 말고
새로운 데이터 소스가 필요하다면, 새로운 collection view와 diffable data source를 만들어서 구성해야 한다.**

* 참고
* https://developer.apple.com/videos/play/wwdc2019/220/?time=2097
