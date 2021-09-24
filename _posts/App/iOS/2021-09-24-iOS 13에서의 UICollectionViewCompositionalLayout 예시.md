---  
layout: post  
title: "[iOS] - iOS 13에서의 UICollectionViewCompositionalLayout 예시"  
subtitle: ""  
categories: app
tags: app-ios UICollectionViewCompositionalLayout
comments: true  
header-img: 

---  
  
> `Move your cells left to right, up and down on iOS 13`  

---

[Move your cells left to right, up and down on iOS 13 — Part 1
](https://medium.com/shopback-tech-blog/move-your-cells-left-to-right-up-and-down-on-ios-13-part-1-1a5e010f48f9)
[Move your cells left to right, up and down on iOS 13 — Part 2
](https://medium.com/shopback-tech-blog/move-your-cells-left-to-right-up-and-down-on-ios-13-part-2-fbc430802227)

복잡한 데이터의 컬렉션을 아주 간단히 구성할 수 있는 두 가지 방법이 있다.

아래와 같은 화면을 보면, 화면 전체는 수직으로 스크롤이 가능한데, 맨 위에 있는 몇 행들은 가로로 스크롤이 가능한 것을 볼 수 있다.

![1_ofe8nyZsPONYzgBvE0drSg](https://user-images.githubusercontent.com/41438361/134630212-6fd58228-3dc2-4722-a934-44d58c9c35c1.gif)


이렇게 가로와 세로로 스크롤이 되는 형태의 인터페이스를 구현하기 위해서는 iOS 13이전에는 `UICollectionView`를 중첩해야 했다. 이때 안의 `UICollectionView`를 밖의 `UICollectionView`의 셀로 지정해야 했을 것이다.(물론 밖의 뷰를 `UITableView`로 지정해줘도 된다.) 이런 구조는 괜찮지만, 각 collection view의 **datasource / delegate**를 지정해줘야 하기 때문에 코드가 지저분해 질 수 있다.

iOS 13은 이런 종류의 인터페이스를 구현할 수 있는 두 가지의 간단한 방법을 제시한다.

하나는 아주 멋진 **SwiftUI**를 쓰는 것이다. SwiftUI는 Swift로 사용자 인터페이스 코드를 작성할 수 있는 선언형 도메인 특화 언어이며, 우리가 오랜 시간동안 iOS 인터페이스를 구현하는 방법을 바꿨다고도 여겨진다.

다른 것은 덜 멋지지만(표현이 그렇다) 스코프가 크지 않은 상황에서도 적용하기 좋은 방법이다. iOS 13은 **`UICollectionViewComopositionalLayout`**이라는 `UiCollectionView`에 적용할 수 있는 새로운 레이아웃을 소개한다. 이 레이아웃은 각 섹션마다 다른 레이아웃을 쉽게 제공할 수 있게 해준다.

## Sample Demo

아래와 같은 화면을 만들어볼 것이다. 마음에 드는 예시다.

![cat](https://user-images.githubusercontent.com/41438361/134630235-b933819e-3ec3-4282-8cb4-f691e640ce58.gif)


간단한 고양이 사료 광고 앱이다. 첫 번째 섹션에는 브랜드 이름의 리스트를 가로 스크롤로 보여줄 것이고, 두 번째 섹션에는 마찬가지로 가로 스크롤로 캔의 이미지들을 보여줄 것이다. 마지막 섹션은 고양이들이 음식을 먹고 있는 귀여운 사진들을 세로로 스크롤링 해서 보여줄 것이다.

## UICollectionViewCompositionalLayout

먼저, `UICollectionViewCompositionalLayout`이 뭔지부터 알아보자. 이미 `UICollectionView`, `UICollectionViewFlowLayout`이 무엇인지는 알고 있다고 가정할 것이다. 위의 화면 예제에서는 

* 브랜드 이름을 담고 있는 배열
* 고양이 음식 이미지를 담고 있는 배열
* 고양이 이미지를 담고 있는 배열

이 있다. 그리고 datasource는 배열에 있는 아이템마다 하나의 `UICollectionViewCell` 인스턴스를 제공한다. 그러면 collection view는 셀들을 세로 축을 따라 레이어링 하기 위해 `UICollectionViewFlowLayout`을 사용한다. 그리고 결과는 아래와 같을 것이다.

![cat2](https://user-images.githubusercontent.com/41438361/134630251-97b28eae-ea93-49af-8ae8-95a1c173ed71.gif)

전체 뷰는 세로 축으로 잘 스크롤링 되지만, 처음 두 섹션은 컨텐츠의 가로가 뷰의 가로와 일치하기 때문에 가로로 스크롤링 할 수 없는 상태다. 

### Constructor

이 레이아웃을 `UICollectionViewCompositionalLayout`으로 바꾸기 위해, 먼저 `UICollectionViewCompositionalLayout` 인스턴스를 만들어준다.

```swift
private lazy var compositionalLayout: UICollectionViewCompositionalLayout = {
    let layout = UICollectionViewCompositionalLayout { [weak self]
        (sectionIndex: Int, layoutEnvironment: NSCollectionLayoutEnvironment) -> NSCollectionLayoutSection? in
        switch Section(rawValue: sectionIndex) {
        case .brandNames:
            return self?.setupBrandNamesSection()
        case .catFoods:
            return self?.setupCatFoodsSection()
        case .cats:
            return self?.setupCatsSection()
        case .none:
            fatalError("Should not be none ")
        }
    }
    return layout
}()

override func viewDidLoad() {
    super.viewDidLoad()

...
    collectionView.collectionViewLayout = compositionalLayout
...
}
```

Delegate 패턴을 사용하게끔 강제했던 전통적인 UITableVIew/UICollectionView API와는 다르게, 우리는 `UICollectionViewCompositionalLayout`을 생성할 때 클로저를 제공할 수 있다. **클로저는 섹션의 인덱스와 environment를 받고, 특정 섹션의 레이아웃을 리턴해준다.** switch 구문에서 우리는 `setup...Section()` 메서드를 통해 레이아웃 정보(나 사이즈 정보)를 반환한다.

#### Brand names section

```swift
func setupBrandNamesSection() -> NSCollectionLayoutSection {
    // 1. Creating section layout. Item -> Group -> Section
    // Item
    let item = NSCollectionLayoutItem(
        layoutSize: NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                           heightDimension: .fractionalHeight(1.0)))
    item.contentInsets = NSDirectionalEdgeInsets(top: 0.0,
                                                 leading: 8.0,
                                                 bottom: 0.0,
                                                 trailing: 8.0)

    // Group
    let group = NSCollectionLayoutGroup.vertical(
        layoutSize: NSCollectionLayoutSize(widthDimension: .estimated(136),
                                           heightDimension: .absolute(44)),
        subitem: item,
        count: 1)

    // Section
    let section = NSCollectionLayoutSection(group: group)
    section.contentInsets = NSDirectionalEdgeInsets(top: 16.0,
                                                    leading: 0.0,
                                                    bottom: 16.0,
                                                    trailing: 0.0)

    // 2. Magic: Horizontal Scroll.
    section.orthogonalScrollingBehavior = .continuousGroupLeadingBoundary

    // 3. Creating header layout
    section.boundarySupplementaryItems = [headerViewSupplementaryItem]

    return section
}
```

```swift
private lazy var headerViewSupplementaryItem: NSCollectionLayoutBoundarySupplementaryItem = {
    let headerViewItem = NSCollectionLayoutBoundarySupplementaryItem(
        layoutSize: NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                           heightDimension: .absolute(44)),
        elementKind: UICollectionView.elementKindSectionHeader,
        alignment: .top)
    headerViewItem.pinToVisibleBounds = true

    return headerViewItem
}()
```

`setBrandNamesSection` 메서드의 첫 부분에서 우리는 섹션의 레이아웃을 구성한다. **섹션은 항상 아이템을 포함한 그룹들을 포함하고 있다.** 이 레이아웃 계층은 아래와 같이 나타낼 수 있다.

![image](https://user-images.githubusercontent.com/41438361/134630329-bff5c9ba-e6dd-42a9-8d98-b56e7ef962f9.png)

이 섹션에서 우리가 한 줄짜리 구조를 가지고 있기 때문에 그룹과 아이템은 단일할 것이다. Grouping에 대해서는 다음 부분에서 더 알아볼 것이다.

이 예제에서 주의해야 할 것은 바로 사이즈를 정의하는 부분이다. 아이템에는 `NSCollectionLayoutDimension.fractionalWidth/Height`을 이용해서 사이즈를 정의한다.

```swift
NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),       
                       heightDimension: .fractionalHeight(1.0))
```

이는 **아이템이 부모 요소의 전체 가로와 세로 값을 채택한다는 것이다.** 여기서 부모는 그룹이 될 것이다.

그리고 그룹은 `NSCollectionLayoutDimension.estimated/absolute`을 사용해서 사이즈를 정의한다.

```swift
NSCollectionLayoutSize(widthDimension: .estimated(136),
                       heightDimension: .absolute(44))
```

`absolute(44)`는 그룹의 높이를 정확히 44 포인트로 지정하겠다는 뜻이다. `estimate(136)`은 그룹의 초기 너비를 136 포인트로 하지만, 컨텐츠 사이즈에 따라 달라질 수도 있다는 뜻이다. (이 부분은 항상 예상한대로만 동작하지는 않는다.)

이제 가로 스크롤을 위해서 한 줄만 더해주면 된다.

```swift
section.orthogonalScrollingBehavior = .continuousGroupLeadingBoundary
```

이 한 줄이 컬렉션 뷰의 스크롤 방향에 직교 방향으로 스크롤 할 수 있게 해준다.(만약 컬렉션 뷰의 메인 스크롤 방향이 가로 방향이었다면, 이 섹션은 수직으로 스크롤 가능했을 것이다.) 
`.continuousGroupLeadingBoundary`은 아이템이 연속적으로 스크롤링되지만, 사용자의 손가락이 화면에서 떨어졌을 때 가장 가까운 그룹의 시작하는 엣지로 휙 지나간다는 뜻이다. 더 많은 동작을 보려면 [여기](https://developer.apple.com/documentation/uikit/uicollectionlayoutsectionorthogonalscrollingbehavior)를 참조하면 된다. 

이제 섹션에서 필요한 부분은 거의 다 됐지만, 헤더 부분을 추가해준다. 섹션 헤더와 푸터 레이아웃은 `NSCollectionLayoutSupplementaryItem`을 사용해서 정의해준다. 사이즈를 정의하기 위해서 이제 위에서 본 `fractionalWidth/Height/estimated/absolute`를 사용하면 된다.

```swift
NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),   
                       heightDimension: .absolute(44))
```

위의 코드를 작성해서, 헤더가 섹션의 전체 가로의 길이를 채택하고, 높이는 44임을 명시적으로 나타낸 것이다. 보통과 다른 레이아웃을 원하지 않는 한 정렬은 헤더는 `.top`, 푸터는 `.bottom`이 되어야 한다.

```swift
headerViewItem.pinToVisibleBounds = true
```

`pinToVisibleBounds`를 사용해서 sticky 헤더를 만들 수도 있다.

최종적으로 헤더는 아래와 같이 동작한다.

![cat3](https://user-images.githubusercontent.com/41438361/134630288-08e285a1-8f5d-4806-81f3-4d5866afcabc.gif)

스크롤링하는 걸 보면 우리가 `continuousGroupLeadingBoundary`를 명시했기때문에 그룹의 시작하는 경계선에서 멈춘다는 것을 확인할 수 있을 것이다.

다시 정리하자면 `UICollectionViewCompositionLayout` 에는 세 가지 다른 클래스, 개념이 있었다.

* NSCollectionLayoutItem
* NSCollectionLayoutGroup
* NSCollectionLayoutSection

`NSCollectionLayoutItem`은 각 `UICollectionViewCell`의 사이즈를 나타내고, `NSCollectionLayoutSection`은 섹션의 사이즈를 나타낸다.
`NSCollectionLayoutGroup`은 각 섹션 내에 유동적인 레이아웃을 생성할 수 있게 해준다.

#### Cat Foods Section

```swift
func setupCatFoodsSection() -> NSCollectionLayoutSection {
    // 1. Configuring Section Layout. Item -> Group -> Section
    // Item
    let item = NSCollectionLayoutItem(
        layoutSize: NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                           heightDimension: .fractionalHeight(0.5)))
    item.contentInsets = NSDirectionalEdgeInsets(top: 4.0,
                                                 leading: 0.0,
                                                 bottom: 4.0,
                                                 trailing: 0.0)

    // Group
    let group = NSCollectionLayoutGroup.vertical(
        layoutSize: NSCollectionLayoutSize(widthDimension: .fractionalWidth(0.5),
                                           heightDimension: .absolute(240)),
        subitem: item,
        count: 2)

    // Section
    let section = NSCollectionLayoutSection(group: group)
    section.contentInsets = NSDirectionalEdgeInsets(top: 16.0,
                                                    leading: 0.0,
                                                    bottom: 16.0,
                                                    trailing: 0.0)

    // 2. Horizontal Scroll
    section.orthogonalScrollingBehavior = .groupPaging

    // 3. Header
    section.boundarySupplementaryItems = [headerViewSupplementaryItem]

    return section
}
```

첫 번째 섹션에서 했던 것과 굉장히 비슷한데, 두 가지 차이가 있다. 첫 번째는 그룹에서 두 아이템을 정의한다는 것이다.

```swift
let group = NSCollectionLayoutGroup.vertical(layoutSize: ...,
                subitem: item,
                count: 2)
```

이는 수직 방향으로 두 개의 인접한 아이템을 그룹으로 묶겠다는 뜻이다. 그래서 구조는 아래와 같이 될 것이다.

![image](https://user-images.githubusercontent.com/41438361/134631008-eec6f3e5-7a68-44b6-80aa-46ba903e0fa8.png)

이렇게 라인 수를 늘리는 것이 그룹핑의 가장 기본적인 쓰임새일 것이다. 그룹은 컬렉션 뷰에서 굉장히 다양한 레이아웃으로 구성될 수 있다. 자세한 내용을 보려면 [영상](https://developer.apple.com/videos/play/wwdc2019/215/)을 참고하자. 

다음은 가로로 스크롤하는 행동에서 차이가 있다.

```swift
section.orthogonalScrollingBehavior = .groupPaging
```

이 섹션에서, 스크롤은 그룹의 시작하는 경계에서 멈춘다. 그래서 결과는 아래와 같이 나올 것이다.

![cat4](https://user-images.githubusercontent.com/41438361/134631313-26256374-5f1a-4e8b-97ff-b008bfd9f1c3.gif)

#### Cats Section

마지막 고양이 section은 아래와 같이 구성한다.

```swift
func setupCatsSection() -> NSCollectionLayoutSection {
    // 1. Configuring Section. Item -> Group -> Section
    // Item
    let item = NSCollectionLayoutItem(
        layoutSize: NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                           heightDimension: .fractionalHeight(1.0)))
    item.contentInsets = NSDirectionalEdgeInsets(top: 6.7, leading: 10.0, bottom: 6.7, trailing: 10.0)

    // Group
    let group = NSCollectionLayoutGroup.vertical(
        layoutSize: NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                           heightDimension: .fractionalWidth(0.67)),
        subitem: item,
        count: 1)

    // Section
    let section = NSCollectionLayoutSection(group: group)
    section.contentInsets = NSDirectionalEdgeInsets(top: 16.0,
                                                    leading: 0.0,
                                                    bottom: 16.0,
                                                    trailing: 0.0)

    // 2. Header
    section.boundarySupplementaryItems = [headerViewSupplementaryItem]

    return section
}
```

구조는 아래와 같이 된다.

![image](https://user-images.githubusercontent.com/41438361/134631421-6f432356-e172-4b4f-bd3f-08583c35374d.png)

여기서 주의할 점은 원래 이미지의 aspect ratio(3:2)를 유지하고 싶어서 그룹의 높이를 `fractionalWidth`를 사용해서 지정한 점이다.

```swift
NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                       heightDimension: .fractionalWidth(0.67))
```

이제 SwiftUI를 사용한 방법도 볼 것이다.

## SwiftUI

SwiftUI 의 장점으로 말 할 수 있는 것은 코드 자체가 뷰 계층을 굉장히 잘 설명할 수 있다는 것이다. 구조는 아래와 같이 된다.

![image](https://user-images.githubusercontent.com/41438361/134631713-0a130255-cd83-457e-9683-91dced8a88c3.png)

전체 뷰는 `VSStack`이고 안에 `ScrollView`를 가지고 있다. 기본 아이디어는 `ScrollView`에 `VStack`을 넣으면 가로로 스크롤 할 수 있다는 것이고, `HStack`을 넣으면 세로로 스크롤 할 수 있다는 것이다.

```swift
ScrollView {
    VStack(alignment: .leading, spacing: 16) {
        // MARK: - Brand Names Section
        HeaderView(headerText: "Brand Names")
        ScrollView(showsHorizontalIndicator: false) {
            HStack {...}
        }
            .frame(height: 64)


        // MARK: - Cat Foods Section
        HeaderView(headerText: "Cat Foods")
        ScrollView(showsHorizontalIndicator: false) {
            HStack {...}
        }
            .frame(height: 120)

        // MARK: - Cats Section
        HeaderView(headerText: "Cats")
        ForEach(cats.identified(by: \.self)) {...}
    }
}
```

위의 코드를 읽는 것만으로도 구조를 파악하기 쉽다. 

브랜드 이름을 정의한 `HStack` 부분은 아래와 같다.

```swift
HStack {
    ForEach(brandNames.identified(by: \.self)) {
        Text("\($0)")
            .frame(width: UIScreen.main.bounds.width / 3 - 30)
            .padding(10)
            .border(Color.black, width: 1, cornerRadius: 22)
    }
}.padding(10)
```

`ForEach`는 데이터 리스트에서 리스트 뷰를 생성하는 UI 요소이다. 이 안에서 데이터의 구분자로 설정할 수 있는 프로퍼티를 설정해주기만 하면 된다.
여기에서 데이터가 문자열이기 때문에, 문자열 자체를 구분자로 설정했다.

`Text` 요소는 `UILabel`과 같은 UI요소를 생성하고, `border(...)` 메서드를 호출해서, 라벨에 둥근 모서리를 적용할 수 있다.

고양이 음식을 나타낸 `HStack`은 아래와 같다. 

```swift
HStack {
    ForEach(stride(from: 0, to: catFoods.count, by: 2).map{ $0 }) { i in
        VStack {
            Image(catFoods[i])
                .resizable()
                .aspectRatio(contentMode: .fit)
                .frame(width: UIScreen.main.bounds.width / 2 - 40, height: 100)
                .padding(EdgeInsets(top: 0, leading: 10, bottom: 0, trailing: 10))
            Image(catFoods[i + 1])
                .resizable()
                .aspectRatio(contentMode: .fit)
                .frame(width: UIScreen.main.bounds.width / 2 - 40, height: 100)
                .padding(EdgeInsets(top: 0, leading: 10, bottom: 0, trailing: 10))
        }
    }
}.padding(10)
```

`ForEach`안에서 `(i, i+1)` 번째 이미지 데이터를 받아 `VStack`에 넣어서 한 열에 두 이미지를 가질 수 있게 했다.

`resizable().aspectRatio(contentMode: .fit)`는 이미지가 프레임 사각형에 적절히 맞게 배치될 수 있게 프레임을 설정하기 전에 호출되어야 한다. SwiftUI에서는 보통 이런 요소를 배치하는 순서가 차이를 만들어낸다.

마지막 섹션은 비슷하게 구현하면 된다.

하지만 UIKit을 이용하면 가로로 스크롤링할 때 컬렉션 아이템의 시작하는 경계에서 멈추지 않는다는 것을 확인할 수 있다. 여기에는 두 가지
요소가 이를 방해하고 있다. 먼저, `UICollectionViewCompositionalLayout`가 하는 것처럼 SwiftUI에서 `ScrollView`가 할 수 있는 
편리한 방법이 제공되지 않는다. 그리고 우리가 `ScrollView`의 offset을 얻을 수 없고, scroll을 방해할 `onScroll` 타입의 콜백을 `ScrollView`에 설정할 수 없기 때문이다. 

이런 단점들도 있지만, 코드가 간결해지면서 같은 동작을 구현할 수 있다는 것이 장점이다.
