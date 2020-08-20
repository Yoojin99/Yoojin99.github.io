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




