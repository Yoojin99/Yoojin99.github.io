---
title:  "React Native - Design - 높이와 너비"
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

component의 높이와 너비는 스크린에서 보여지는 사이즈를 결정한다.

## 고정된 치수

component의 치수를 설정하는 일반적인 방법은 style에 고정된 `width`와 `height`를 추가하는 것이다. RN의 모든 치수는
단위가 없고, 밀도-독립적인 픽셀을 나타낸다.

```js
import React from 'react';
import { View } from 'react-native';

const FixedDimensionsBasics = () => {
    return (
      <View>
        <View style={{width: 50, height: 50, backgroundColor: 'powderblue'}} />
        <View style={{width: 100, height: 100, backgroundColor: 'skyblue'}} />
        <View style={{width: 150, height: 150, backgroundColor: 'steelblue'}} />
      </View>
    );
};

export default FixedDimensionsBasics;
```
스크린 치수에 상관없이 항상 같은 사이즈로 렌더링 되어야 할 때 이런식으로 치수를 설정하는 것은 일반적인 방법이다.

## 유동적인 치수

component가 가용 공간에 따라 동적으로 늘고 줄게 하기 위해 component의 스타일에서 `flex`을 이용해라.
보통 같은 부모를 가진 다른 요소들끼리 공유되는 가용 공간을 component가 모두 채우라는 뜻인 `flex:1`을 사용한다.
더 큰 `flex`가 주어질 수록, 형제들에 비해 더 넓은 비중을 차지하게 된다.

component는 부모가 0보다 큰 치수를 가지고 있을 때만 가용 공간을 채우기 위해 확장될 수 있다. 만약 부모가 고정된 `width`나 
`height`나 `flex`중 하나라도 가지고 있지 않다면, 부모는 치수가 0일 것이고 `flex` 자식은 보여지지 않을 것이다.

```js
import React from 'react';
import { View } from 'react-native';

const FlexDimensionsBasics = () => {
    return (
      // Try removing the `flex: 1` on the parent View.
      // The parent will not have dimensions, so the children can't expand.
      // What if you add `height: 300` instead of `flex: 1`?
      <View style={{flex: 1}}>
        <View style={{flex: 1, backgroundColor: 'powderblue'}} />
        <View style={{flex: 2, backgroundColor: 'skyblue'}} />
        <View style={{flex: 3, backgroundColor: 'steelblue'}} />
      </View>
    );
};

export default FlexDimensionsBasics;
```


