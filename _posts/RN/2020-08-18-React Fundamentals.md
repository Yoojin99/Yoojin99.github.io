---
title:  "React Native - React 기초"
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

React Native는 JavaScript로 사용자 인터페이스를 만드는 유명한 오픈 소스 라이브러리 React를 기반으로 동작한다. React Native를 하기 전에, React에 대해 이해하는 것이 도움이 된다.

React의 핵심 개념은 다음과 같다.

* Components
* JSX
* Props
* state

## Component

여기서 들 예시들은 고양이이다.

**Function Component Example**

```javascript
import React from 'react';
import { Text } from 'react-native';

const Cat = () => {
  return (
    <Text>Hello, I am your cat!</Text>
  );
}

export default Cat;
```

먼저 JavaScript의 `import`를 이용해서 React를 import하고 React Native의 `Text` Core component를 불러왔다.

그리고 component는 함수로 다음과 같이 작동한다.

`const Cat = () => {};`

Component를 청사진으로 생각할 수 있다. 함수 component가 리턴하는 것은 어떤 것이든 React 요소로 렌더링된다. 
React 요소는 어떤 것을 화면에서 보고 싶은 건지 구체화할 수 있게 해준다.

다시 위의 코드를 보면, `Cat` component는 `<Text>` 요소를 만들것이다. 함수 component는 JavaScript의 `export default`를 통해 export할 수 있다.

`return`문을 자세히 보자. `<Text>Hello, I am your cat!</Text>`은 일종의 JavaScript 문법을 이용하고 있다. 이것은 element를 작성하는 것을 편리하게 한다:JSX.


**Class Component Example**

```javascript
import React, { Component } from 'react';
import { Text } from 'react-native';

class Cat extends Component {
  render() {
    return (
      <Text>Hello, I am your cat!</Text>
    );
  }
}

export default Cat;
```

React에서 `Component`를 import 한다. 그리고 component는 `Component`를 연장한 class로 시작한다.

`class Cat extends Component {}`

Class component는 `render()` 함수를 가지고 있다. 이 안에서 리턴되는 것은 어떤 것이든 react element로 만들어진다.

함수 component와 마찬가지로, class component를 export 할 수 있다.

그리고 함수 component와 마찬가지로, class component 또한 export할 수 있다.

`return`문에 주목해보자. `<Text>Hello, I am your cat!</Text>`는 element들을 편리하게 작성할 수 있게 해주는 JavaScript 문법이다: JSX.

## JSX

리액트와 리액트 네이티브는 JSX, `<Text>Hello, I am your cat!</Text>`같은 element를 JavaScript로 쓸 수 있게 해주는 문법이다. JSX는 JavaScript이기 때문에, 다양한 변수들을 안에서 사용할 수 있다. 여기서는 cat의 이름을 `name`으로 선언해서 중괄호로 감싸 `<Text>`안에 넣고 있다.

```js
import React from 'react';
import {Text} from 'react-native';

const Cat = () => {
  const name = "Maru";
  return(
    <Text>Hello, I am {name}!</Text>
  );
}

export default Cat;
```

모든 JavaScript 표현은 `{getFullName("Rum", "Tum", "Tugger")}`처럼 함수를 실행하는 것과 같이 중괄호 안에서 작동한다. 

```js
import React from 'react';
import {Text} from 'react-native';

const getFullName = (firstName, secondName, thirdName) => {
  return firstName + " " + secondName + " " + thirdName;
}

const Cat = () => {
  return (
    <Text>
      Hello, I am {getFullName("Rum", "Tum", "Tugger")}!
    </Text>
  );
}
```

중괄호가 JSX안에 JS 기능성을 사용할 수 있게 포탈을 열어논 거라고 생각할 수 있다. JSX는 React 라이브러리에 포함되어 있기 때문에 `import React from 'react'`를 하지 않으면 작동하지 않는다.

## Custom Component

React는 Core component를 서로의 안에 중첨되어서 넣을 수 있게 해서 새로운 component를 만들 수 있게 한다. 이런 중첩가능하고, 재사용가능한 요소들은 React 패러다임의 핵심이다.

예를 들어, Text와 TextInput을 View안에 넣을 수 있고, React Native은 이것을 같이 렌더링할 것이다.

