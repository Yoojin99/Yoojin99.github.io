---  
layout: post  
title: "[iOS] - UICollectionView란?"  
subtitle: ""  
categories: app
tags: app-ios uicollectionview
comments: true  
header-img: 

---  
  
> `UICollectionView에 대해 자세히 알고 넘어가자`  

---

UICollectionView 없이 근사한 UI를 가진 앱을 만들기가 힘들다. UICollectionView를 이용해서 화면을 구현하는 데 헷갈리는 점도 많고 막히는 점도 많아 
이 기회에 개념과 막히는 부분들을 중점으로 짚고 넘어가 보려고 한다.

# [UICollectionView](https://developer.apple.com/documentation/uikit/uicollectionview?language=objc)

https://developer.apple.com/documentation/uikit/uicollectionview?language=objc

공식문서에 따르면, UICollectionView는 데이터 아이템들의 순서가 있는 컬렉션을 관리하고, 이들을 커스터마이징이 가능한 레이아웃을 사용해서 출력하는 객체다.
즉 순서 개념이 있는 아이템들을 관리하고, 레이아웃을 통해 출력하는데 이 레이아웃을 사용자가 마음대로 구현할 수 있다는 뜻이다.

컬렉션 뷰를 사용자 인터페이스에 추가할 때, 앱이 해야 하는 주요 작업은 컬렉션 뷰와 관련있는 데이터를 관리하는 것이다. 컬렉션 뷰는 자신의 [`dataSource`](https://developer.apple.com/documentation/uikit/uicollectionview/1618091-datasource?language=objc) 프로퍼티에 저장된 데이터들을 가져온다.
Data source에는 [`UICollectionViewDiffableDataSource`](https://developer.apple.com/documentation/uikit/uicollectionviewdiffabledatasource?language=objc)를 사용할 수 있는데,
이 `DiffableDataSource` 객체는 iOS 13 이상부터 사용이 가능하다. 이 객체는 컬렉션 뷰의 데이터에 변경사항이 생겼을 때 뷰의 데이터와 사용자 인터페이스에 생기는 변화를 효과적으로 관리하게 해준다.
대신, `UICollectionViewDataSource` 프로토콜을 채택해서 커스텀 data source 객체를 만들 수 있다. 

컬렉션 뷰의 데이터는 화면에 출력할 때 섹션으로 묶어 그룹으로 나눌 수 있는 개별 아이템으로 구성된다. 아이템은 출려가고 싶은 데이터의 가장 작은 단위이다. 예를 들어
사진 앨범과 같은 앱에서 아이템은 사진이 될 것이다. 컬렉션 뷰는 data source가 구성하고 제공하는 `UICollectionViewCell`의 인스턴스인 cell을 사용해서 아이템을
화면에 출력한다. 

![image](https://user-images.githubusercontent.com/41438361/133023765-c68e97a7-366c-482e-938e-b4c59f6916a2.png)

Cell에 추가로, 컬렉션 뷰는 다른 타입의 뷰를 사용해서 데이터를 출력할 수도 있다. 이런 보조적인 뷰(위의 사진에서 supplementary view로 나와있는 부분이다.)들은 예를 들어 개별 cell과는
다르지만 여전히 정보를 포함하고 있는 섹션 헤더와 푸터가 될 수 있다. 위의 그림에서도 사진들은 개별 아이템인 cell로 출력이 되고 있고, 금색 제목 영역인 characters 나 items는
섹션 헤더로 보조적인 역할을 수행하는 뷰라고 할 수 있다. 이런 보조적인 뷰를 지원하는 것은 선택적이며 컬렉션 뷰의 레이아웃 객체에 의해 정의된다. 이 레이아웃 객체는
이런 뷰들의 배치까지도 정의한다.

![image](https://user-images.githubusercontent.com/41438361/133024739-e953e73a-9c2c-4de7-8b48-8777adc8341f.png)

`UICollectionView`를 사용자 인터페이스에 임베딩할 때, 화면에 출력되는 아이템의 순서가 data source 객체에 있는 순서와 일치하는 것을 보장하게 컬렉션 뷰의 메서드를 구현해야 한다.
`UICollectionViewDiffableDataSource`(앞에서 나온 iOS13 이상부터 쓸 수 있었던 data source) 객체는 이런 작업을 자동으로 해준다. 만약 커스텀 data source를 사용하고 있다면,
컬렉션의 데이터를 추가하고, 삭제하거나 재배치를 할 때, `UICollectionView`의 메서드를 사용해서 일치하는 셀들을 삽입, 삭제, 그리고 재배치해야 한다는 것이다. 이게 정확히 무슨 말이냐면,
data source에서 컬렉션 뷰가 데이터를 가져오고 이를 바탕으로 화면에 실제로 출력하는 작업을 하기 때문에 data source가 변경되면 컬렉션 뷰는 변경사항을 화면에 업데이트 해줘야 한다.

![image](https://user-images.githubusercontent.com/41438361/133025364-cc68ee4e-97bc-41aa-89f3-66e545505439.png)

추가로 아이템들을 선택하는 행위는 [`delegate`](https://developer.apple.com/documentation/uikit/uicollectionview/1618033-delegate?language=objc)객체를 통해 관리하지만,
선택된 객체를 관리하는 작업은 컬렉션 뷰 객체를 이용해 관리한다.

## Layout

위에서 보조적인 뷰를 지원하고 뷰들의 배치를 레이아웃 객체가 관리한다고 했을 때 잠깐 레이아웃에 대한 얘기가 나왔다. 

레이아웃 객체는 이름에서도 알 수 있듯이 컬렉션 뷰에 있는 컨텐츠의 시각적인 배치를 정의한다. [`UICollectionViewLayout`](https://developer.apple.com/documentation/uikit/uicollectionviewlayout?language=objc)의 
하위 크래스인 레이아웃 객체는 컬렉션 뷰 내의 모든 셀과 보조적인 뷰의 구성과 위치를 정의한다. 컬렉션 뷰는 셀들과 보조적인 뷰들의 생성이 컬렉션 뷰와 data source 객체 사이의
연결하는 과정에 개입하기 때문에 레이아웃 정보를 레이아웃을 적용할 적절한 뷰들에 적용한다. 레이아웃 객체는 다른 data source와 비슷하지만, 아이템의 데이터 대신에 시각적인 정보를 제공한다는 점에서 다르다.

일반적으로 컬렉션 뷰를 생성할 때 레이아웃 객체를 생성하지만, 컬렉션 뷰의 레이아웃을 동적으로 변경할 수도 있다. 레이아웃 객체는 [`collectionViewLayout`](https://developer.apple.com/documentation/uikit/uicollectionview/1618047-collectionviewlayout?language=objc)에 저장되어 있다.
이 프로퍼티를 직접 설정하면 애니메이션을 적용하지 않고 레이아웃을 즉각적으로 업데이트 한다. 만약 변화를 애니메이션을 통해 보여주고 싶다면 [`setCollectionViewLayout:animated:completion:`](https://developer.apple.com/documentation/uikit/uicollectionview/1618017-setcollectionviewlayout?language=objc)메서드를 호출하면 된다.

Gesture recognizer나 터치 이벤트에 의한 상호작용 transition을 생성하려면 [`startInteractiveTransitionToCollectionViewLayout:completion:`](https://developer.apple.com/documentation/uikit/uicollectionview/1618098-startinteractivetransitiontocoll?language=objc) 메서드를 사용해서
레이아웃 객체를 변화시키면 된다. 이 메서드는 transition progress를 추적하기 위한 gesture recognizer나 event-handling 코드와 함께 작동하는 중간의 레이아웃 객체를 
설치한다. 이벤트 핸들링 코드가 transition이 끝났음을 알게 되면, 앞서 받았던 이 중간의 객체를 삭제하고 intented target layout 객체를 설치하기 위해 [`finishInteractiveTransition`](https://developer.apple.com/documentation/uikit/uicollectionview/1618080-finishinteractivetransition?language=objc)나
[`cancelInteractiveTransition`] 메서드를 호출한다.

더 많은 것을 보려면 [`Layouts`](https://developer.apple.com/documentation/uikit/views_and_controls/collection_views/layouts?language=objc)를 참고하면 된다.

## Cell과 Supplementary View

컬렉션 뷰의 data source 객체는 아이템을 위한 컨텐츠와 이 컨텐츠를 출력하기 위해 사용되는 뷰를 같이 제공한다. 컬렉션뷰가 컨텐츠를 처음 로딩했을 때, data source에게 
각각의 화면에 보여져야 할 아이템을 위한 뷰를 제공해달라고 요청한다. 컬렉션 뷰는 data source가 재사용하겠다고 마크한 뷰 객체들의 큐나 리스트를 관리한다. 코드에서 명시적으로
새로운 뷰들을 생성하는 것 대신에, deque된 뷰를 사용하는 것이 좋다. 즉 컬렉션 뷰는 cell에 해당하는 뷰를 재사용하는데, 이 뷰들을 관리하는 큐/리스트를 가지고 있다는 소리다.

재사용할 뷰를 dequeing 하는 두 가지의 메서드가 있다. 요청된 뷰가 어떤 타입인지에 따라서 다른 메서드를 요청하면 된다.

* 컬렉션 뷰의 아이템의 cell을 가져오려면 [`dequeueReusableCellWithReuseIdentifier:forIndexPath:`](https://developer.apple.com/documentation/uikit/uicollectionview/1618063-dequeuereusablecellwithreuseiden?language=objc)를 사용하면 된다.
* 레이아웃 객체에 의해 요청된 supplementary view를 가져오기 위해 [`dequeueReusableSupplementaryViewOfKind:withReuseIdentifier:forIndexPath:`](https://developer.apple.com/documentation/uikit/uicollectionview/1618068-dequeuereusablesupplementaryview?language=objc)를 사용하면 된다.

이 메서드들을 호출하기 이전에, 만약 뷰가 존재하지 않는 상태였다면 이 뷰를 어떻게 생성하는지 컬렉션 뷰에 알려줘야 한다. 이를 위해서 컬렉션 뷰에
클래스나 nib 파일을 등록해야 한다. 예를 들어 cell을 등록할 때, 클래스를 등록하기 위해서 [`registerClass:forCellWithReuseIdentifier:`](https://developer.apple.com/documentation/uikit/uicollectionview/1618089-registerclass?language=objc) 메서드를 사용하고,
nib 파일을 등록하려면 [`registerNib:forCellWithReuseIdentifier:`](https://developer.apple.com/documentation/uikit/uicollectionview/1618083-registernib?language=objc)를 사용하면 된다.
등록하는 과정의 일부로, 뷰의 목적을 잘 나타내는 reuse identifier를 정의해야 한다. 이 identifier를 이용해서 나중에 뷰를 dequeing한다.

Data source 메서드에서 적절한 뷰를 dequeueing핬다면, 이 뷰의 컨텐츠를 채우고, 이를 컬렉션 뷰에 리턴해서 사용하게 한다. 레이아웃 객체에서 레이아웃 정보를 얻은 후, 
컬렉션 뷰가 뷰에 적용해서 출력할 것이다. 

![image](https://user-images.githubusercontent.com/41438361/133027949-ab6732be-64ef-462f-a531-d9c1575648b9.png)

## Data Prefetching

컬렉션 뷰는 반응성을 향상시키기 위해 두 가지의 prefetching 기술을 제공한다. 하나는 뷰를, 하나는 데이터를 미리 불러오는 기술이다.

* Cell prefetching은 셀이 요청되기 이전에 cell을 항상 준비해 놓는 것이다. 컬렉션 뷰가 많은 수의 셀들을 갑자기 한 번에 요청할 때, 예를 들어 grid layout의 새로운 행의 cell들은 display를 위해 요청된 때보다 더 먼저 요청된다. 셀 렌더링은 그래서 여러 레이아웃을 통해 전달되고, 더 스크롤링이 부드럽게 될 수 있다. 이 Cell prefetching은 기본적으로 활성화 되어 있는 상태이다.
* Data prefetching은 셀을 요청하기 앞서서 컬렉션 뷰에 데이터 요청이 들어왔을 때 일어난다. 셀의 컨텐츠가 굉장히 오래걸리는 데이터 로딩 프로세스(네트워크 프로세스같은 애들)에 의존할 때 유용하다. 셀을 위한 데이터를 언제 prefetch할 지 알림을 받고 싶다면 [`UICollectionViewDataSourcePrefetching`](https://developer.apple.com/documentation/uikit/uicollectionviewdatasourceprefetching?language=objc)프로토콜을 채택한 객체를 [`prefetchDataSource`](https://developer.apple.com/documentation/uikit/uicollectionview/1771768-prefetchdatasource?language=objc) 프로퍼티에 할당한다.

이 cell prefetching과 data prefetching도 별개로 설명하고 있는 애플 공식문서가 있으니 참고해봐도 좋을 것 같다. 

https://developer.apple.com/documentation/uikit/uicollectionviewdatasourceprefetching/prefetching_collection_view_data?language=objc

## 상호작용으로 아이템 재배치하기

컬렉션 뷰는 사용자 상호작용에 따라서 아이템을 이동할 수 있게 한다. 보통 컬렉션 뷰의 아이템의 순서는 data source에서 결정된다. 만약 사용자가 아이템을 재배치하게 했다면,
컬렉션 뷰의 아이템과 사용자의 상호작용을 추적할 수 있는 gesture recognizer를 구성하고 아이템의 위치를 업데이트 할 수 있다.

아이템을 상호작용하는 것으로 재배치하기 위해, 컬렉션 뷰의 [`beginInteractiveMovementForItemAtIndexPath:`](https://developer.apple.com/documentation/uikit/uicollectionview/1618019-begininteractivemovementforitema?language=objc)를 호출한다.
Gesture recognizer가 터치 이벤트를 추적할 때, 터치하는 위치의 변화를 알리기 위해 [`updateInteractiveMovementTargetPosition:`](https://developer.apple.com/documentation/uikit/uicollectionview/1618079-updateinteractivemovementtargetp?language=objc) 메서드를 호출한다.
Gesture를 추적하는 것을 마쳤다면 상호작용을 종료하고 컬렉션 뷰를 업데이트 하기위해 [`endInteractiveMovement`](https://developer.apple.com/documentation/uikit/uicollectionview/1618082-endinteractivemovement?language=objc)나 [`cancelInteractiveMovement`](https://developer.apple.com/documentation/uikit/uicollectionview/1618076-cancelinteractivemovement?language=objc) 메서드를 호출한다.

상호작용 도중에, 컬렉션 뷰는 아이템의 현재 위치를 반영하기 위해 레이아웃을 동적으로 비활성화한다. 만약 아무것도 하지 않으면, 기본 레이아웃 변경이 아이템에 적용될 것이지만
원한다면 레이아웃 애니메이션을 커스터마이징 할 수 있다. 만약 상호작용이 끝나면, 컬렉션 뷰는 아이템의 새로운 위치와 함께 data source를 업데이트 한다.

`UICollectionViewController` 클래스는 관리되는 컬렉션 뷰 내의 아이템을 재배치하는데 사용할 수 있는 기본 gesture recognizer를 제공한다. 이 gesture recognizer를 설치하려면
컬렉션 뷰의 `installsStandardGestureForInteractiveMovement` 프로퍼티를 YES로 설정한다.

# [UICollectionViewFlowLayout](https://developer.apple.com/documentation/uikit/uicollectionviewflowlayout?language=objc)

UICollectionView의 레이아웃을 정의할 때 `UICollectionViewFlowLayout`을 자주 사용한다. `UICollectionViewFlowLayout`은 각 섹션에 아이템들을
그리드로 구성한다. 여기에 헤더와 푸터 뷰가 추가될 수 있다.

Flow layout은 collection view layout의 한 타입이다. 컬렉션 뷰 안에 있는 아이템들은 scrolling direction에 따라 한 행이나 열 내에 나열되고, 각 행은
최대한 많은 셀들로 채워진다. 셀들은 같은 사이즈일 수 있고, 다른 사이즈일 수 있다. 

Flow layout은 각 섹션과 그리드에서 아이템, 헤더, 푸터의 사이즈를 결정하기 위해 컬렉션 뷰의 delegate 객체와 같이 작동한다. 이 Delegate 객체는 꼭 `UICollectionViewDelegateFlowLayout` 프로토콜을 채택하고 있어야 한다.
이 delegate를 사용하면 레이아웃 정보를 동적으로 적용할 수 있게 된다. 예를 들어, delegate 객체를 사용해서 그리드에 있는 아이템들에 각각 다른 사이즈를 적용할 수 있다.
만약 delegate를 제공하지 않으면 flow layout은 클래스의 프로퍼티에 설정한 기본 값을 사용하게 된다.

Flow layout은 한 방향에서의 고정된 거리와 다른 쪽의 scrollable 거리를 이용해서 컨텐츠를 배치한다. 예를 들어 수직으로 스크롤할 수 있는 그리드라면, 그리드 컨텐츠의
너비는 컬렉션 뷰의 너비에 맞춰질 것이고, 높이는 그리드에 있는 섹션과 아이템의 개수에 따라 동적으로 조정될 것이다. 레이아웃은 기본적으로 수직으로 스크롤링 되지만,
 `scrollDirection` 프로퍼티를 사용해서 스크롤링되는 방향을 조정할 수 있다.
 
 Flow layout에 있는 각 섹션은 커스텀 헤더와 푸터를 가질 수 있다. 뷰의 헤더와 푸터를 구성하기 위해서, 헤더와 푸터의 사이즈를 non-zero 값으로 조정해야 한다.
 
 ## [UICollectionViewFlowLayoutAutomaticSize](https://developer.apple.com/documentation/uikit/uicollectionviewflowlayoutautomaticsize?language=objc)
 
 `estimatedItemSize` 프로퍼티를 이 값으로 설정해서 컬렉션 뷰에서 셀들이 알아서 사이즈가 정해질 수 있도록 한다. 이 값은 non-zero이고, 컬렉션 뷰가 셀의 실제 사이즈를 얻기 위해 셀의 [`preferredLayoutAttributesFittingAttributes:`](https://developer.apple.com/documentation/uikit/uicollectionreusableview/1620132-preferredlayoutattributesfitting?language=objc) 메서드를 사용하게 하는 placeholder 값이다.
 
 
 
