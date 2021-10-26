---  
layout: post  
title: "[iOS] - modalPresentationStyle 비교"  
subtitle: ""  
categories: app
tags: app-ios modalPresentationStyle
comments: true  
header-img: 

---  
  
> `UIViewController를 띄울 때 설정할 수 있는 modalPresentationStyle을 비교해보자.`  

---

UIViewController를 띄울 때 아래와 같이 띄운다.

```swift
present(vc, animated: false, completion: nil)
```

이렇게 띄우면 기본적으로 아래와 같이 뷰 컨트롤러가 띄워진다.

![image](https://user-images.githubusercontent.com/41438361/138827465-548d8f38-60d8-443c-a829-2d458101ef38.png)

초록색 화면이 원래 UIViewController이고, 그 위에 떠져 있는 약간 투명한 검은색 뷰가 새로 띄운 UIViewController이다. 밑에 있는 UIViewController를 보기 위해서
띄우는 뷰를 일부러 약간 투명하게 만들었다.

하지만 이렇게 기존의 화면 위에 새로운 화면이 올라오면서 기존의 화면이 보이는 스타일 말고도 다른 스타일로 UIVIewController를 띄울 수 있다.

이를 UIViewController의 인스턴스 프로퍼티인 `modalPresentationStyle`로 바꿀 수 있다.

## modalPresentationStyle

![image](https://user-images.githubusercontent.com/41438361/138827886-f2f642eb-d370-44d0-bafc-49679a21db35.png)

공식 문서에 의하면 modal view controller를 위한 presentation style을 설정할 수 있다고 한다. 그렇다면 modal은 무엇일까?

Modal은 사용자의 이목을 끌기 위한 화면 전환 기법이다. 모달에는 흐름을 이어지는 컨텐츠를 담기보다는 흐름이 끊어져서 딱 눈에 들어올 수 있는 컨텐츠를 담는데 사용하면 좋다고 한다. 

별로 중요한 것은 아니니 여기까지 보고, presentation style을 어떻게 바꿀 수 있는지 보도록 하겠다.

modalPresentationStyle을 지정해서 UIViewController를 present 하려면 아래와 같이 원하는 스타일을 지정해서 띄워주면 된다.

```swift
vc.modalPresentationStyle = .automatic
present(vc, animated: false, completion: nil)
```

**automatic**

시스템에 의해 선택된 기본 presentation style이다.

![image](https://user-images.githubusercontent.com/41438361/138831656-6e5b2c1a-a700-425e-b781-eaeddce2fb67.png)

위에서 봤던 그 스타일이다. 

**none**

설명을 직역하자면 수정이 없어야 함을 나타내는 프레젠테이션 스타일이라고 한다. 그래서 설정해서 실행했더니 아래와 같이 에러가 발생했다. 

![image](https://user-images.githubusercontent.com/41438361/138831893-4dbc33e9-279d-4082-9742-42d476c86609.png)

이유를 찾기 어려웠는데, 뷰 컨트롤러를 present할 때 이 스타일을 사용하면 안된다고 한다. 


**fullScreen**

새로 띄워지는 뷰가 화면 전체를 커버하는 스타일이다.

![image](https://user-images.githubusercontent.com/41438361/138832895-31d73d7b-d525-43a7-9b25-cdebacbb834e.png)

기존의 초록색 화면이 아예 보이질 않는다.

**pageSheet**

새로 띄워지는 뷰가 부분적으로 아래의 컨텐츠를 커버하는 스타일이다.

![image](https://user-images.githubusercontent.com/41438361/138833049-5d75f6be-04e7-4ec1-96f5-3e494e201f0c.png)

default 스타일이랑 같은 스타일이다.

**formSheet**

컨텐츠를 화면 중앙에 나타내는 스타일이다.

![image](https://user-images.githubusercontent.com/41438361/138833294-5144c1e5-5075-425e-84a3-c5d26c969df5.png)

위의 pageSheet랑 별 다른 차이점이 없어 보이는데, 가로 방향일 때 조정이 되어 표시되는 스타일인 것 같다. 덮이지 않는 영역은 사용자가 상호작용 못하게
흐리게 표시된다.

![image](https://user-images.githubusercontent.com/41438361/138833842-f3e441eb-5123-42e2-94a1-9727c7305fdb.png)

**currentContext**

컨텐츠가 다른 뷰 컨트롤러의 컨텐츠 위에 표시되는 스타일이다.

![image](https://user-images.githubusercontent.com/41438361/138833970-ff479a67-5404-4d13-acac-16eee67dfbf7.png)

**custom**

custom으로 지정하게 될 경우 custom presentation controller와 하나 이상의 custom animator 객체를 이용해서 띄우게 된다.
이를 통해 내가 원하는 스타일, 애니메이션 대로 화면을 띄울 수 있다.

**overFullScreen**

스크린을 커버하면서 띄우는 스타일이다.

![image](https://user-images.githubusercontent.com/41438361/138834276-db0a1964-63a6-4a26-912c-490797ea8497.png)

새로 띄워지는 화면이 화면 전체를 커버하는 것을 볼 수 있다.

**popOver**

popoverview에 컨텐츠가 출력되는 스타일이다.

![image](https://user-images.githubusercontent.com/41438361/138834525-f4030454-40c8-48d3-b5f6-b1f54996f904.png)




