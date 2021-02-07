---
title:  "JavaScript - binding과 call, apply, bind의 차이"
excerpt: ""

categories:
  - js
tags:
  - binding
  - call
  - apply
  - bind
last_modified_at: 2020-10-19
---

javascript의 함수는 각자 자신만의 this를 정의한다. 예를 들어 다음과 같은 함수가 있다고 해보자.

```js
const say = function() {
  console.log(this); // 이 this는 무엇일까?
  console.log("Hello, my name is " + this.name);
}

say();
```

위를 실행하면 this는 window임을 알 수 있다. 기본적으로 this는 window이다. 하지만 이 this는 바뀔 수도 있는데,
객체 내부, 객체 메서드 호출시, 생성자 new 호출시, 명시적 bind시에 따라 바뀐다.

하여간 이 say 함수에서 window 객체를 사용하고 싶지 않다. this를 그때그때 알맞은 객체로 바꿔서 this에 따라 인사말이 
바뀌도록 하는 것이다. 이것이 this의 binding이다. 명시적으로 this가 window가 아닌 객체로 바꿔주는 함수가 call, apply, bind이다.

## call 과 apply

위의 say 함수의 this를 바꾸고 싶다면 물론 this를 대체할 객체가 있어야 한다.

```js
const obj = { name: 'Tom' };

const say = function(city) {
  console.log(`Hi, my name is ${this.name}, I live in ${city}`);
}

say("seoul"); // Hi, my name is , I live in seoul
say.call(obj, "seoul"); // Hi, my name is Tom, I live in seoul
say.apply(obj, ["seoul"]); // Hi, my name is Tom, I live in seoul
```

`call`과 `apply`는 함수를 호출하는 함수이다. 그냥 실행하지 않고 첫 번째 인자에 this로 만들어버리고 싶은
객체를 넘겨서 this를 바꿔주고 실행시킨 것이다.

`call`과 `apply`의 차이점은, 첫 번째 인자(this로 세팅할 값)을 제외하고 실제 함수(say)에 필요한 parameter를 입력하는 방식이다.
`call`과 다르게 `apply`는 두 번째 인자부터 모두 배열에 넣어야 한다.

## bind

```js
const obj = { name: 'Tom' };

const say = function(city) {
  console.log(`Hi, my name is ${this.name}, I live in ${city}`);
}

const boundSay = say.bind(obj);
boundSay("seoul"); // Hi, my name is Tom, I live in seoul
```

`bind` 함수가 `call`, `apply`와 다른 점은 함수를 실행하지 않는다느 점이다. 대신 bound 함수를 리턴한다.

이 bound 함수(boundSay)는 이제부터 this를 obj로 갖고 있기 때문에 나중에 사용해도 된다. bind에 사용하는 나머지 rest 파라미터는 call과 apply와 동일하다.
