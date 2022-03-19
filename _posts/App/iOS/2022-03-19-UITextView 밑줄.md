---  
layout: post  
title: "[iOS] - UITextView에 Underline추가, line height 임의로 지정하기"  
subtitle: ""  
categories: app
tags: app-ios uitextview
comments: true  
header-img: 

---  
  
> UITextView에 밑줄을 넣고 줄 높이를 바꿔보자. 

---

참 UITextView가 사람을 화나게 만든다. 삽질하는 과정을 자세하게 기록해보려고 한다. 내가 만드려는 UITextView는 아래와 같다.

<img width="855" alt="image" src="https://user-images.githubusercontent.com/41438361/159124393-52567ae7-ddbf-472c-a33d-0f00a14921c9.png">

UITextView가 있고, 모든 줄마다 underline이 있다. 텍스트가 없는 부분에도 밑줄이 쳐져 있어서 메모지 같은 모양이 된다. 그리고 textview의 크기가 제한되어 있어서 줄 수도 미리 정해져 있다.

일단 먼저 UITextView를 하나 만들어보자.

```swift
class ViewController: UIViewController {
    
    private var textView: UITextView!

    override func viewDidLoad() {
        super.viewDidLoad()
        
        view.backgroundColor = UIColor.systemBlue
        
        setupTextView()
        setLayoutConstraintsOfTextView()
    }

    private func setupTextView() {
        textView = UITextView()
        textView.translatesAutoresizingMaskIntoConstraints = false
        
        view.addSubview(textView)
    }
    
    private func setLayoutConstraintsOfTextView() {
        textView.widthAnchor.constraint(equalToConstant: 300).isActive = true
        textView.heightAnchor.constraint(equalToConstant: 220).isActive = true
        textView.centerXAnchor.constraint(equalTo: view.centerXAnchor).isActive = true
        textView.centerYAnchor.constraint(equalTo: view.centerYAnchor).isActive = true
    }
}
```

단순한 코드로 아래와 같은 UITextView 하나를 만들었다. 

