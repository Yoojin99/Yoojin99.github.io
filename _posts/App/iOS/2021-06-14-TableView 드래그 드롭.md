---  
layout: post  
title: "[iOS] - Table View에서 드래그 드롭하기"  
subtitle: ""  
categories: app
tags: app-ios 
comments: true  
header-img: img/dev/app/ios/XcodeImg.png

---  
  
> `TableView에서 드래그 드롭을 하는 방법을 알아보자.`  

---

이 글에서 설명하는 내용은 iOS 11 이상부터 지원되는 내용이다. iOS 11 밑의 버전에서는 어떻게 테이블 뷰의 셀을 드래그 드롭할 수 있는지 공부 후 추가해두겠다.

## 개요

테이블 뷰는 행이 출력되는 것을 다루는 API를 통해 드래그 드롭을 지원한다. 드래그를 지원하기 위해, drag delegate 객체(UITableViewDragDelegate 프로토콜을 채택한 객체)를 정의하고 테이블 뷰의 `dragDelegate` 프로퍼티에 설정한다.
드롭을 다루기 위해, drop delegate 객체(UITableViewDropDelegate 프로토콜을 채택한 객체)를 정의하고 이를 테이블 뷰의 `dropDelegate` 프로퍼티에 할당한다.

## 테이블 뷰에서 행들 드래그하기

테이블 뷰는 대부분의 드래그와 관련된 상호작용을 관리하지만, 개발자가 어떤 행을 드래그 할 지를 명시해야 한다. 드래그하는 제스쳐를 했을 때, 테이블 뷰는 드래그 세션을 생성하고 drag delegate 객체의 `tableView(_:itemsForBeginning:at:)`메서드를 호출한다.
(사용자가 선택된 행을 드래그했을 때, 이 메서드는 선택된 행마다 한 번 호출된다. 만약 아무런 행도 선택되지 않았을 때, 메서드는 밑에 깔려진 행에 대해서 한 번 호출된다.) 만약 비어있지 않은 배열을 리턴했을 때,
테이블 뷰는 내가 명시한 행을 드래그하기 시작한다. 사용자가 특정 index path에 있는 컨텐츠를 드래그하는 것을 허용하고 싶지 않으면 빈 배열을 리턴해라. 

*추가적인 드래그와 관련된 상호작용을 관리하기 위해 `UITableViewDragDelegate` 프로토콜의 다른 메서드를 사용해라. 예를 들어, 드래그 되는 행들의 외형을 바꿀 수도 있고, 드래그 세션 도중에 아이템을 추가하게 할 수도 있다.*

`tableView(_:itemsForBeginning:at:)` 메서드 구현에서, 아래의 것들을 한다.

* 하나 이상의 `NSItemProvider` 객체를 생성한다. Item provider 들을 이용해서 테이블의 행에 들어갈 데이터를 표현한다.
* 각 item provider 객체를 `UIDragItem` 객체로 감싼다.
* 각 드래그 아이템의 `localObject` 프로퍼티에 값을 할당하는 것을 고려한다. 이 단계는 안해도 되지만 앱에서 드래그 드롭을 하는 것을 더 빠르게 만들어줄 수 있다.
* 드래그 아이템을 반환한다.

제공된 index path를 이용해서 드래그 아이템을 생성할 행을 결정한다. 만약 타겟 행이 현재 선택된 행들 중 하나라면, 테이블 뷰는 자동으로 선택된 모든 행들을 드래그 한다. 만약 행이 현재 선택된 것의 일부가 아니라면
테이블 뷰는 그것을 이미 활성화 된 drag operation에 추가한다.

## 드롭된 컨텐츠 받기

컨텐츠가 경계 안에서 드래그 될 때, 테이블 뷰는 drop delegate에게 드래그된 데이터를 받을 수 있는지를 물어본다. 처음에, 테이블 뷰는 drop delegate의 `tableView(_:canHandle:)`를 호출해서 특정 데이터를 데이터 소스에 합칠 수 있는지 판단한다.
만약 포함시킬 수 있으면, 테이블 뷰는 데이터가 드롭될 수 있는 곳을 판단하기 위해 다른 메서드를 호출하기 시작한다.

사용자의 손가락이 움직일 때, 테이블 뷰는 가능한 드롭 장소를 추적하고 `tableView(_:dropSessionDidUpdate:withDestinationIndexPath:)` 메서드를 호출해서 변화를 알린다. 이 메서드를 구현하는 것도 옵션이지만,
드래그된 컨텐츠가 어떻게 합쳐질지 시각적인 피드백을 테이블 뷰가 제공할 수 있기 때문에 구현하는 것이 권장된다. 구현에서, 특정 index path에 드롭 하는 것에 어떻게 반응할지에 대한 정보와 함께 `UITableViewDropProposal` 객체를 생성한다.
예를 들어, 데이터 소스에 새로운 행으로 컨텐츠를 삽입하고 싶을 수도 있고 특정 index path에 있는 행에 데이터를 추가하고 싶을 수도 있다. 메서드가 자주 호출되기 때문에, 가능한 빨리 개발자가 제안하는 바를 리턴해야 한다.
만약 이 메서드를 구현하지 않는다면, 테이블 뷰는 드롭할 때 시각적인 피드백을 주지 않는다.

