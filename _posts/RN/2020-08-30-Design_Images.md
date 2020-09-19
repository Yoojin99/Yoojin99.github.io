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

## Static Image Resources

RN은 안드로이드와 iOS 앱들에서 이미지들과 다른 미디어 에셋들을 관리하는 통합된 방법을 제공한다. 앱에 정적인 이미지를 추가하고 싶으면,
소스 코드 트리에 추가하고 아래와 같이 참조해라.

```js
<Image source={require('./my-icon.png')} />
```

이미지 이름은 JS 모듈들 처럼 같은 방법으로 불러오면 된다. 위의 예제에서, bundler는 component가 요구하는대로 
같은 폴더 내의 `my-icon.png`를 찾을 것이다. 또한, 만약 `my-icon.ios.png`와 `my-icon.android.png`를 가지고 있다면 bundler는 플랫폼에 맞는 파일을 선택할 것이다.

또한 `@2x`와 `@3x` 꼬리말을 사용해서 다른 스크린 밀도를 위한 이미지를 제공할 수도 있다. 만약 아래와 같이 파일을 가지고 있다고 해보자.

```
.
├── button.js
└── img
    ├── check.png
    ├── check@2x.png
    └── check@3x.png
```

그리고 `buuton.js`가 아래 코드를 포함하고 있다.

```js
<Image source={require('./img/check.png')}>
```

bundler는 이미지를 기기의 스크린 밀도와 일치하는 이미지를 묶고 제공할 것이다. 예를들어 `check@2x.png`는 아이폰7에서,
`check@3x.png`는 아이폰7이나 넥서스 5에 사용될 것이다. 만약 스크린 밀도와 일치하는 이미지가 없다면, 그나마 적합한 이미지가 선택된다.

윈도우에서, 프로젝트에 새로운 이미지를 추가했을 때 bundler를 재시작해야 할 수도 있다.

이로 얻을 수 있는 이점은 다음과 같다:

1. 안드로이드와 iOS에서 같은 시스템으로 동작한다.
2. 이미지를 JavaScript 코드와 같은 폴더에 저장할 수 있다. Component들은 그냥 포함된것이다(?)
3. 글로벌 namespace가 필요없다. 이름 충돌에 대해 걱정할 필요가 없다.
4. 실제 사용되는 이미지만이 앱에 패키징될 것이다.
5. 이미지를 추가하고 변경한다고 앱을 다시 컴파일 할 필요가 없다. 그냥 시뮬레이터를 새로고침하면 된다.
6. bundler가 이미지 치수를 알기때문에 코드에서 다시 번복할 필요가 없다.
7. 이미지는 npm 패키지로 배포될 수 있다.

이것이 동작할 수 있도록 하기 위해, `require`에서 이미지 제목이 정적으로 선언되어야 한다.

```js
// GOOD
<Image source={require('./my-icon.png')} />;

// BAD
var icon = this.props.active
  ? 'my-icon-active'
  : 'my-icon-inactive';
<Image source={require('./' + icon + '.png')} />;

// GOOD
var icon = this.props.active
  ? require('./my-icon-active.png')
  : require('./my-icon-inactive.png');
<Image source={icon} />;
```

이런식으로 만들어진 이미지 소스들은 이미지를 위한 사이즈 정보가 포함된다. 만약 이미지 사이즈를 동적으로(flex를 통해서)
조정하고 싶다면, `{width:undefined, height:undefinde}`를 스타일 속성에서 직접 설정해야 한다.

## 정적인 Non-Image Resources

위의 `require` 문법은 프로젝트에 오디오, 비디오, 문서도 포함할 수 있다. 일반적으로 `.mp3`, `.wav`, `.mp4`,
`.mov`, `.html`, `.pdf`를 포함하는 파일 타입들이 지원된다. 

Metro configuration에서 `assetExts` resolver option을 추가함으로써 다른 타입들을 지원하게 할 수 있다.

비디오는 사이즈 정보가 현재 non-image asset을 위해 전달되지 않으므로 `flexGrow`대신에 정확한 포지셔닝을 사용해야 한다.
이런 경우는 Xcode나 안드로이드를 위한 Asset 폴더에 직접적으로 링크될때는 발생하지 않는다.

## Images From Hybrid App's Resources

만약 하이브리드 앱을 개발하고 있다면(어떤 UI는 RN, 어떤 UI는 플랫폼 코드로) 이미 앱에 포함된 이미지들을 여전히 사용할 수 있다.

Xcode aaset catalog나 안드로이드 drawable 폴더에 포함된 이미지는 extension없이 이미지 이름을 사용하면 된다.

```js
<Image
  source={{ uri: 'app_icon' }}
  style={{ width: 40, height: 40 }}
/>
```

안드로이드 에셋 폴더에 있는 이미지로는 `asset:/` 식을 이용하면 된다.

```js
<Image
  source={{ uri: 'asset:/app_icon.png' }}
  style={{ width: 40, height: 40 }}
/>
```

이런 접근은 saftey check를 제공하지 않는다. 이 이미지가 앱에서 제대로 동작하는 것을 보장하는 것은 나한테 달렸다. 이미지 치수또한 직접 설정해야 한다.

## Network Images

