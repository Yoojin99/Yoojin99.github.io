---  
layout: post  
title: "[iOS] - WWDC19 Advances in UI Data Sources"  
subtitle: ""  
categories: app
tags: app-ios wwdc
comments: true  
header-img: 

---  

[Advances in UI Data Sources](https://developer.apple.com/videos/play/wwdc2019/220/?time=36)

# Current state-of-the-art

<img width="1534" alt="image" src="https://user-images.githubusercontent.com/41438361/175437384-d9c03189-6938-442d-85fb-886824c0afde.png">

당시 `UICollectionView` data source를 구현하기 위해서는 위와 같이 구현해야 했다. 

* 섹션의 수
* 각 섹션 내의 아이템의 수

를 제공하고 있다. 만약 1차원 혹은 2차원 data source를 가지고 작업한다면 특징은 아래와 같다.

* Simple : 단순히 배열 내 요소를 iterate 한다
* Flexible : data source에 정해진 타입이 없다.

<img width="725" alt="image" src="https://user-images.githubusercontent.com/41438361/175437742-8089b66a-b498-4d8c-a2ed-6f4105649b40.png">

하지만 1차원, 2차원 배열보다 더 복잡한 경우가 있다. 앱들은 점점 더 복잡해지고, 기능 요구사항도 많아진다. 

* data source가 앱 내의 복잡한 controller에 의해 뒷받침 될 때가 있다.
* 복잡한 controller는 다양한 작업을 한다.
* Core data와 상호작용하고, 웹 서비스를 이용할 수 있다.

## Conversation between UI layer and Controller

Controller layer는 데이터를 가져오기 위한 복잡한 작업들을 한다.

<img width="1114" alt="image" src="https://user-images.githubusercontent.com/41438361/175437976-80dcaf57-a439-4e48-ad51-d34cadb0cc39.png">

UI layer가 Controller layer에 화면에 출력할 데이터나 cell을 요청한다.

<img width="1097" alt="image" src="https://user-images.githubusercontent.com/41438361/175438164-e861a3e4-71de-4bd2-8757-a73aa0c9b12b.png">

더 복잡한 경우를 생각해보자. 

1. Controller가 웹 서비스 요청 후 응답을 받았다.
2. Controller가 자신이 변했음(자신 내부의 무언가가 변했음)을 알린다.
3. UI layer가 무언가가 변했음을 알고 업데이트를 한다.

## Impoerfect world

<img width="1727" alt="image" src="https://user-images.githubusercontent.com/41438361/175438335-73164631-74fd-4a78-9e2b-111e70f7b3a9.png">

Batch 업데이트를 에러 없이 수행하기 위해 노력해도 안 되는 경우가 있다. 개발하며 위와 같은 에러를 마주한 적이 있을 것이다. 이를 해결하기 위해 `reloadData`를 호출하면 애니메이션 없이 reload가 수행된다.

## Where is our truth?

<img width="925" alt="image" src="https://user-images.githubusercontent.com/41438361/175438552-0a58358d-6a65-4fe9-96c1-b324c13df8b3.png">

문제는 "truth"가 어디있냐는 것이다. 여러 군데에서 업데이트가 발생하고 데이터가 업데이트 됐을 때 layer 별로 특정 시간대에 다른 정보를 가질 수 있다. 이 때 정답은 누가 가지고 있느냐가 문제인 것이다.

* 위 상황에서 문제는 **data controller(data source)가 시간마다 변하는 자신만의 truth를 가진다는 것이다.** 그리고 UI도 자신만의 truth를 가지고 있다.
truth는 하냐여야 하는데 data고 ui고 자신만의 truth를 가지고 있으니 문제가 된다.
* UI layer 코드는 이 두 버전의 truth가 항상 동기화되어야 하는 것에 대한 책임이 있다. 하지만 위 에러에서도 봤듯이 어려운 상황이 있다. 따라서 이런 접근 방식은 에러가 발생하기 쉽다.
* 즉 **Centralized truth**가 없다는 것이 문제가 된다.

# A new approach

<img width="1749" alt="image" src="https://user-images.githubusercontent.com/41438361/175439058-c70d5f49-c67d-4ba3-99a6-c4c15bb48e7c.png">

새로운 접근 방식인 DiffableDataSource가 등장했다.

<img width="1114" alt="image" src="https://user-images.githubusercontent.com/41438361/175439121-dd266a5f-69b6-4b42-9c79-0cdd8505827d.png">

DiffableDataSource는

* `performBatchUpdates`가 없다. : 모든 crash, hassle, 복잡도가 없어졌다.
* `apply` 메서드가 생겼다. : 간단하고, 자동적이고, hassle이 없는 diffing(차이를 적용하는 작업)이다.

## Snapshot

<img width="839" alt="image" src="https://user-images.githubusercontent.com/41438361/175439255-f7f2a5e8-6346-41fb-bb7a-1b4464c986da.png">

Snapshot이라는 새로운 구조를 사용할 것이다. 

* 현재 UI 상태의 truth다.
* `IndexPath` 대신, 유일한 섹션 식별자와 아이템 식별자를 가지고 있다.
* `IndexPath` 대신 아이템 식별자로 업데이트한다.

<img width="800" alt="image" src="https://user-images.githubusercontent.com/41438361/175439853-55312d60-ef68-491d-b8aa-224861059840.png">

1. 현재 화면에 FOO, BAR, BIF를 식별자로 가진 것들이 출력되고 있다고 하자.
2. Controller가 변했다. 왼쪽의 새로운 snapshot을 적용하고 싶다.
3. `apply()`를 통해 새로운 상태를 현재 상태에 반영한다.

## Diffable Data Source

<img width="652" alt="image" src="https://user-images.githubusercontent.com/41438361/175440094-c68683d6-3953-4f54-9f04-5015d950c521.png">

모든 플랫폼에 4개의 클래스가 있다.

* iOS, TVoS : `UICollectionViewDiffableDataSource`, `UITableViewDiffableDataSource`
* mac : `NSCollectionViewDiffableDataSource`

모든 플랫폼에 현재 UI 상태에 대한 책임이 있는 `NSDiffableDataSourceSnapshot`가 있다.

# Demos

3단계 작업으로 이루어진다.

1. 변화가 생긴 후, snapshot을 생성한다. 변화를 collectionView나 tableView에 적용하기 위한 Snapshot이다.
2. Snapshot에 업데이트 사이클에 출력하고 싶은 아이템들에 대한 내용을 포함시킨다.
3. `apply()`를 통해 UI를 자동으로 바꾼다.

DiffableDataSource는 차이와 변화를 UI에 반영하는 것들을 관리해준다.

## Demo 1

![mountain](https://user-images.githubusercontent.com/41438361/175467809-ea1565ab-a9bc-47c6-94b1-97500710f5c5.gif)

산 이름을 검색할 수 있는 앱이다. 검색창에 검색어를 입력할때마다 하단에 나오는 산의 목록이 자동으로 업데이트 된다.

<img width="780" alt="image" src="https://user-images.githubusercontent.com/41438361/175441133-83d3dc25-9e24-4777-9eae-d1a4d9574a11.png">

### 1. Snapshot 생성

```swift
var snapshot = NSDiffableDataSourceSnapshot<Section, MountainsController.Mountain>()
```

Snapshot은 Swift의 제네릭 클래스다. 사용하고자 하는 `SectionIdentifierType`과 `ItemIdentifierType`을 파라미터로 가지고 있다. 
여기에서 사용된 section, item identifier type을 보자.

#### SectionIdentifierType

```swift
enum Section: CaseIterable {
    case main
}
```

하나의 섹션만 필요할 때 위와 같이 enum으로 만들어주면 유용하다. Enum이 좋은 것은 자동으로 hashable 하다는 것이다. 그래서 하나의 case가 있는 enum을 생성해서 SectionIdentifierType을 만들었다.

#### ItemIdentifierType

```swift
struct Mountain: Hashable {
    let name: String
    let height: Int
    let identifier = UUID()
    func hash(into hasher: inout Hasher) {
        hasher.combine(identifier)
    }
    static func == (lhs: Mountain, rhs: Mountain) -> Bool {
        return lhs.identifier == rhs.identifier
    }
    func contains(_ filter: String?) -> Bool {
        guard let filterText = filter else { return true }
        if filterText.isEmpty { return true }
        let lowercasedFilter = filterText.lowercased()
        return name.lowercased().contains(lowercasedFilter)
    }
}
```

`Mountain` struct를 생성했다. 이 구조체가 hashable하게 해서 identifier를 명시적으로 전달하는 대신 사용할 수 있게 했다. 중요한 것은 각 mountain이 hash 값에 의해 식별돼서 DiffableDataSource에서 사용할 수 있다는 것이다.
위 코드에서는 hash 값으로 identifier를 그냥 제공하고 있다. 이 identifier는 DiffableDataSource가 이전에서 다음으로 업데이트 할 때 바뀐 게 무엇인지를 추적하기 위해 사용할 수 있는 식별자다.
그래서 모든 mountain은 고유한 identifier를 가지고 있어야 한다.

### 2. 다음 업데이트에 출력하고 싶은 아이템의 identifier 추가

```swift
snapshot.appendSections([.main])
snapshot.appendItems(mountains)
```

`snapshot`은 처음에 완전 비어있는 상태이기 때문에 여기에 우리가 원하는 섹션과 아이템을 추가해야 한다. 이 경우 하나의 섹션만 필요하기 때문에 `main`이라는 섹션 하나를 추가했다.

그리고 아이템들의 identifier를 추가했다. 주로 여기에서는 identifier의 배열을 전달한다. Swift에서는 여기에 내 **커스텀 타입을 사용할 수 있다.**

**Native 타입(struct, enum과 같은 값 타입도 포함)을 hashable하게 만들면 객체 자체를 전달할 수 있다.**

### 3. apply

```swift
dataSource.apply(snapshot, animatingDifferences: true)
```

Snapshot을 적용한다. 이러면 DiffableDataSource는 자동으로 이전 업데이트에서 어떤 것이 바뀌는지 찾는다.

* idexPath : fragile, ephemeral. 특정 업데이트를 참조하고, 다른 의미와 다른 업데이트를 갖는다.
* identifier : robust, enduring. 간단하다.

### collectionView와 연결

<img width="851" alt="image" src="https://user-images.githubusercontent.com/41438361/175465402-acc8d370-bac1-4ecd-9add-5f268baeffa3.png">

`configureDataSource`라는 메서드를 생성해서 datasource를 설정했다. UICollectionView로 작업하고 있기 때문에 UICollectionViewDiffableDataSource 객체를 만들었다. 여기에 섹션, 아이템 타이템 타입을 인자로 전달했다.

```swift
dataSource = UICollectionViewDiffableDataSource<Section, MountainsController.Mountain>(collectionView: mountainsCollectionView) {
    (collectionView: UICollectionView, indexPath: IndexPath, identifier: MountainsController.Mountain) -> UICollectionViewCell? in
    // Return the cell.
    return collectionView.dequeueConfiguredReusableCell(using: cellRegistration, for: indexPath, item: identifier)
}
```

같이 사용하고자 하는 collectionView를 가리키는 포인터를 전달한다. DiffableDataSource는 이 포인터를 받아 해당 collectionView를 위한 data source로서 자신을 연결한다.

클로저 안에 있는 부분은 `cellForItemAt IndexPath` 메서드를 구현할 때 작성하던 부분과 비슷하다. CollectionView에 출력하고자 하는 적절한 cell과 데이터를 받는다.

여기에서 편리한 점은 아이템의 IndexPath를 사용하지 않고 identifier를 사용해서(여기에서는 native Swift 값 타입을 사용해서) 우리가 출력하고 싶은 아이템을 표시한 것이다.

```swift
let cellRegistration = UICollectionView.CellRegistration<LabelCell, MountainsController.Mountain> { (cell, indexPath, mountain) in
    // Populate the cell with our item description.
    cell.label.text = mountain.name
}
```

위 `cellRegisteration`을 사용해서 cell이 주어졌을 때 데이터를 어떻게 바인딩할지를 나타냈다.

## Demo 2

![wifi](https://user-images.githubusercontent.com/41438361/175467838-7cd89150-2a9b-4e25-91e1-f65c0af8f8fc.gif)

iOS의 와이파이 설정 부분을 mockup 한 것이다. 이전 데모와 다른 점은 **섹션이 두개라는 것이다.** 

* config section : 상단 섹션. 와이파이를 끄거나 킬 수 있다. 와이파이를 끄면 하단 섹션이 화면에 나타나지 않는다.
* wifi section : 하단 섹션. 현재 연결할 수 있는 네트워크들이 계속 동적으로 업데이트 되며 나타난다.

여기도 마찬가지로 세 단계로 이루어진다.

<img width="847" alt="image" src="https://user-images.githubusercontent.com/41438361/175470496-f6dcc77c-af08-4c8b-af08-bd1e5fff46ee.png">

### 1. Snapshot 생성

```swift
let configItems = configurationItems.filter { !($0.type == .currentNetwork && !controller.wifiEnabled) }

currentSnapshot = NSDiffableDataSourceSnapshot<Section, Item>()
```

출력하고 싶은 데이터를 받는다. 그리고 Snapshot을 생성한다. 이 Snapshot은 초기에 비어있는 상태이기 때문에 우리가 출력하고 싶은 부분으로 snapshot을 구성해야 한다.

#### SectionIdentifierType

<img width="223" alt="image" src="https://user-images.githubusercontent.com/41438361/175471135-609cc1e4-3f42-4e18-9c00-8f965dc9f167.png">

섹션을 enum으로 생성해서 자동으로 hashable하게 만들었다.

#### ItemIdentifierType

<img width="545" alt="image" src="https://user-images.githubusercontent.com/41438361/175471223-fdb0f463-19e9-4887-b228-06ae2cba3d42.png">

Demo 1의 Mountain과 비슷하게, hashable한 struct를 만들었다. hash 함수에서는 고유한 identifier 값을 기반으로 연산하고 있어서 각 아이템을 구별할 수 있게 해준다.

### 2. 다음 업데이트에 출력하고 싶은 아이템의 identifier 추가

```swift
currentSnapshot.appendSections([.config])
currentSnapshot.appendItems(configItems, toSection: .config)

if controller.wifiEnabled {
    let sortedNetworks = controller.availableNetworks.sorted { $0.name < $1.name }
    let networkItems = sortedNetworks.map { Item(network: $0) }
    currentSnapshot.appendSections([.networks])
    currentSnapshot.appendItems(networkItems, toSection: .networks)
}
```

먼저 상단의 config 섹션을 추가하고 아이템을 추가한다. 

와이파이가 활성화되어 있으면 model layer에 연결 가능한 와이파이 목록을 요청한다. 네트워크 섹션을 추가하고, 모델 레이어에서 받은 아이템을 해당 섹션에 추가한다.
우리가 아이템을 추가할 섹션을 명시적으로 나타낼 수 있다.

이제 화면에 출력하고 싶은 모든 것을 다 작성했다.

### 3. apply

```swift
self.dataSource.apply(currentSnapshot, animatingDifferences: animated)
```

DiffableDataSource에게 변화를 적용할 것을 요청했다. 

### collectionView와 연결

자세히 볼 필요는 없다. Demo1에서 한 것과 크게 다르지 않은데, 코드가 긴 것은 셀 타입에 따라 cell을 구성하는 코드가 길기 때문이다.

```swift
func configureDataSource() {
    wifiController = WiFiController { [weak self] (controller: WiFiController) in
        guard let self = self else { return }
        self.updateUI()
    }

    self.dataSource = UITableViewDiffableDataSource
        <Section, Item>(tableView: tableView) { [weak self]
            (tableView: UITableView, indexPath: IndexPath, item: Item) -> UITableViewCell? in
        guard let self = self, let wifiController = self.wifiController else { return nil }

        let cell = tableView.dequeueReusableCell(
            withIdentifier: WiFiSettingsViewController.reuseIdentifier,
            for: indexPath)

        var content = cell.defaultContentConfiguration()
        // network cell
        if item.isNetwork {
            content.text = item.title
            cell.accessoryType = .detailDisclosureButton
            cell.accessoryView = nil

        // configuration cells
        } else if item.isConfig {
            content.text = item.title
            if item.type == .wifiEnabled {
                let enableWifiSwitch = UISwitch()
                enableWifiSwitch.isOn = wifiController.wifiEnabled
                enableWifiSwitch.addTarget(self, action: #selector(self.toggleWifi(_:)), for: .touchUpInside)
                cell.accessoryView = enableWifiSwitch
            } else {
                cell.accessoryView = nil
                cell.accessoryType = .detailDisclosureButton
            }
        } else {
            fatalError("Unknown item type!")
        }
        cell.contentConfiguration = content
        return cell
    }
    self.dataSource.defaultRowAnimation = .fade

    wifiController.scanForNetworks = true
}
```

위에서 클로저 안에 존재하는 긴 코드는 diffableDataSource를 사용하지 않더라도 `cellForItem IndexPath` 메서드 안에 존재했을 코드다.

## Demo 3

![color](https://user-images.githubusercontent.com/41438361/175472134-5017a315-2d47-451e-9b31-d117b69b6585.gif)

처음에 색이 랜덤하게 존재하는데, sort 버튼을 누르면 무지개 순으로 색이 정렬된다. 이전 데모와 다른 점은 초기의 랜덤한 상태에서, 최종 상태(무지개)로 바로 건너뛰는 것이 아니라,
각 정렬 과정을 중간에 볼 수 있다는 것이다. 즉 초기 상태에서 바로 최종 상태로 건너 뛰는 것이 아니라, 각 단계마다 정렬되는 과정을 UI에 반영해야 한다는 것이다.
이에 맞게 각 정렬 순간마다 Snapshot을 생성하고 적용해야 한다.

Demo 3에서 3단계를 진행하는 코드는 아래와 같다.

```swift
func performSortStep() {
    if !isSorting {
        return
    }

    var sectionCountNeedingSort = 0

    // Get the current state of the UI from the data source.
    var updatedSnapshot = dataSource.snapshot()

    // For each section, if needed, step through and perform the next sorting step.
    updatedSnapshot.sectionIdentifiers.forEach {
        let section = $0
        if !section.isSorted {

            // Step the sort algorithm.
            section.sortNext()
            let items = section.values

            // Replace the items for this section with the newly sorted items.
            updatedSnapshot.deleteItems(items)
            updatedSnapshot.appendItems(items, toSection: section)

            sectionCountNeedingSort += 1
        }
    }

    var shouldReset = false
    var delay = 125
    if sectionCountNeedingSort > 0 {
        dataSource.apply(updatedSnapshot)
    } else {
        delay = 1000
        shouldReset = true
    }
    let bounds = insertionCollectionView.bounds
    DispatchQueue.main.asyncAfter(deadline: .now() + .milliseconds(delay)) {
        if shouldReset {
            let snapshot = self.randomizedSnapshot(for: bounds)
            self.dataSource.apply(snapshot, animatingDifferences: false)
        }
        self.performSortStep()
    }
}
```

### 1. Snapshot 생성

```swift
// Get the current state of the UI from the data source.
var updatedSnapshot = dataSource.snapshot()
```

이전 Demo들에서는 새로운 snapshot을 생성했는데, 여기에서는 현재 snapshot을 가져왔다. 이 snapshot은 현재 UICollectionView가 보여지고 있는, 가지고 있는 현재의 truth가 된다.
이렇게 하는 이유는 우리는 정렬되는 모든 과정을 다 보여줘야 하기 때문이다. 이를 통해 완전히 새로 시작하지 않고 중간 상태에서 시작해서 다음 상태로 넘어갈 수 있다.

#### SectionIdentifierType

#### ItemIdentifierType

### 2. 다음 업데이트에 출력하고 싶은 아이템의 identifier 추가

```swift
// For each section, if needed, step through and perform the next sorting step.
updatedSnapshot.sectionIdentifiers.forEach {
    let section = $0
    if !section.isSorted {

        // Step the sort algorithm.
        section.sortNext()
        let items = section.values

        // Replace the items for this section with the newly sorted items.
        updatedSnapshot.deleteItems(items)
        updatedSnapshot.appendItems(items, toSection: section)

        sectionCountNeedingSort += 1
    }
}
```

`appendItems`, `deleteItems` 등 snapshot을 수정할 수 있는 다양한 함수들이 존재한다. 이들을 사용해서 item들의 순서를 바꿀 수 있다.
여기에서도 이전 데모들과 마찬가지로 indexPath가 아닌 identifier를 사용해서 우리가 원하는 새로운 상태로 snapshot을 설정한다.

### 3. apply

```swift
var shouldReset = false
var delay = 125
if sectionCountNeedingSort > 0 {
    dataSource.apply(updatedSnapshot)
} else {
    delay = 1000
    shouldReset = true
}
let bounds = insertionCollectionView.bounds
DispatchQueue.main.asyncAfter(deadline: .now() + .milliseconds(delay)) {
    if shouldReset {
        let snapshot = self.randomizedSnapshot(for: bounds)
        self.dataSource.apply(snapshot, animatingDifferences: false)
    }
    self.performSortStep()
}
```

# Considerations
