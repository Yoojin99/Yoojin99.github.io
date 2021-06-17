---  
layout: post  
title: "[iOS] - UITableView expand, 접었다 펴서 확장하고 축소하기"  
subtitle: ""  
categories: app
tags: app-ios 
comments: true  
header-img: img/dev/app/ios/XcodeImg.png

---  
  
> `TableView에서 특정 행을 클릭해서 접었다 폈다 할 수 있는 방법을 알아보자.`  

---

![functionalApp-1](https://user-images.githubusercontent.com/41438361/122309164-bf5bee00-cf48-11eb-89da-cfcc94b0f526.gif)

위와 같이 특정 section을 터치하면 해당 섹션의 행들이 접히고 펴지는 방법에 대해 볼 것이다. 이것을 구현하기 위한 방법은 크게 두 가지가 있는데,

1. 하나의 tableView를 두고, section 부분을 button 등으로 만들어 터치했을 때 섹션에 포함된 행들이 사라지고 나타나게 하기
2. tableView와 stackView를 같이 써서 stackView의 hide 기능으로 특정 셀을 터치했을 때 stackView의 뷰들이 나타나고 사라지게 하기

원래 나는 2번과 같은 방법으로 구현했는데, 1번이 훨씬 간단하게 구현이 가능하여 1번에 대해 적어보려고 한다.

## 1. UITableViewDelegate, UITableViewDataSource 프로토콜 채택하기

```swift
class ViewController: UIViewController, UITableViewDataSource, UITableViewDelegate
```

## 2. ViewController에 UITableView 추가하고 layout 설정하기

ViewController에 UITableView를 추가하고 layout 설정을 한다. Storyboard를 이용해도 되고 코드 상으로 해도 된다.

## 3. 데이터와 감춰진 데이터 상태를 저장할 집합 생성

먼저 테이블 뷰에 들어갈 데이터가 있어야 할 것이다.

```swift
let tableViewData = [
    ["1","2","3","4","5"],
    ["1","2","3","4","5"],
    ["1","2","3","4","5"],
    ["1","2","3","4","5"],
    ["1","2","3","4","5"],
]
```

그리고 감춰진 섹션을 저장할, 즉 상태를 저장할 변수를 하나 둔다.

```swift
var hiddenSections = Set<Int>()
```

이를 집합으로 선언한 이유는 다음과 같다.

1. 어떤 섹션이 감춰져 있다는 정보는 순서가 중요하지 않고, `Set`은 순서가 없으므로 정렬이 여기서 이슈가 되지 않는다.
2. `Set`은 중복되는 요소가 없기 때문에 실수로 중복되는 요소를 넣는 것을 방지할 수 있다.
3. `Set`을 사용하기 위해서는 hash로 만들 수 있는 타입을 사용해야 한다. 이게 빠르게 탐색하게 하는 것을 가능하기 때문에 이 집합이 커져도 탐색 시간은 오래 걸리지 않을 것이다.

## 4. TableView datasource와 delegatae 메서드 구현하기

먼저 섹션의 개수가 얼마나 있는지를 알려줘야 한다.

```swift
func numberOfSections(in tableView: UITableView) -> Int {
    return self.tableViewData.count
}
```

그리고 각 cell에 대한 정보를 알려준다. 여기서 cell은 register 한 것을 dequeue 해도 되고, 자기 마음대로 구성하면 될 것 같다.

```swift
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = UITableViewCell()
    cell.textLabel?.text = self.tableViewData[indexPath.section][indexPath.row]
    
    return cell
}
```

다음으로 각 섹션에 행이 얼마나 있을지를 알려줘야 한다. 위에서 선언한 감춰진 섹션 집합에 섹션이 포함되면 0을 반환해서 행이 노출되지 않게 해야 하고, 그렇지 않으면 가진 데이터의 개수만큼 노출시키면 된다.

```swift
func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    // 섹션이 hidden이므로 행을 노출시키지 않는다.
    if self.hiddenSections.contains(section) {
        return 0
    }
    
    // 가진 데이터의 개수만큼 노출시킨다.
    return self.tableViewData[section].count
}
```

이 상태로 앱을 실행시키면 아래와 같이 뜬다. 왜냐하면 secion을 어떻게 나타낼지에 대한 정보를 아직 주지 않았기 때문이다.

![image](https://user-images.githubusercontent.com/41438361/122310283-064ae300-cf4b-11eb-85f8-c4e29a24c1a1.png)

이제 section view를 어떻게 구성할지 알려준다. 여기에서는 section 자체를 UIButton으로 구성하는데, 이것도 마음대로 구성하면 될 것 같다.

```swift
func tableView(_ tableView: UITableView, viewForHeaderInSection section: Int) -> UIView? {
    // UIButton 생성
    let sectionButton = UIButton()
    
    // section 제목
    sectionButton.setTitle(String(section),
                           for: .normal)
    
    // section 배경 색
    sectionButton.backgroundColor = .systemBlue
    
    // tag로 섹션을 구분할 것이다.
    sectionButton.tag = section
    
    // section을 터치했을 때 실행할 메서드 설정(밑에서 구현한다.)
    sectionButton.addTarget(self,
                            action: #selector(self.hideSection(sender:)),
                            for: .touchUpInside)

    return sectionButton
}
```

위에서 추가한 `hideSection` 메서드를 구현할 것이다.

```swift
@objc
private func hideSection(sender: UIButton) {
    // section의 tag 정보를 가져와서 어느 섹션인지 구분한다.
    let section = sender.tag
    
    // 특정 섹션에 속한 행들의 IndexPath들을 리턴하는 메서드
    func indexPathsForSection() -> [IndexPath] {
        var indexPaths = [IndexPath]()
        
        for row in 0..<self.tableViewData[section].count {
            indexPaths.append(IndexPath(row: row,
                                        section: section))
        }
        
        return indexPaths
    }
    
    // 가져온 section이 원래 감춰져 있었다면
    if self.hiddenSections.contains(section) {
        // section을 다시 노출시킨다.
        self.hiddenSections.remove(section)
        self.tableView.insertRows(at: indexPathsForSection(),
                                  with: .fade)
    } else {
        // section이 원래 노출되어 있었다면 행들을 감춘다.
        self.hiddenSections.insert(section)
        self.tableView.deleteRows(at: indexPathsForSection(),
                                  with: .fade)
    }
}
```

이제 앱을 다시 실행하면 아래와 같이 특정 section을 누르는 것을 반복했을 때 섹션에 포함된 행들이 나타나고 사라지기를 반복하는 것을 볼 수 있다.

![functionalApp-1](https://user-images.githubusercontent.com/41438361/122310900-f089ed80-cf4b-11eb-8764-ab2aee240e6d.gif)


그런데 여기까지 해도 아직 부족한 점이 있다. 아래의 상황을 보자.

![test](https://user-images.githubusercontent.com/41438361/122311190-99384d00-cf4c-11eb-814e-254b4dc35da2.gif)

감춰져 있는 섹션을 다시 클릭했을 때 나타나는 행들이 화면에 노출되었으면 좋겠는데, 섹션을 열었다 닫아도 테이블 뷰의 현재 노출 위치가 바뀌지 않아서 행들이 나타나는지 안나타나는지 한 눈에 알기 어렵다.

그래서 바로 위에서 구현했던 `hideSection` 메서드의 섹션을 노출시키는 부분에 코드 한 줄을 추가한다.

```swift
...

if self.hiddenSections.contains(section) {
    self.hiddenSections.remove(section)
    self.tableView.insertRows(at: indexPathsForSection(), with: .fade)
    // 섹션을 노출시킬때 원래 감춰져 있던 행들이 다 보일 수 있게 한다.
    self.tableView.scrollToRow(at: IndexPath(row: self.tableViewData[section].count - 1, 
                    section: section), at: UITableView.ScrollPosition.bottom, animated: true)
    
} else {
    self.hiddenSections.insert(section)
    self.tableView.deleteRows(at: indexPathsForSection(),
                              with: .fade)
}

...
```

이제 앱을 실행시키면 아래와 같이 섹션을 노출시킬 때 감춰져 있던 행까지 다 뜬다.

![test2](https://user-images.githubusercontent.com/41438361/122311528-5cb92100-cf4d-11eb-9c83-7fc1290ecd90.gif)


출처

* https://programmingwithswift.com/expand-collapse-uitableview-section-with-swift/
