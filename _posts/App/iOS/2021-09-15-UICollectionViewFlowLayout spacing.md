---  
layout: post  
title: "[iOS] - UICollectionViewFlowLayout spacing에 대한 고찰"  
subtitle: ""  
categories: app
tags: app-ios UICollectionViewFlowLayout spacing
comments: true  
header-img: 

---  
  
> `UICollectionViewFlowLayout에서 minimum spacing을 적용할 때 일어나는 변화에 대해 알아보자.`  

---

UICollectionView를 이용해서 아래와 같이 가로로 아이템을 스크롤 할 수 있는 뷰를 만들고 있었다.

![image](https://user-images.githubusercontent.com/41438361/133384076-83b6fbdb-1bae-41eb-9bad-eb73dc61ac70.png)

UICollectionViewFlowLayout의 `minimumLineSpacing`과 `minimumInteritemSpacing`은 값을 따로 지정하지 않은 상태이므로 둘 다 default 값인 10인 상태이고, 이때 UICollectionView의 contentSize를 찍어보니 아래와 같았다.

![image](https://user-images.githubusercontent.com/41438361/133385112-86231269-2b61-4768-ac43-255cfd950ba8.png)

* width: 914
* height: 120

참고로 한 아이템의 가로 길이는 74이다. 여기에서 세로 길이는 별로 중요하지 않아서 높이에 대해서는 자세히 보지 않겠다. 그리고 아이템이 11개이며,
아이템 사이의 간격은 디폴트가 10이므로 74(아이템 너비) * 11(아이템 개수) + 10(아이템 사이의 공백) * 10(공백 개수) =  814 + 100 = 914이 되므로 contentSize의 width가 914로 나온 것은 정확하게 사이즈가 잘 계산되었다는 것을 의미한다. Spacing에 따라 contentSize에 어떻게 달라지는지 확인하기 위해 추가적인 작업들을 해봤다. 

레이아웃을 정의할 때 `UICollectionViewFlowLayout`을 사용했는데, 이 상태에서 아이템 간의 간격을 더 좁히기 위해 아래와 같이 코드를 작성했다.
참고로 아무런 값을 설정하지 않았을 때 기본 값은 10.0이다.

```swift
collectionViewLayout.minimumLineSpacing = 4
```

UIScrollView의 scrollDirection이 horizontal인 상태라 lineSpacing을 변화시켰다. 위와 같이 설정하고 실행시켰더니 아래와 같이 오른쪽에 추가 여백이 생기는 것을 확인할 수 있었다. 

![image](https://user-images.githubusercontent.com/41438361/133384604-63c1aac4-96d3-49b1-bc63-2461367afdb4.png)

참고로 이때 collectionView의 contentsize를 출력하니 아래와 같이 나왔다.

![image](https://user-images.githubusercontent.com/41438361/133385884-88869e55-0ab2-43ed-a6d9-9b59d1e4e4e2.png)

* width: 914
* height: 120

spacing을 변경해서 contentSize가 줄어든 spacing에 맞춰 width가 줄어들 것으로 예상했으나, contentSize는 변하지 않았다. contentSize가 변하지 않은 상태로
lineSpacing은 10에서 4로 줄어들었으니 오른쪽에 여백이 생기는 것은 당연해 보인다. 그렇다면 lineSpacing을 반대로 늘리면 어떻게 될까?

```swift
collectionViewLayout.minimumLineSpacing = 20
```

![image](https://user-images.githubusercontent.com/41438361/133386770-96f1dfaf-a9f6-42cb-8b16-6780cdc408c2.png)

마지막 11번째 아이템이 노출되지 않았다. 

이 상태로 contentSize를 찍어보니 아래와 같이 나왔다.

![image](https://user-images.githubusercontent.com/41438361/133386625-5b65c88c-a7bb-432c-a0b7-ccbbdb72b7d5.png)

* width: 914
* height: 120

마찬가지로 contentSize는 변하지 않고, 화면에서 보이는 아이템 사이의 간격만 변했다. 그래서 contentSize의 너비는 일정한데 아이템 사이의 공백 간격이 커져서 마지막 아이템이 보이지 않게 되는 것이다.

lineSpacing은 그렇다 치고, interitemSpacing을 변화시키면 어떻게 될까? lineSpacing을 설정한 코드를 삭제하고, 아래와 같이 코드를 작성했다. InterItemSpacing을 10에서 4로 줄였다. 

```swift
collectionViewLayout.minimumInteritemSpacing = 4
```

![image](https://user-images.githubusercontent.com/41438361/133388506-0ecb17fa-0cde-4540-8d97-8f62292a0a6a.png)

이 친구도 마지막 아이템이 잘려서 출력된다.

그리고 contentSize를 찍어보니 아래와 같이 나왔다.

![image](https://user-images.githubusercontent.com/41438361/133387889-81e41366-74fb-4148-a069-1e090cf88db2.png)

* width: 854
* height: 120

contentSize가 바뀌었다! 914에서 854로 줄어들었는데, 60만큼 줄어들었다. 아이템 간의 간격을 10에서 4로 조정했는데, 60만큼 줄어들었다는 것은
감소된 6만큼이 10번 줄어들었다는 것이다. 즉 줄어든 아이템 사이의 간격에 맞게 contentSize가 잘 줄어들었다는 것이다. contentSize가 줄어들었는데 
아이템 사이의 spacing(lineSpacing: 여기서는 scrollDirection이 horizontal이므로 line의 기준이 열방향이므로 아이템 사이의 간격을 lineSpacing이라고도 할 수 있다.)은 10이므로 마지막 아이템이 잘려 출력되는 것이 맞다.

그렇다면 반대로 spacing을 늘리면 어떻게 될까?

```swift
collectionViewLayout.minimumInteritemSpacing = 20
```

![image](https://user-images.githubusercontent.com/41438361/133391276-e5415eae-43a8-4d00-82de-b0618e2ac28e.png)

마지막에 여백이 노출되어 출력이 된다.

그리고 contentSize를 찍으니 아래와 같이 나왔다.

![image](https://user-images.githubusercontent.com/41438361/133391177-0798c298-b0ea-4ac4-b82b-d2f232bd370e.png)

* width: 1014
* height: 120

여기에서도 마찬가지로 contentSize에 변화가 생겼다. 아무것도 설정하지 않았을 때 가로 길이가 914였고, 여기에서는 1014가 되었으니 100만큼 늘어난 것이다. 늘어난 spacing 10만큼 10번이 늘어난 것이고, 여기에서도 마찬가지로 아이템 사이의 변화된 간격이 contentSize에 잘 반영이 된 것을 확인할 수 있다. 하지만 lineSpacing은 그대로 10을 유지하고 있어서 화면 상에는 아이템 사이의 간격이 10으로 나오지만, 실제 contentSize는 늘어났으므로 늘어난만큼 오른쪽에 여백이 노출된 것이다.

그래서 위의 작업들을 통해 횡스크롤 1줄 짜리의 collectionView에서는 `minimumInteritemSpacing`는 실제 contentSize에 영향을 주지만 
화면 상에 보이는 아이템 사이의 실제 간격에는 영향을 주지 않고, 반대로 `minimumLineSpacing`은 실제 contentSize에 영향을 주지 않지만
화면 상에 보이는 아이템 사이의 간격(여기에서는 아이템 사이의 간격이 lineSpacing이 맞다.)에는 영향을 준다는 것이다.

즉 내가 원하는 대로 아이템 사이의 간격에도 변화를 주고 정상적으로 화면에 아이템이 보이게끔 하려면 아래와 같이 `minimumInteritemSpacing`과 `minimumLineSpacing`의 값을 모두 변화시켜야 한다는 것이다.

```swift
collectionViewLayout.minimumLineSpacing = 4
collectionViewLayout.minimumInteritemSpacing = 4
```

여기서 잠시 `minimumInteritemSpacing`과 `minimumLineSpacing`의 개념을 짚고 넘어가면 아래와 같다.

### minimumInteritemSpacing

![image](https://user-images.githubusercontent.com/41438361/133392092-f3f08e34-a250-4054-84d0-b62d993b7868.png)

**같은 줄**안에 있는 아이템 간의 최소 간격을 의미한다.

세로로 스크롤하는 그리드에서 이 값은 같은 줄(행이 될 것이다) 내의 아이템 사이의 최소 간격을 의미한다. 가로로 스크롤하는 그리드에서 이 값은 같은 줄(열) 내의 최소 간격이 될 것이다. 이 간격은 한 줄에 얼마나 많은 아이템이 들어갈 수 있는 지 계산하는데 쓰인다.

### minimumLineSpacing

![image](https://user-images.githubusercontent.com/41438361/133392452-1e79ecef-4e9f-473f-b0f1-d63d5797b9ab.png)

**줄 간의** 최소 간격을 의미한다. InteritemSpacing이 같은 줄 내의 아이템 사이의 간격이었다면, lineSpacing은 줄 간의 간격이 된다.

세로로 스크롤하는 그리드에서 이 값은 연속하는 줄(행) 간의 최소 간격이 될 것이고, 가로로 스크롤하는 그리드에서 이 값은 연속하는 줄(열) 간의 최소 간격이 될 것이다. 이 간격은 헤더와 첫 번째 줄, 그리고 마지막 줄과 푸터에 적용되는 값은 아니다. 

여기까지 봤을 때, 가로로 스크롤하는 collectionView에서의 간격을 그림으로 그려보자면 아래와 같다.

![image](https://user-images.githubusercontent.com/41438361/133393251-f6efb878-0359-479b-ba23-9652c0b614fb.png)

위의 그림을 봤을 때, 지금까지 작업했던 1줄짜리 collectionView에서 아이템 사이의 간격(lineSpacing)을 늘리기 위해서는 

```swift
collectionViewLayout.minimumLineSpacing = 20
```

위와 같이만 작성해도 될 것처럼 보인다. 하지만 실제로 위와 같이만 작성했을 때, lineSpacing이 contentSize에 반영되지 않아 contentSize는 그대로인 반면에 화면에 보이는 lineSpacing은 증가했기 때문에 아이템이 잘려보이는 현상이 발생했다. 그렇다면 왜 `minimumLineSpacing`은 contentSize에 반영이 되지 않고 `minimumInteritemSpacing`이 contentSize에 반영이 되는 것일까? 그리고 작업한 collectionView는 한 줄짜리라 interitemSpacing을 변경해도 contentSize의 width가 아닌 height에 적용되어야 할 것 같은데, 왜 width가 늘어나는 것일까?

대체 왜 이런 현상이 일어나는지 찾아보기 위해 UICollectionView가 spacing을 가지고 contentSize를 계산하는 방법에 대해 구글링을 해봤으나 
관련된 공식문서나 자료를 많이 찾을 수 없었다. 다만 내가 겪은 현상과 똑같은 현상에 대해 질문한 [stackOverflow의 글](https://stackoverflow.com/questions/42980842/minimuminteritemspacing-is-adding-some-strange-content-at-the-end-of-collection)을 찾을 수 있었다. 하지만 여기에서도 명확한 답을 찾을 수는 없었다.

없으면 답답한 사람이 직접 찾아야 한다고 collectionView의 scrollDirection이 horizontal과 vertical일 때 spacing의 변화가 contentSize에 어떤 영향을 미치는지 직접 알아보기로 했다. ^__^

## Vertical Scroll UICollectionView

먼저 익숙한 세로로 스크롤링 되는 UICollectionView를 만들고, 화면 밑에 눌렀을 때 collectionView의 contentSize를 출력하게 하는 버튼을 추가했다. UICollectionView는 남은 화면에 꽉 채워지게 설정했다.

![image](https://user-images.githubusercontent.com/41438361/133407075-533abf0e-2203-41d8-b120-0271ee6d789c.png)

이 상태에서 contentSize를 확인하니 아래와 같이 출력되었다.

![image](https://user-images.githubusercontent.com/41438361/133407192-0b98cdf2-0dce-4a35-9bcd-e3af3e656d57.png)

참고로 저 아이템은 50 x 50 인데, 높이를 보았을 때 50 * 3 + 10(linespacing) * 2 = 170이므로 contentSize의 높이가 170이 나오는 것이 맞다.

이제 lineSpacing을 먼저 바꿔보자. 변화를 알아보기 쉽게 lineSpacing을 50으로 설정했다.

![image](https://user-images.githubusercontent.com/41438361/133408566-1aa16045-3544-4249-8f74-dca1feaa96e1.png)

그리고 contentSize를 출력하니 아래와 같았다.

![image](https://user-images.githubusercontent.com/41438361/133408650-39509ef1-c283-471a-b453-ac61c0c539f3.png)

50 * 3 + 50 * 2 = 250이므로 높이가 250이 되는 것이 맞다.

이번에는 itemSpacing을 바꿨다. lineSpacing을 다시 기본값으로 바꾸고, interitemSpacing을 50으로 설정했다.

![image](https://user-images.githubusercontent.com/41438361/133562472-6e2f2884-cd1d-4338-a320-e82285bdbe3b.png)

![image](https://user-images.githubusercontent.com/41438361/133562502-66d7ae00-a999-4c9d-a063-992d335cbf0e.png)

interItemSpacing이 50으로 설정되어 있으니 한 줄에는 최대 4개의 아이템이 들어가는 것이 맞다. 그리고 변경된 interItemSpacing에 따라 contentSize의 height도 290으로 잘 조정되었다.



## Horizontal Scroll UICollectionView

![image](https://user-images.githubusercontent.com/41438361/133407513-4a9ce867-0cbd-4a5a-9e50-48a022e1c22b.png)

아이템 사이즈가 50 x 50일 때의 화면이다. vertical scroll UICollectionView와 달라진 점은, vertical grid에서는 한 행이 채워지면 다음 행으로 넘어가는데, horizontal grid에서는 한 열이 먼저 채워지면 다음 열로 넘어가는 것이다. 즉 줄의 기준이 행에서 열로 바뀐 것을 알 수 있다.

이 때 contentSize를 출력하니 아래와 같았다.

![image](https://user-images.githubusercontent.com/41438361/133407761-2221de55-a2a8-48d8-9451-64019694edff.png)

가로의 길이는 50 * 2 + 10 = 110이 되는 것이 맞다.

이제 lineSpacing을 50으로 설정했다.

![image](https://user-images.githubusercontent.com/41438361/133561958-681069fe-5673-4f35-986b-ab354b8a66e5.png)

contentSize를 출력하니 아래와 같았다.

![image](https://user-images.githubusercontent.com/41438361/133562056-8e35c50c-e523-482d-8648-c945448b53db.png)

* width: 110
* height: 701

contentSize의 width가 변하지 않았다. 하지만 화면에 실제 보이는 것으로는 lineSpace가 증가한 것이 맞고, 심지어 둘 째 줄의 아이템 사이의 간격이 증가한 형태로 나타났다.



