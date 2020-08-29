---
title:  "React Native - ScrollView 사용하기"
excerpt: ""

toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
header:
  teaser: 
  
  
categories:
  - ReactNative
  
tags:
  - 공부
  - React Native
  
last_modified_at: 
---

![image](https://user-images.githubusercontent.com/41438361/90485901-2404bb80-e173-11ea-8aaf-141b0555134b.png)
 
 ScrollView는 많은 component와 view를 포함할 수 있는 스크롤 컨테이너이다. 스크롤 가능한 아이템들은 무조건 같을 필요는 없고,
 가로와 세로 방향 모두 스크롤 가능하다.(`horizontal` property를 설정하면 된다.)
 
 아래의 예제는 이미지와 텍스트를 함께 세로방향의 `ScrollView`를 생성하여 보여준다.
 
```js
import React from 'react';
import { Image, ScrollView, Text } from 'react-native';

const logo = {
  uri: 'https://reactnative.dev/img/tiny_logo.png',
  width: 64,
  height: 64
};

export default App = () => {
  <ScrollView>
    <Text style={{ fontSize: 96 }}>Scroll me plz</Text>
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Text style={{ fontSize: 96 }}>If you like</Text>
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Text style={{ fontSize: 96 }}>Scrolling down</Text>
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Text style={{ fontSize: 96 }}>What's the best</Text>
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Text style={{ fontSize: 96 }}>Framework around?</Text>
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Text style={{ fontSize: 80 }}>React Native</Text>
  </ScrollView>
}
```

참 예제들을 보면 정말 만들면서 재밌어 했을 것 같다...

ScrollView는 `pagingEnabled` prop을 이용해서 swiping 제스처를 사용해서 view들 사이를 왔다갔다 할 수 있게 하는데 구성할 수 있다.
View들 사이를 가로방향으로 swiping하는 것은 안드로이드에서 [`ViewPager`](https://github.com/react-native-community/react-native-viewpager)라는 component를 사용해서 구현할 수 있다.

iOS에서 단 하나의 요소만 가지고 있는 ScrollView는 확대할 수 있게도 할 수 있다. `maximumZoomScale`과 `minimumZoomScale` prop을 
설정하면 사용자가 줄이고 늘리는 제스처를 사용해서 확대하거나 축소할 수도 있다.

ScrollView는 작은 양의 것들을 한정된 사이즈로 보여주는 것에 유용하다. 모든 `ScrollView`의 요소들과 뷰들은 스크린에 현재 보여지고 있지 않더라도
 렌더링된다. 만약 더 많은 아이템들이 있다면, `FlatList`를 사용하는 것을 추천한다.
 
 
 