앱에 표시하게 되는 많은 이미지들은 컴파일 타임에 유효하지 않을 수도 있고, binary 사이즈를 줄이기 위해 몇개는
동적으로 로딩하고 싶을 것이다. 정적인 리소스들과는 다르게, 이미지의 치수를 직접 설정해야 한다.
iOS의 App Transport Security 요구사항을 만족시키기 위해 https를 사용하는 것이 추천된다.

```js
// GOOD
<Image source={{uri: '이미지 url'}} style={{width: 400, height: 400}} />

// BAD
<Image source={{uri: '이미지 url'}} />
```

### Network Request for Images

만약 http-verb를 사용하고 싶다면, 소스 객체에서 이런 property들을 설정함으로써 이미지 요청에 헤더나 바디를 추가시킨다.

```js
<Image
  source={{
    uri: 'https://reactjs.org/logo-og.png',
    method: 'POST',
    headers: {
      Pragma: 'no-cache'
    },
    body: 'Your Body goes here'
  }}
  style={{ width: 400, height: 400 }}
/>
```

### Uri Data Images 

가끔, REST API 콜로 인코딩된 이미지 데이터를 가져올 수도 있다. 이런 이미지들을 사용하기 위해서는 `data:` uri 식을 사용하면 된다. 
네트워크 리소스에서도 똑같이, 직접 이미지의 수치를 지정해야 한다.

이거는 DB의 리스트에 있는 아이콘같이 굉장히 작고 동적인 이미지들에 쓰는 것을 추천한다.

```js
// include at least width and height!
<Image
  style={{
    width: 51,
    height: 51,
    resizeMode: 'contain'
  }}
  source={{
    uri:
      'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADMAAAAzCAYAAAA6oTAqAAAAEXRFWHRTb2Z0d2FyZQBwbmdjcnVzaEB1SfMAAABQSURBVGje7dSxCQBACARB+2/ab8BEeQNhFi6WSYzYLYudDQYGBgYGBgYGBgYGBgYGBgZmcvDqYGBgmhivGQYGBgYGBgYGBgYGBgYGBgbmQw+P/eMrC5UTVAAAAABJRU5ErkJggg=='
  }}
/>
```

### Cache Control (iOS Only)

이미 로컬 캐시에 있을 때만 이미지를 출력하고 싶을 수 있다. 아미녀 BW를 절약하기 위해 outdated 이미지를 출력하고 싶을 수 있다.
`cache` 소스 property는 어떻게 network layer가 cache와 소통할지 컨트롤 할 수 있게 한다.

* `default` 
* `reload`
* `force-cache`
* `only-if-cached`

```js
<Image
  source={{
    uri: 'https://reactjs.org/logo-og.png',
    cache: 'only-if-cached'
  }}
  style={{ width: 400, height: 400 }}
/>
```

## 사이즈 설정 자동화

브라우저에서 만약 이미지에 사이즈 값을 지정하지 않는다면, 브라우저는 0x0 element를 렌더링하고 다운로드 받은 후에 올바른 사이즈로 렌더링 할 것이다.
이렇게 하는 것은 사용성을 안 좋게 만든다는 것이다.

RN에서 이런거는 내부적으로 구현되어 있지 않다. 미리 원격 이미지의 수치를 아는 것은 개발자의 일이지만, 우리는 이게 더 좋은 사용성으로 이어진다고 생각한다.
`require('./my-icon.png')` 문법으로 앱 번들에서 로드된 정적 이미지는 그것들의 수치가 mounting을 할 때 즉시 사용가능하기 때문에 동적으로 사이즈가 정해질 수 있다

예를 들어, `require('./my-icon.png')`의 결과는 다음과 같을 것이다.

`{"__packager_asset":true,"uri":"my-icon.png","width":591,"height":573}`

## Source as an object

RN에서, 흥미로운 결정 하나는 `src` 속성이 `source`로 되어 있는 것과 문자열을 받지 않고 `uri` 속성을 가진 객체로 받는다는 것이다.

```js
<Image source={{ uri: 'something.jpg' }} />
```

Infrastructure 적인 측면에서 이렇게 하는 이유는 객체에 메타데이터를 달 수 있기 때문이다. 예를 들어 `require('./my-icon.png')`를 사용한다고 하면,
우리는 실제 위치와 사이즈(후에 바뀔 수 있다고 한다)에 대한 정보를 추가한다. 이것도 미래에 개발 될 수 있는 거긴 한데, 예를 들어 특정 지점에 sprite를 출력하고 싶다면
`{uri: ...}` 대신에 `{uri: ..., crop: {left:10, top:50, width:20, height:40}}`으로 설정하고 모든 이 이미지를 부르는 곳에서 투명하게 스프라이트를 지원할 수 있다.

## Background Image via Nesting

개발자들로부터 흔히 요청되는 거는 `backgroun-image`이다. 이거를 사용하기 위해, `<Image>`와 같은 prop들을 가지고 있는 `<ImageBackground>` component를 사용하고 이미지 위에 올려놓을 자식들만 추가하면 된다.

```js
return (
  <ImageBackground source={...} style={{width: '100%', height: '100%'}}>
    <Text>Inside</Text>
  </ImageBackground>
);
```

높이와 너비 속성을 지정해야 한다는 것을 기억해라.