![image](https://user-images.githubusercontent.com/41438361/159124698-43a4e5b5-9043-4345-a839-99102591ac34.png)

입력도 잘되는데 글씨 크기가 너무 작다.

![Simulator Screen Recording - iPhone 12 Pro Max - 2022-03-19 at 23 20 36](https://user-images.githubusercontent.com/41438361/159124883-ddfa66b1-0628-431c-a41e-c36f25d0191f.gif)

아래와 같이 UITextView의 font를 크기 30짜리로 설정해 대충 입력했을 때 5줄 정도면 UITextView를 채울 수 있게 했다.

```swift
textView.font = UIFont.systemFont(ofSize: 30)
```

![image](https://user-images.githubusercontent.com/41438361/159124978-16877034-29af-4dcb-8d9d-1714a2b3998b.png)

그리고 임의로 테스트를 해봤는데, 각 줄에 해당하는 부분을 터치하면 해당 줄의 맨 앞, 혹은 맨 뒤로 커서가 이동한다. 그런데 예를 들어 현재 커서가 첫 번째 줄에 있고, 꾹 누른 상태에서 드래그를 하면 내
손가락이 두 번째 줄로 이동해도 두 번째 줄까지 선택이 되지 않는다. 정확하게는 손가락이 두 번째 줄보다 더 많이 이동해야 커서가 더 아래로 이동해서 대략 두 번째 줄까지 선택이 된다. 왜 드래그할 때
커서의 위치를 손가락의 위치대로 이동시키지 않는지는 갤럭시에 익숙한 사람으로써 이해하기 어렵지만 아마 아주 작은 글씨의 TextView에서 몇 줄을 정확하게 드래그하기에는 손가락이 미세하게 움직이는 걸 조정하기 힘들어서 이렇게 만든게 아닌가 싶다... 

그리고 또 이렇게 위 아래 줄에 위치한 커서가 겹치게 된다. 이게 기본 상태인 것으로 보인다.

<img width="94" alt="image" src="https://user-images.githubusercontent.com/41438361/159125240-636a76dd-8a44-4c09-bddc-6170c2a81b7b.png">

커서가 line height보다 큰 것 같아서 궁금해서 커서의 높이를 출력해봤다. UITextView에는 특정 위치에서 커서를 그리는 `caretRect(for:)` 라는 함수가 있다. 이를 아래와 같이 오버라이딩해서
커서의 높이가 얼마나 나오는 지 확인해봤다.

```swift
override func caretRect(for position: UITextPosition) -> CGRect {
    print(super.caretRect(for: position))
    return super.caretRect(for: position)
}
```

커서의 높이는 37.6... 정도가 나왔고, 추가로 `textView.font.lineHeight`를 출력하니 35.8... 가 나왔다. 커서의 높이는 원래 줄의 높이보다 크게 잡히는 것 같다. 

## UITextView underline

먼저 UITextView는 최대 5줄이 있다고 해보자. UITextView의 줄 수를 아래와 같이 제한했다.

```swift
textView.textContainer.maximumNumberOfLines = 5
```

![Simulator Screen Recording - iPhone 12 Pro Max - 2022-03-19 at 23 48 01](https://user-images.githubusercontent.com/41438361/159125805-9dbef434-f332-48e8-91d8-6e5b954a5299.gif)

이러면 다섯번째 줄에서 엔터를 눌러도 여섯 번째 줄을 만들 수 없다. 그런데 문제는 5번째 줄에서 계속 입력해서 UITextView를 벗어나게 되면 커서가 화면에서 보이지 않아서 포커스가 해제된 것처럼 보이는데,
delete 버튼을 계속해서 누르면 다시 커서가 UITextView에서 보이는 것이다. 엔터를 해도 마찬가지였는데, 6번째 줄이 보이지는 않지만 엔터를 치고 지웠을 때도 똑같이 커서가 안보였다가 다시 보였다.

![Simulator Screen Recording - iPhone 12 Pro Max - 2022-03-19 at 23 52 16](https://user-images.githubusercontent.com/41438361/159125995-7c0f2d29-0125-4947-aa04-884c88bb7df7.gif)

위 스샷에서 다섯 번째 줄에서 `ㅇ`을 계속해서 입력하면 커서가 화면에서 보이지 않는데, 이 상태에서 백스페이스를 계속 누르니 다시 글자가 지워지면서 커서가 보이는 걸 확인할 수 있다. 왜 이렇게 구현이 된 
걸까 이해는 잘 안되지만 다 이유가 있을 것이다. 그래서 5줄을 만드려는데 글자수를 제한한다던가 줄 수를 제한하려면 UITextViewDelegate를 사용해야 한다.

`textView`의 delegate를 UIViewController로 설정해주고,

```swift
textView.delegate = self
```

UIViewController의 extension을 사용해 UITextViewDelegate를 따르게 한다.

```swift
extension ViewController: UITextViewDelegate {
    
}
```

줄 수/글자 수를 제한하기 위해 사용할 UITextViewDelegate의 함수는 `textView(_:shouldChangeTextIn:replacementText:)` 함수다. text를 바꿀지 말지를 결정하는 함수로, 아래와 같이 구현해준다.
일단은 글자 수만 제한해줬다.

```swift
func textView(_ textView: UITextView, shouldChangeTextIn range: NSRange, replacementText text: String) -> Bool {
    let newText = (textView.text as NSString).replacingCharacters(in: range, with: text)
    let numberOfChars = newText.count
    return numberOfChars <= 50
}
```

이러면 개행 문자를 포함해서 총 50자까지만 입력할 수 있다.

![image](https://user-images.githubusercontent.com/41438361/159128680-3303b85e-881a-47ed-ad8e-b737b70a5f39.png)

이제 밑줄을 추가해보자. 밑줄은 `draw` 메서드를 오버라이딩해서 만들 수 있다.

```swift
override func draw(_ rect: CGRect) {
    let context = UIGraphicsGetCurrentContext()
    context?.setStrokeColor(UIColor.black.cgColor)

    let topInset = textContainerInset.top
    let lineHeight = (rect.height) / CGFloat(textContainer.maximumNumberOfLines)

    for i in 1...textContainer.maximumNumberOfLines {
        let y = topInset + CGFloat(i) * lineHeight

        let line = CGMutablePath()
        line.move(to: CGPoint(x: 0, y: y))
        line.addLine(to: CGPoint(x: rect.width, y: y))
        context?.addPath(line)
    }

    context?.strokePath()

    super.draw(rect)
}
```

rect의 height를 앞에서 설정한 maximumNumberOfLines로 나눠줘서 밑줄을 그을 간격을 구했다.

![image](https://user-images.githubusercontent.com/41438361/159128817-1dea1fcd-805c-4db6-90bf-a5d8d680bde5.png)

근데 밑줄이 4개밖에 안나온다! 이는 UITextView의 inset때문이다. UITextView의 textContainer inset은 기본적으로 위 아래 8씩 있다. 따라서 그릴때 y 위치에 topInset을 더해줘서
마지막 줄은 더 아래에 그어져서 화면에 나타나지 않는 것이다. 나는 UITextView에 inset이 있는걸 원하지 않아 아래와 같이 설정했다.

```swift
textView.textContainerInset = UIEdgeInsets(top: 0, left: 0, bottom: 0, right: 0)
```

![image](https://user-images.githubusercontent.com/41438361/159129027-bea598fb-0f72-4df2-932f-7ebd10c27d5b.png)

이러면 밑에 줄이 경계선때문에 잘 안보이긴 한데 마지막 밑줄까지 잘 나온다.

## lineHeight

![image](https://user-images.githubusercontent.com/41438361/159129057-357c1f3c-3943-4a2e-9069-2c8efc71b98e.png)

ㅋㅋ 밑줄간의 간격은 44정도 되는데, lineHeight는 35이니 당연히 위와 같이 나올 수 밖에 없다. 구글링해보니 line 높이를 변경하려면 attributedText와 NSMutableParagraphStyle을 사용해야 한다고 한다.

코드를 아래와 같이 수정했다.

```swift
// textView.font = UIFont.systemFont(ofSize: 30)

let attributes: [NSAttributedString.Key : Any] = [
    NSAttributedString.Key.font : UIFont.systemFont(ofSize: 30)
]

textView.attributedText = NSAttributedString(string: "", attributes: attributes)
```

이러니까 `textView.font?.lineHeight`는 nil이 나오는데 이건 string을 ""로 설정해서 그런거고, 다른 string을 넣으면 35.8..로 그냥 `textView.font = 어쩌구`로 설정했을 때와 똑같이 나온다.



sdf



sdfsdfsd




sdfsdf

sdfsdfs
