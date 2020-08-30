---
title:  "React Native - Design - Flexbox와 Layout"
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

component는 Flexbox 알고리즘을 이용해서 자식들의 레이아웃을 특정지을 수 있다. Flexbox는 다른 스크린 사이즈에서 일괄적인 레이아웃을 제공하기 위해 디자인되었다.

일반적으로 적당한 레이아웃을 설정하기 위해 `flexDirection`, `alignItems`, `justifyContent`를 조합해서 사용한다.

Flexbox는 몇가지 예외를 제외하고 CSS가 웹에서 동작하는 것처럼 RN에서 같은 방식으로 작동한다. 디폴트는 `flexDirection`이
`row` 대신에 `column`을 사용한다는 것이고, `flex` 파라미터가 오직 숫자값 하나만 지원한다는 점이 다르다.

## Flex

`flex` 는 주축을 따라 사용가능한 공간을 아이템들이 어떻게 **채울지** 정의한다. 공간은 각 요소들의 flex property에 따라 나누어질 것이다.

아래의 예제에서는 빨강, 노랑, 그리고 초록 view들은 `flex: 1`을 가지고 있는 컨테이너 view의 자식들이다.
빨강 view는 `flex:1`, 노란색 view는 `flex:2`, 그리고 초록 view는 `flex:3`을 이용한다. **1+2+3 =6**이므로
빨간색 뷰는 공간의 1/6, 노란색은 2/6, 초록색은 3/6만큼 차지한다.

