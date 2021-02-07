---
title:  "React Native - 핵심 요소와 Native 요소"
excerpt: ""

toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
header:
  teaser: 
  
  
categories:
  - react
  
tags:
  - react-rn
  - React Native
  
last_modified_at: 
---

![image](https://user-images.githubusercontent.com/41438361/90485901-2404bb80-e173-11ea-8aaf-141b0555134b.png)

React Native은 안드로이드와 iOS 앱들을 React를 사용해서 만들기 위한 오픈 소스 프레임워크이다. React Native로, Javasciprt를 이용해서
플랫폼의 API에 접근할 수 있고 React Component(재사용 가능하고, 중첩 가능한 code)의 동작과 표현까지 설명할 수 있다.

먼저, React Native에서 어떻게 요소들이 작동하는지 보겠다.

## View 와 모바일 개발

Android와 iOS 개발에서 `view`는 기본적인 UI block이다. (화면에 text, image, 혹은 사용자 입력에 대응할 수 있는 것을 보여줄 수 있는
사각형 요소이다.) 심지어 텍스트 한 줄이나 버튼같이 가장 작은 시각적 요소들도 view의 일종이다.

어떤 view는 다른 view를 포함할 수 있다.

![image](https://user-images.githubusercontent.com/41438361/90486852-8f9b5880-e174-11ea-9e87-f95afe227a86.png)

위의 이미지는 안드로이드와 iOS 앱에서 많은 view들이 사용된다는 것을 보여준다.

## Native 요소들

안드로이드 개발에서, view는 Kotlin이나 Java를 통해 작성한다. iOS 개발에서는 Swift나 Objective-C를 이용한다.

React Native에서는 이런 view들을 React 요소들을 이용하여 JavaScript로 만들 수 있다. Runtime때, React Native는 이런 요소들과 일치하는 안드로이드와 iOS view들을 생성한다.

React Native 요소들이 Android와 iOS것과 같은 view들의 지원을 받았기 때문에 react native app은 다른 앱들과 똑같은 look and feel을 지원한다.
이런 플랫폼-지원 요소들을 **Native Component**라고 부른다.

React Native는 자신만의 Native Component를 만들 수 있도록 한다. 또한 Community-contributed-components(깃헙같은 곳에서 사람들이 자신이 만든 요소들을 올려놓는 것 같았다.)

이는 [Native Directory](https://reactnative.directory/) 여기서 확인할 수 있다. 보니까 채팅 플랫폼 처럼 꾸며놓은 것과 같이 많은 요소들이 있다.

React Native는 또한 필수적인, 바로 사용가능한 Native Component들을 가지고 있다. 이것들이 React Native의 **핵심 요소**들이다.

## Core Components

React Native는 form controls부터 activity indicators까지 많은 핵심 요소들을 가지고 있다. 이것들은 모두
[여기](https://reactnative.dev/docs/components-and-apis)에서 찾을 수 있다. 주로 사용되는 핵심 요소들은 아래와 같다.

*나는 안드로이드 유저이기 때문에 안드로이드에 관한 것만 적도록 하겠다. iOS에 관한 것까지 확인하고 싶다면 [여기](https://reactnative.dev/docs/intro-react-native-components)에서 확인할 수 있다.*

|React Native UI 요소들|안드로이드 View|Web Analog|설명|
|----------------------|---------------|----------|----|
|`<View>`|`<ViewGroup>`|non-scrolling `<div>`|flexbox, touch handling, 접근성 관리와 함께 레이아웃을 지원하는 컨테이너|
|`<Text>`|`<TextView>`|`<p>`|텍스트. 터치 이벤트까지 다룰 수 있다.|
|`<Image>`|`<ImageView>`|`<img>`|다양한 타입의 이미지를 전시한다.|
|`<ScrollView>`|`<ScrollView>`|`<div>`|다양한 요소들과 view를 포함할 수 있는 스크롤 가능한 컨테이너|
|`<TextInput>`|`<EditText>`|`<input type="text">`|유저들이 텍스트를 입력할 수 있게 해준다.|

예시 코드는 아래와 같다.

```javascript
import React from 'react';
import { View, Text, Image, ScrollView, TextInput } from 'react-native';

const App = () => {
  return (
    <ScrollView>
      <Text>Some text</Text>
      <View>
        <Text>Some more text</Text>
        <Image
          source={{
            uri: 'https://reactnative.dev/docs/assets/p_cat2.png',
          }}
          style={{ width: 200, height: 200 }}
        />
      </View>
      <TextInput
        style={{
          height: 40,
          borderColor: 'gray',
          borderWidth: 1
        }}
        defaultValue="You can type in me"
      />
    </ScrollView>
  );
}

export default App;
```

React Native가 React 요소들과 같은 API 구조를 사용하고 있기 때문에, 먼저 React component API들을 이해해야 한다. 

![image](https://user-images.githubusercontent.com/41438361/90488745-50bad200-e177-11ea-9089-b07c20a6a0dd.png)



