---
title:  "React - LifeCycle API"
excerpt: ""

toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
header:
  teaser: 
  
  
categories:
  - React
  
tags:
  - 공부
  - React
  
last_modified_at: 
---

## LifeCycle API

생명주기.

Component가

1. 나타날 때
2. 업데이트 될 때
3. 사라질 때

중간중간에서 사용하고 싶을 때 사용한다.

종류가 많다.

constructor -> render -> 등등,,, 하나하나가 다 함수이다.

## Mounting

브라우저 상에 나타날 때.

constructor : 만들어질 때 가장 처음 됨. state

getDerivedStateFromProps

render 

componentDidMount : 외부 라이브러리에서 특정 DOM에 그려달라, API 요청같은거 할 때? 우리가 만든 컴포넌트가 브라우저에 나타난 시점에 어떤걸 하냐.

## Updating

getDerivedStateFromProps

shouldComponentUpdate : 렌더링 작업이 불필요해질 때(업데이트 된 것만 바뀌어야 한다.) Virtual DOM에 그리는 것도 아끼기 위해. true나 false 값 반환.true이면 렌더링 하고 false이면 렌더링 안한다.

render

getSnapshotBeforeUpdate : 브라우저에 반영되기 직전에 호출되는 거.

componentDidUpdate : 작업을 마치고 업데이트 되었을 떄 호출. 이전의 상태와 지금의 상태가 바뀌었을 때 사용 등등

## Unmounting 

componentWillUnmount : 설정한 리스너 없애기

브라우저 상에서 사라질 때.



# 2.

## 컴포넌트 초기 생성

컴포넌트가 브라우저에 나타나기 전, 후에 호출되는 API가 있다.


### constructor

```
constructor(props){
  super(props);
}
```

###  componentDidMount

```js
componentDidMount(){
  //외부 라이브러리 연동 : D3, masonry, etc
  //컴포넌트에서 필요한 데이터 요청: Ajax, GraphQL, etc
  //DOM에 관련된 작업 : 스크롤 설정, 크기 읽어오기 등
}
```
컴포넌트가 화면에 나타나게 됐을 때 호출된다.

## 컴포넌트 업데이트

### getDerivedStateFromProps()

```js
static getDerivedStateFromProps(nextProps, prevState){
  //여기서는 setState를 하는 것이 아니라 특정 props가 바뀔 때 설정하고 싶은 state 값을 리턴하는 형태로 사용.
  /*
  if(nextProps.value !== prevState.value){
    return{ value: nextProps.value };
  }
  
  return null; //null을 리턴하면 따로 업데이트 할 것은 없다라는 의미
  */
}
```

### shouldComponenetUpdate

```js
shouldComponentUpdate(nextProps, nextState){
  //return false 하면 업데이트를 안함
  // return this.props.checked !==nextProps.checked
  
  return true; //업데이트를 함.
}
```

컴포넌트를 최적화하는 작업을 할 때 유용하다. 리액트에서는 변화가 발생하는 부분만 업데이틀 해줘서 성능이 좋은데, 변화가 발생한 부분만 감지해내기 위해서는 Virtual Dom에 그려줘야 한다.

현재 컴포넌트의 상태가 업데이트 되지 않아도, 부모 컴포넌트가 리렌더링 되면, 자식 컴포넌트들도 렌더링된다. 이 "렌더링"된다는 것은 render()함수를 호출한다는 뜻이다.

### getSnapshotBeforeUpdate()

이 API가 발생하는 시점은

render()가 되고 난 후, 실제 DOM에 변화 발생 전이다. 

DOM 변화가 일어나기 직전의 DOM 상태를 가져오고, 리턴하는 값은 componentDidUpdate의 3번쨰 파라미터로 받아올 수 있다.

```js
getSnapshotBeforeUpdate(prevProps, prevState){
  //
}
```


### componentWillUnmount

```js
componentWillUnmount() {
    console.log('Good Bye');
  }
```

해당 컴포넌트가 없어질 때 실행된다.

### componentDidCatch

원래는 render 함수에서 에러가 발생하면 리액트 앱이 crash 된다. 이때 이걸 사용하면 된다.

```js
componentDidCatch(error, info){
//error는 무슨 에러가 났는지 확인할 수 있다.
  this.setState({
    error : true
  })
}
```

에러가 발생할 수 있는 컴포넌트의 부모 컴포넌트에서 사용해야 한다.


App.js

```js
import React, { Component } from 'react';
import MyComponent from './MyComponent';

class App extends Component {
  state = {
    counter: 1
  };

  constructor(props) {
    super(props); //Component를 상속하는데 컴포넌트의 생성자를 먼저 호출해주는 거.
    console.log('constructor');
  }

  componentDidMount() {
    console.log('componentDidMount');
  }

  handleClick = () => {
    this.setState({
      counter: this.state.counter + 1
    });
  };

  render() {
    return (
      <div>
        {this.state.counter < 10 && <MyComponent value={this.state.counter} />}
        <button onClick={this.handleClick}>Click Me</button>
      </div>
    );
  }
}

export default App;
```

MyComponent.js

```js
import React, { Component } from 'react';

class MyComponent extends Component {
  state = {
    value: 0
  };

  static getDerivedStateFromProps(nextProps, prevState) {
    if (prevState.value !== nextProps.value) {
      return {
        value: nextProps.value
      };
    }
    return null;
  }

  shouldComponentUpdate(nextProps, nextState) {
    if (nextProps.value === 10) return false;
    return true;
  }

  componentDidUpdate(prevProps, prevState) {
    if (this.props.value !== prevProps.value) {
      console.log('value 값이 바뀌었다!', this.props.value);
    }
  }

  componentWillUnmount() {
    console.log('Good Bye');
  }

  render() {
    return (
      <div>
        {/*this.props.missing.something */}
        <p>props: {this.props.value}</p>
        <p>state: {this.state.value}</p>
      </div>
    );
  }
}

export default MyComponent;
```
