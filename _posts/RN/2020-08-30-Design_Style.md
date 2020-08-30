---
title:  "React Native - Design - 스타일"
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

RN에서는, JavaScript로 앱을 스타일링한다. 모든 core component들은 `style`이란 이름의 prop을 사용한다.
스타일 이름과 값들은 이름이 camel casing으로 써져있는 것만 제외하고 주로 CSS가 web에서 동작하는 것과 비슷하다.
예를 들어 `background-color`보다는 `backgroundColor`를 사용한다.

`style` prop은 일반적인 옛날 JavaScript 객체이다. 또한 style들을 배열로 패스할 수도 있다 - 배열의 마지막 스타일은 우선적이므로 이걸로 
스타일을 지정할 수 있다.

component가 복잡해지므로 여러 스타일을 한 곳에서 정의하기 위해 `StyleSheet.create`를 사용하는 것이 깔끔해보인다.

```js
import React from 'react';
import { StyleSheet, Text, View } from 'react-native';

const LotsOfStyles = () => {
  return(
    <View style={styles.container}>
        <Text style={styles.red}>just red</Text>
        <Text style={styles.bigBlue}>just bigBlue</Text>
        <Text style={[styles.bigBlue, styles.red]}>bigBlue, then red</Text>
        <Text style={[styles.red, styles.bigBlue]}>red, then bigBlue</Text>
      </View>
  );
}

const styles = StyleSheet.create({
  container: {
    marginTop: 50,
  },
  bigBlue: {
    color: 'blue',
    fontWeight: 'bold',
    fontSize: 30,
  },
  red: {
    color: 'red',
  }
});

export default LotsOfStyles;
```

일반적인 패턴은 하위 component들을 차례로 스타일링하는데 사용되는 `style` porp을 받아들이게 하는 것이다.
CSS에서 했던 것처럼 이런 스타일을 "연쇄적으로" 만드는데 이걸 사용할 수 있다.

텍스트 스타일을 커스터마이징 하는데 많은 방법이 있다. 그건 [여기서](https://reactnative.dev/docs/text)확인 가능하다.



