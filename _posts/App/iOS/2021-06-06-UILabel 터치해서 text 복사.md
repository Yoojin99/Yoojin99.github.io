---  
layout: post  
title: "[iOS] - UILabel 텍스트 터치해서 복사하기"  
subtitle: ""  
categories: app
tags: app-ios 
comments: true  
header-img: img/dev/app/ios/XcodeImg.png

---  
  
> `UILabel의 text를 터치해서 복사하는 법을 알아보자.`  

---

UILabel을 터치했을 때 텍스트가 클립보드에 복사되는 것을 구현했다. 나는 UILabel의 extension으로 구현했다.

```swift
extension UILabel {
    func enableCopyOnTouch() {
        let tapGesture = UITapGestureRecognizer(target: self, action: #selector(labelTapped(sender:))) // 터치 시 실행될 함수를 연결한다.

        self.isUserInteractionEnabled = true
        self.addGestureRecognizer(tapGesture) // tapGestureRecognizer를 라벨에 붙인다.
    }
    
    @objc
    private func labelTapped(sender: UITapGestureRecognizer) { // 라벨이 터치되었을 때 호출됨
        guard let label = sender.view as? UILabel else {
            return
        }
        UIPasteboard.general.string = label.text // 텍스트가 복사됨
    }
}
```

아래와 같이 UILabel을 설정하면 라벨을 터치했을 때 해당 라벨의 텍스트가 클립보드에 복사된다.

```swift
let myLabel = UILabel()
myLabel.enableCopyOnTouch()
```
