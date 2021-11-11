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
