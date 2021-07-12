---  
layout: post  
title: "[iOS] - UITableView performBatchUpdates - row와 section를 일괄적으로 업데이트하기"  
subtitle: ""  
categories: app
tags: app-ios uitableview
comments: true  
header-img: 

---  
  
> `Row와 Section을 일괄적으로 업데이트하는 방법에 대해 알아보자.`  

---

UITableView에서 행과 section을 일괄적으로 업데이트 하는 batch update를 적용하는 방법을 애플의 공식 문서에서는 3가지 메서드를 통해 소개하고 있다.

* `beginUpdates()`
* `endUpdates()`
* `performBatchUpdates(_:completion)`

이름에서 짐작할 수 있듯이 `beginUpdates()`와 `endUpdates()`가 한 쌍이다. 이 둘은 iOS 2.0부터 가능한 반면에, 
`performBatchUpdates`은 iOS 11.0부터 사용이 가능하다. 공식문서에서는 가능한 한 `beginUpdates`, `endUpdates`대신에
`performBatchUpdates(_:completion:)`을 사용하라고 가이드하고 있다.

![image](https://user-images.githubusercontent.com/41438361/125220393-99382c80-e301-11eb-90ab-a85ec55251a0.png)

## `performBatchUpdates(_:completion:)`

여러개의 삽입, 삭제, reload, 그리고 이동하는 동작을 **그룹으로** 애니메이션을 통해 보여준다.

정의를 보면 아래와 같이 되어 있다.

![image](https://user-images.githubusercontent.com/41438361/125220600-faf89680-e301-11eb-89b0-aa4c17b053cd.png)

* `updates` : 위에서 언급한 삽입, 삭제, reload, 이동하는 동작을 수행하는 블럭이다. 테이블의 행을 수정하는 것에 추가로, 이런 수정 사항을 반영하기 위해 테이블의 data source를 업데이트 한다. 이 블럭은 return 값이 없고 인자를 가지지 않는다.
* `completion` : 이름에서 알 수 있듯이 모든 동작이 끝나면 실행할 블럭이다. 얘는 아래의 파라미터를 갖는다.
  * `finished` : 애니메이션이 성공적으로 끝났는지를 나타내는 Bool 값이다. 애니메이션이 어떤 이유로든 방해받아서 끝나지 못했다면 이 파라미터의 값은 false가 된다.

이 메서드는 테이블 뷰에 가해지는 여러 변화들이 각각 별도의 애니메이션을 통해 보여지는 게 아니라 하나의 애니메이션으로 보여주고 싶을 때 사용한다.
하나의 애니메이션으로 보여주고 싶은 모든 동작들을 `updates` 블럭안에 넣으면 된다.

참고로 삭제하는 동작은 삽입하는 동작보다 먼저 처리된다. 이거는 batch operation 전에 삭제를 위한 index들이 테이블 뷰의 상태에 대한 index를 제공하는 것과 비슷하게 처리되고
삽입에 대한 index들이 모든 삭제하는 동작 이후에 처리된다는 것을 의미한다. `updates` 내에 코드 순서가 어떻게 되어 있든 간에 삭제는 항상 삽입하는 동작 전에
일어나게 된다.

## `beginUpdates()`

삽입, 삭제, 행이나 섹션을 선택하는 메서드들 호출을 시작하는 메서드이다.

위에서도 언급했지만 이 대신에 가능한 한 `performBatchUpdates(_:completion:)`을 사용하라고 애플의 공식문서에 나와 있다.
삽입, 삭제, 그리고 선택하는 동작들(ex `cellForRow(at:)`, `indexPathsForVisibleRows`)이 동시에 애니메이팅 되게 하기 위해 이 메서드를 호출한다.
위에서 봤던 `performBatchUpdates`에서는 reload하는 동작을 포함했는데, 여기서는 포함하고 있지 않는 것을 볼 수 있다.

`beginUpdates()`를 쓰고, 바로 뒤에 `endUpdates()`를 이어 써서 cell을 reloading 하지 않고 행의 높이를 바꾸는 것을 애니메이션을 통해 보여줄 수 있다.
만약 여러개의 메서드들을 묶을 거라면 꼭 `endUpdates()`로 끝나야 한다. 다만 그 메서드안에 `reloadData()`를 호출하면 안된다. 만약 호출했다면, 모든 애니메이션을
스스로 수행해야 한다.



