---
title:  "React Native - React 튜토리얼"
excerpt: "React Tutorial Korean - 리액트 튜토리얼 한국"

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

RN을 공부하기 전에 javascript 강의를 듣는 것이 좋을 것 같아 javascript 기본 강의를 들었다.
이제 ReactNative의 공식 문서를 보고 공부하고, 또 그 전에 ReactNative에 대해서도 좀 공부를 해야 할 것 같다.
먼저 React Native 튜토리얼 문서를 보고 문제 없이 잘 이해되면 굳이 React 강의를 보지 않고 react native 공식 문서들을 
보고 공부하려고 한다.

아래는 내가 [React Native Tutorial 공식문서](https://reactnative.dev/docs/tutorial)를 보고 임의로 해석한 것이다.

## Learn the Basics

React Native는 React와 비슷하지만, web component 대신에 building block과 같은 web component를 이용한다. 그래서
React Native 앱의 기본적인 구조를 이해하려면, JSX, component, state, prop과 같은 기본적인 몇 React 개념들을 알고 있어야 한다.
이미 React를 알고 있다해도 native component와 같이 몇 React-Native-specific 요소들을 배워야 한다. 이제 밑에 나올 튜토리얼은
리액트 경험과 상관 없이 진행할 수 있다.

## Hello World!

전통에 따라, 첫 빌드는 "Hello, world!"를 말하는 앱을 출력해 볼 것이다. ^^

```js
import React from 'react';
import {Text, View} from 'react-native';

const HelloWorldApp = () => {
  return{
    <View
      style={{
        flex: 1,
        justifyContent: "center", //vertical
        alignItems: "center" //horizontal
      }}>
      <Text>Hello, world!</Text>
    </View>
  }
}

export default HelloWorldApp;
```

이 코드를 `App.js` 파일에 위 코드를 붙여서 가지고 있는 기기로 확인할 수도 있고, 홈페이지 상의 web simulator를 통해 코드를 변경해서 확인도 가능하다.

## 어떻게 동작을 하는 것인가?

1. 먼저, 후에 각 플랫폼에서 native component로 바뀔 `JSX`를 이용하기 위해 `React`를 import 해야 한다.
2. 2번째 줄에서, `react-native`에서 `Text`와 `View` 요소를 import 했다.

그리고 다음에 나오는 `HelloWordlApp` 함수는 `functional component`이고 React가 web에서 동작하는 것과 똑같이 동작한다. 이 함수는 몇개의 style와 `Text`를 가진 `View` 요소를 리턴한다.

`Text` 요소는 text를 렌더링 하는 것을 허용하고, `View` 요소는 컨테이너를 렌더링 한다.
이 컨테이너는여러 스타일들이 적용되어 있는 것을 확인할 수 있다.

첫번째 스타일은 `flex: 1`이다. `flex` prop은 주축(세로축)을 따라 허용된 공간에 어떻게 아이템을 "fill"할지 정의한다. 지금 위에 코드에는 하나의 컨테이너밖에 없기 때문에, 부모 요소에서 가능한 모든 공간을 다 차지할 것이다. 

다음 스타일은 `justifyContent: "center"`이다. 이건 컨테이너의 자식들을 컨테이너의 주축의 가운데에 배열한다. 그리고 `alignItems: "center"`는 컨테이너의 가로축의 가운데에 자식들을 배치한다.

여기에 있는 것들 중에는 JavaScript 처럼 보이지 않을 수도 있다. 하지만 패닉하지 마라. 이게 미래가 될 것이다. ^^

먼저, ES2015(ES6라고 알려져 있다) 는 현재 공식 표준의 일부가 된 JavaScript의 개선점들이다. 하지만 모든 브라우저에서 지원되는 것은 아니라 web 개발쪽에서는 아직 사용되고 있지 않다. React Native은 ES2015를 지원하기 때문에, 이런 요소들을 호환성을 걱정하지 않고 사용하면 된다. 위의 예시에 있는 `import`, `from`, `class`, `extend`와 같은 요소들은 ES2015 feature들이다. 만약 ES2015에 익숙하지 않다면, 이 튜토리얼 코드를 보며 배울 수 있을 것이다. 

이 코드 예시에서 또 특이한 점은 바로 `<View><Text>Hello world!</Text><View>`이다. 이건 JSX - XML을 JavaScript로 임베딩하기 위한 문법이다. 많은 프레임워크는 markup language 안에서 코드를 임베딩 할 수 있게 해주는 특화된 템플릿 언어를 사용한다. React에서는 반대로 작용한다. JSX는 코드 안에서 markup language를 사용할 수 있게 해준다. `<div>`나 `<span>`대신에 React 요소를 사용하는 것을 제외하고, web에서는 HTML로 보인다. 이 경우에, `<Text>`는 텍스트를 보여주는 Core Component이고 `View`는 `<div>`나 `<span>`와 같다.

*functional component란? - React 공식 홈페이지에서 참조*

### Function와 Class component 

요소를 정의하기 위한 가장 간단한 방법은 JavaScript 함수를 작성하는 것이다.

```javascript
function Welcom(props){
  return <h1>Hello, {props.name}</h1>;
}
```

이 함수는 하나의 "props"(properties를 타나내는 거) 객체 변수를 데이터와 함께 받아 React 요소를 리턴했기 때문에 React component라고 할 수 있다. 우리는 얘네들이 말 그대로 JavaScript 함수이기 때문에 이런 요소들을 "function component"라고 부른다.

또한 ES6 class를 이용해서 요소를 정의할 수도 있다.

```javascript
class Welcom extends React.Component{
  render(){
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

위의 두 요소는 React의 관점에서 같은 요소이다.

## Component

그래서 결론 적으로 이 코드는 `HelloWorldApp`, 새로운 `Component`를 정의하고 있다. React Native App을 개발할 때, 많은 요소들을 만들게 될 것이다. 화면에서 보는 것은 어떤 종류든간에 다 요소에 속한다.

## Props

대부분의 요소들은 다른 파라미터로 생성될 때 커스터마이징 할 수 있다. 이런 생성 파라미터를 props라고 부른다.

사용자 지정의 요소도도 `props`를 이용할 수 있다. 이거는 앱에서 다양하게 쓰이는 특정 요소를 각 사용되는 곳에서 미묘하게 다른 특성을 갖게끔 할 수 있다. 함수 요소 안의 `props.{Name}`나 `this.props.{Name}`을 보도록 하겠다.

```javascript
import React from 'react';
import { Text, View, StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  center: {
    alignItems: 'center'
  }
})