```js
import React from 'react';
import { Text, TextInput, View } from 'react-native';

const Cat = () => {
  return(
    <View>
      <Text>Hello, I am..</Text>
      <TextInput 
        style={{
          height:40,
          borderColor: 'gray',
          borderWidth: 1
        }}
        defaultValue="Name me!"
      />
    </View>
  )
}

export default Cat;
```

안드로이드에서는 주로 view들을 `LinearLayout`, `FrameLayout`, `RelativeLayout`등 안에 넣어서 view들의 자식들이 스크린상으로 어떻게 배치될 지 정의한다. RN에서는, `View`는 자식 레이아웃에 Flexbox를 사용한다.

또한 이런 요소들을 여러번, 여러 곳에 `<Cat>`을 쓰는 것을 반복해서 렌더링 할 수 있다.

```js
import React from 'react';
import { Text, TextInput, View } from 'react-native';

const Cat = () => {
  return (
    <View>
      <Text>I am also a cat!</Text>
    </View>
  );
}

const Cafe = () => {
  return(
    <View>
      <Text>Welcome!</Text>
      <Cat />
      <Cat />
      <Cat />
    </View>
  );
}

export default Cafe;
```

다른 component를 렌더링 하는 component들은 부모 component이다. 여기서, `Cafe`는 부모 component이고 각 `Cat`은 자식 component이다.

위처럼 카페에 원하는 만큼 고양이를 넣을 수 있다. 각 `<Cat>`은 **props**로 커스터마이징 할 수 있는 고유한 element를 렌더링한다.

## Props

Props는 "properties"의 줄임말이다. Props는 React component를 커스터마이징 할 수 있게 해준다. 아래의 코드에서는, 각 `<Cat>`에 다른 `name`을 줄 것이다.

```js
import React from 'react';
import { Text, View } from 'react-native';

const Cat = (props) => {
  return (
    <View>
      <Text>Hello, I am {props.name}!</Text>
    </View>
  );
}

const Cafe = () => {
  return (
    <View>
      <Cat name="Maru" />
      <Cat name="Jellylorum" />
      <Cat name="Spot" />
    </View>
  );
}

export default Cafe;
```

대부분의 RN Core component들은 prop으로 커스터마이징 될 수 있다. 예를 들어, Image를 이용할 때 source라는 이름의 prop을 전달할 수 있다.

```js
import React from 'react';
import { Text, View, Image } from 'react-native';

const CatApp = () => {
  return (
    <View>
      <Image
        source={{uri: "https://reactnative.dev/docs/assets/p_cat1.png"}}
        style={{width: 200, height: 200}}
      />
      <Text>Hello, I am your cat!</Text>
    </View>
  );
}

export default CatApp;
```

Image는 style을 포함해서 다양한 prop을 가지고 있다. 

styld의 가로, 세로를 감싸는 더블 중괄호를 주목해라. JSX에서는, JavaScript 값은 `{}`으로 감싸진다. 이거는 배열이나 숫자같은 걸 전달할 때 편하다. (`<Cat food={["fish", "kibble"] age={2}}>`) 그런데, JS 객체들도 마찬가지로 중괄호로 감싸진다 : `{width: 200, height: 200}`. 그래서 JS 객체를 JSX에서 전달하기 위해서는 객체를 **또 다른** 중괄호로 감싸야 한다. `{{width: 200, height: 200}}`

Text, Image, View 같은 Core Component 와 props를 이용해서 많은 것을 만들 수 있다. 하지만 상호작용가능한 것을 만드려면, state가 필요하다.

## State

Props를 어떻게 component들이 렌더링 될지를 구성하는데 쓴다면, state는 component의 데이터 저장공간과 같다. State은 사용자 인터렉션에 따라 바뀌는 데이터들을 다루는데 유용하다. State는 component에 메모리를 부여한다!

일반적으로, props는 component가 렌더링 되었을 때를 구성하는데 쓰고, State는 시간에 따라 바뀌는 component 데이터를 유지하기 위해 사용해라.

밑에 나오는 예제는 고양이 카페에서 두 배고픈 고양이들이 밥 먹기를 기다리는 예제이다. 그들의 이름과는 다르게, 배고픔은 시간에 따라 바뀐다는 것을 우리는 알 수 있고, 이것을 state로 저장할 수 있다. 고양이에게 밥을 먹이기 위해서는, 버튼을 클릭해서 그들의 state를 업데이트 한다.

### 1. state with Function Component

