---  
layout: post  
title: "[iOS] - URL Scheme 파싱하기"  
subtitle: ""  
categories: app
tags: app-ios 
comments: true  
header-img: img/dev/app/ios/XcodeImg.png

---  
  
> `URL Scheme으로 앱을 호출했을 때 파싱하는 법을 알아보자.`  

---

URL Scheme으로 외부에서 앱을 호출하면 호출한 url을 parsing하거나 원하는 작업을 할 수 있다. url의 쿼리 부분이나 host, path 부분을 이용해서
앱의 원하는 기능을 동작하게 할 수도 있다. 그런데 이 들어온 url을 parsing을 어떻게 하면 좋을지 처음에는 고민이 됐다.

만약 현재 기능 요구 사항에서 url은 `appScheme://load/user`와 같이 단순한 형태로 들어온다고 되어있다고 해서
단순히 String에서 특정 단어나 문자가 언급되는 것을 찾아 split을 해서 잘라 사용할 수도 있지만 추후 더 복잡한 url이 들어오지 않는다는 보장도 없었고,
따라서 url이 어떻게 들어오든 이에 대한 대비를 할 필요를 할 수 있었다. 그래서 `?`과 같이 url의 query 부분에서 split의 기준점으로 사용할 수 있는
애들을 기준으로 잘라 dictionary를 만들어볼까도 생각해봤지만, 복잡한 쿼리문이 들어오면 아무 쓸모가 없어지게 된다.

구글링을 하는데 좋은 방법으로 이를 처리한 사람이 [StackOverflow](https://stackoverflow.com/questions/8756683/best-way-to-parse-url-string-to-get-values-for-keys) 있어 이를 사용했다. 

바로 `URLComponents`의 `queryItems`를 사용하는 방법이었다.

```swift
let url = "http://example.com?param1=value1&param2=param2"
let queryItems = URLComponents(string: url)?.queryItems // [URLQueryItem]와 같은 형태의 배열이 나온다.
// ["param1=value1", "param2=param2"]와 같은 형태로 나오게 된다.

let param1 = queryItems?.filter({$0.name == "param1"}).first // URLQueryItem에는 name이라는 프로퍼티가 있다. 위의 예에서는 param1, param2가 될 것이다.
print(param1?.value) // URLQueryItem에는 value라는 프로퍼티가 있다. 위의 예에서는 value1, param2가 될 것이다.
```

이렇게 끝내도 되지만 여러 곳에서도 사용할 수 있게 URL의 extension을 만드는 것이 더 좋은 방법이라고 생각한다.

```swift
extension URL {
    var queryParameters: QueryParameters { 
      return QueryParameters(url: self) // 연산프로퍼티
    }
}

class QueryParameters {
    let queryItems: [URLQueryItem]
    
    init(url: URL?) {
        queryItems = URLComponents(string: url?.absoluteString ?? "")?.queryItems ?? []
        print(queryItems)
    }
    
    subscript(name: String) -> String? {
        return queryItems.first(where: { $0.name == name })?.value
    }
}
```

위의 코드는 `URL.queryParameters["어쩌구"]`와 같은 형태로 접근할 수 있게 해준다.

그래서 아래와 같이 코드를 작성할 수 있다.

```swift
let url = URL(string: "http://example.com?param1=value1&param2=param2")!
print(url.queryParameters["param1"])
```
