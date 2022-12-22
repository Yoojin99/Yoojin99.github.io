---  
layout: post  
title: "[iOS] - What's new in TextKit and text views"  
subtitle: ""  
categories: app
tags: app-ios wwdc
comments: true  
header-img: 

---  

# What's new in TextKit and text views

[WWDC 영상](https://developer.apple.com/videos/play/wwdc2022/10090?time=30)

이전 [WWDC21 TextKit 2 영상](https://developer.apple.com/videos/play/wwdc2021/10061/?time=2433)을 미리 보는 것이 이해에 도움이 된다.

이전 WWDC 영상에서 언급된 TextKit 2의 여러 특징들을 다시 짚어보면 다음과 같다.

* TextKit 2의 viewport-based layout 아키텍처 (noncontiguous layout을 사용하여 viewport에 보이는 부분만 layout 하는 아키텍처)를 통해 대용량 문서를 layout 할 때 좋은 성능을 유지할 수 있다.
* TextKit 2는 glyph를 사용하지 않음으로서 다양한 언어에 대해 더 정확하게 텍스트를 나타낼 수 있었고, OpenType이나 Variable Font와 같은 현대 font 기술들도 지원할 수 있게 됐다.
* 또한 value semantics를 사용한 높은 레벨의 객체를 사용해서 작업하기 때문에 더 적은 코드로 텍스트의 layout을 커스터마이징 할 수 있다.

![image](https://user-images.githubusercontent.com/41438361/208803619-931378de-4fc3-41fa-a75d-7e667ab5fcc2.png)

여기서 더욱 나아가서 TextKit 2 엔진은 모든 Apple 플랫폼의 text layout과 rendering의 기본이 된다. 그리고 앞으로의 성능 향상 업데이트 등의 업데이트는 TextKit 2 엔진에 초점을 맞출 것이라고 한다.

![image](https://user-images.githubusercontent.com/41438361/208805570-c2ac9a8e-4d5a-4393-ac82-0c3302902ccb.png)

이제 iOS 16과 macOS Ventura에서 UIKIt과 AppKit의 모든 text control은 TextKit 2를 사용한다. 

대부분의 앱에서 TextKit 2로 전환할 때 아무런 추가 작업이 필요가 없다고 한다.

## What's new in TextKit 2

![image](https://user-images.githubusercontent.com/41438361/208805969-92c4375e-7ddc-4d0a-9227-a670b31a92aa.png)

TextKit 2는 iOS 15에서 처음으로 UIKit에서 등장했고, iOS 16에서는 완전히 TextKit 2를 사용하게끔 되어있다. 기본적으로 TextKit 2를 사용하게 되어 있는데, textView를 사용할 때 TextKit 2를 바로 사용할 수 없는 몇 가지 케이스는 영상의 뒷부분에서 다룬다.

macOS Ventura에서 모든 text control은 기본적으로 TextKit2를 사용한다. 대부분의 NSTextView도 TextKit 2를 기본적으로 사용하게 되어있다.

### TextEdit

![image](https://user-images.githubusercontent.com/41438361/208807023-cf15d927-21b1-4ff3-97e8-b3c39038f9f0.png)

NSTextView의 wrapper인 TextEdit도 원래 macOS Big Sur에서는 plain text mode일 때만 TextKit2를 사용했는데, macOS Ventura에서는 rich text mode일때도 TextKit 2를 사용한다.

### Convenience initializer

![image](https://user-images.githubusercontent.com/41438361/208807258-f8f7f923-53ed-4d19-880f-b9c9c240090d.png)

TextKit 2가 새로운 표준이기 때문에, UITextView와 NSTextView를 위한 convenience 이니셜라이저가 추가되었다. 만약 TextKit 2를 사용하는 text view를 만드려면 `usingTextLayoutManager` 인자에 true를 전달하고, TextKit 1을 사용해야 한다면 false를 전달하면 된다.

### Interface Builder option

![image](https://user-images.githubusercontent.com/41438361/208811196-2321aece-eeb8-4ee5-a96b-929b5d7b02e6.png)

Interface Builder에서 TextView의 Text Layout을 위한 옵션도 있다. 이 새로운 옵션을 써서 TextKit 1을 사용할 건지, TextKit 2를 사용할 건지 고를 수 있다.

### exclusionPaths

![image](https://user-images.githubusercontent.com/41438361/208811382-cef84878-28b1-4759-9254-c6119e953c96.png)

TextKit 2는 복잡한 text container도 지원한다. 복잡한 text container는 내부에 구멍이나 여백이 있을 수 있다. 이는 텍스트가 특정 이미지나 내부 컨텐츠를 감싸는 형태로 존재할 수 있게 한다. 이런 복잡한 text container를 만드려면 NSTextContainer의 exclusionPaths 프로퍼티를 사용해서 텍스트가 위치하면 안되는 곳을 정의해야 한다. 

### Line Breaking

![image](https://user-images.githubusercontent.com/41438361/208812878-dfb5f11a-81a2-4f19-8f59-c3033d465522.png)

또한 TextKit 2에서 줄 바꿈 엔진을 개선했다. TextKit 2에서는 별도의 작업 없이 왼쪽의 형태가 오른쪽과 같이 개선된 것을 확인할 수 있다.

### Text lists

![image](https://user-images.githubusercontent.com/41438361/208813191-320e5a78-6f71-4474-87d0-250def4a2e86.png)

모든 플랫폼에서 TextKit 2에서 text list가 지원된다. Text list로 text view에 출력할 숫자형 / 목록형 리스트를 만들 수 있다. NSTextList는 원래 AppKit에서만 사용할 수 있었지만,  iOS 16에서는 UIKit에서도 사용 가능하다.

![image](https://user-images.githubusercontent.com/41438361/208813542-98d2b30e-cf3f-403a-bf09-c040331e209f.png)

Text storage 안에 어떤 paragraph가 출력될 때 list형으로 출력되어야 한다면 이를 나타내기 위해 `NSTextList`와 `NSMutableParagraphStyle`을 함께 사용하면 된다. TextView는 text storage에서 이런 attribute를 가져와서 목록처럼 보이게 paragraph 컨텐츠를 재구성 할 수 있다.

![image](https://user-images.githubusercontent.com/41438361/208823343-48a42c21-0a53-4bb5-ad99-902e4da05415.png)

List가 중첩된 아이템들을 가질 수 있기 때문에, 트리형으로 표현하는 것이 자연스럽다. TextKit 2에서는 이런 형태를 트리 형태로 구성해서 자식과 부모 요소에 접근할 수 있게 했다.

![image](https://user-images.githubusercontent.com/41438361/208823690-e9c8bb6f-9043-439f-9579-8e70f503e322.png)

`NSTextListElement`라는 새로운 element 자식 클래스가 추가됐다. Content manager가 text 컨텐츠에서 `NSTextList`를 보게 되면, 이를 리스트 안에 있는 아이템으로 표현하기 위해 `NSTextListElement`들을 생성하게 된다.

### Attachment

![image](https://user-images.githubusercontent.com/41438361/208836262-8baca6a1-c7a7-43bb-afc1-9a1106742cf1.png)

Text attachment API는 UI나 NSView를 text attachment로 사용할 수 있게 해주고, 이 view들이 이벤트를 직접 제어할 수 있게 해준다. 이는 text attachment로 이벤트를 다루는 것을 더 간단하게 해주는데 TextKit 2에서만 가능한 부분이라고 한다. 

## Compatibility mode

*들어가기 앞서 compatibility mode 부분에 나오는 내용들은 이전 WWDC 21 Meet TextKit 2에서 나온 내용과 동일하므로 이전 영상을 본 사람들은 넘어가도 된다.*

![image](https://user-images.githubusercontent.com/41438361/208837436-7de25c55-1301-4e11-a701-1a0573509826.png)

TextKit1에 강하게 의존하는 것도 TextKit2로 잘 작동할 수 있게 `UITextView`와 `NSTextView`를 위한 TextKit1 compatibility mode가 추가되었다. (이전 wwdc영상에서도 언급됐다.) 

명시적으로 `NSLayoutManager` API를 호출하면 text view는 `NSTextLayoutManager`를 `NSLayoutManager`로 바꾸고 TextKit1을 사용하게 된다. TextView가 TextKit2에 의해 지원되지 않는 attribute를 사용하게 될 때도 이런 fallback이 나타난다.

### Check fallback

![image](https://user-images.githubusercontent.com/41438361/208837911-2a5624ef-c091-418e-a104-225edb7df65a.png)

만약 의도하지 않았는데 TextKit 1으로 런타임 상에서 fallback이 일어난 경우 이런 변경된 내용에 대한 로그를 확인할 수 있다. `_UITextViewEnablingCompatibilityMode` 심볼에 breakpoint를 걸고 stack trace를 확인해서 디버깅 할 수 있다.

![image](https://user-images.githubusercontent.com/41438361/208838277-c7c3f5a9-cd08-49ba-8939-63c5b81be2f8.png)

NSTextView의 경우 `willSwitchTo~`나 `didSwitchTo~` notification을 구독해서 런타임 fallback을 탐지할 수 있다.

### Opt out strategies

1. TextKit1 layout manager 사용

![image](https://user-images.githubusercontent.com/41438361/208838429-30349cb6-93c4-48a6-8b88-75c2ac50d327.png)

만약 TextKit 1으로 fallback해야 하는 경우 생성 시점에 코드로 설정하는 것이 제일 좋다. Text container와 TextKit 1 layout manager를 사용해서 TextKit 1을 사용할 수 있다.

2. Convenience 생성자 사용

![image](https://user-images.githubusercontent.com/41438361/208838681-7b75781d-bb7f-4fc7-812b-e07af72007af.png)

다른 방법은 위의 convenience 생성자를 사용해서 `usingTextLayoutManager`에 false를 전달하면 TextKit 1을 사용하게 된다.

3. Interface Builder

![image](https://user-images.githubusercontent.com/41438361/208839282-2267f722-2243-48ed-b6d0-569b4251ba02.png)

Interface Builder의 `Text Layout` 옵션을 TextKit1으로 설정할 수 있다.

![image](https://user-images.githubusercontent.com/41438361/208839416-7ef9f6d0-47be-4fd4-8003-3deb28398213.png)

만약 생성 이후나 도중 text container의 layout manager를 swap out 시키면 textView는 TextKit 1을 사용하도록 fall back될 것이다. 이후에 버려질 TextKit 2 객체들을 초기화 때 생성하는 것은 비효율적이다.

또한 타이밍에 따라서 사용자가 타이핑하고 있는 도중에 fallback이 일어나게 되면 text view는 focus를 잃고 사용자가 다시 입력하기 위해 text view를 선택해야 하는 상황이 발생할 수도 있다.

따라서 위와 같은 문제가 발생하지 않도록 아예 생성 시점에 textView가 TextKit1을 사용하도록 해야 한다.

## Modernization strategies

![image](https://user-images.githubusercontent.com/41438361/209015720-07265335-9f3a-4cca-b6d9-43faf180e831.png)

먼저 **text view 하나에는 오로지 하나의 layout manager만 존재할 수 있다.** Text view는 `NSTextLayoutManager`와 `NSLayoutManager`를 함께 가질 수 없다.

![image](https://user-images.githubusercontent.com/41438361/209016164-ec5be3fd-4ef3-4a14-b75e-057a9c84dc16.png)

**Text view가 한 번 TextKit1으로 전환되면 다시 TextKIt2로 돌아갈 수 없다. Layout 시스템을 바꾸는데 많은 작업이 필요하고, 전환이 일어나게 되면 그 때 가지고 있던 모든 UI 상태를 잃게 된다.**(Text view의 경우 포커싱이 되어 있는데 전환이 일어났을 때 포커싱을 잃을 것이다.) 그리고 성능과 사용성을 위해 시스템은 TextKit1에서 TextKit2로 전환하지 않는다. 

![image](https://user-images.githubusercontent.com/41438361/209016670-894aa4f6-4fff-40f7-9ec3-d0cdb37bdb9b.png)

그래서 **compatibility mode를 가능한 한 사용하지 않는 것이 중요**하다. Compatibility mode로 진입하게 되는 가장 큰 이유는 text view의 `layoutManager` 프로퍼티에 접근하는 것이다. 

![image](https://user-images.githubusercontent.com/41438361/209016879-9a470fb1-8d0f-4e70-bdf8-a91e25d108c9.png)

또한 textView의 `textContainer`를 통해서도 `layoutManager`에 접근하지 않도록 한다. 이렇게 사용하는 코드들에 대해 같은 기능을 하면서도 TextKit 2를 사용할 수 있는 코드로 바꿔야 한다.

![image](https://user-images.githubusercontent.com/41438361/209017085-60fe1061-84c9-40c8-bc6e-c73e792402e0.png)

만약 TextKit 2를 사용하지 않는 오래된 OS 버전에 배포한다면 `layoutManager` 코드를 완전히 지우기 쉽지 않을 것이다. 이 경우 위와 같이 **textView의 `NSTextLayoutManager`가 있는지 먼저 확인하면 TextKit 2를 쓸 수 있는 곳에서는 TextKit 2를 쓸 것이고, 그렇지 않은 경우에는 `layoutManager` 쿼리를 실행하기 때문에 TextKit 1으로 fallback 할 것이다.**

### 1. Updating NSLayoutManager code

`NSLayoutManager`에 접근하는 코드를 확인했다면 `NSTextLayoutManager`를 사용하는 API들을 TextKit 2 버전으로 바꿔야 한다.

![image](https://user-images.githubusercontent.com/41438361/209018217-71cc87ec-c65a-4fb8-a54c-011bf30ba581.png)

위처럼 TextKit 1과 2버전에서 API들은 비슷한 이름을 갖고 있다. 

하지만 TextKit 1 의 어떤 API들은 TextKit 2로 직접 변환할 수 없다. 그 이유는 TextKit 1에서는 glyph를 사용한 glyph 기반의 API를 사용하는데, glyph들을 기반으로 텍스트의 범위를 알기 어려운 언어들이 있기 때문이다.

![image](https://user-images.githubusercontent.com/41438361/209019118-69057d49-f959-42ef-ba74-41ec9061a02e.png)

위와 같이 glyph는 분리되거나 재정렬 되거나 다양한 작업을 통해 다르게 쓰여질 수 있고, 따라서 glyph를 기반으로 glyph range를 찾기는 어렵다. **TextKit 2에서는 glyph API 가 전혀 없다.** TextKit 1에서 사용하던 glyph 기반의 API들을 다른 방법을 통해 사용해야 한다.

#### Updating glyph-based code

![image](https://user-images.githubusercontent.com/41438361/209019460-cc618cd0-3afb-40da-9510-b75764d3a893.png)

Glyph 기반의 코드를 업데이트 하는 방법은 다음과 같다.

1. 어떤 glyph API를 사용하고 있는지 확인한다.

![image](https://user-images.githubusercontent.com/41438361/209019964-22b5ea60-9134-43dc-940d-553c55f4ea3a.png)

2. 이 API들을 어떻게 사용하고 있는지 보고 높은 레벨에서 무엇을 하려는지 정의한다. Glyph 기반의 코드는 굉장히 낮은 레벨이기 때문에 높은 레벨의 관점으로 봤을 때 작업과 관련 없는 많은 디테일이 있을 것이다.

![image](https://user-images.githubusercontent.com/41438361/209020085-a89955b0-679a-4434-a0ff-0bc4234320e0.png)

위의 예시 코드에서는 TextKit 1을 사용해서 문서 내의 모든 glyph를 iterate한 후 line fragment rect를 세고 있다. 높은 레벨의 작업은 text view가 감싸고 있는 줄 수를 세는 것이 될 것이다.

3. 높은 레벨 작업을 정의했다면 layout fragment, line fragment, text selection 과 같이 TextKit 2에서 사용할 수 있는 객체를 찾는다.

![image](https://user-images.githubusercontent.com/41438361/209020368-5dcfa9f6-de27-4e23-967d-ccd293ec543d.png)

위의 예시 코드를 봤을 때 line fragment rect를 가지고 무언가 하고 있기 때문에 TextKit 2를 사용한다면 `NSTextLineFragment`와 `NSTextLayoutFragment`를 사용할 수 있을 것이다.

![image](https://user-images.githubusercontent.com/41438361/209020446-b80e7d27-a949-4e71-95ae-a0009ce356fd.png)

그리고 앞선 3단계를 거치고 나서 TextKit 2를 사용하게끔 바꾼 코드는 위와 같다. Glyph를 iterating하는 것 대신에 문서 내의 text layout fragment를 enumerating하고 각 layout fragment 내의 text line fragment를 모두 세는 클로저를 제공했다.

### 2. Updating NSRange code

#### NSRange

![image](https://user-images.githubusercontent.com/41438361/209022002-0ea157a0-63de-4cbb-b7df-ad5ea6914109.png)

TextKit 1은 `NSRange`를 사용해서 텍스트 컨텐츠에 인덱스를 붙인다. `NSRange`는 문자의 linear index다. 위의 예시를 보면 "TextKit 2!"에 대한 `NSRange`는 location 6과 length 10을 갖고 있는데, 6번째 문자에서 시작해서 10 글자이기 때문이다. 이런 linear model은 이해하기 쉽고, 문자열에 인덱스를 붙이기 좋다.

![image](https://user-images.githubusercontent.com/41438361/209023109-e0f450fd-5767-40ee-bd44-bcfeaeaaf598.png)

하지만 linear model은 단순 문자열이 아닌 구조에 인덱스를 붙이기에는 적합하지 않다. 위의 예시를 보면 HTML 문서는 트리 구조로 표현되어 있다. `NSRange`로는 "TextKit 2!"의 위치가 span tag안에 3 단계 하위에 중첩되어 있는 위치에 있다고 표현할 수 있는 방법이 없다. 

#### NSTextRange

![image](https://user-images.githubusercontent.com/41438361/209025133-20c0545e-dd74-4c2a-9613-6c10f09e08f5.png)

**HTML 문서와 같이 중첩된 구조를 가지고 있는 곳에서는 `NSRange`로 인덱스를 표현하기가 어렵기 때문에 TextKit 2에서는 텍스트 컨텐츠의 범위를 표현하기 위한 새로운 타입을 추가했다.**

* `NSTextLocation` : Text 컨텐츠 내의 한 위치를 표현하는 **객체**
* `NSTextRange` : 시작, 끝 지점으로 구성되어 있고, 맨 마지막 위치는 범위에 포함시키지 않는다. 

![image](https://user-images.githubusercontent.com/41438361/209025734-dae6af9d-2108-4f71-b7a2-cbcd63941304.png)

위의 새로운 타입을 사용해서 중첩된 구조를 가진 문서의 DOM node와 문자 offset으로서 위치를 표현할 수 있다.

`NSTextLocation`이 프로토콜이기 때문에, 프로토콜을 채택하는 커스텀 객체를 만들 수 있다. 이는 모델에서 구조화된 데이터를 가진 다양한 타입의 backing store를 지원할 때 중요하다.

![image](https://user-images.githubusercontent.com/41438361/209026035-c6ac3547-04f4-406e-9738-e7b7799f740c.png)

하지만 **`NSAttributedString` backing store를 기반으로 만들어진 text view는 이런 구조를 가지고 있지 않다. 그래서 `selectedRange`나 `scrollRangeToVisible`과 같은 textView API를 사용하면 `NSRange`를 써야 한다. TextKit 2 layout manager나 content manager를 사용할 때 `NSRange`와 `NSTextRange`를 서로 변환하는 작업을 해야 한다.**

#### Converting

**NSRange ➡️ NSTextRange**

![image](https://user-images.githubusercontent.com/41438361/209027764-17b56455-87f2-4bdb-962f-4d61e599999c.png)

Text view의 `NSRange`를 `NSTextRange`로 변경하려면 `location`을 시작 location, `length`를 `NSRange`의 `location`과 `length`를 더해서 변환한다.

![image](https://user-images.githubusercontent.com/41438361/209028069-d267fdb1-1b53-43c4-a9c3-274530e217e6.png)

실제로 코드를 작성하면 `NSTextLocation`은 객체여야 하기 때문에 위와 같이 될 것이다.

**NSTextRange ➡️ NSRange**

![image](https://user-images.githubusercontent.com/41438361/209028288-745a578b-40a2-4201-b44b-5dd607e830d0.png)

반대로 변환하기 위해서는 `textContentManager`를 사용해서 두 개의 offset을 받는다.

**UITextRange ↔️ NSTextRange**

![image](https://user-images.githubusercontent.com/41438361/209028563-7e6b9322-54f0-4c39-a680-f0f7116c9f88.png)

UITextView와 UITextField는 UITextInput 프로토콜을 채택하고 있는데, UITextInput 프로토콜은 `UITextPosition`과 `UITextRange`를 사용한다. 대부분의 경우 UITextView나 UITextField를 사용할 때 `UITextRange`를 `NSTextRange`로 직접 변환하지는 않지만 만약 하게 된다면 정수 offset을 사용해서 변환한다.

![image](https://user-images.githubusercontent.com/41438361/209028838-184cf090-1c95-4458-89de-ad2f79a91b8e.png)

만약 UITextInput을 사용하는 커스텀 view가 있다면 뷰 안의 `UITextPosition`, `UITextRange` 하위 클래스를 직접 통제할 수 있을 것이다. 이때 `UITextPosition` 하위 클래스가 `NSTextLocation`을 채택하게 해서 `NSTextRange`를 바로 생성할 수 있게 할 수 있다.

![image](https://user-images.githubusercontent.com/41438361/209029093-f900d9c6-098c-44f2-9d10-79f3774e7b26.png)

참고로 다양한 뷰에서 `UITextPosition` 객체를 재사용하는 것을 피해야 한다고 한다. 내부의 컨텐츠가 비슷해도, `UITextPosition`은 그것을 생성한 뷰에서만 정상적으로 동작하게 되어 있다.
