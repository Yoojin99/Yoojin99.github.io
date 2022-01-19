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

이 모든 동작을 보면 사용자가 검색창에 타이핑을 시작할 때 시작한다. 그래서 `searchBarTextDidChange` 라는 callback이 있고, 이 callback은 controller에 전달된다.

```swift
extension MountainsWindowController: NSSearchFieldDelegate {
    func controlTextDidChange(_ obj: Notification) {
        performQuery(with: searchField.stringValue)
    }
}
```

`performQuery` 함수에는 검색창에 있는 검색어를 전달해준다. `performQuery` 함수 자체는 매우 간단하다. `MountainsController`라는 모델 레이어 객체를 호출한다. 그리고 검색된 단어에 맞는 산들만 필터링 된 결과를 요청한다. 이러면 우리는 산들의 리스트를 받을 수 있다. 그리고 위에서 언급한 세단계를 수행한다. **먼저 새로운 NSDiffableDataSourceSnapshot을 생성한다.** 이 Snapshot은 처음에는 비어있다. 그래서 우리가 원하는 섹션과 아이템을 채우는 것은 우리의 몫이다. 이 케이스에서 우리는 출력할 섹션이 하나밖에 없기 때문에 하나의 섹션을 추가하고, 이를 main section이라 부를 것이다. 그리고 이번 업데이트에 출력하고 싶은 아이템들의 identifier들을 추가한다. 여기서의 예제를 보면 identifier들의 배열을 전달하고 있다. 하지만 Swift에서는 자신만의 native type들을 사용해서 더 고급스럽게 표현할 수도 있다. Native type이 될 수도 있고, 값 타입(struct/enum)이 될 수도 있는데 이 타입을 hashable하게 만든다면 이 객체들을 전달할 수도 있다. 이제 Snapshot은 다 만들었으니 DiffableDataSource에 이 snapshot을 적용해서 차이를 애니메이션으로 보여달라고 요청한다. DiffableDataSource는 자동적으로 이전 업데이트와 다음 업데이 사이에 바뀐 점들을 찾아낸다.

```swift
private func performQuery(with filter: String?) {
    let mountains = self.mountainsController.filteredMountains(with: filter).sorted { $0.name < $1.name }

    var snapshot = NSDiffableDataSourceSnapshot<Section, MountainsController.Mountain>()
    snapshot.appendSections([.main])
    snapshot.appendItems(mountains)
    dataSource.apply(snapshot, animatingDifferences: true)
}
```

Snapshot은 Swift의 generic 클래스이기 때문에, SectionIdentifierType과 ItemIdentifierType을 파라미터로 전달해줘야 한다. 여기서는 Section을 보면 단일하면서 딱히 쓸모가 없다. 이럴 때 자주 사용할 수 있는 기술은 이를 위한 enum 타입을 하나 생성하는 것이다. 그래서 SectionIdentifierType을 보면 enum으로 정의되어 있다. 그리고 Swift에서 enum이 유용한 점 하나는 자동으로 hashable이라는 것이다.

