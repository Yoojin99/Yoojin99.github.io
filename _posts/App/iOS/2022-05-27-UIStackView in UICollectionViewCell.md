---  
layout: post  
title: "[iOS] - UIStackView with Dynamic height in UICollectionViewCell"  
subtitle: ""  
categories: app
tags: app-ios stackview collection
comments: true  
header-img: 

---  
  
> `collectionview cell 안에 stackview를 넣고, stackview의 높이를 동적으로 변경해봤다.`  

---

UIStackView는 내부의 컨텐츠의 높이에 맞게 자기가 알아서 높이를 조정한다. 내부의 `arrangedSubview`가 stackview에서 remove되면 그에 맞게 stackview의 높이가 동적으로 변할거라고 예상해볼 수 있다.
