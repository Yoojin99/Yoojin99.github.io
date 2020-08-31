---
title:  "React Native - Design - Screen 사이 이동하기"
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

모바일앱이 단일 화면으로 만들어진 경우는 드물다. 여러 스크린을 출력하고 이동할 수 있게 하는 거는 네비게이터에 의해 관리된다.

React Navigation은 안드로이드와 iOS 모두에서 일반적인 stack navigation과 tabbed navigation pattern과 함께 직접적인 네비게이션 해결책을 제시한다.

만약 안드로이드와 iOS 모두에서 native look을 보여지게 하고 싶거나, 이미 네비게이션을 natively하게 관리학 있는 앱에 RN을 통합하고 있다면, 
react-native-navigation 라이브러리가 두 플랫폼 모두에 native navigation을 제공할 수 있다.

## React Navigation

Navigation을 하기 위한 커뮤니티의 해결책은 개발자가 몇줄의 코드로 앱의 스크린을 설정할 수 있는 독립적인 라이브러리를 사용하는 것이다.

### 설치와 설정

먼저, 프로젝트에 다운받아야 한다.

```
npm install @react-navigation/native @react-navigation/stack
```

그리고 관련있는 것들을 다운받아라. 프로젝트가 Expo 관리 프로젝트인지 아니면 그냥 없는 RN 프로젝트인지에 따라 다른 커맨드를 입력해야 한다.

* 만약 Expo managed project라면, `expo`와 관련있는 것들을 다운받아라:
  `expo install react-native-reanimated react-native-gesture-handler react-native-screens react-native-safe-area-context @react-native-community/masked-view`
* 만약 없는 RN 프로젝트라면 `npm`과 관련 있는 것을 다운받아라:
  `npm install react-native-reanimated react-native-gesture-handler react-native-screens react-native-safe-area-context @react-native-community/masked-view`
  
bare RN project를 iOS로 하려면, Cocoapods를 다운 받아야 한다. pds는 다음과 같이 다운받을 수 있다.

```
cd ios
pod install
cd ..
```

`react-native-gesture-handler` 설치를 마치기 위해, 아래의 코드를 entry 파일(`index.js`나 `App.js`)맨 위에(가장 위에!!) 붙여라.

```js
import 'react-native-gesture-handler';
```

그리고 앱을 통째로 `NavigationContainer`에 넣어라. 주로 이거는 `index.js`나 `App.js`같은 entry 파일에 한다.

```js
import 'react-native-gesture-handler';
import * as React from 'react';
import { NavigationContainer } from '@react-navigation/native';

const App = () => {
  return (
    <NavigationContainer>
      {/* Rest of your app code */}
    </NavigationContainer>
  );
};

export default App;
```

이제 기기/시뮬레이터에서 실행할 준비가 되었다.

### 사용

이제 홈화면과 프로필 화면으로 구성된 앱을 만들 수 있다.

```js
import * as React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';

const Stack = createStackNavigator();

const MyStack = () => {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen
          name="Home"
          component={HomeScreen}
          options={{ title: 'Welcome' }}
        />
        <Stack.Screen name="Profile" component={ProfileScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
};
```

이 예제에서는 `Stack.Screen` component를 이용해서 2개의 화면(`Home`과 `Profile`)을 만들었다. 비슷하게, 
얼마든지 많은 화면을 만들 수 있다.

각 화면의 이름을 `Stack.Screen`의 `options` prop을 설정할 수도 있다.

각 화면은 React component인 `component` prop을 가지고 있다. 이 component들은 다른 스크린으로 링크할 수 있는 메소들을 많이 가진
`navigation`이라는 prop을 받는다. 예를 들어, `navigation.navigate`를 이용해서 `Profile` 화면으로 넘어갈 수 있다:

```js
const HomeScreen = ({ navigation }) => {
  return (
    <Button
      title="Go to Jane's profile"
      onPress={() =>
        navigation.navigate('Profile', { name: 'Jane' })
      }
    />
  );
};
const ProfileScreen = () => {
  return <Text>This is Jane's profile</Text>;
};
```

stack navigator에 있는 view들은 native component과 native thread에서 동작하고 있는 60fps 애니메이션을 전달하기 위해 Animated 라이브러리를 사용한다.
추가로, 애니메이션과 제스쳐도 커스터마이징이 가능하다.

RN은 tab과 drawer같이 다양한 종류의 네비게이터를 위한 패키지를 가지고 있다.