![image](https://user-images.githubusercontent.com/41438361/150049576-13842c3f-13b2-4fda-8fab-f3a28b5dc702.png)

MountainType은 MountainsController(model layer)에서 확인할 수 있다. 여기에서는 Mountain들을 struct로 구성했다. 그리고 이 struct type을 hashable하게 만들어 단순히 identifier를 전달하는 것이 아니라 그 자체를 DiffableDataSource에 전달할 수 있게 했다. 여기에서 중요한 것은 각 산들이 hash 값으로 각각 유일하게 정의가 되어야 한다는 것이다.

![image](https://user-images.githubusercontent.com/41438361/150050756-f2a3c473-19bc-4cca-b0b4-80fc3214986e.png)

다시 MountainsViewController로 돌아와서, `configureDataSource`라는 메서드를 보면 data source를 구성하고 있다. 우리는 이 예제에서 UICollectionView를 사용하고 있기 때문에 `UICollectionViewDiffableDataSource`를 초기화했고, 우리의 section과 item 타입으로 파라미터를 전달해줬다. 그리고 diffableDataSource에 연결하고 싶은 collectionview를 전달한다. 마지막으로 끝의 클로저부분에서 `cellForItemAt IndexPath` 메서드를 구현할 때 작성했던 코드들을 작성한다. 우리는 collectionView에 callback을 하고 적절한 타입의 cell을 요청한다. 그리고 cell을 생성한 다음 리턴해준다. 여기에서 달라진 것은 우리가 요청한 IndexPath가 주어지는 것에 추가로 identifier가 주어진다는 것이다. 

```swift
func configureDataSource() {
        
    let cellRegistration = UICollectionView.CellRegistration
    <LabelCell, MountainsController.Mountain> { (cell, indexPath, mountain) in
        // Populate the cell with our item description.
        cell.label.text = mountain.name
    }

    dataSource = UICollectionViewDiffableDataSource<Section, MountainsController.Mountain>(collectionView: mountainsCollectionView) {
        (collectionView: UICollectionView, indexPath: IndexPath, identifier: MountainsController.Mountain) -> UICollectionViewCell? in
        // Return the cell.
        return collectionView.dequeueConfiguredReusableCell(using: cellRegistration, for: indexPath, item: identifier)
    }
}
```

또 다른 예제를 보자. 이번에는 iOS의 Wi-Fi 설정 화면과 비슷한 화면이다. 이게 이전 예제보다 좀 더 복잡한 이유는 두 개의 섹션이 있기 때문이다. 첫 번째 섹션은 Config 섹션으로, 와이파이를 끄고 켤 수 있다. 그리고 그 밑에는 우리가 연결할 수 있는 와이파이 목록을 동적으로 보여주는 섹션이 있다. 그리고 만약 와이파이 끄기 버튼을 토글시킨다면 UI가 애니메이션으로 사라지거나 나타나는 걸 볼 수 있고, 이것은 모두 DiffableDataSource로 할 수 있다. 이 동적 UI가 어떻게 구현되어 있는지 볼 것이다.

![wifi1](https://user-images.githubusercontent.com/41438361/150053134-18220669-0781-405d-b34d-1481ed48d84e.gif)

WifiSettingsViewController를 보면 `updateUI`라는 메서드가 있다. 그리고 이 메서드는 우리가 출력하고 싶은 것에 변화가 있을 때마다 호출되도록
했다. 거의 항상 현재 사용 가능한 네트워크가 달라지기 때문에 이와 같이 만들었다. 이 말고도 사용자가 와이파이 켜기/끄기 버튼을 토글할 때도 변화가 일어날 수 있다. 언제건 간에 UI를 변화시켜야 할 때마다 이 메서드가 호출될 것이다. 그리고 여기서도 마찬가지로 위에서 언급한 세 단계를 거친다. 우리가 출력하고 싶은 데이터를 받은 다음, 스냅샷을 생성하고 스냅샷은 처음에 비어있기 때문에 데이터로 이를 구성할 것이다. 먼저 첫번째 섹션인 config 섹션을 추가하고, 아이템을 추가한다. 그래서 여기에는 와이파이가 켜져있는지 꺼져있는지에 따라 하나 혹은 두 개의 아이템이 있을 수 있다. 와이파이가 켜지면 우리의 model layer에서 현재 사용가능한 네트워크 리스트를 받는다. 그리고 이 리스트를 특정 타입의 아이템으로 감쌀 건데, 이는 뒤에서 볼 것이다. 
네트워크 리스트를 위한 섹션을 추가하고, 아이템들을 추가한다. 여기에서 우리가 두 개의 다른 섹션을 가지고 작업하고 있기 때문에, 우리가 어떤 섹션에 아이템을 추가하는지를 명시할 수 있다. 이제 DiffableDataSource에 이 변화들을 적용해달라고 요청한다. 이때 애니메이션을 적용하지 않고 싶을 때도 있기 때문에 이런 사항은 옵션으로 넘겨주면 되겠다.

```swift
func updateUI(animated: Bool = true) {
    guard let controller = self.wifiController else { return }

    let configItems = configurationItems.filter { !($0.type == .currentNetwork && !controller.wifiEnabled) }

    currentSnapshot = NSDiffableDataSourceSnapshot<Section, Item>()

    currentSnapshot.appendSections([.config])
    currentSnapshot.appendItems(configItems, toSection: .config)

    if controller.wifiEnabled {
        let sortedNetworks = controller.availableNetworks.sorted { $0.name < $1.name }
        let networkItems = sortedNetworks.map { Item(network: $0) }
        currentSnapshot.appendSections([.networks])
        currentSnapshot.appendItems(networkItems, toSection: .networks)
    }

    self.dataSource.apply(currentSnapshot, animatingDifferences: animated)
}
```

그리고 위 코드에서 `Secion`과 `Item` 타입을 보자. section은 enum 타입이고, 두 개의 섹션이 필요하니 케이스를 두 개 만든다. 위에서도 잠깐 언급했는데 enum type은 Swfit에서 hashable 하기 때문에 이를 바로 사용할 수 있다. Item 타입은 산 예제에서 봤던 것과 같이 struct type이고, 이것이 hashable을 따르도록 구현한다. 그리고 여기에서 Item type을 정의한 이유는 우리의 리스트를 보면 대부분 네트워크 목록을 포함하고 있다. 하지만 맨 처음 아이템을 보면 와아피아 켜기/끄기 스위치가 있는데 이는 네트워크 아이템이 아니다. 그래서 여기서 단일 리스트를 가지고 있다. 그리고 각 아이템들을 이 generic wrapper type으로 감싼다. 하지만 이 wrapper type이 우리가 DiffableDataSource에 넘겨줄 타입이기 때문에 이게 hashable을 따르게 하고 아이템들이 hash 값에 의해 유일하게 정의되어야 함을 보장해줘야 한다.

![image](https://user-images.githubusercontent.com/41438361/150055361-003ab94e-31db-45fb-b53d-8e17275e3371.png)
![image](https://user-images.githubusercontent.com/41438361/150055526-04a379b8-07a3-4690-8e26-533b99d37e8c.png)

이제 data source를 구성하는 부분을 볼 것이다. 이전에 했던 것과 비슷한데, 여기에서는 UITableView로 작업을 한다. 하지만 snapshot을 생성하는 관점에서 보면 API가 굉장히 비슷하기 때문에 신경쓰지 않아도 된다. `UITableViewDiffalbeDataSource`를 초기화하는데, 우리가 사용할 섹션과 아이템 타입들을 인자로 넘겨준다. 그리고 같이 사용할 table view를 넘겨준다. 마지막으로 item provider closure를 구현한다. 확실히 이전 산 예제에 비해서 복잡한데, 이는 우리가 다양한 타입의 아이템들을 갖고 있기 때문이다. 세 개의 아이템 타입이 있기 때문에 이를 다르게 다뤄주고 있다. 하지만 이 코드는 DiffableDataSource를 사용하고 있지 않아도 작성해야 할 코드다. `cellForItem IndexPath` 메서드에 있을 코드다. 

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

마지막 예제는 흥미로운 예제인데, 다른 예제들과 update를 생성하고 commit하는 방법에 있어서 약간의 차이가 있다. 이 예제는 정렬 과정을 각 단계마다 볼 수 있는 예제이다. 이를 위해서 단계를 거쳐 진행되는 메서드가 있고, 한 단계마다의 결과를 받을 수 있게 했다. 얘는 연속적인 새로운 상태를 제공하고, 우리는 이게 제공될때마다 snapshot을 생성하고 적용한다. 

![colorsort](https://user-images.githubusercontent.com/41438361/150058347-e3a80a63-ee8b-47bc-8b2d-9692cdd7941e.gif

InsertionSortViewController를 보면 `performSortStep` 메서드가 있다. 앞에서도 계속 말했듯이, 세 단계 사이클이 있다. 스냅샷을 생성하고, 채우고, 적용한다. 하지만 이 예제에서 새로운, 비어있는 스냅샷을 생성하지 않고 DiffableDataSource에게 현재의 Snapshot을 요청할 것이다. 이제 Snapshot은 현재 UICollectionView에 보여지는 현재의 truth로 미리 채워져 있다. 우리는 이 상태에서 다음 단계를 연산해주면 된다. 코드를 보면 Snapshot을 채울 때 예전에 사용했던 `appendItems` 메서드를 볼 수 있다. 추가로 `deleteItems` 함수도 볼 수 있다. 그리고 Snapshot API를 보면 존재하는 스냅샷을 수정하기 위한 다양한 함수들이 존재한다.

![image](https://user-images.githubusercontent.com/41438361/150073480-797ca5f3-17e1-4f89-a1f7-ff5414592c36.png)

하지만 다른 측면에서 이 코드에서 하고 있는 작업은 위의 다른 예제들에서 했던 것과 매우 비슷하다. 우리가 출력하고 싶은 최종 상태를 만들고 있는 것이다. 


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

그리고 DiffableDataSource를 설정하는 코드는 아래와 같다. 

```swift
func configureDataSource() {
        
    let cellRegistration = UICollectionView.CellRegistration
    <UICollectionViewCell, InsertionSortArray.SortNode> { (cell, indexPath, sortNode) in
        // Populate the cell with our item description.
        cell.backgroundColor = sortNode.color
    }

    dataSource = UICollectionViewDiffableDataSource<InsertionSortArray, InsertionSortArray.SortNode>(collectionView: insertionCollectionView) {
        (collectionView: UICollectionView, indexPath: IndexPath, node: InsertionSortArray.SortNode) -> UICollectionViewCell? in
        // Return the cell.
        return collectionView.dequeueConfiguredReusableCell(using: cellRegistration, for: indexPath, item: node)
    }

    let bounds = insertionCollectionView.bounds
    let snapshot = randomizedSnapshot(for: bounds)
    dataSource.apply(snapshot)
}
```

### 4. Considerations

위의 예제들에서 봤듯이 세 단계로 DiffableDataSource를 사용할 수 있다. 

1. Snapshot 생성
2. Snapshot 채우기
3. Apply

그래서 이제 `performBatchUpdates` 대신 `apply` 메서드를 호출하고 있다. 

스냅샷을 생성하는 데는 두 가지 방법이 있었다. 가장 일반적인 방법은 빈 스냅샷을 만들고, Snapshot을 섹션과 아이템으로 채운다. 또한 현재 상태에서 스냅샷을 생성할 수도 있다. 이는 수정사항이 작을 때 유용하다. 스냅샷을 생성하면 truth의 복사본을 얻는 거라 이를 수정할 수 있는데 이런 수정은 data source에 영향을 주지 않는다. 

스냅샷을 생성하면 아이템이 얼마나 있는지, 섹션이 얼마나 있는지, identifier들에 어떤 것들이 있는지를 API를 통해 확인할 수 있다. API에서 더는 IndexPath를 사용하지 않는다. 그래서 아이템을 추가하고 섹션을 추가하는 과정을 통해 봤던 방식을 사용하면 된다. 이런 작업들을 insert, move, delete를 통해 수행할 수 있다. 

![image](https://user-images.githubusercontent.com/41438361/150075831-f14f0cdd-73b9-4de9-8ccc-457d89e90668.png)


* 참고
* https://developer.apple.com/videos/play/wwdc2019/220/?time=2097
