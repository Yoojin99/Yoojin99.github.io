---
title:  "React Native - 플랫폼 특정적인 코드"
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

만약 cross-platform 앱을 만들고 있다면, 코드를 가능한 많이 재사용하고 싶을 것이다. 안드로이드와 iOS를 위한 시각적인 component를 구현하고 싶은데
코드가 다를때 문제 상황이 생기게 된다.

RN은 코드를 구성하고 플랫폼 별로 나누는데 두가지 방법을 제시한다.

*  `Platform` 모듈을 사용한다.
* 플랫폼 특정적인 파일 extension을 사용한다.

어떤 component들은 특정 플랫폼에서만 동작하는 property를 가질 수 있다. 이 모든 prop들은 `@platform`으로 주석이 달아져 있고 웹사이트에서 옆에 조그만 배지를 옆에 표시하게 된다.

## 플랫폼 모듈

RN은 현재 앱이 동작하고 있는 플랫폼이 무엇인지 탐지하는 모듈을 제공한다. 이 탐지 로직을 이용해서 플래폼-특정적인 코드를 구현할 수 있다.
오직 작은 부분이 플랫폼 특정적일 때 이걸 활용하라.

```js
import { Platform, StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  height: Platform.OS === 'ios' ? 200 : 100
});
```

`Platform.OS`는 iOS에서는 `ios`가 될 것이고 안드로이드 상에서는 `android`가 될 것이다.

또한 `'ios' | 'android' | 'native' | 'default'`중에 오직 하나만 키로 가지고 있는 객체를 가지고 현재 동작중인 플랫폼에 가장 맞는 값을 리턴하는  `Platform.select` 메서드가 있다.
즉 만약 폰에서 작동중이라면, `ios`랑 `android` 키가 선호될 것이다. 만약 특정지어지지 않는다면 `naitve` 키 다음으로 `default` 키가 사용될 것이다.

```js
import {Platform, StyleSheet} from 'react-native';

const styles = StyleSheet.create({
  container: {
    flex:1,
    ...Platform.select({
      ios: {
        backgroundColor: 'red'
      },
      android: {
        backgroundColor: 'green'
      },
      default: {
        //다른 플랫폼. web같은거
        backgroundColor: 'blue'
      }
    })
  }
});
```

이거는 모든 플랫폼에서 `flex: 1`을 가지고 있을 것이고 iOS에서는 빨간색 배경, 안드로이드에서는 초록 배경화면, 그리고 다른 플랫폼에서는 파란색 배경을 가지게 될 것이다.

`any` 값을 가질 수 있게 된 때부터, 아래처럼 플랫폼 특정적인 component를 반환하는데 사요ㅇ할 수도 있다.

```js
const Component = Platform.select({
  ios: () => require('ComponentIOS'),
  android: () => require('ComponentAndroid')
})();

<Component />;
```

```js
const Component = Platform.select({
  native: () => require('ComponentForNative'),
  default: () => require('ComponentForWeb')
})();

<Component />;
```

### 안드로이드 버전 탐지하기

안드로이드에서, `Platform` 모듈은 현재 앱이 동작중인 안드로이드 플랫폼의 버전을 탐지할 수 있다.

```js
import {Platform} from 'react-native;

if(Platform.Version === 25){
  console.log('Running on Nougat!');
}
```

### iOS 버전 탐지하기

iOS에선, `Version`은 현재 동작하는 시스템의 버전을 나타내는 문자열인 `-[UIDevice systemVersion]`의 결과이다. 아래의 예제에서 시스템의 버전은 "10.3"이다.
예를 들어, iOS에서 메인 버전 번호를 찾으려면 아래와 같이 하면 된다.

```js
import {Platform} from 'react-native';

const majorVersionIOS = parseInt(Platform.Version, 10);
if(majorVersionIOS <= 9){
  console.log('Wrok around a change in behaviour');
}
```

## 플랫폼-특정 extensions

만약 플랫폼 특정적인 코드가 더 복잡하다면, 이 코드를 별도의 파일로 따로 저장하는 것을 고려하는 것이 좋다.
RN은 `.ios.`나 `.android.` extension을 가지고 있는 파일을 찾을 것이고 다른 component에서 요구될 때 관련있는 플랫폼 파일을 로드할 것이다.

예를 들어, 프로젝트에 다음과 같은 파일들이 있다고 해보자.

```
BigButton.ios.js
BigButton.android.js
```

그러면 아래와 같이 component를 요구할 수 있다.

```js
import BigButton from './BigButton';
```

RN은 자동으로 현재 동작하고 있는 플랫폼에 맞는 파일을 선택할 것이다.

## Native-특정적인 extensions (즉 NodeJS와 Web과 코드를 공유하는 것)

모듈이 NodeJS/Web 과 RN 사이에 공유되어야 하지만 Android/iOS 간 차이가 없을 때 `.native.js` extension을 사용할 수 있다.
이거는 RN과 React 사이에 공유되어야 하는 코드가 있는 프로젝트에서 유용하게 사용될 수 있다.

예를 들어, 프로젝트에 아래와 같은 파일들이 있다고 해보자.

```
Container.js #Webpack, Rollup이나 다른 Web bundler 의해 picked up됨
Container.native.js # 안드로이드, iOS를 모두 위한 RN bundler에 의해 pickedup
```

`.native` extension 없이고 쓸 수 있다.

```js
import Container from './Container';
```

프로  :사용되지 않는 코드가 production bundle에서 포함되지 않게 해서 최종 번들 사이즈를 줄이기 위해 Web bundler가 `.native.js`를 무시하게끔 구성해라.
