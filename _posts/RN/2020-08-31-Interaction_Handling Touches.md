---
title:  "React Native - Design - Images"
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

유저들은 주로 모바일 앱을 터치하며 상호작용한다. 버튼에 탭하거나, 리스트를 스크롤하거나, 지도를 줌하는것과 같이 여러 제스처의 조합을 사용한다.
RN은 gesture responder system같이 유명한 제스처를 다룰 수 있는 component들을 제공하지만, 아마 가장 관심있어할 것은 기본 버튼이라고 생각한다.

## Displaying a basic button

버튼은 모든 플랫폼에서 잘 렌더링 되는 기본 버튼 component를 제공한다. 버튼을 출력하는 간단한 예제는 아래와 같다.

```js
<Button
  onPress={()=>{
    alert('You tapped the button!');
  }}
  title="Press me"
/>
```

이거는 iOS에서는 파랑 라벨, 안드로이드에서는 밝은 텍스트와 파란색 사각형을 렌더링 할 것이다. 
이 버튼을 클릭하면 "onPress" 함수가 호출되며 팝업을 띄울 것이다. 만약 원한다면 "color" prop을 설정해서 버튼의 색을 바꿀 수도 있다.

기본 코드는 아래와 같다.

```js
import React, {Component} from 'react';
import {Button, StyleSheet, View} from 'react-native';

export default class ButtonBasics extends Component{
  _onPressButton(){
    alert('You tapped the button')
  }
  
  render(){
    return(
      <View style={styles.container}>
        <View style={style.buttonContainer}>
          <Button
            onPress={this._onPressButton}
            title="Press Me"
          />
        </View>
        <View style={styles.buttonContainer}>
          <Button
            onPress={this._onPressButton}
            title="Press Me"
            color="#841584"
          />
        </View>
        <View style={styles.alternativeLayoutButtonContainer}>
          <Button
            onPress={this._onPressButton}
            title="This looks great!"
          />
          <Button
            onPress={this._onPressButton}
            title="OK!"
            color="#841584"
          />
        </View>
      </View>
    );
  }
}

const styles = StyleSheet.create({
  container: {
   flex: 1,
   justifyContent: 'center',
  },
  buttonContainer: {
    margin: 20
  },
  alternativeLayoutButtonContainer: {
    margin: 20,
    flexDirection: 'row',
    justifyContent: 'space-between'
  }
});
```

## Touchables

만약 기본 버튼이 앱에서 잘 동작하지 않는다면 RN에서 제공하는 "Touchable" component를 이용해서 자신만의 버튼을 만들수도 있다.
"Touchable" component는 탭하는 제스처를 캐치할 수 있으며 제스처를 인식했을 때 피드백을 할 수도 있다. 
이런 component들은 디폴트 스타일링을 제공하지 않기 때문에 예쁘게 보여지게 하기 위해서는 작업을 좀 해야 한다.

"Touchable" component는 어떤 종류의 피드백을 주고 싶은지에 따라 달렸다.

* 주로, TouchableHighlight는 버튼이나 웹에 있는 링크에 사용할 수 있다. 사용자가 버튼을 누르면 view의 바탕이 어두워진다.
* 안드로이드에서 TouchableNativeFeedback을 해서 사용자가 터치하면 표현이 파동이 이는 것처럼 설정할 수도 있다.
* TouchableOpacity는 버튼의 불투명성을 줄임으로써 피드백을 줄 수 있다.
* 만약 탭 제스처를 피드백을 주지 않고 핸들링하고 싶으면 TouchableWithoutFeedback을 사용해라.

일정 시간동안 탭한 상태에서 유지하는 경우를 탐지하고 싶을 수도 있다. 이렇게 길게 누르는 것은 "Touchable" component의 `onLongPress` props로 전달해서 다룰 수 있다.

```js
import React, { Component } from 'react';
import { Platform, StyleSheet, Text, TouchableHighlight, TouchableOpacity, TouchableNativeFeedback, TouchableWithoutFeedback, View } from 'react-native';

export default class Touchables extends Component {
  _onPressButton() {
    alert('You tapped the button!')
  }

  _onLongPressButton() {
    alert('You long-pressed the button!')
  }


  render() {
    return (
      <View style={styles.container}>
        <TouchableHighlight onPress={this._onPressButton} underlayColor="white">
          <View style={styles.button}>
            <Text style={styles.buttonText}>TouchableHighlight</Text>
          </View>
        </TouchableHighlight>
        <TouchableOpacity onPress={this._onPressButton}>
          <View style={styles.button}>
            <Text style={styles.buttonText}>TouchableOpacity</Text>
          </View>
        </TouchableOpacity>
        <TouchableNativeFeedback
            onPress={this._onPressButton}
            background={Platform.OS === 'android' ? TouchableNativeFeedback.SelectableBackground() : ''}>
          <View style={styles.button}>
            <Text style={styles.buttonText}>TouchableNativeFeedback {Platform.OS !== 'android' ? '(Android only)' : ''}</Text>
          </View>
        </TouchableNativeFeedback>
        <TouchableWithoutFeedback
            onPress={this._onPressButton}
            >
          <View style={styles.button}>
            <Text style={styles.buttonText}>TouchableWithoutFeedback</Text>
          </View>
        </TouchableWithoutFeedback>
        <TouchableHighlight onPress={this._onPressButton} onLongPress={this._onLongPressButton} underlayColor="white">
          <View style={styles.button}>
            <Text style={styles.buttonText}>Touchable with Long Press</Text>
          </View>
        </TouchableHighlight>
      </View>
    );
  }
}

const styles = StyleSheet.create({
  container: {
    paddingTop: 60,
    alignItems: 'center'
  },
  button: {
    marginBottom: 30,
    width: 260,
    alignItems: 'center',
    backgroundColor: '#2196F3'
  },
  buttonText: {
    textAlign: 'center',
    padding: 20,
    color: 'white'
  }
});
```

## Scrolling and swiping

제스처들은 swipe과 pan을 할 수 있는 터치 가능한 스크린에 주로 사용된다. 사용자가 아이템 리스트를 스크롤 하거나, 페이지 사이를 스와이핑 할 수 있게 한다. 이는 ScrollView를 이용하면 된다. 




