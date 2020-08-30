---
title:  "React Native - List View 사용하기"
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

RN은 데이터들의 리스트를 나타내기 위한 component들을 제공한다. 보통 FlatList나 SectionList를 사용한다.

`FlatList` component는 바뀌지만 비슷하게 구성된 데이터들의 scrolling 리스트를 화면에 보여준다. `FlatList`는 몇개의 데이터가 시간에 따라 바뀌는 긴 목록의 데이터들에 적합하다.
일반적인 ScrollView와 다르게, `FlatList`는 모든 요소들을 한번에 렌더링하지 않고 현재 화면에 보여지고 있는 요소들만 렌더링한다.

`FlatList` component는 두가지 prop을 가지고 있다 : `data`와 `renderItem`. `data`는 리스트의 정보이다. `renderItem`은 source에서 하나를 골라서 렌더링하기 위한 정형화된 component를 리턴한다.

아래의 예제는 하드코딩된 데이터들의 `FlatList`를 생성한다. `data` prop에 있는 각 아이템은 `Text` component로 렌더링된다.
`FlatListBasics` component는 그러면 `FlatList`와 모든 `Text` component를 렌더링한다.

```js
import React from 'react';
import { FlatList, StyleSheet, Text, View } from 'react-native';

const styles = StyleSheet.create({
  container: {
    flex: 1,
    paddingTop: 22
  },
  item: {
    padding: 10,
    fontSize: 18,
    height: 44,
  }
});

const FlatListBasics = () => {
  return(
    <View style={styles.container}>
      <FlatList
        data={[
          {key: 'Devin'},
          {key: 'Dan'},
          {key: 'Dominic'},
          {key: 'Jackson'},
          {key: 'James'},
          {key: 'Joel'},
          {key: 'John'},
          {key: 'Jillian'},
          {key: 'Jimmy'},
          {key: 'Julie'},
        ]}
        renderItem={({item}) => <Text style={styles.item}>{item.key}</Text>}
      />
    </View>
  );
}

export default FlatListBasics;
```

만약 섹션 헤더와 함께 논리적인 섹션으로 데이터들을 나누고 싶다면, iOS의 `UITableView`와 비슷하게 하려면 SectionList를 사용해라.

```js
import React from 'react';
import { SectionList, StyleSheet, Text, View } from 'react-native';

const styles = StyleSheet.create({
  container: {
   flex: 1,
   paddingTop: 22
  },
  sectionHeader: {
    paddingTop: 2,
    paddingLeft: 10,
    paddingRight: 10,
    paddingBottom: 2,
    fontSize: 14,
    fontWeight: 'bold',
    backgroundColor: 'rgba(247,247,247,1.0)',
  },
  item: {
    padding: 10,
    fontSize: 18,
    height: 44,
  },
})

const SectionListBasics = () => {
    return (
      <View style={styles.container}>
        <SectionList
          sections={[
            {title: 'D', data: ['Devin', 'Dan', 'Dominic']},
            {title: 'J', data: ['Jackson', 'James', 'Jillian', 'Jimmy', 'Joel', 'John', 'Julie']},
          ]}
          renderItem={({item}) => <Text style={styles.item}>{item}</Text>}
          renderSectionHeader={({section}) => <Text style={styles.sectionHeader}>{section.title}</Text>}
          keyExtractor={(item, index) => index}
        />
      </View>
    );
}
```