React의 `useState` Hook을 불러와서 state를 component에 추가할 수 있다. Hook는 React 피쳐들에 "Hook into"할 수 있게 해주는 함수같은 것이다. 예를 들어, `useState`는 state를 function component에 추가해줄 수 있는 Hook이다. 

```js
import React, {useState} from 'react';
import {Button, Text, View} from 'react-native';

const Cat = (props) => {
  const [isHungry, setIsHungry] = useState(true);
  
  return(
    <View>
      <Text>
        I am {props.name}, and I am {isHungry? "hungry" : "full"}!
      </Text>
      <Button
        onPress={()=>{setIsHungry(false);}}
        disabled={!isHungry}
        title={isHungry? "Pour me some milk, please!" : "Thank you!"}
      />
    </View>
  );
}

const Cafe = ()=>{
  return(
    <>
      <Cat name="Nabi" />
      <Cat name="Spot" />
    </>
  );
}

export default Cafe;
```

먼저, `uesState`를 함수 안에서 불러서 component의 state를 선언한다. 이 예제에서는 `useState`는 `isHungry` state 변수를 생성했다.

`useState`를 이용해서 문자열, 숫자, Boolean, 배열, 객체와 같이 다양한 데이터를 추적할 수 있다. 예를 들어, 고양이를 몇 번 쓰다듬었는지는 `const [timesPetted, setTimesPetted] = useState(0)`를 이용하면 된다.

`useState`를 부르는 것은 두가지 일을 한다:

* 초기값과 함께 "state variable"을 생성한다-여기서는 state varialbe은 `isHungry`이고 초기값은 `true`이다.
* state 변수의 값을 설정하는 함수를 만든다-`setIsHungry`

어떤 이름을 사용하는지는 상관없다. 하지만 패턴은 다음과 같이 존재한다. `[<getter>, <setter>] = useState(<initialValue>)`

그리고 Button Core Component를 추가하고 `onPress` prp을 준다. 이러면 누가 버튼을 눌렀을때, `onPress`가 실행되며 `setIsHungry(false)`를 실행시킨다. 이거는 `isHungry` 상태 변수를 `false`로 설정한다. `isHungry`가 false이면, `Button`의 `disabled` porp이 `true`로 바뀌며 `title`또한 바뀐다.

여기서는 근데 `isHungry`가 const인데, 재할당할 수 있다는 것을 화인할 수 있을 것이다. state-setting 함수인 `setIsHungry`가 불려졌을 때, 이 component는 다시 렌더링 된다. 이때 `Cat` 함수는 다시 실행되고 이때 `useState`는 `isHungry`의 다음 값을 줄 것이다.

마지막으로 고양이를 `Cafe` component에 넣는다. `<>`와 `</>`와 같은 JSX는 fragment이다. 인접한 JSX 요소들은 무조건 닫히는 tag로 감싸져야 한다. Fragment는 `View`와 같은 걸로 불필요하게 감싸지 않고 중첩을 가능하게 해준다.

### 2. State with Class Components

```js
import React, { Component } from "react";
import { Button, Text, View } from "react-native";

class Cat extends Component{
  state = { isHungry: true };
  
  render(props){
    return(
       <View>
        <Text>
          I am {this.props.name}, and I am
          {this.state.isHungry ? " hungry" : " full"}!
        </Text>
        <Button
          onPress={() => {
            this.setState({ isHungry: false });
          }}
          disabled={!this.state.isHungry}
          title={
            this.state.isHungry ? "Pour me some milk, please!" : "Thank you!"
          }
        />
      </View>
    );
  }
}

class Cafe extends Component {
  render() {
    return (
      <>
        <Cat name="Munkustrap" />
        <Cat name="Spot" />
      </>
    );
  }
}

export default  Cafe;
```

클래스 component에서, 상태는 state 객체에 저장된다.

props를 `this.props`로 접근하는 것과 같이, `this.state`로 component 안에 있는 객체로 접근할 수 있다.

그리고 키와 값 쌍으로 존재하는 상태 객체 안에 각각의 값을 설정한다. 그리고 `this.setState()`로 새로운 값을 설정한다. `this.state.hunger = false`로 바로 새로운 값을 할당하지 말아라. `this.setState()`를 호출하는 것은 React가 rerendering을 실행시키는 state에 생기는 변화를 추적할 수 있게 해준다. state를 직접 수저하는 것은 반응성을 깨트리는 것이다.