![image](https://user-images.githubusercontent.com/41438361/91653416-a5e1d680-eadb-11ea-855b-e5ee130a7dba.png)

## Flex Direction

`flexDirection`은 노드의 자식들이 어느 방향으로 펼쳐지는지를 조정한다. 이거는 또한 주축을 참조한다.
가로축은 주축에 수직이다.

* `row` : 왼쪽에서 오른쪽으로 자식들을 정렬한다. 만약 wrapping이 되어 있지 않다면, 첫번째 아이템의 아래부분의 컨테이너의 왼쪽부분에서 다음줄이 시작된다.
* `column`(default) : 자식들을 위에서 아래로 정렬한다. 만약 wrapping이 되어 있지 않다면, 다음줄은 첫번째 아이템의 오른쪽에서 시작된다.
* `row-reverse` : 자식들을 오른쪽에서 왼쪽으로 정렬한다. 만약 wrapping이 되어 있지않다면, 다음 줄은 첫번째 아이템의 아래의 오른쪽에서 시작될 것이다.
* `column-reverse` : 자식들을 아래에서 위로 정렬한다. 만약 wrapping이 되어 있지 않다면, 다음 줄은 첫번째 아이템의 오른쪽, 컨테이너의 바닥에서 시작될 것이다.

```js
import React from 'react';
import { View } from 'react-native';

const FlexDirectionBasics = () => {
    return (
      // Try setting `flexDirection` to `column`.
      <View style={{flex: 1, flexDirection: 'row'}}>
        <View style={{width: 50, height: 50, backgroundColor: 'powderblue'}} />
        <View style={{width: 50, height: 50, backgroundColor: 'skyblue'}} />
        <View style={{width: 50, height: 50, backgroundColor: 'steelblue'}} />
      </View>
    );
};

export default FlexDirectionBasics;
```

![image](https://user-images.githubusercontent.com/41438361/91653585-2228e980-eadd-11ea-851a-9addbf06b1dd.png)

## Layoout Direction

Layout direction은 계층에서 자식들과 텍스트가 어떤 방향으로 정렬되어야 하는지 정의한다. Layout dirctiondms `start`와 `end`가 참조할 엣지에 영향을 미친다.
기본적으로 RN은 LTR layout 방향을 이용한다. 이 모드에서는 `start`는 왼쪽을, `end`는 오른쪽을 참조한다.

* `LTR`(디폴트) : 텍스트와 자식들이 왼쪽에서 오른쪽으로 펼쳐진다. 마진과 패딩은 왼쪽 부분에 요소들의 시작지점에 적용된다.
* `RTL` : 텍스트와 자식들이 오른쪽에서 왼쪽으로 펼쳐진다. 마진과 패딩은 오른쪽 부분에 요소들의 시작지점에 적용된다.

## Justify Content

`justifyContent`는 자식들이 그들의 컨테이너의 주축에 어떻게 배치될지를 정의한다. 예를 들어, 자식을 컨테이너 상에 가로 방향에서 중앙에 위치하도록 
`flexDirection`으로 `row`를 설정하거나 세로방향으로는 `column`을 설정하기 위해 `flexDirection`을 사용할 수 있다.

* `flex-start`(디폴트) : 자식들을 컨테이너의 주축의 시작부분에 배치한다.
* `flex-end` : 자식들을 컨테이너 주축의 마지막 부분에 배치한다.
* `center` : 자식들을 컨테이너 주축의 중간에 배치한다.
* `space-between` : 컨테이너의 주축에서 자식 사이 사이 공간을 두고 배치한다.
* `space-around` : `space-between`과 비교했을 때, 자식 사이에 띄우게 되는 공간은 첫번째 자식의 시작 부분과 마지막 자식의 마지막 부분에 배치된다.
* `space-evenly` 

```js
import React from 'react';
import { View } from 'react-native';

const JustifyContentBasics = () => {
    return (
      // Try setting `justifyContent` to `center`.
      // Try setting `flexDirection` to `row`.
      <View style={{
        flex: 1,
        flexDirection: 'column',
        justifyContent: 'space-between',
      }}>
        <View style={{width: 50, height: 50, backgroundColor: 'powderblue'}} />
        <View style={{width: 50, height: 50, backgroundColor: 'skyblue'}} />
        <View style={{width: 50, height: 50, backgroundColor: 'steelblue'}} />
      </View>
    );
};

export default JustifyContentBasics;
```

![image](https://user-images.githubusercontent.com/41438361/91653781-96b05800-eade-11ea-9df0-b4649f18dfc9.png)

## Align Items

`alignItems` 는 컨테이너의 가로축을 따라 어떻게 자식들을 배치하는지를 정의한다. Align items는 `justifyContent`와 매우 비슷하지만 주축 대신에 가로축을 적용한다

* `stretch` (기본값) : 컨테이너의 자식들을 컨테이너의 가로축의 `height`에 맞추기 위해 확장한다.
* `flex-start` : 컨테이너의 자식들을 컨테이너의 가로축의 시작부분에 배치한다.
* `flex-end` : 컨테이너의 자식들을 컨테이너이 가로축의 마지막 부분에 배치한다.
* `center` : 컨테이너의 자식들을 컨테이너의 가로축의 중심부분에 배치한다.
* `baseline` : 일반적인 baseline을 따라 자식들을 배치한다. 각각의 자식들은 부모의 참조 baseline이 되도록 설정될 수 있다.

`stretch` 가 효과적으로 작동하기 위해, 자식들은 보조축에 대한 고정적인 수치를 가지면 안된다. 아래의 예제에서는 `alignItems: stretch`는 `width:50`이 지워지기 전까지 아무런 동작도 하지 않는다.

```js
import React from 'react';
import { View } from 'react-native';

const AlignItemsBasics = () => {
    return (
      // Try setting `alignItems` to 'flex-start'
      // Try setting `justifyContent` to `flex-end`.
      // Try setting `flexDirection` to `row`.
      <View style={{
        flex: 1,
        flexDirection: 'column',
        justifyContent: 'center',
        alignItems: 'stretch',
      }}>
        <View style={{width: 50, height: 50, backgroundColor: 'powderblue'}} />
        <View style={{height: 50, backgroundColor: 'skyblue'}} />
        <View style={{height: 100, backgroundColor: 'steelblue'}} />
      </View>
    );
};

export default AlignItemsBasics;
```

![image](https://user-images.githubusercontent.com/41438361/91659751-d857f780-eb0c-11ea-8239-6a4bb0deca83.png)

## Align Self

`alignSelf`는 `alignItems`와 같은 효과와 옵션을 가지고 있지만 컨테이너에 있는 자식들에 영향을 미치는 것 말고 이거를 특정 자식에 대해 property를 바꿀 수 있다. `alignSelf`는 `alignItems`로 부모의 옵션을 덮어씌운다.

![image](https://user-images.githubusercontent.com/41438361/91659797-1d7c2980-eb0d-11ea-8a5a-f955c5b7bf66.png)

## Align Content

alignContent는 가로축의 분포를 결정짓는다. 이거는 아이템들이 `flexWrap`을 사용하는 많은 줄들로 둘러쌓였을 경우 효과가 있다.

![image](https://user-images.githubusercontent.com/41438361/91659845-587e5d00-eb0d-11ea-8f36-ef147b76f7a2.png)

## Flex Wrap

flexWrap property는 컨테이너에 적용되며 자식이 컨테이너의 주축보다 사이즈가 커질경우를 컨트롤 한다. 디폴트로, 자식들은 한줄(요소들을 줄일 수 있다.)로 들어가도록 되어있다. 만약 wrapping이 되어있다면, 아이템들은 주축을 따라 여러줄로 구성될 수 있다.

줄들을 감쌀때, `alignContent`는 줄들이 컨테이너에 어떻게 위치하는지 사용될 수 있다. 

![image](https://user-images.githubusercontent.com/41438361/91659906-b743d680-eb0d-11ea-8d9c-90efbe7fc7dd.png)

## Flex Basis, Grow, and Shrink

## Width and Height

## Absolute & Relative Layout







