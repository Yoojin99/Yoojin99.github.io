---  
layout: post  
title: "[iOS] - WWDC21 Meet TextKit 2"  
subtitle: ""  
categories: app
tags: app-ios wwdc
comments: true  
header-img: 

---  

# Meet TextKit 2

[WWDC 영상](https://developer.apple.com/videos/play/wwdc2021/10061)

TextKit 2는 새롭게 사용되는 Apple의 텍스트 엔진이다.

# TextKit 1

기존에 사용되는 TextKit이었던 **TextKit 1은 모든 Apple의 플랫폼의 텍스트의 레이아웃을 조정하고 화면에 보여주는 텍스트 레이아웃 엔진**이다. UIKit과 AppKit에서는 TextKit을 사용해서 텍스트의 레이아웃과 저장소를 관리했다.

![image](https://user-images.githubusercontent.com/41438361/207508959-991166ba-a2c2-4e7b-9aa7-6f7ad1d9ca44.png)

TextKit 1은 20년 전에 OpenStep이라는 시스템에서 처음 등장했고, macOS 10.0에서 iOS 7부터 macOS 11과 iOS 14까지 오랜 시간 쓰여왔고, 또 발전했다. 하지만 오래된만큼 **오래된 원칙에 기반되어 구현되어 있기 때문에 좋은 성능을 유지하면서 새로운 기술과 결합해 API를 제공하기가 점점 어려워졌고, 결국 TextKit 2가 등장하게 되었다.**

# TextKit 2 

**TextKit 2는 새로운 버전의 텍스트 엔진으로, 미래지향적인 디자인 원칙에 기반**해 구현되었다. 

사실 이미 알든 모르든 TextKit 2를 사용하고 있는 사람들도 있을텐데, Big Sur에서는 많은 OS 컴포넌트들에 TextKit 2를 사용하고 있고 macOS 11부터도 TextKit 2를 사용하고 있다. 

## Architecture

![image](https://user-images.githubusercontent.com/41438361/207510067-1ca89d29-8d7c-4960-8f37-bad7e8ce4c08.png)

TextKit2 는 TextKit1과 공존한다. TextKit2도 `Foundation`, `Quartz`, `Core Text` 를 바탕으로 구현되었다. UIKit과 AppKit의 text control도 TextKit2를 기반으로 동작한다.

**TextKit1의 MVC 디자인**

![image](https://user-images.githubusercontent.com/41438361/207510395-5f45e216-e8de-44c9-99be-0ae2ae892c9a.png)

**TextKit2의 MVC 디자인**

![image](https://user-images.githubusercontent.com/41438361/207510678-219e7798-3cd4-4623-88c6-6c51c2936cec.png)

TextKit2는 TextKit1의 MVC 디자인을 약하게(loose하게) 따르고 있다. View부분은 UIKit과 AppKit 프레임워크의 뷰 객체이고, Model과 Controller쪽에 새로운 버전들이 추가됐다. 이 새로운 디자인은 원하는 것을 위해 **어떻게** 할 지보다 **무엇**을 원하는지를 표현하기 더 쉽게 만든다고 한다.

## Design Principles

![image](https://user-images.githubusercontent.com/41438361/207515220-cb3b9b02-ec6f-4348-8454-2dec6b7854df.png)

TextKit2의 핵심 디자인 원칙은 **1. 정확성, 2. 안정성, 3. 성능**이다. 모두 중요하기 때문에 어떤 것이 다른 것보다 중요하다고 할 수 없다. 이 세 디자인 원칙에 따라 TextKit2에서 디자인적으로 변한 부분들이 있다.

1. **Correctness : *Glyph handling*을 사용하지 않는다. (영상에서는 "abstracts away"라 하는데 이게 일반적인 의미에서 사용하지 않는다는 것인지, cs에서 생각하는 추상화를 시킨다는 말인지는 정확하게 알기 어렵지만 문맥상 사용하지 않게 한다는 게 더 정확한 것 같다. )**
2. **Safety : *Value semantics*에 집중한다.**
3. **Performance : *Viewport* 기반의 레이아웃과 렌더링을 사용한다.**

## 1. Correctness

**정확도를 높이기 위해서 glyph (글리프)를 사용하지 않는다.**

### Glyph

Glyph는 여러 문자의 **시각적인 표현**이다. 시각적인 표현이라는 것은 문자의 모양, 디자인, 표현 방법을 모두 표현한다. 

![image](https://user-images.githubusercontent.com/41438361/207742651-f156181d-5510-4e2d-b66b-419a012b28a0.png)

예를 들어 문자 "a"라는 것이 있을 때 위와 같이 다양한 시각적인 형태로 "a"를 표현할 수 있는데, 위는 "a"를 나타내는 glyph들의 일부를 모아놓은 것이다.

많은 서양 언어는 위의 "a"와 같이 주로 한 개의 glyph가 한 문자를 표현하지만 항상 한 문자에 한 개의 glyph가 사용되는 건 아니다.

![image](https://user-images.githubusercontent.com/41438361/207742989-3bb466d4-08ec-4479-9717-8fee9acf551e.png)
![image](https://user-images.githubusercontent.com/41438361/207743094-5f2ffba1-9570-451a-ab44-74c2e92dae96.png)
![image](https://user-images.githubusercontent.com/41438361/207743317-8fdee99e-5ef0-4cab-8bd1-7564c7a5b3a5.png)

위와 같이 한 문자에 여러 개의 glyph가 사용될 수도 있고, 여러 문자에 한 개의 glyph가 사용될 수도 있다.

### Ligature

![image](https://user-images.githubusercontent.com/41438361/207743389-2f483ef9-9b7d-453c-b908-7cf471c26ca9.png)

그리고 위와 같이 여러 개의 문자를 한 개의 glyph로 표현한 것을 **ligature**라고 한다. 서양 언어에는 이런 ligature들이 많이 없고, 가독성을 해치지도 않는다. 위의 ligature도 f 두 개, i 하나가 붙었음을 바로 알 수 있다.

하지만 **ligature가 많고, 또 해석하는데 영향을 주는 언어들이 많다.**

![image](https://user-images.githubusercontent.com/41438361/207743700-59702568-ecd2-4a5d-8528-780a9914e6b2.png)

위 사진에 있는 단어는 urdu 어로 '순간'이라는 뜻이다. 왼쪽과 같이 개별적인 문자로 작성된 것과 오른쪽과 같이 하나의 ligature로 작성된 것이 굉장히 달라보이는데, 실제 urdu어를 사용하는 사람들은 왼쪽의 개별적인 문자로 표현되어 있는 것을 가독성이 떨어진다(기호를 해석하지 못한다)고 느낀다고 한다. 이렇게 ligature가 영향력이 큰 언어들이 있다.

### Glyph APIs

![image](https://user-images.githubusercontent.com/41438361/207744206-16807866-6851-4cd9-9e1a-fc7793f07e63.png)

위 사진은 TextKit1의 API로, **TextKit 1의 많은 API들이 glyph 인덱스, 범위를 사용하고 있다.** 그래서 위의 예제에서처럼 특정 텍스트의 bound를 알기 위해 glyph 범위를 알아야 했다.

![image](https://user-images.githubusercontent.com/41438361/207744446-a9c584a2-5c3d-4808-9072-7c6280d2c523.png)

영어의 경우 glyph 범위를 아는 건 쉽다. 앞에서도 봤듯이 문자 하나 당 주로 glyph 한 개가 사용되었기 때문에, 위 사진을 봤을 때 "shen"이라는 텍스트의 glyph 범위도 0번째부터 시작해서 4만큼의 길이를 가지고 있을 것이라는 것을 바로 알 수 있다.

![image](https://user-images.githubusercontent.com/41438361/207744667-9567ff1d-f92b-44bb-b268-640bf8b1f97a.png)

하지만 다른 언어의 경우 그렇지 않다. 위 언어는 Kannada어를 쓴 것으로, 이 언어에는 ligature도 많고, glyph들이 재배치되거나 다양한 방법으로 조합될 수도 있다. 위 사진에서 볼 수 있듯이 하나의 문자에 사용된 glyph들이 쪼개지기도 하고, 문자들이 합쳐질 때 사용되는 표현법으로 다른 glyph로 표현되기도 하고 하여간 glyph 범위를 알기가 굉장히 어렵다는 것을 알 수 있다. **이런 언어에서는 glyph 범위를 알 수 없다. 그래서 이런 언어에서 TextKit 1 API를 사용하게 되면 layout이나 rendering이 잘못될 가능성이 있다.**

**그래서 TextKit2는 glyph handling을 사용하지 않는다. TextKit2는 모든 텍스트를 Core Text로 렌더링하기 때문에 복잡한 스크립트를 렌더링 할 때 문제가 없다.**

여기서부터 볼 selection, location, range는 TextKit2에서 glyph대신 사용하는 높은 레벨의 객체다.

### Selection

![image](https://user-images.githubusercontent.com/41438361/207745471-3c874d45-f3f0-4423-b71b-0c496b7384ec.png)

**TextKit2는 glyph를 사용하는 대신 더 높은 레벨의 객체를 사용해서 텍스트 layout과 상호작용을 관리한다.** 

`NSTextSelection`은 이런 높은 레벨 객체 중 하나로, 텍스트를 선택할 때 관련된 내용을 포함하고 있다. 

![image](https://user-images.githubusercontent.com/41438361/207747184-81db797f-2696-4afe-8dd2-9ef84386a08f.png)

`NSTextSelection`의 프로퍼티들은 읽기 전용이라 수정할 수 없다. 대신, `NSTextSelectionNavigation`의 인스턴스를 사용해서 텍스트 선택과 관련된 작업들을 하고, 그 작업들의 결과로 또 새로운 `NSTextSelection` 인스턴스들을 받을 수 있다.

![image](https://user-images.githubusercontent.com/41438361/207747464-6dfd3d06-5b18-47f8-9c16-a1ff0a369c30.png)

텍스트 선택을 할 때 navigation 객체를 사용할 수 있다.

* `textSelections(interactingAt: containerLocation: ...)` : 화면에서 탭이나 클릭해서 생성된 선택된 부분들을 받는다.
* `textSelection(for:enclosing:)` : 특정 지점에서 문법적으로 의미가 있는 부분까지 확대해서 선택한다.

이 방법으로 올바르게 한 단어 별로 텍스트를 선택하고 오른쪽에서 왼쪽으로 정렬되는 언어들에 대해서도 올바르게 동작할 수 있다고 한다. 

### Location and Range

위의 `NSTextSelectionNavigation`의 첫 번째 메서드를 보면 `NSTextLocation` 객체를 인자로 받고 있는 것을 알 수 있다. `NSTextLocation`은 TextKit 2에서 등장한 새로운 객체다.

![image](https://user-images.githubusercontent.com/41438361/207748425-9a5d8ddc-1a4b-45d3-9a75-376997371cc1.png)

얘네들은 UIKit의 `UITextPosition`과 `UITextRange` 클래스와 굉장히 비슷하지만 상속할 수는 없다. 거의 항상 TextKit 2에서는 이 location, range 객체들을 사용하게 된다.

![image](https://user-images.githubusercontent.com/41438361/207748814-d44d00b9-7f2c-42c4-8e49-49d24ccac262.png)

정수 대신에 객체를 사용하는 것은 더 표현적인(expressive) document model에서 효과적인데, 그 이유는 텍스트의 범위가 상대적인 위치에 따라 결정되기 때문이다. 위의 HTML 예시를 보면 중첩된 요소들이 있고, 텍스트의 위치 정보는 문서 내의 정확한 위치와 실제 보여지는 텍스트 내의 위치에 대한 정보를 모두 포함하고 있어야 한다. 이런 정보는 정수 하나로만으로는 표현할 수 없다.

## 2. Safety

### Value semantics

TextKit2는 **value semantics**를 중심으로 디자인되어 Swift와 SwiftUI과 같은 기술들의 목표와 맞출 수 있게 했다.

> Value Semantics?
> Value Semantics는 Swift에서 흔히 얘기하는 값 타입, 참조 타입 할 때 말하는 "값"이다. 값 자체에 집중한다는 뜻이다. 

**값 타입은 데이터가 불변성을 갖고 있기 때문에 예상치 못한 동작이나 의도하지 않은 공유가 되는 것을 막아서 더 안전한 코드를 작성할 수 있다.** 여기에 더불어 immutable 클래스는 초기화 이후에 프로퍼티의 값을 수정할 수 없기 때문에 데이터가 변하는 것을 막을 수 있다. 이런 **immutable 클래스들은 값 타입처럼 동작하기 때문에 "value semantics"를 가지고 있다고 한다.**

만약 이런 객체의 데이터를 바꾸고 싶다면 새로운 인스턴스를 만들어서 원래의 것을 대체해야 한다. TextKit2의 많은 클래스는 이런 방식으로 디자인되어 있고, "value semantics"를 가지고 있다고 할 수 있다. 

### TextKit1의 디자인

TextKit2에서 클래스들이 value semantics를 가지게 된 것의 장점이 무엇인지 알려면 TextKit1이 어떻게 디자인됐는지 이해할 필요가 있다. 

![image](https://user-images.githubusercontent.com/41438361/207780294-80f68d1c-d260-49e9-a84f-b9e0117fb214.png)

위 사진은 TextKit1의 디자인을 그림으로 그린 것으로, **TextStorage를 업데이트하게 되면 glyph 들을 생성하고, 위치시킨다음, 뷰에 직접 그리게 된다. 이런 방법으로 glyph 들을 뷰에 직접그리게 되면 커스텀으로 무언가를 그릴 때 여백을 만들기 위해 텍스트의 어디를 분리해야 하는지 알기가 어렵다.**

"위 방법으로 glyph를 직접 그리는 방법을 사용하게 되면 여백을 만들기 위해 텍스트의 어디를 분리해야 하는지 알기가 어렵다" 라는 말이 무슨 말인지 알기 위해 아래의 예제 앱을 볼 것이다.

![image](https://user-images.githubusercontent.com/41438361/207780811-152cab6e-22f7-457e-8924-e8178ea373d0.png)

위 예시 앱에서는 레시피들을 보여주고 있고, 밑에 보라색으로 표시된 부분과 같이 개별적으로 버블모양의 커맨트를 달 수 있다. 위와 같이 커맨트를 올바른 위치에, 다른 텍스트와 구별되게끔 하기 위해 어떻게 해야 할까?

![image](https://user-images.githubusercontent.com/41438361/207781213-b9eec585-9b87-41f9-bad1-b3e60bcdb3b3.png)

대략적으로 레시피 텍스트를 의미있는 단위나 요소로 나누고, 각 커멘트를 각 요소(레시피)에 붙인 다음, 각 커멘트를 연관된 레시피에 위치시키고, 커멘트를 그리기 위한 어떤 동작을 실행시키면 되지 않을까하고 생각해볼 수 있다.

하지만 TextKit1에서는 실제로는 굉장히 복잡한 과정을 거쳐야 한다. 

![image](https://user-images.githubusercontent.com/41438361/207781686-2dca39a9-0fe4-4da4-ba7b-c246a7922ac7.png)

위에서 볼 수 있는 많은 디테일까지 신경쓰며 개발하고 싶지 않을 것이다. 

![image](https://user-images.githubusercontent.com/41438361/207782170-8e150f34-edca-4a5b-9e5b-9a346b5957d8.png)

TextKit2에서는 앞서 상상했던 것과 같은 방식을 사용해서 구현할 수 있다. 

### TextKit2의 디자인

![image](https://user-images.githubusercontent.com/41438361/207782501-d6361096-d681-45eb-ab5a-03b72acdac81.png)

위 그림은 TextKit 2가 어떤 플로우로 동작하는지를 나타냈다.

1. ContentManager라는 새로운 객체를 통해 text storage를 업데이트 하게 된다. ContentManager는 텍스트들을 요소들로 나눈 다음 이 요소들을 추적한다.
2. 레이아웃할 때 LayoutManager가 ContentManager에 요소들을 요청해서 받는다. LayoutManager는 텍스트 컨테이너 안에 요소들을 배치하고, 레이아웃과 위치 정보를 갖고 있는 LayoutFragment를 생성하게 된다.
3. 화면에 출력해야 할 때, 앞서 만든 LayoutFragment들이 ViewportLayoutController에 넘겨지게 되고, ViewportLayoutController는 rendering surface(view/layer)에 이 fragment들을 위치시키고 레이어링한다.

TextKit1에는 없었던 새로운 객체들이 많이 등장하는데, 여기에서 **value semantics**가 강조된다.

빨간색의 "Your App"이라고 표시되어 있는 부분에서는 올바른 시점에 시스템에 hooking을 통해 텍스트의 layout과 display를 제어할 수 있음을 나타낸 것이다. 이때 **Value semantics를 사용하는 객체, 즉 value 타입처럼 동작하는 immutable한 클래스 객체들에서 받을 수 있는 정보를 시스템에 전달하게 된다.** 

<img width="737" alt="image" src="https://user-images.githubusercontent.com/41438361/208023894-b99defc0-a3f8-4e55-bf4d-71c13dba79d9.png">

**변경사항이 있다면 변경된 내용이 적용된 새로운 value 객체들을 생성한 후에 시스템에 전달해야 하고, 시스템이 이 새롭게 만들어진 객체들의 정보를 사용해서 layout하고, display한다.**

위 TextKit2 디자인에서 등장한 새로운 객체들에 대해 자세히 보자.

### Storage Objects

<img width="532" alt="image" src="https://user-images.githubusercontent.com/41438361/208033211-a9954094-19c8-4ce6-92c8-1429257d67c4.png">

먼저 storage 객체들을 먼저 보겠다.

1. Elements

<img width="1817" alt="image" src="https://user-images.githubusercontent.com/41438361/208024194-b3a837ae-6a5b-4d88-999d-1e696a0212ea.png">

**Elements**는 문서를 구성하는 블럭이다. 

* Element는 컨텐츠의 일부를 나타낸다.
* Element는 문서 내의 위치에 대한 범위 정보를 갖고 있다. **범위를 포함한 프로퍼티는 immutable하기 때문에 element 생성 후에 바뀔 수 없다.**

<img width="1756" alt="image" src="https://user-images.githubusercontent.com/41438361/208024468-748f9eb7-3134-4e8c-843f-8d7e28fcbe9c.png">

문서를 문자열들로 구성하는 것보다 element들로 모델링하는 이유는 element가 어떤 종류의 컨텐츠를 표현하는지 쉽게 알 수 있기 때문이다. 위 사진에서도 element는 텍스트 문단, attachment, custom type 등 다양한 종류가 있다. 그래서 종류에 따라서 어떻게 element를 layout할지를 결정할 수 있다.

2. Content Manager

<img width="1819" alt="image" src="https://user-images.githubusercontent.com/41438361/208024934-c72c7992-4192-410e-bd81-d08a187edda6.png">

* Content manager는 텍스트 컨텐츠에서 요소를 생성하고, 이 요소들을 관리, 추적한다.
* Backing store를 다루고 **backing store에 있는 컨텐츠가 바뀔 때 바뀐 범위에 포함되는 새로운 요소들을 생성**한다. 단순히 Content Manger는 backing store의 wrapper라고 생각하면 된다.

`NSTextContentManager`와 `NSTextElement`는 모두 추상 타입이기 때문에 custom 문서 모델이나 custom backing store가 필요할 때 상속해서 사용하면 된다. 대부분의 경우에는 TextKit2에서 주어지는 기본 객체를 사용하면 된다.

3. Content Storage, Paragraph

<img width="1799" alt="image" src="https://user-images.githubusercontent.com/41438361/208027372-07c7e99c-f429-4062-b0c6-d554b3707dbb.png">

`NSTextContentStorage`는 `NSTextContnetManager`를 상속한 클래스로, `NSTextStorage`를 backing store로 사용하는 컨텐트 매니저다. **텍스트 컨텐츠를 요소로 나눠 `NSTextParagraph` 인스턴스들을 생성한다.**

### Changing content storage

<img width="1862" alt="image" src="https://user-images.githubusercontent.com/41438361/208027717-8124880f-0752-48ee-b09d-e5aa26500064.png">

Text storage에 변경사항을 만들면, 이 변경사항을 `performEditingTransaction`에 전달해야 한다. 이 메서드를 사용해서 TextKit2 시스템의 다른 부분이 변경사항이 있음을 알 수 있다.



<img width="736" alt="image" src="https://user-images.githubusercontent.com/41438361/208032457-101caab0-84ca-4ca3-8ddb-4bed6904a813.png">

여기까지 TextKit2가 어떻게 텍스트 컨텐츠에서 요소들을 생성하는지 봤고, 이는 앞에서 봤던 4단계의 첫 번째 두 단계를 커버한다. Content storage가 자동으로 텍스트를 paragraph element들로 나누고, 새로운 커멘트가 달린 부분에 대해 새로운 paragraph(NSTextParagraph 인스턴스를 의미하는 것으로 보인다)를 만들었다.

### Layout Objects

<img width="528" alt="image" src="https://user-images.githubusercontent.com/41438361/208033111-bb86aba7-bc2f-4b45-9c86-c1378151e2c6.png">

앞서 **생성한 커멘트 element들을 배치할 layout 정보를 알아야 한다.**

1. Text Layout Manager

<img width="1819" alt="image" src="https://user-images.githubusercontent.com/41438361/208033402-d7ca2df9-3b84-47b8-a2d7-c6729d1f9e0c.png">

* 텍스트 layout 프로세스를 제어한다.
* **Glyph를 사용하지 않는다. 대신 텍스트 element들을 받아 text container에 배치하고, element들의 layout fragment들을 생성한다.** Layout fragment를 사용해서 text element들의 layout 정보를 받는다.

2. Layout Fragment

<img width="1797" alt="image" src="https://user-images.githubusercontent.com/41438361/208033824-ef7d1623-0121-4bd1-86f5-ae646004007e.png">

* 하나 이상의 element들의 layout  정보를 갖고 있다.
* **Value semantics를 사용하고 프로퍼티는 immutable하다.**

<img width="1854" alt="image" src="https://user-images.githubusercontent.com/41438361/208034082-40fd27e2-3ed4-4f41-a8fa-7a3d5dcc954b.png">

Layout fragment는 세 개의 프로퍼티를 통해 layout 정보를 주고 받는다. Layout을 커스터마이징하거나 바꾸고 싶다면 이 프로퍼티들을 알고 있어야 한다.

1. `textLineFragments`
2. `layoutFragmentFrame`
3. `renderingSurfaceBounds`

**textLineFragments**

<img width="1807" alt="image" src="https://user-images.githubusercontent.com/41438361/208034421-9da44ba5-eeef-4281-9ebd-ca0c30223ade.png">

Line fragment는 layout fragment 안의 텍스트의 각 줄에 대한 수치 정보를 가지고 있다. 특정 줄의 위치 정보나 줄 수를 세는데 사용한다.

**layoutFragmentFrame**

Layout fragment frame은 layout fragment 안의 텍스트가 텍스트 container 영역에 어떻게 레이아웃되는지를 설명한다. TextKit2에서는 기본적으로 container 내에 layout fragment frame들이 스택처럼 쌓이게 된다. 

<img width="1594" alt="image" src="https://user-images.githubusercontent.com/41438361/208034688-272350af-d48a-416f-9120-e05a0da0884c.png">

위 사진이 스택처럼 쌓이는 걸 보여주고 있는데, frame을 타일처럼 생각하면 시스템이 text container영역을 타일로 나눠서 스택처럼 쌓았다고 생각할 수 있다. 단지 각 타일이 layout fragment인 것이다. 빈 줄의 경우도 layout fragment frame을 갖고 있다. Layout fragment frame은 텍스트 컨텐츠의 전체 높이를 계산하거나 fragment 컨텐츠 근처에 다른 view를 위치시킬 때 유용하다.

**renderingSurfaceBounds**

앞서 봤던 frame은 텍스트를 그려야 하는 공간을 정확하게 나타내지는 않는다. 이를 표현하기 위한 프로퍼티가 바로 `renderingSurfaceBounds`다.

<img width="1639" alt="image" src="https://user-images.githubusercontent.com/41438361/208035954-0e61e7c5-1582-4964-a017-efcae186c743.png">

Rendering surface bounds는 텍스트를 그리기 위한 영역을 나타낸다. 텍스트가 화면에 나타나는 영역을 알고 싶을 때가 있는데, 위 사진에서처럼 layout fragment frame과 텍스트가 나타나는 영역이 다르다.

Layout fragment frame이 보라색 점선으로 표시되어 있는 부분인데, 맨 앞 글자인 j의 끝 부분이 살짝 frame을 벗어난 것을 확인할 수 있다. 주로 이태릭체를 쓸 때, descender가 긴 폰트에서 이런 상황이 많이 발생한다. 그래서 Rendering surface bounds는 주황색 실선으로 표시된 영역으로 설정돼서 텍스트가 그려진 부분을 모두 커버할 수 있다.

### Customizing layout fragments

여기까지 layout fragment가 제공하는 layout 정보들을 봤고, 이 정보들을 사용해서 텍스트 element들의 layout을 어떻게 커스터마이징하는지 볼 것이다.

**Layout framgent들도 element와 같이 value semantics를 사용하기 때문에 immutable하고, fragment에 직접 접근해서 layout 정보를 바꿀 수 없다.**

<img width="1781" alt="image" src="https://user-images.githubusercontent.com/41438361/208037067-7bf46f3f-74c6-487c-a111-52b9b8f480de.png">

Flow 다이어그램을 다시 보면, layout 프로세스가 일어날 때 우리가 **변경하고 싶은 정보들로 새로운 `NSTextLayoutFragment` 인스턴스들을 생성해야 한다.**

<img width="1856" alt="image" src="https://user-images.githubusercontent.com/41438361/208037325-e7995c05-0ff4-4c13-bfd0-b38b54e3e74d.png">

그리고 layout 프로세스에 hooking할 때 `NSTextLayoutManager`의 delegate method를 사용한다. 이 메서드는 text layout manager가 element들에서 layout fragment들을 생성할 때 호출된다. 여기에서 자신만의 커스텀 layout fragment를 생성할 수 있다.

<img width="734" alt="image" src="https://user-images.githubusercontent.com/41438361/208037672-e255197f-6d3a-45a1-83de-9998144147ac.png">

여기까지 앞서 본 4단계의 나머지 두 단계를 커버했다. `NSTextlayoutFragment`의 자식 클래스를 사용하고 text layout manager delegate에 커스텀 fragment의 인스턴스들을 제공해서 layout fragment들을 위치시키고, 커스텀하게 그렸다.

여기까지 다양한 클래스가 새로 등장했는데, value semantics를 사용하면서 작업이 이루어지고 있는 것을 여러 곳에서 확인할 수 있었다.

## 3. Performance

텍스트 엔진의 성능은 중요하다. 수백 MB의 문서를 렌더링하고, 스크롤 해야하는 경우에는 **noncontigous한 text layout**이 좋은 성능을 위해 필수적이다.

### Noncontiguous vs contiguous

<img width="1801" alt="image" src="https://user-images.githubusercontent.com/41438361/208039928-deb746a3-c147-4d80-8ed2-77911236822a.png">

Conginuous layout은 문서의 처음부터 시작해서 텍스트의 끝까지 순서대로 나타난다. 그래서 **문서의 중간 지점까지 스크롤하게 되면 contiguous layout은 그 지점 이전까지의 모든 텍스트를 layout하게 된다. 이게 문제인 이유는 이미 스크롤돼서 화면 밖으로 나간 텍스트들에 대해서도 layout하기 때문이다. 텍스트가 굉장히 많다면 성능이 떨어지고, 스크롤할 때 animation hiccup이 발생할 수도 있고 멈출 수도 있다.**

<img width="1826" alt="image" src="https://user-images.githubusercontent.com/41438361/208040611-5ee18a87-0ea6-4fb0-89d7-5b126e1b6037.png">

반면 **noncontiguous layout은 문서의 어떤 위치에 있는 텍스트 조각을 그 조각 이전에 오는 다른 조각들을 layout하지 않고서도 layout할 수 있다는 뜻이다. 그래서 문서 중간까지 스크롤하면 실제로 보여지는 부분에 대해서만 layout이 일어난다. 화면에 보여지는 부분(조금 벗어나는 부분까지)만 layout을 수행하기 때문에 성능이 좋다.**

### TextKit1, TextKit2 layout 비교

**TextKit2의 layout은 항상 noncontiguous하다.** 반면 TextKit1에서는 이를 옵션으로 설정할 수 있었다.

<img width="1770" alt="image" src="https://user-images.githubusercontent.com/41438361/208041147-48780eeb-b07d-47f4-a585-9db6ddce6172.png">

TextKit1에서의 noncontiguous layout 관련 API가 굉장히 간단했기 때문에 layout 정보를 요청할 때 충분한 정보를 표현하지 못했다. 

Noncontiguous layout은 문서의 다른 부분들이 layout 됐을 때 변할 수 있는 어떤 추정치들에 의존하는데, TextKit1에서는 오직 noncontiguous layout 옵션을 킬건지 끌건지만 설정할 수 있었다.

TextKit2 API는 더 많은 정보를 표현할 수 있다. 보여지는 영역의 element들의 layout 정보를 지속적으로 제공하고 보여지는 영역에 layout update가 생기면 알려준다.

<img width="1755" alt="image" src="https://user-images.githubusercontent.com/41438361/208041986-19e19964-0242-4f3b-b3b3-3446b91c17bd.png">

위 그림에서  **보여지는 영역을 viewport**라고 한다. Viewport를 조정하고 재배치해서 관리할 수 있고, viewport가 layout하기 전, 하는 도중, 그리고 하고 난 후에 callback을 받을 수 있다.

성능을 위해서 코드에서는 **viewport 영역 내의 layout 정보들에 집중해야 한다.** 가능한 한 viewport 밖에 있는 요소들의 layout 정보를 요청하지 말아야 한다. 밖에 있는 요소들의 정보는 부정확할 수도 있고, 정보를 받는 것도 성능상 좋지 않다.

### Viewport Objects

<img width="609" alt="image" src="https://user-images.githubusercontent.com/41438361/208042626-b7bd11b5-875d-49de-b008-d6f0715e31c3.png">

앞서 봤던 flow 다이어그램에서는 viewport를 관리하기 위한 새로운 controller 클래스가 있다.

1. Viewport Layout Controller

<img width="1806" alt="image" src="https://user-images.githubusercontent.com/41438361/208042781-593669d9-c1ad-4b2e-8b57-b33fb49ef79e.png">

* Viewport layout 정보의 source of truth 이다. Text layout manager에 viewport 영역 내 요소들의 layout fragment를 요청한다.
* Text layout manager의 프로퍼티로 viewport layout controller에 접근할 수 있다.

### Viewport layout process

![image](https://user-images.githubusercontent.com/41438361/208325500-adc9614b-f773-40e6-b4fd-8b84a74bff39.png)

Viewport layout controller는 viewport layout 프로세스가 일어나는 동안 위 세개의 메서드를 호출한다.

* `willLayout` : Viewport에 요소들을 layout 하기 전에 호출한다. Layout을 위해 미리 해야하는 작업들을 할 수 있다. (e.g. view나 layer의 컨텐츠를 비워두기)
* `configureRenderingSurface` : Viewport에서 볼 수 있는 모든 layout fragment에 대해 호출한다. 각 fragment view나 layer의 기하적인 특성을 조정한다.
* `didLayout` : viewport내에서 볼 수 있는 모든 Layout fragment들을 layout한 후에 호출한다.

![image](https://user-images.githubusercontent.com/41438361/208325768-f3fb674e-87c6-4506-9729-061eda860c5f.png)

위 사진은 viewport layout이 끝난 후에, 마지막 요소를 화면에 보이게끔 해서 다시 업데이트가 일어날 때 호출되는 메서드들을 나타낸 것이다.

# Modernization

![image](https://user-images.githubusercontent.com/41438361/208793631-8598f7d1-c900-4f0e-b9d5-cacbf4ec367c.png)

TextKit2의 새로운 클래스들은 iOS 15 이상, macOS 12 이상에서 UIKit에서 사용할 수 있다. 

## Compatibility

많은 앱에서는 TextView와 같이 내장된 text control을 사용하고 있다. 그리고 이 중 일부는 TextKit2로 업데이트 됐다. 이전에 TextKit1을 사용하던 곳에서 새롭게 도입된 TextKit2로 바뀌었을 때 기능이 그대로 잘 작동할 수 있게 하는 것이 중요하다. 이런 compatibility 때문에 iOS 15와 macOS 12의 일부 control만 TextKit2를 자동으로 사용하고 있는 것이다.

![image](https://user-images.githubusercontent.com/41438361/208794251-806fe957-9a4d-4db0-b7d2-8e6dc287a7ef.png)

어떤 OS 버전에서는 TextKit 2를 사용하려면 추가 작업이 필요하다. AppKit에서 NSTextView는 TextKit2를 자동으로 사용하지 않는다. 만약 NSTextView에 TextKit2를 사용하고 싶다면 생성할 때 코드상으로 위와 같이 해야 한다.

1. Text layout manager를 생성한다.
2. Text container를 생성한다.
3. NSTextLayoutManager의 textContainer 프로퍼티를 사용해서 text layout manager와 text container를 연결한다.

위 방법으로 TextKit 2를 사용하는 text view를 사용할 수 있다. 

![image](https://user-images.githubusercontent.com/41438361/208795352-b25b3ebd-3614-4026-904b-e6fde8475745.png)

그리고 NSTextView의 `layoutManager` 프로퍼티에 접근할 때 주의해야 한다. `layoutManager` 프로퍼티를 호출하면 NSLayoutManager를 가져오고 설정할 수 있게 하는데, **NSLayoutManager는 TextKit1 객체고 TextKit2에서는 사용할 수 없다.** 그래서 text view는 `textLayoutManager`와 `layoutManager`를 동시에 가지고 있을 수 없다. 

### Compatibility mode

![image](https://user-images.githubusercontent.com/41438361/208795702-9982c3d7-3844-4380-bcb6-bdbff39cdc96.png)

그래서 필요한 경우 NSTextView가 TextKit1을 사용하게끔 변경할 수 있는 compatibility 모드를 추가했다. 만약에 `layoutManager`를 사용하고 있는 게 감지되면 text view는 자동으로 `NSTextLayoutManager`를 `NSLayoutManager`로 대체해서 사용한다. 

![image](https://user-images.githubusercontent.com/41438361/208795920-b3328c3e-7652-46c9-b377-fe09f7dfa239.png)

TextKit 2를 사용하는 것을 기반으로 코드를 작성한다 해도 `layoutManager` 프로퍼티를 호출하면 자동으로 TextKit 1으로 대체될 것이다. TextKit 2에서는 지원되지 않거나 TextKit 1을 필요로 하는 동작이 감지될 경우 TextView는 자동으로 TextKit1을 사용하게 된다.

![image](https://user-images.githubusercontent.com/41438361/208796454-b64386eb-1e84-429a-84e5-6179a2d283d9.png)

NSTextField의 경우 기본적으로 TextKit 2를 사용하는데, 마찬가지로 fiedlEditor의 layoutManager를 호출하게 된다면 field editor는 자동으로 TextKit 1을 사용하게 된다. 

### NSTextView compatibility notifications

![image](https://user-images.githubusercontent.com/41438361/208796629-f4f60b47-264f-441f-a5d9-7461a205a435.png)

시스템은 text view가 TextKit1으로 바뀌기 전에 notification을 보낼 수 있다. 이 notification을 사용해서 textkit 1으로 바꿔서 사용하게 됐음을 알 수 있다.

### UIKit

![image](https://user-images.githubusercontent.com/41438361/208796857-19b46f93-ebbb-4cdb-8d46-bfb206737cc2.png)

UIKit의 경우 UITextField는 iOS 15부터 자동으로 TextKit 2를 사용한다. 하지만 iOS 15에서 UITextView가 TextKit 2를 사용하게 할 수 없다. 

# Demo

<img width="698" alt="image" src="https://user-images.githubusercontent.com/41438361/208328344-fab9d964-f75a-49af-8177-7a72abfcf894.png">

1. Text layout manager가 어떤 이유(변화가 생김/container 크기가 변함/새로운 부분이 viewport에 나타남 등)로 인해 문서를 layout하기 전에 viewport layout delegate의 `textViewportLayoutControllerWillLayout` 메서드를 호출한다. 여기에서 모든 텍스트 sublayer를 지우고 animation transaction을 시작한다.

<img width="835" alt="image" src="https://user-images.githubusercontent.com/41438361/208328525-5f20900b-9fe0-4a69-9a01-29263527a7f0.png">

2. Text layout manager가 layout하는 각 텍스트 요소에 대해 `textViewportLayoutController(configureRenderingSurfaceFor:)` 메서드를 호출한다. 여기에서는 text layout fragment를 출력하기 위한 layer를 받고, 기하적 특성을 업데이트하고, 새로운 위치로 애니메이션을 수행한 다음, sublayer로 추가한다.

![image](https://user-images.githubusercontent.com/41438361/208329175-9717069a-43a6-411a-b488-a2600a79a7f4.png)

3. Layout manager가 layout을 마쳤다면 `textViewportLayoutControllerDidLayout` 메서드를 호출한다. Animation transaction을 commit하고, selection highlight을 업데이트 한 후, content size를 업데이트해서 scroll thumb이 올바른 곳에 위치하도록 한다.

## Comment

문서의 comment를 받아서 font나 color와 같은 custom attribute를 적용하고, 바탕에 버블을 그리는 방법을 알아볼 것이다.

### NSTextContentStorageDelegate
<img width="955" alt="image" src="https://user-images.githubusercontent.com/41438361/208791690-b250ad09-6a1f-4370-8b18-b3419394a181.png">

문서의 각 paragraph마다, text content storage는 paragraph의 attribute를 delegate에서 커스터마이징 할 수 있게 한다. 구현부를 보면 커멘트일 경우 새로운 attribute를 적용하고 있다.

### NSTextContentManagerDelegate

<img width="912" alt="image" src="https://user-images.githubusercontent.com/41438361/208792079-fb614654-b15f-4ba5-8dc8-0ca9313e2981.png">

Text content manager는 layout 도중에 어떤 text element들이 text layout manager에 보여야 하는지를 delegate에서 결정할 수 있게 한다. 구현부에서 false를 리턴하는 것은 화면에 보여지지 않게끔 한다. 그래서 false를 리턴하게 되면 실제로 text storage에서 지우는 것이 아니라 커멘트를 감추는 효과를 나타내게 된다. 

### NSTextLayoutManagerDelegate

<img width="1011" alt="image" src="https://user-images.githubusercontent.com/41438361/208792392-e4fcfe88-2656-4f54-8f3f-92ae460aafde.png">

Text layout manager도 delegate가 있다. `textLayoutManager(_:textLayoutFragmentFor:in:)` 메서드를 구현하면 delegate는 주어진 NSTextElement에 대해 기본적인 NSTextLayoutFragment 인스턴스 대신에 커스텀 TextLayoutFragment를 생성할 수 있다. 여기에서 NSTextElement가 커멘트일 경우 NSTextLayoutFragment의 자식 클래스인 BubbleLayoutFragment를 생성하게 된다.

### BubbleLayoutFragment

<img width="540" alt="image" src="https://user-images.githubusercontent.com/41438361/208792764-203c4a3a-6dbd-4fd2-8972-484504c460ea.png">

BubbleLayoutFragment에서는 NSTextLayoutFragment의 draw 메서드를 오버라이딩해서 맨 위에 텍스트를 그리기 전에 바탕에 버블을 그리도록하고 있다.

* 참고
* https://www.creativelive.com/blog/6-typography-terms-that-get-confused/
* https://hackingcpp.com/cs/value_vs_reference_semantics.html
