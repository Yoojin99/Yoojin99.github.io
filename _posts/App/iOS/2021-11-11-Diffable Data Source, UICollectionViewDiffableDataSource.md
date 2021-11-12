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

Ordered collection diffing은 값을 비교할 수 있는 요소들을 가진 컬렉션에 적용된 변화를 파악하고, 변화를 적용하기 쉽게 해준다. 그렇다면 Diffable DataSource는
대략적으로 Datasource에 diffable한 기능을 더한 것이라고 생각해볼 수 있다.

이 Diffable Datasource를 다룬 [WWDC19](https://developer.apple.com/videos/play/wwdc2019/220/)영상이 있다.

이 Diffable Datasource가 등장하게 된 배경을 보자. UITableView/CollectionView를 가지고 작업을 했다면 아래와 같은 코드를 작성한 적이 있을 것이다.

![image](https://user-images.githubusercontent.com/41438361/141249120-89b08aae-3c2d-47be-ae41-98b7a1129ca1.png)

이 메서드들을 통해 섹션의 수와 섹션 안의 아이템의 수를 제공하고, 이 작업은 굉장히 간단하다. 이 메서드들은 간단하기도 하지만, 굉장히 유연하다. 그 이유는
데이터 소스에 특별한 데이터 구조를 사용할 필요가 없기 때문이다. 데이터 소스는 일차원, 좀 더 복잡해지면 이차원의 배열을 사용할 수도 있을 것이다.

하지만 앱은 실제로 일차원이나 이차원 배열보다 훨씬 더 복잡하다. 데이터 소스는 앱 내의 복잡한 controller에 의해 뒷받침 되는 경우도 있다. 또한
컨트롤러들은 굉장히 다양한 작업들을 하는데, Core Data와 상호작용할 수도 있고 web service를 다루고 있을 수도 있다. 그리고 이런 작업들을 굉장히
빨리 시각화하고 싶은 것이다.

데이터를 가져오기 위해 UI layer와 controller layer 사이의 소통을 보면, UI 레이어에서 Controller layer쪽으로 아이템과 cell을 달라고 요청한다.

![image](https://user-images.githubusercontent.com/41438361/141251056-1ced328c-7c88-4fe0-a46f-687d3383e9a8.png)

하지만 이보다 더 복잡한 상황들이 많이 생길 수 있다. 이 controller가 응답을 받는 web service request를 가지고 있다고 해보자. 응답값, 데이터를 받으면
controller layer는 무언가가 변했음을 알린다. 그리고 이런 변화를 UI layer에 바녕해야 한다. 그리고 이런 변화들은 TableView와 CollectionView에 
도 적용되어야 한다. 이런 작업들은 확실히 이전에 봤던 작업보다 복잡하다. 

![image](https://user-images.githubusercontent.com/41438361/141251508-9edbe953-95b9-4bf9-a45b-70a49ba43b0b.png)

그리고 이런 상황에서 아래의 에러를 본 사람들도 많을 것이다.

![image](https://user-images.githubusercontent.com/41438361/141251668-75161882-5ed4-443e-9374-61b128dfb00a.png)

설명을 보니 collection view가 알고 있는 자신의 데이터의 개수와 실제로 주어진 데이터의 개수가 달라 발생한 문제로 보인다. 이 상황에서의 해결법은 reloadData를 호출하는 것이지만,
이러면 애니메이션이 적용되지 않은 상태로 화면이 나타나게 된다. 이는 사용성을 굉장히 떨어뜨린다. 

그렇다면 앞에서 봤던 UI가 들고 있는 정보, 또 실제로 변화한 데이터들을 가지고 있는 Data source(Data controller)가 있다. 그니까 UI도 자신만의
버전을 들고 있고, controller도 자신만의 정보를 들고 있다. 그리고 UILayerCode가 이 둘의 sync를 맞추기 위한 책임이 있다. 하지만 앞에서 봤던 
에러가 뜨는 것처럼 이게 쉽지는 않다. 

그래서 이 발표가 나온 시점에서 이 문제에 대한 최점단 접근 방법은 에러 자체를 방지하는 것이었고, 이는 중앙화된 버전에 대한 개념이 없기 때문이었다.

그리고, 새로운 접근 방법인 **DiffableDataSource**가 나오게 되었다. 

![image](https://user-images.githubusercontent.com/41438361/141253196-96eb0af8-4e30-4241-8591-348781c1de17.png)

DiffableDataSource는

1. PerformBatchUpdates가 없다.
2. 크래시, 혼란, 복잡함, 그리고 처리하고 싶지 않았던 모든 것들이 제거되었다.

위의 사항 대신, Apply라는 단일 메서드를 가지게 되었다. Apply는 간단하고, 자동적이며, 앞에서 말한 복잡한 것들이 사라진 diffing이다.

![image](https://user-images.githubusercontent.com/41438361/141412490-fca59185-c87d-4408-9297-d25e759682c5.png)

대신, Snapshot이라고 부르는 새로운 구조로 이를 수행한다. 스냅샷은 현재 UI 상태의 진실한 버전을 가지고 있다.

그리고 IndexPath대신에 identifier를 사용하는 것 같다. 그래서 update를 IndexPath로 하지 않고, identifier로 하게 된다.

![image](https://user-images.githubusercontent.com/41438361/141412774-1ac54db6-923a-44ff-b1f4-35062d7c9b58.png)

여기 예시가 있다. FOO, BAR, BIF가 화면에 있고, 얘네들은 앱에서의 identifier들이다. 그리고 우리의 컨트롤러가 바뀌었다고 해보자.
그러면 우리는 적용하고 싶은 새로운 Snapshot을 가지게 된다. 새로운 Snapshot과 이전의 Snapshot을 비교해서, 몇 개는 순서가 바뀌고, 새로운
아이템이 들어온 것을 알 수 있다.

그래서 개념적으로는 Apply는 현재의 상태를 알고 새로운 상태를 알고 있다. 그래서 변경된 내용을 적용하면 끝이다. 

iOS에서 TVoS에서는 UICollectionViewDiffableDataSource, 그리고 UITableViewDiffableDataSource가 있다.

그리고 모든 플랫폼에서 공통되는 것은 현재의 UIState에 대한 책임을 지고 있는 NSDiffableDataSourceSnapshot이 있다.

코드를 보면 세 단계로 이를 수행할 수 있다. 내가 Collection View와 UITableView의 data에 변화를 주고 싶다면, Snapshot을 만들면 된다. Snapshot을 내가 update cycle에서 출력하고 싶은 아이템들에 대한 설명과 함께 Snapshot을 생성하면 된다. 그리고 Snapshot을 변화를 UI를 적용하기 위해 apply한다. DiffableDataSource는 UI 요소에 대한 diffing과 변화를 관리한다.

![diff1](https://user-images.githubusercontent.com/41438361/141413978-c327cc7b-0602-4ac6-855d-4b915175c585.gif)

위의 예시는 이 WWDC 발표에서 예시로 발표한 영상인데, 보면 검색하는 단어가 달라질때마다 아래에 나타나는 결과가 애니메이션과 함께 다르게 보여진다.

![image](https://user-images.githubusercontent.com/41438361/141414249-17cca811-b217-4ef8-bbdd-e0ed30089d98.png)

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
