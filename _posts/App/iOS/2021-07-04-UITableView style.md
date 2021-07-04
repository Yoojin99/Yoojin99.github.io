---  
layout: post  
title: "[iOS] - UITableView style, headerView 같이 드래그하기"  
subtitle: ""  
categories: app
tags: app-ios uiTableView style
comments: true  
header-img: 

---  
  
> `UITableView의 style을 설정해서 headerView가 drag할 때 같이 드래그 되도록 한다.`  

---

## 기본

UITableView를 생성할 때 UITableView의 생성자를 통해 아래와 같이 테이블 뷰를 생성할 수 있다.

```swift
let tableView = UITableView()
```

이를 통해 테이블 뷰를 생성하고, 헤더 뷰를 설정했다는 가정하에 테이블 뷰를 스크롤 하면 아래와 같이 헤더는 고정이고, row들만 스크롤된다.

![notScroll](https://user-images.githubusercontent.com/41438361/124389542-90a08e80-dd22-11eb-8d20-62ad3717c1b7.gif)


## Grouped

이 헤더를 스크롤 할 떄 같이 스크롤 하기 위해서는 style을 지정해줘야한다.
UITableView를 생성할 때 style을 아래와 같이 지정해서 생성할 수 있다.

```swift
let tableView = UITableView(frame: CGRect.zero, style: .grouped)
```

그러면 스크롤 할 때 아래와 같이 테이블 뷰의 헤더가 스크롤할 때 같이 스크롤된다.

![scroll](https://user-images.githubusercontent.com/41438361/124389576-c2b1f080-dd22-11eb-8246-464b405ec0b1.gif)

## InsetGrouped

궁금해서 아래와 같이 tableView 의 style을 insetGroupped로 설정하고 생성하니 

```swift
let tableView = UITableView(frame: CGRect.zero, style: .insetGrouped)
```

아래와 같이 예쁜 모양으로 테이블 뷰가 노출된다.

![inset](https://user-images.githubusercontent.com/41438361/124389687-33f1a380-dd23-11eb-8d74-f1546d550b43.gif)