사용자가 스크린에서 손가락을 뗄 때, 테이블 뷰는 drop delegate의 `tableView(_:performDropWith)` 메서드를 호출한다. 이 메서드를 구현해서 드롭된 데이터를 다룬다. 구현에서는, 드래그 된 데이터를 가져오고,
테이블 뷰의 데이터 소스를 업데이트 하고, 필요하면 테이블 뷰에 새로운 행을 추가한다. 만약 컨텐츠가 테이블 뷰에서 가져와진 거라면, 테이블 뷰 API를 이용해서 드래그된 행을 원래 있던 곳에서 삭제하고 새로운 곳에 넣을 수 있다.
테이블 뷰 밖에서 온 컨텐츠에 대해서는 `localObject` 프로퍼티(앱 안의 컨텐츠)를 사용하거나 데이터를 가져오기 위해 `NSItemProvider` 객체를 이용하고 데이터를 삽입한다.

`tableView(_:performDropWith:)` 메서드에서, 아래의 것들을 한다.

1. 제공된 drop coordinator 객체의 `items` 프로퍼티를 반복한다.
2. 각 아이템에서, 컨텐츠를 어떻게 관리하고 싶은지를 결정한다.
  * 아이템의 `sourceIndexPath` 가 값을 포함하고 있다면, 아이템은 테이블 뷰에서 온 것이다. 아이템을 현재 위치에서 삭제하고 새로운 index path에 넣기 위해 batch update를 한다.
  * 만약 드래그된 아이템의 `localObject` 프로퍼티가 집합이라면, 컨텐츠는 앱의 어디선가 온 것이고, 앱은 새로운 행을 추가하거나 이미 존재하는 행을 업데이트 해야 한다.
  * 다른 옵션이 없을 경우, 드래그 아이템의 `itemProvider`의 `NSItemProvider`를 이용해서 데이터를 비동기적으로 가져오고 아이템을 삽입하거나 업데이트 한다.
3. 데이터 소스를 업데이트 하고, 테이블 뷰에 필요한 아이템을 삽입하거나 이동시킨다.

앱에 이미 있는 컨텐츠 같은 경우에는 테이블 뷰의 데이터 소스와 인터페이스를 직접 바로 업데이트 시킬 수 있다. 예를 들어 테이블 뷰에서 드래그 된 아이템을 삭제하고 새로 넣기 위해 batch update를 사용할 수 있다. 완료되면,
drop coordinator의 `drop(_:toRowAt:)` 메서드를 호출해서 테이블 뷰에 드래그된 콘텐츠에 삽입하는 것을 애니메이팅 한다.

`NSItemProvider` 객체를 이용해서 받아진 데이터의 경우에, 실제 데이터를 받을 수 있기 전까지 테이블 뷰에 placeholder를 삽입한다. placeholder를 삽입하는 것은 테이블 뷰에 새로운 행을 추가할 때만 필요하다. placeholder는
임시 행처럼 행동하며, 실제 데이터를 가져오기 전까지 default 컨텐츠를 출력한다. 예를 들어, 컨텐츠가 현재 로딩중이라는 것을 나타내는 텍스트 필드를 placeholder에 표시할 수 있다.

Placeholder를 테이블 뷰에 삽입하려면 아래의 것들을 한다.

1. 테이블 뷰에 placeholder 행을 삽입하기 위해 제공된 `UITableViewDropCoordinator` 객체의 `drop(_:toPlaceholderInsertedAt:withReuseIdentifier:rowHeight:cellUpdateHandler:)` 메서드를 호추란다. Placeholder 셀의 컨텐츠를 구성하기 위해서 `cellUpdateHandler` 파라미터의 블럭을 이용한다.
2. `NSItemProvider` 객체에서 데이터를 비동기적으로 로딩한다.

`NSItemProvider` 객체가 실제 데이터를 리턴할 때, 삽입을 하고 placeholder cell을 최종 cell로 바꿔치기 한다. Placeholder를 생성하고 나서 컨텍스트 객체의 `commitInsertion(dataSourceUpdates:)` 메서드를 호출한다.
해당 메서드에 넘기는 블럭에서, 모델 객체와 테이블 뷰의 데이터 소스를 업데이트 한다. 이 메서드가 리턴되면, 테이블 뷰는 자동으로 placeholder를 삭제하고 최종 행을 삽입하고, drop coordinator의 `destinationIndexPath` 프로퍼티에 의대 명시된
장소에 새로 삽입된 셀의 데이터에 반영된다. 


출처

* https://developer.apple.com/documentation/uikit/views_and_controls/table_views/supporting_drag_and_drop_in_table_views
* https://developer.apple.com/documentation/uikit/drag_and_drop/adopting_drag_and_drop_in_a_table_view
