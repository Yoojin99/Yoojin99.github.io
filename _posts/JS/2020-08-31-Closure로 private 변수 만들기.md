---
title:  "JavaScript - Closure로 private member 만들기"
excerpt: ""

categories:
  - JS
tags:
  -
last_modified_at: 
---

1. 외부로부터의 접근 제어
2. 전역스코프의 변수 최소화

```js
var createCar = function(f, p){
  var fuel = f;
  var power = p;
  var total = 0;
  
  return{
    run: function(km){
      var wasteFuel = km/ power;
    }
  }

}
```

외부에는 run만 노출됨.

1. 함수에서 지역변수 및 내부함수 등을 생성한다.
2. 외부에 노출시키고자 하는 멤버들로 구성된 객체를 리턴한다.

이러면 리턴 객체에 포함 안된애들은 private, 객체에 포함된 애들은 public이다.

return function : 최초 선언시의 정보를 유지한다. 그래서 함수 안의 함수로 리턴시키면 그 리턴된 애의 밖에 있는 애는 모를 수밖에 없다.
