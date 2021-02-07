---
title:  "React Native - Text Input 다루기"
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
 
TextInput은 사용자가 텍스트를 칠 수 있게 해주는 Core component이다. 매번 텍스트가 바뀌었을 때마다 호출되는 
`onChangeText` prop과 매번 text가 submit되었을 때마다 호출되는 `onSumbitEditing` prop을 가지고 있다.

예를 들어, 유저가 타이핑 한 거를 다른 언어로 번역하려고 해본다고 하자. 이 새로운 언어에서는, 하나 하나의 단어가 
🍕로 쓰인다. 그래서 "Hello there Bob"은 "🍕🍕🍕"으로 번역될 것이다.

```js
import React, { useState } from 'react';
import { Text, TextInput, View } from 'react-native';

const PizzaTranslator = () =>{
  const [text, setText] = useState('');
  return(
    <View style={{padding: 10}}>
      <TextInput
        style={{height: 40}}
        placeholder="Type here to translate!"
        onChangeText={text=>setText(text)}
        defaultValue={text}
      />
      <Text style={{padding: 10, fontSize: 42}}>
        {text.split(' ').map((word)=> word && '🍕').join(' ')}
      </Text>
    </View>
  );
}

export default PizzaTranslator;
```

이 예제에서 우리는 `text`를 state에 저장한다. (시간에 따라 바뀌기 때문)

Text input을 가지고 굉장히 많은 것들을 할 수 있다.





