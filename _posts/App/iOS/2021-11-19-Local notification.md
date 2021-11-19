---  
layout: post  
title: "[iOS] - User Notificaation으로 Local notification 알림 띄우기"  
subtitle: ""  
categories: app
tags: app-ios UserNotifications local notification 알림
comments: true  
header-img: 

---  
  
> `UserNotifications를 이용해서 local notification을 띄우는 방법에 대해 알아보자.`  

---

![image](https://user-images.githubusercontent.com/41438361/142576728-4320f977-eb07-4671-b700-10ac4c337858.png)

위의 사진과 같은 Local notification을 띄워보려고 한다. 참고로 notification 은 local/remote 이 두 가지 방법이 존재한다. 그 중에서도 remote로 앱이 위와 같은 noti를 띄우게 하는 걸
push 알림이라고 한다. 

하지만 그 중에서도 오늘은 local notification을 띄우는 방법에 대해 알아보려고 한다.

## 1. 사용자 권한 요청하기

![image](https://user-images.githubusercontent.com/41438361/142576906-d596c815-3d35-412a-a764-1b2be97ce570.png)

Local notification을 띄우든 push notification을 띄우든 사용자의 권한을 얻는 작업은 꼭 필요하다. 

![image](https://user-images.githubusercontent.com/41438361/142576976-71b06eef-3896-43a4-b9e8-55dfce2ae22d.png)

알림을 띄울 때 알림을 띄울지, 소리도 같이 낼 지, 앱 아이콘에 배지를 달 수 있는지에 대한 권한을 요청하는데, 이런 권한을 요청하는 이유는 사용자가 이런 알림들에 의해 방해받는다는 느낌을 받을 수 있기 때문이다.

위와 같이 명시적으로 사용자에게 권한을 요청하려면 

1. `UNUserNotificationCenter`의 공유 인스턴스를 불러와서
2. 이 인스턴스의 `requestAuthorization(options:completionHandler:)` 메서드를 호출한다. 이때 앱이 사용할 모든 상호작용 타입을 명시한다. (배지, 소리 등등)

이런 권한 요청은 처음 요청될 때 사용자에게 허용할지, 혹은 거절할지를 선택하게 하고, 이를 기억해뒀다가 이후 이런 권한 요청을 하지 않는다.

먼저 이를 위해 프로젝트를 하나 만들었다. 그리고 기본으로 생성된 `ViewController` UIViewController 클래스에 아래 메서드를 추가해줬다.

```swift
private func askPermissionForNotification() {
    let center = UNUserNotificationCenter.current()
    center.requestAuthorization(options: [.alert, .sound, .badge]) { granted, error in
        if let error = error {
            print("Error(\(error)) occured during asking permission for notification")
        }
    }
}
```

여기에서 설정해주는 `.alert`, `.sound`, `.badge`는 `UNAuthorizationOptions` 라는 구조체의 전역변수로 각각 다른 형태의 notification을 나타낸다.

그리고 이를 `viewDidLoad()` 메서드에서 호출해줬다.

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    // Do any additional setup after loading the view.

    view.backgroundColor = .white

    askPermissionForNotification()
}
```

그리고 simulator를 실행했더니 아래와 같이 알림에 대한 권한을 요청하는 화면이 뜨는 것을 확인할 수 있다. 


이 알림 메세지에 뜨는 문구가 내가 요청한 authorizationOption 타입에 따라 달라지는지도 테스트해봤는데 바뀌지는 않았고, 고정 문구로 나왔다.

그리고 이제 여기에서 "Allow"를 누르고 다시 simulator를 실행시키면 simulator가 이전에 권한을 허용할지 말지를 선택한 걸 기억하고 있어서 다시 이런 권한을 요청하는 메세지가 뜨지 않는다. 

## 2. Schedule Notification 

알림을 띄울 때 알림을 띄우거나, 소리를 내거나, 앱 아이콘에 배지를 달 수도 있다. 시스템은 내가 명시한 시간과 장소에 따라 알림을 띄울 수 있다. 만약 앱이 실행되고 있지 않거나 백그라운드에 있는 상태일 때 알림을
띄워야 한다면 시스템은 대신 사용자와 대신 상호작용 할 수 있고, 만약 앱이 띄워져 있는 상태에서 알림을 띄운다면 시스템은 알림을 앱에서 띄워준다.

알림을 만드는 방법을 보자.

### 1. 알림의 Content 설정하기

`UNMutableNotificationContent` 객체의 프로퍼티를 명시해서 알림의 여러 기능들을 설정한다. 여기에서 정의하는 내용들은 시스템이 알림을 어떻게 전달할지를 정의한다. 

```swift
private func createNotificationContent(title: String, subTitle: String) -> UNMutableNotificationContent {
    let content = UNMutableNotificationContent()
    content.title = title
    content.subtitle = subTitle
    content.sound = .default
    return content
}
```

![image](https://user-images.githubusercontent.com/41438361/142582922-9626038f-d9dc-41a1-becc-5603d6fd294f.png)

이 content에 굉장히 많은 디테일한 내용들을 설정할 수 있다.

굳이 따로 함수로 만들 필요는 없는데, 그냥 만들어봤다.

### 2. 알림을 띄울 조건 설정하기

```swift
private func createNotificationTrigger(day: Int, hour: Int) -> UNCalendarNotificationTrigger {
    var dateComponents = DateComponents()
    dateComponents.calendar = Calendar.current

    dateComponents.weekday = day // 3일 경우 목요일
    dateComponents.hour = hour // 14일 경우 오후 2시(14시)

    return UNCalendarNotificationTrigger(dateMatching: dateComponents, repeats: true)
}
```

`UNCalendarNotificationTrigger`, `UNTimeIntervalNotificationTrigger`이나 `UNLocationNotificationTrigger` 객체를 이용해서 알림을 띄울 조건을 만든다.
이 각 trigger 객체는 다른 파라미터를 가지고 있다.

### 3. 알림 request 생성하고 등록하기

먼저 notification request를 생성한다. 앞서 만든 content와 trigger를 주입한다.

```swift
private func createNotificationRequest(content: UNMutableNotificationContent, trigger: UNCalendarNotificationTrigger?) -> UNNotificationRequest {
    let uuidString = UUID().uuidString
    return UNNotificationRequest(identifier: uuidString, content: content, trigger: trigger)
}
```

참고로 저 `trigger` 부분에 nil이 들어가면 즉시 알림을 띄우게 된다.

만든 request를 notificationCenter에 등록한다.

```swift
private func scheduleNotification(title: String, subTitle: String, day: Int?, hour: Int?) {
    let content = createNotificationContent(title: title, subTitle: subTitle)
    let trigger = createNotificationTrigger(day: day, hour: hour)

    let request = createNotificationRequest(content: content, trigger: trigger)

    //register
    let notificationCenter = UNUserNotificationCenter.current()
    notificationCenter.add(request) { (error) in
       if let error = error {
           print("Error(\(error)) occured during scheduling notification")
       }
    }
}
```

그리고 `viewDidLoad()` 메서드에 아래의 코드를 추가해줬다.

```swift
scheduleNotification(title: "🎉테스트 타이틀🎉", subTitle: "테스트 내용", day: nil, hour: nil)
```

이렇게 하고 실행해도 알림이 뜨지 않는다. 그 이유는 앱이 띄워져 있을 때 알림을 띄우려면 `UNUserNotificationCenterDelegate`를 상속해서 메서드를 구현해야 한다.

## 3. UNUserNotificationCenterDelegate 상속하기

알림과 관련된 작업들을 관리하기 위해 `UNUserNotificationCenterDelegate`을 상속하게 한다. 그리고 항상 **이 프로토콜을 상속한 객체의 `delegate` 프로퍼티를 이 delegate와 상호작용할 만한 작업이 실행되기 전에 할당해야 한다.**

크게 이 delegate는 

1. 사용자가 선택한 내용 다루기
  ![image](https://user-images.githubusercontent.com/41438361/142587072-c7db79c6-3647-417e-bab1-98d34d2cf64a.png)
2. 앱이 foreground에 있을 때 알림 띄우기

를 할 수 있다.

일단 앱이 띄워져 있을 때 알림을 띄우는 작업부터 해결을 해보자.

### 1. 앱이 foreground일 때 알림 띄우기

위에서 우리가 작업한 내용은 알림의 content, trigger를 생성하고, request를 만들고 이를 등록까지 했다. 하지만 앱이 foreground에 있었기 때문에 알림이 뜨지 않았다. 

먼저 프로토콜을 상속해준다.

```swift
extension ViewController: UNUserNotificationCenterDelegate {
}
```

delegate를 설정한다. `viewDidLoad()` 메서드 안에서 설정해줬지만, 이렇게 설정하는 것은 좋은 방법이 아니라고 한다. 이 delegate 는 앱이 완전히 가동되기 전에 할당되어야 한다고 한다.

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    // Do any additional setup after loading the view.

    view.backgroundColor = .white

    UNUserNotificationCenter.current().delegate = self

    askPermissionForNotification()
    scheduleNotification(title: "🎉테스트 타이틀🎉", subTitle: "테스트 내용", day: nil, hour: nil)
}
```

이제 프로토콜의 메서드를 구현해준다.

```swift
func userNotificationCenter(_ center: UNUserNotificationCenter, willPresent notification: UNNotification, withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
    completionHandler([.alert, .sound])
}
```

저 `completionHandler` 안에 알림을 띄울 옵션들을 설정해서 넣어준다.

그리고 이제 실행하면

![image](https://user-images.githubusercontent.com/41438361/142589125-4ec20e0e-0bab-426a-b1e4-7712316b3bad.png)

앱이 foreground에 있는 상태에서도 알림이 잘 뜬다! 다만 앱이 꺼져도 알림이 남아있는 것을 원했는데, 앱을 끄면 바로 알림도 꺼지게 된다.



* 참조
* https://developer.apple.com/documentation/usernotifications
* https://stackoverflow.com/questions/41884922/ios-10-local-notifications-not-showing
