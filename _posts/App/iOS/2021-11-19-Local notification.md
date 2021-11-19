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


* 참조
* https://developer.apple.com/documentation/usernotifications
