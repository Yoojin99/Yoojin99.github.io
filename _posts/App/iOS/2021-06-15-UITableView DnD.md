---  
layout: post  
title: "[iOS] - UITableView 드래그 드롭으로 행 순서 바꾸기"  
subtitle: ""  
categories: app
tags: app-ios 
comments: true  
header-img: img/dev/app/ios/XcodeImg.png

---  
  
> `TableView에서 드래그 드롭으로 행의 순서를 바꾸는 방법을 알아보자.`  

---

# UITableView DnD(Drag and Drop)로 행 순서 바꾸기

UITableView에서 cell을 드래그 드롭해서 할 수 있는 작업은 여러가지인데, 그 중에서도 UITableView 내의 행의 순서를 바꾸는 방법들에 대해 보려고 합니다.

1. UITableView의 cell을 조금 길게 눌러서 드래그 드롭하기
2. UITableView가 editing mode일 때 cell의 오른쪽에 나타나는 버튼(三 ◀️ 이런 모양의 버튼) 눌러서 드래그 드롭하기

두 방법에는 약간 차이가 있습니다.

## 1\. UITableView의 cell을 눌러서 드래그 드롭

![image](https://user-images.githubusercontent.com/41438361/121973080-7de80900-cdb7-11eb-841d-7f73c4b95af4.png)

UITableViewCell이 위와 같이 있다고 할 때, cell의 어느 지점을 눌러도 드래그 드롭이 가능합니다. 셀 범주 안에 있는 부분이기만 하다면 위에 표시한 동그라미들처럼 어느 지점을 눌러도 드래그 드롭이 가능합니다.

다만 드래그 드롭을 하기 위해 살짝 길게 눌러서 '이제 드래그 할거야'라는 제스처를 해야 합니다.

## 2\. UITableView의 editing mode 이용해 드래그 드롭

![image](https://user-images.githubusercontent.com/41438361/121973096-84768080-cdb7-11eb-9b72-50ab7a745e09.png)

위 사진은 UITableView가 editing mode일 때 cell 우측에 셀을 드래그 드롭할 수 있는 버튼이 뜬 화면입니다. 이 버튼을 눌러야만 드래그 드롭을 할 수 있고, 셀 내의 다른 부분을 클릭해서 드래그 드롭할 수 없습니다.

위의 방법 1에서는 드래그 드롭을 하기 위해 약간의 누르는 시간이 필요했지만, 이 방법에서는 즉시 해당 버튼을 눌러 드래그 드롭 할 수 있습니다.

## 비교

요구사항이 명확할 경우에는 두 가지 방법 중 하나만 선택하는 것이 맞지만, 그렇지 않은 경우에 둘 중 하나를 선택해야 할 것입니다. 두 가지 방법을 모두 구현해보며 개인적으로 느꼈던 차이를 비교하자면 아래와 같습니다.

* 간단함 : 1 << 2 - 2번 방법이 구현하기 훨씬 간단합니다.
* 응용성 : 1 >> 2 - 1번 방법이 더욱 다양한 기능을 위해 사용될 수 있습니다. (다른 테이블 뷰 간의 드래그-드롭 등등을 지원합니다.)

## 구현

구현하는 것을 보기에 앞서서 NavigationController의 NavigationBar가 화면에 존재한다는 가정을 하고 보겠습니다. NavigationBar의 오른쪽에는 `editButtonItem`이 있어서 Edit ↔️ Done 으로 상태를 변화시킬 수 있습니다. Edit을 누르면 테이블 뷰의 행의 순서를 바꿀 수 있는 모드로 진입했다고 하고, Done을 누를 경우 테이블 뷰의 행의 순서를 바꿀 수 없게 한다고 가정합니다.

* 버튼이 Edit으로 떠 있을 때 (현재 상태에서는 테이블 뷰를 드래그 드롭 할 수 없음)
![image](https://user-images.githubusercontent.com/41438361/121973125-8d675200-cdb7-11eb-940d-e8e18073db35.png)
* 버튼이 Done으로 떠 있을 때 (현재 상태에서는 테이블 뷰를 드래그 드롭해서 행의 순서를 바꿀 수 있음)
![image](https://user-images.githubusercontent.com/41438361/121973148-93f5c980-cdb7-11eb-8152-5db4749b3f9c.png)

### 1\. UITableView의 cell을 눌러서 드래그 드롭

Cell을 눌러 드래그 드롭하기 위해 중요한 프로토콜이 두 개가 있습니다.

1. `UITableViewDragDelegate` : 셀을 드래그 할 때 필요한 프로토콜
2. `UITableViewDropDelegate` : 셀을 드롭 할 때 필요한 프로토콜

#### 1. `tableView.dragInteractionEnabled` 프로퍼티 설정

먼저 `tableView.dragInteractionEnabled` 프로퍼티를 설정해야 합니다.

NavigationBar의 'Edit' 버튼을 눌러 테이블 뷰의 행의 순서를 바꿀 수 있고, 'Done' 버튼을 눌러 테이블 뷰의 행의 순서를 바꿀 수 없다고 가정했습니다.

NavigationBar의 오른쪽 버튼(현재 'Edit'버튼)은 UIViewController의 `editButtonItem`로 할당이 되어 있는데, `editButtonItem`은 클릭이 되었을 때
`setEditing(:animated)` 메서드를 호출합니다. 따라서 뷰 컨트롤러에서 이 메서드를 override 해서 버튼이 클릭될 때마다 `dragInteractionEnabled` 의 값을 true/false로 전환해 드래그 드롭을 활성화하고 비활성화 할 것입니다.

``` swift
override func setEditing(_ editing: Bool, animated: Bool) {
    super.setEditing(editing, animated: animated)
    tableView.dragInteractionEnabled = editing
}
```

#### 2. `UITableViewDelegate` 구현하기

``` swift
// 위에서 tableView.delegate = self 와 같이 delegate 설정을 해줘야 합니다.

extension ViewController: UITableViewDelegate{
    func tableView(_ tableView: UITableView, canMoveRowAt indexPath: IndexPath) -> Bool {
        return true
    }

    func tableView(_ tableView: UITableView, moveRowAt sourceIndexPath: IndexPath, to destinationIndexPath: IndexPath) {
        model.moveItem(at: sourceIndexPath.row, to: destinationIndexPath.row) // 임의로 구현한 메서드로, dataSource의 데이터 순서를 바꾸는 메서드입니다. 드래그 한 아이템과 드롭한 위치에 있는 아이템을 바꿉니다.
    }
}
```

#### 3. `UITableViewDragDelegate` 구현하기

``` swift
// 위에서 tableView.dragDelegate = self와 같이 dragDelegate 설정을 해줘야 합니다.

extension ViewController: UITableViewDragDelegate {
    func tableView(_ tableView: UITableView, itemsForBeginning session: UIDragSession, at indexPath: IndexPath) -> [UIDragItem] {
        return [UIDragItem(itemProvider: NSItemProvider())] // 애플의 공식 문서에서는 사용자가 특정 행을 드래그하는 것을 원하지 않으면 빈 배열을 리턴하라고 했는데, 빈 배열을 리턴했을 때도 드래그가 가능했습니다. 이 부분은 더 자세히 알아봐야 할 것 같습니다.

    }

    // 드래그를 시작했을 때 네비게이션바 오른쪽 버튼을 조작할 수 없게
    func tableView(_ tableView: UITableView, dragSessionWillBegin session: UIDragSession) {
        navigationItem.rightBarButtonItem?.isEnabled = false
    }

    // 드래그를 끝냈을 때 네비게이션바 오른쪽 버튼을 조작할 수 있게
    func tableView(_ tableView: UITableView, dragSessionDidEnd session: UIDragSession) {
        navigationItem.rightBarButtonItem?.isEnabled = true
    }
}
```

#### 4. `UITableViewDropDelegate` 구현하기

``` swift
// 위에서 tableView.dropDelegate = self 와 같이 dropDelegate 설정을 해줘야 합니다.

extension ViewController: UITableViewDropDelegate {
    // 손가락을 화면에서 뗐을 때. 드롭한 데이터를 불러와서 data source를 업데이트 하고, 필요하면 새로운 행을 추가한다.
    func tableView(_ tableView: UITableView, performDropWith coordinator: UITableViewDropCoordinator) {
        let destinationIndexPath: IndexPath

        if let indexPath = coordinator.destinationIndexPath {
            destinationIndexPath = indexPath
        } else {
            let section = tableView.numberOfSections - 1
            let row = tableView.numberOfRows(inSection: section)
            destinationIndexPath = IndexPath(row: row, section: section)
        }

        coordinator.session.loadObjects(ofClass: NSString.self, completion: { items in
            let stringItems = items as! [String]

            var indexPaths = [IndexPath](undefined "undefined")
            for (index, item) in stringItems.enumerated() {
                let indexPath = IndexPath(row: destinationIndexPath.row + index, section: destinationIndexPath.section)
                self.model.addItem(item, at: indexPath.row)
                indexPaths.append(indexPath)
            }

            tableView.insertRows(at: indexPaths, with: .automatic)
        })
    }

    func tableView(_ tableView: UITableView, canHandle session: UIDropSession) -> Bool {
        return model.canHandle(session)
    }

    // 드래그할 떄 (손가락을 화면에 대고 있을 때)
    func tableView(_ tableView: UITableView, dropSessionDidUpdate session: UIDropSession, withDestinationIndexPath destinationIndexPath: IndexPath?) -> UITableViewDropProposal {
        var dropProposal = UITableViewDropProposal(operation: .cancel)

        guard session.items.count == 1 else { return dropProposal }

        if tableView.hasActiveDrag {
            // 현재 edit 모드이면 드래그 하는 애를 이동시킨다.(move)
            if isEditing {
                dropProposal = UITableViewDropProposal(operation: .move, intent: .insertAtDestinationIndexPath)
            }
        } else {
            dropProposal = UITableViewDropProposal(operation: .copy, intent: .insertAtDestinationIndexPath)
        }

        return dropProposal
    }
}
```

*추측*

여기서 굉장히 헷갈렸던 부분은, 이 코드를 구현하는 시나리오에서는 `UITableViewDropDelegate`의 `func tableView(_ tableView: UITableView, performDropWith coordinator: UITableViewDropCoordinator)` 메서드가 호출되지 않는다는 것이었습니다. 한 테이블 뷰 내에서 단순히 드래그 드롭할 때는 `UITableViewDelegate`의 `func tableView(_ tableView: UITableView, moveRowAt sourceIndexPath: IndexPath, to destinationIndexPath: IndexPath)` 메서드가 호출됩니다. 하지만 앱이나 뷰 간에 드래그 드롭을 할 때에는 이 메서드가 호출되어 data source를 업데이트 시킬 것으로 생각됩니다.

위와 같이 구현한 후 앱을 실행시키면 아래의 화면에서 셀들을 길게 눌러 드래그-드롭할 수 있습니다.

![image](https://user-images.githubusercontent.com/41438361/121973180-a5d76c80-cdb7-11eb-9a23-8abae48a09ed.png)
![Jun-14-2021 18-32-25](https://user-images.githubusercontent.com/41438361/121973211-b556b580-cdb7-11eb-9abf-46616d7490d0.gif)


### 2\. UITableView의 editing mode 이용해 드래그 드롭

해야 하는 것은 크게 두 가지입니다.

1. UITableView의 `setEditing(_:animated:_)` [메서드](https://developer.apple.com/documentation/uikit/uitableview/1614876-setediting)를 호출해 UITableView를 editing mode로 설정하기.
2. `UITableViewDelegate` 구현하기

#### 1\. UITableView를 editing mode로 설정하기

NavigationBar의 'Edit' 버튼을 눌러 테이블 뷰의 행의 순서를 바꿀 수 있고, 'Done' 버튼을 눌러 테이블 뷰의 행의 순서를 바꿀 수 없다고 가정했습니다.

NavigationBar의 오른쪽 버튼(현재 'Edit'버튼)은 UIViewController의 `editButtonItem`로 할당이 되어 있는데, `editButtonItem`은 클릭이 되었을 때
`setEditing(:animated)` 메서드를 호출합니다. 따라서 뷰 컨트롤러에서 이 메서드를 override 해서 버튼이 클릭될 때마다 tableView를 editing 모드로 전환하고 다시 바꾸려고 합니다.

``` swift
// 뷰 컨트롤러에서 override
override func setEditing(_ editing: Bool, animated: Bool) {
    super.setEditing(editing, animated: animated)
    tableView.setEditing(editing, animated: true) // 테이블 뷰의 editing mode를 설정합니다.
    // tableView.dragInteractionEnabled = editing // dragInteractionEnabled를 설정하지 않아도 됩니다.
}
```

이러고 앱을 실행했을 때, NavigationBar의 edit 버튼을 누르는 것을 반복하면 tableview가 editing mode로 전환되었다가 다시 원래대로 돌아오는 것을 반복하는 것을 확인할 수 있습니다.

![image](https://user-images.githubusercontent.com/41438361/121973230-c0114a80-cdb7-11eb-9542-b4389f2437d4.png)

그런데 tableView가 editing mode로 전환되는 것은 눈에 보이는데 각 cell의 앞 부분에 delete 버튼은 보이지만, 뒷 부분에 원했던 위치를 바꾸는 버튼이 보이지 않습니다. 이 버튼을 보이게 하고, 또 이 버튼을 눌러 드래그 드롭을 가능하게 하기 위해 `UITableViewDelegate`를 구현합니다.

#### 2. `UITableViewDelegate` 구현하기

``` swift
// 물론 위에서 tableView.delegate = self 등으로 delegate 설정을 해줘야 합니다.

extension ViewController: UITableViewDelegate {
    // 테이블 뷰의 행의 순서를 바꿀 수 있는 버튼과, 버튼을 드래그 드롭해서 순서를 바꿀 수 있게 하는 메서드입니다.
    func tableView(_ tableView: UITableView, moveRowAt sourceIndexPath: IndexPath, to destinationIndexPath: IndexPath) {
        model.moveItem(at: sourceIndexPath.row, to: destinationIndexPath.row) // dataSource의 순서를 바꿔주는 메서드입니다. 처음 드래그 했던 행과 드롭하는 위치에 있는 행의 데이터의 순서를 바꿉니다. 이 코드를 작성하지 않아도 행 간의 위치는 바꿀 수 있었지만, 바꿀 때 데이터가 중복이 되는 형상이 일어나는 경우가 있었습니다.
    }

    // 아래 두 메서드는 editing mode로 전환되었을 때 cell 앞 부분의 delete button이 뜨지 않게 해 줍니다.
    func tableView(_ tableView: UITableView, editingStyleForRowAt indexPath: IndexPath) -> UITableViewCell.EditingStyle {
        return .none
    }

    func tableView(_ tableView: UITableView, shouldIndentWhileEditingRowAt indexPath: IndexPath) -> Bool {
        return false
    }
}
```

위의 코드에서 언급한 '데이터가 중복이 되는 현상'은 아래와 같이 행 간 위치를 바꿀 때 특정 데이터가 중복되어 나타나는 현상이 생기는 경우를 말합니다.

![image](https://user-images.githubusercontent.com/41438361/121973238-c6072b80-cdb7-11eb-8a11-385939ad9a0c.png)

이제 앱을 실행시키면 원하던대로 'Edit' 버튼을 눌러서 테이블의 행의 순서를 바꿀 수 있게 되었습니다.

![image](https://user-images.githubusercontent.com/41438361/121973246-cb647600-cdb7-11eb-8cf9-ac31a5bdf091.png)

### 그 외

TableView에서 DnD를 하는 방법을 서술한 애플의 공식문서([https://developer.apple.com/documentation/uikit/drag\_and\_drop/adopting\_drag\_and\_drop\_in\_a\_table\_view](https://developer.apple.com/documentation/uikit/drag_and_drop/adopting_drag_and_drop_in_a_table_view)를 보면 iOS 13+ 로 버전이 표기되어 있는데, TableView에서 드래그 드롭을 지원하는 게 iOS 11+([https://developer.apple.com/documentation/uikit/views\_and\_controls/table\_views/supporting\_drag\_and\_drop\_in\_table\_views](https://developer.apple.com/documentation/uikit/views_and_controls/table_views/supporting_drag_and_drop_in_table_views)인 것과 차이가 있지만 이는 iOS 13 버전에서 새롭게 업데이트가 된 것이 있는 것이 아니라 iOS 13+으로 표기된 애플의 공식문서가 제공하는 프로젝트가 iOS 13 버전 이상을 타겟으로 하기 때문입니다. 애플이 제공하는 프로젝트에서 AppDelegate에 `var window: UIwindow?`를 해주면 iOS 11 버전에서도 문제 없이 동작합니다.

출처

* [https://stackoverflow.com/questions/7334126/how-can-i-identify-self-editbuttonitem-buttons-clicked-event](https://stackoverflow.com/questions/7334126/how-can-i-identify-self-editbuttonitem-buttons-clicked-event)
* [https://stackoverflow.com/questions/33946836/remove-the-delete-button-️-on-table-rows-in-edit-mode/47646795](https://stackoverflow.com/questions/33946836/remove-the-delete-button-️-on-table-rows-in-edit-mode/47646795)
* [https://stackoverflow.com/questions/60270449/how-can-i-use-drag-and-drop-to-reorder-a-uitableview](https://stackoverflow.com/questions/60270449/how-can-i-use-drag-and-drop-to-reorder-a-uitableview)
