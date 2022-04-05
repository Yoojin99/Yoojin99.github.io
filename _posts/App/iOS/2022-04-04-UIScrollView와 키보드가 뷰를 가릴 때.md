---  
layout: post  
title: "[iOS] - 키보드가 특정 뷰를 가릴 때 UIScrollView로 뷰를 위로 올리기"  
subtitle: ""  
categories: app
tags: app-ios keyboard 
comments: true  
header-img: 

---  
  
> 간혹 텍스트 입력을 위해 키보드가 필요한 뷰에서 키보드가 올라와 사용자에게 보여야 할 뷰가 가려질 때가 있다. UIScrollView를 사용해서 이런 현상을 어떻게 해결할 지 보자.  

---

![image](https://user-images.githubusercontent.com/41438361/161485625-eb3a2377-844a-4e1a-8d9f-17b1f3713953.png)

위의 화면과 같이 정중앙에 UITextView를 만들었다. 텍스트 뷰에 뭐를 입력하려고 텍스트 뷰 안을 터치하면 아래와 같이 키보드가 올라온다.

![image](https://user-images.githubusercontent.com/41438361/161485864-1b886327-1070-43f7-882d-4eaaeaf930ca.png)

화면을 보면 키보드의 윗부분이 텍스트 뷰를 가리게 된다. 아래와 같이 키보드가 올라와 있지 않을 때는 텍스트 뷰가 원래의 위치(화면 중앙)에 있고,
키보드가 올라왔을 때는 키보드 위로 텍스트 뷰를 올리고 싶으면 어떻게 해야 할까?

<img width="430" alt="image" src="https://user-images.githubusercontent.com/41438361/161487402-2157fdfe-16f6-4c5a-8152-cad5654f15c8.png">

키보드가 올라왔을 때 뷰의 위치를 조정하는 방법에는 여러 방법이 있을 수 있다. 이런 상황에서는 UIScrollView를 사용하는 것이 가장 간단하다.

심지어 애플은 [Apple Developer Doucment](https://developer.apple.com/library/archive/documentation/StringsTextFonts/Conceptual/TextAndWebiPhoneOS/KeyboardManagement/KeyboardManagement.html)에
이런 상황을 어떻게 처리할 지에 대한 코드도 남겨놓았다. 이런 것에 대한 공식 문서가 있을 정도로 이런 케이스가 많다는 걸 의미할 것이다.

"Moving Content That Is Located Under the Keyboard" 부분을 보면 키보드가 올라왔을 때 앱 컨텐츠의 일부를 가리는 현상과 그것을 해결하는 방법에
대해 나와있다. 키보드가 컨텐츠를 가리게 되면 이 컨텐츠를 키보드 위로 올려야 한다. 키보드와 UITextView와 같은 텍스트를 다루는 객체를 관리하는 가장 간단한
방법은 위에서도 잠깐 언급했지만 UITableView와 같은 UIScrollView 객체를 사용하는 것이 가장 간단하다.

## 키보드 위로 뷰 올리기

그러면 UIScrollView를 사용해서 키보드 위로 컨텐츠를 어떻게 올릴까?

1. 키보드 사이즈를 구한다.
2. UIScrollView의 하단 content inset을 키보드의 높이와 같게 한다.
3. 필요하다면 스크롤 뷰를 스크롤한다.

### 0. 스크롤 뷰 생성

![image](https://user-images.githubusercontent.com/41438361/161490473-5aec0f3e-82bf-4ab8-a8aa-7cad325c8a36.png)

만들어 둔 이 화면에서 시작해보자. 먼저 UIScrollView가 화면을 꽉 채우게 하고, 안에 화면 사이즈만한 컨텐츠 뷰를 만든다음, 컨텐츠 뷰에 텍스트 뷰를 넣을 것이다.

뷰 계층을 보면 UIScrollView 안에 UIView가 있고, UIView에 UITextView가 아래와 같이 있다.

<img width="324" alt="image" src="https://user-images.githubusercontent.com/41438361/161654679-7ff7832b-b788-46de-9c85-2cd635d1a273.png">
<img width="243" alt="image" src="https://user-images.githubusercontent.com/41438361/161654744-7078927d-9fec-4e9c-bf68-20c6818bc497.png">

이러면 UIView는 UIScrollView의 크기와 같기 때문에 스크롤이 활성화되어 있지 않다.

![image](https://user-images.githubusercontent.com/41438361/161654760-5fa2bd66-72b8-4f42-8017-4a7c0ed6321c.png)

키보드를 올렸을 때 아래와 같이 키보드가 텍스트 뷰 밑을 가리고, 스크롤은 여전히 되지 않는다.

![image](https://user-images.githubusercontent.com/41438361/161654828-29976f81-8781-441f-a510-b06b6536f694.png)

### 1. 키보드 사이즈 구하기

먼저 스크롤 뷰를 관리하는 뷰 컨트롤러 등에서 키보드가 화면에 올라오고, 다시 해제되는 것에 대한 notification을 다룰 handler method를 구현한다.

```swift
private func registerForKeyboardNotifications() {
        NotificationCenter.default.addObserver(self, selector: #selector(keyboardWasShown(_:)), name: UIResponder.keyboardDidShowNotification, object: nil)
    }
    
    @objc private func keyboardWasShown(_ notification: Notification) {
        let keyboardFrame = notification.userInfo?[UIResponder.keyboardFrameEndUserInfoKey] as? NSValue
        
        if let keyboardSize = keyboardFrame?.cgRectValue {
            // 여기에서 keyboardSize.height로 키보드 높이에 접근할 수 있다.
        }
    }
```

`registerForKeyboardNotifications()` method에서 키보드가 완전히 올라왔을 때를 알 수 있게 observer를 등록했다.
참고로 키보드는 아래에서 위로 올라오고, 이동안 키보드의 높이가 변하기 때문에 키보드가 완전히 올라왔을 때 높이를 구하려면 `UIKeyboardFrameEndUserInfoKey`를 사용해야 한다.

그리고 `keyboardWasShown(_:)` method에서 전달받은 notification에서 키보드에 대한 정보에 접근해서 `keyboardSize`를 구했다.

### 2. UIScrollView의 contentInset 조정

코드를 보기 전에 contentInset이 뭔지를 간단하게 보자. contentInset은 이름에서부터 짐작할 수 있듯이 컨텐츠의 inset, 즉 내부에 삽입하는
것과 관련된 것이구나 생각할 수 있다.

<img width="515" alt="image" src="https://user-images.githubusercontent.com/41438361/161656497-419b249f-30b0-4b32-a2e4-f81262aeba10.png">

contentInset은 safe area나 scroll view 끝에서 content view가 안에서 얼마나 떨어졌는지에 대한 거리를 의미한다. contentInset은 `UIEdgeInset` 타입으로,
아래와 같이 아래, 왼쪽, 오른쪽, 위쪽에서 얼마나 떨어져 있는지를 나타낸다.

<img width="283" alt="image" src="https://user-images.githubusercontent.com/41438361/161663667-182faffe-a96a-43ff-89c1-ce4c537ea845.png">

참고로 contentInset의 기본값은 `UIEdgeInsetsZero`다.

아무튼 scrollView의 contentInset을 어떻게 조정해야 할까? 우리가 하려는 것은 키보드가 올라왔을 때 키보드 아래에 가려지는 부분을 키보드 위로
**올리는** 것이다.

그러면 scrollView가 있고 내부에 content view가 있다면 이 contentView를 위로 올려야 한다. 이는 scrollView의 contentInset의 bottom에
키보드 높이만큼의 값을 줘서 할 수 있다. 그림으로 나타내면 아래와 같다.

<img width="315" alt="image" src="https://user-images.githubusercontent.com/41438361/161664444-1ea89d22-f010-4d95-82cc-8955b46f03b1.png">

더 정확하게 표현하면 아래와 같이 될 것이다.

<img width="192" alt="image" src="https://user-images.githubusercontent.com/41438361/161664765-a490e43d-0a4c-4295-ae9b-b9addfa87fa4.png">

녹색은 화면을 표현한 것이고, content view는 scrollView의 content view를 표현했다. content view의 길이가 길어서 스크롤 뷰에서
스크롤을 통해 볼 컨텐츠의 위치를 조정한다. 만약 위 화면에서 사용자가 아래로 스크롤하면 아래와 같이 될 것이다.

<img width="176" alt="image" src="https://user-images.githubusercontent.com/41438361/161664910-8a545e77-3f2c-41d0-88c8-b0a3e3299377.png">

화면상에서 하단 부분에는 contentInset이 보일텐데, 실제로 저 부분은 여백이므로 빈 부분으로 보일 것이다. 그렇다면 키보드가 올라왔을 때 어떻게 될까?
우리는 contentInset을 키보드의 높이만큼 줬다. 따라서 키보드가 올라와 있으면 아래와 같이 될 것이다.

<img width="176" alt="image" src="https://user-images.githubusercontent.com/41438361/161665277-c0dc3e6a-bd70-46d2-b6fb-c298e651566f.png">

즉 우리가 줬던 contentInset 만큼의 빈 공간은 키보드가 덮어버리게 되고, contentView는 실제로 키보드 위로 올라간 것처럼 보이게 되는 것이다.

코드 상으로는 아래와 같이 조정한다. 위에서 작성한 `keyboardWasShown(_:)` 메서드 내에 scrollView의 contentInset과 scrollIndicatorInset의 bottom 값을
키보드의 높이만큼 설정해준다. 참고로 scrollIndicatorInset은 스크롤할 때 나타나는 스크롤 바의 위치가 스크롤 뷰 내에서 얼마나 떨어져 있는지를 나타낸다.

```swift
@objc private func keyboardWasShown(_ notification: Notification) {
    let keyboardFrame = notification.userInfo?[UIResponder.keyboardFrameEndUserInfoKey] as? NSValue

    if let keyboardSize = keyboardFrame?.cgRectValue {
        let inset = UIEdgeInsets(top: 0, left: 0, bottom: keyboardSize.height, right: 0)
        scrollView.contentInset = inset
        scrollView.scrollIndicatorInsets = inset
    }
}
```

그리고 앱을 실행시키고, textView 내부를 클릭해보면 아래와 같이 나온다.

![image](https://user-images.githubusercontent.com/41438361/161665599-d528f0a6-e7bb-4690-aee1-44cbea6c0702.png)

텍스트 뷰가 안 올라간게 당연하다. 우리는 content view의 하단 여백이 얼마나 떨어졌을지만을 조정했지 실제로 content view를
얼마만큼 위로 스크롤 할 지 설정하지 않았기 때문이다. 즉 현재 아래와 같은 상황이다.

<img width="183" alt="image" src="https://user-images.githubusercontent.com/41438361/161665866-d27eccb5-84aa-4c59-b839-71588584626a.png">

content view의 밑에 padding을 줬고, content view에 우리가 위로 올리고 싶은 textView가 있다. 그리고 textView를 클릭해서 키보드가 올라오면
textView가 가려지게 된다. 우리가 여기에서 추가로 해줘야 할 것은 scrollView를 실제로 얼마나 스크롤 할 지 추가로 지정해줘서 textView를 올려야 하는 것이다.

추가로, 키보드가 내려갔을 때 contentInset을 다시 원래대로 설정해야 한다. 먼저 키보드가 내려갔을 때 contentInset을 모두 0으로 만들어주는
`keyboardWillBeHidden` method를 작성한다.

```swift
NotificationCenter.default.addObserver(self, selector: #selector(keyboardWillBeHidden(_:)), name: UIResponder.keyboardWillHideNotification, object: nil)
```

## 3. UIScrollView의 contentOffset 조정하기

위에서 얘기한 scrollView를 얼마나 스크롤할지에 대한 것은 contentOffset을 통해 조정할 수 있다. 

contentOffset은 무엇일까?

<img width="520" alt="image" src="https://user-images.githubusercontent.com/41438361/161666331-02d23d7a-cdf8-4768-ae2c-1ba9744ef51d.png">

contentOffset은 content view의 origin이 scroll view의 origin으로부터 얼마나 떨어져 있는지를 나타낸다. 얘도 마찬가지로 기본값은 CGPointZero다.
UIScrollView는 세로로 스크롤 하고 싶을 때 contentOffset.y의 값을 조정하여 컨텐츠의 어떤 부분을 보여줄 지를 결정한다.

다시 본론으로 돌아와서 키보드가 올라왔을 때 UITextView를 키보드 위에서 20만큼 떨어진 곳까지 스크롤 하고 싶으면 어떻게 해야 할까?

```swift

```

* 출처
* https://developer.apple.com/library/archive/documentation/StringsTextFonts/Conceptual/TextAndWebiPhoneOS/KeyboardManagement/KeyboardManagement.html