const Greeting = (props) => {
  return (
    <View style={styles.center}>
      <Text>Hello {props.name}!</Text>
    </View>
  );
}

const LotsOfGreetings = () => {
  return (
    <View style={[styles.center, {top: 50}]}>
      <Greeting name='Rexxar' />
      <Greeting name='Jaina' />
      <Greeting name='Valeera' />
    </View>
  );
}

export default LotsOfGreetings;
```

`name`을 prop으로 사용하는 것을 통해 `Greeting` 요소를 커스터마이징할 수 있었고, 이 요소들을 재사용하면서 사용할 수 있었다. 이 예제는 또한 JSX의 `Greeting` 요소를 사용한다.

여기서 새로운 점은 `View` 요소이다. `View`는 스타일과 레이아웃을 위해 다른 요소들의 컨테이너로 유용하게 사용된다.

`props`와 기본 Text, Image, 그리고 View 요소로, 한정된 스크린을 다양하게 꾸밀 수 있다. 앱이 시간에 따라 바뀌는 것을 배우기 위해, State를 알아야 한다.

## State

오직 읽기만 가능하고 수정이 불가한 props와 달리, `state`는 React component들이 시간에 따라 유저의 행동, 네트워크 응답이나 다른 것들에 반응하기 위해 바뀌는 것을 허용한다.

**React에서 state와 prop의 차이는 무엇일까?**

React 요소에서, prop은 우리가 부모 요소에서 자식 요소로 전달하는 변수이다. 비슷하게, state도 변수이지만, parameter로 전달되지 않고, 내부에서 요소를 초기화 시키고 관리한다.

**React와 React Native에서 state를 다루는데 차이점이 있나?**

차이가 없다. ^^ class와 함수 요소 모두에서 hook을 이용해서 state를 이용할 수 있다.

밑에는 class를 이용해서 클릭한 횟수를 보여주는 코드 예제이다.

```javascript
import React, { Component } from 'react'
import {
  StyleSheet,
  TouchableOpacity,
  Text,
  View,
} from 'react-native'

class App extends Component {
  state = {
    count: 0
  }

  onPress = () => {
    this.setState({
      count: this.state.count + 1
    })
  }

 render() {
    return (
      <View style={styles.container}>
        <TouchableOpacity
         style={styles.button}
         onPress={this.onPress}
        >
         <Text>Click me</Text>
        </TouchableOpacity>
        <View>
          <Text>
            You clicked { this.state.count } times
          </Text>
        </View>
      </View>
    )
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  button: {
    alignItems: 'center',
    backgroundColor: '#DDDDDD',
    padding: 10,
    marginBottom: 10
  }
})

export default App;

```






