---  
layout: post  
title: "[iOS] NSAttributedString html"  
subtitle: ""  
categories: app
tags: app-ios 
comments: true  
header-img: 
--- 

어느날 `[_WebSafeForwarder forwardInvocation:]` 라는 크래시가 발생했다. 이에 대해 구글링을 해보니 여러 Stack Overflow 글에서 NSAttributedString 의 메서드를 사용해서 html 을 파싱할 때 간헐적으로 발생하는 크래시인 것으로 보여 관련해서 iOS 에서 NSAttributedString 으로 html 을 파싱할 때 내부적으로 어떻게 동작하는지 확인하고, 왜 크래시가 발생하는지 확인했다.

# 1\. iOS 내부 동작

## NSAttributedString 생성자

`initWithData:options:documentAttributes:error:`

명시된 data 객체에서 attributed string을 생성하는 메서드. 디코딩 되지 않았을 경우 nil, 그 외의 경우 attributed string 객체 리턴.

[공식 문서](https://developer.apple.com/documentation/foundation/nsattributedstring/1524613-initwithdata)

* **HTML importer를 background thread에서 호출하면 안됨** : HTML importer(`options` 딕셔너리가 `NSDocumentTypeDocumentAttribute`를 `NSHTMLTextDocumentType`으로 지정한 것) background thread에서 호출한 경우 **main thread와 동기화를 하려고 할 때 fail, time out 발생**.
* Main thread에서 호출할 경우 정상적으로 동작 : HTML이 외부 리소스를 포함하고 있는 경우 time out 발생 가능
* 에러 처리 : 실패할 경우 `throws`로 error를 던짐. `do`-`catch` 문 내에서 `try`를 함께 붙여 에러를 처리한다.

## NSAttributedString 내부 동작

1. **html 렌더링 시 webkit 사용**
2. 생성자 호출시 **내부적으로 `CFRunLoopRun` 호출**

**생성자 호출 시 stack trace**

```
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 1.1
frame #0: 0x000000010f111b81 SampleProject`closure #1 in ViewController.dispatchStuff(wait=0) at ViewController.swift:20:23
frame #1: 0x000000010f111dc8 SampleProject`thunk for @escaping @callee_guaranteed () -> () at <compiler-generated>:0
frame #2: 0x000000010f5f3d18 libdispatch.dylib`_dispatch_call_block_and_release + 12
frame #3: 0x000000010f5f4f5b libdispatch.dylib`_dispatch_client_callout + 8
frame #4: 0x000000010f605d55 libdispatch.dylib`_dispatch_main_queue_drain + 1463
frame #5: 0x000000010f605790 libdispatch.dylib`_dispatch_main_queue_callback_4CF + 31
frame #6: 0x00007ff800387b1f CoreFoundation`__CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__ + 9
frame #7: 0x00007ff800382436 CoreFoundation`__CFRunLoopRun + 2482
frame #8: 0x00007ff8003816a7 CoreFoundation`CFRunLoopRunSpecific + 560
frame #9: 0x00007ff804671f4c UIFoundation`-[NSHTMLReader _loadUsingWebKit] + 1843
frame #10: 0x00007ff804672df8 UIFoundation`-[NSHTMLReader attributedString] + 22
frame #11: 0x00007ff8045e5ccc UIFoundation`_NSReadAttributedStringFromURLOrData + 5837
frame #12: 0x00007ff8045e458a UIFoundation`-[NSAttributedString(NSAttributedStringUIFoundationAdditions) initWithData:options:documentAttributes:error:] + 144
frame #13: 0x000000010f113fbe SampleProject`@nonobjc NSAttributedString.init(data:options:documentAttributes:) at <compiler-generated>:0
frame #14: 0x000000010f113912 SampleProject`NSAttributedString.init(data:options:documentAttributes:) at <compiler-generated>:0
frame #15: 0x000000010f11343a SampleProject`ViewController.parseHTML(self=0x00007fa8a8407000) at ViewController.swift:54:31
frame #16: 0x000000010f1116a4 SampleProject`ViewController.viewDidLoad(self=0x00007fa8a8407000) at ViewController.swift:11:13
...
```

### 1\. NSAttributedString html 렌더링

<img width="761" alt="image" src="https://user-images.githubusercontent.com/41438361/212313718-4ebce04c-d13c-407d-a63c-e9e9eb2573d9.png">

* TextKit을 사용하지 않고 내부적으로 WebKit을 사용. 메인 스레드에서 실행하지 않는다면 SIGTRAP과 함께 크래시가 난다.
* WebKit에 의존하고 있기 때문에 비동기 작업이 수행되고 있는 도중에 runloop를 spinning하게 된다. *하나의 thread가 연속적으로 두 개의 이벤트를 실행하는데, 두 이벤트가 event loop와 연관된 callback을 하게 될 경우 버그가 발생할 수 있다.*
* HTML 렌더링을 할 때 WebKit을 사용하기 때문에 Background thread에서 호출하게 될 경우, main thread로 작업을 옮기게 된다. 이는 호출 도중에 메인 스레드가 run loop를 실행해야 함을 의미한다. 이 과정에서 문제가 많을 수 있지만 이렇게 하는 이유는 HTML이 로딩이 필요한 외부 리소스를 참조하고 있을 수 있기 떄문이다.

*main thread는 user-interactive qos를 가지지만, 역은 성립하지 않음. 참고 : [https://developer.apple.com/library/content/documentation/Performance/Conceptual/EnergyGuide-iOS/PrioritizeWorkWithQoS.html](https://developer.apple.com/library/content/documentation/Performance/Conceptual/EnergyGuide-iOS/PrioritizeWorkWithQoS.html) [https://developer.apple.com/videos/play/wwdc2015/718/](https://developer.apple.com/videos/play/wwdc2015/718/)*

### 2\. CFRunLoopRun

1. Html 렌더링 시 WebKit 을 사용하고 있고, WebKit 은 main thread 에서 사용해야 함.
2. global queue 에서 생성자를 호출할 때 main thread 로 전환하기 위해 `performSelectorOnMainThread:withObject:waitUntilDone:` 를 호출하고 있다. 이 메서드를 Secondary thread 에서 호출할 경우 명시적으로 run loop 를 시작해야 한다.
![image](https://user-images.githubusercontent.com/41438361/210683087-dacb85ca-6a6a-41f0-8ea3-695ebb1c2197.png)
3. Run loop 를 명시적으로 호출하는 `_CFRunLoopRun` 을 호출하고 있다.
<img width="858" alt="image" src="https://user-images.githubusercontent.com/41438361/212313877-38fcd585-bac6-4d3d-81fc-9f99e60a3635.png">

`CFRunLoopRun` : 현재 thread 의 Run loop 객체를 무기한으로 실행. 실행된 run loop는 `CFRunLoopStop`이 호출되거나 run loop에 있는 모든 코드와 timer가 제거되기 전까지 실행됨.

생성자 호출 시 `_CFRunLoopRun`을 내부적으로 호출. 이 때문에 특정 큐에서 비동기적으로 동작하는 것으로 보이게 된다. 메인 스레드에서는 기본 run loop가 이미 실행되고 있기 때문에 `CFRunLoopRun`을 실행해도 의미가 없다.

## 정리

(Secondary thread 에서 `NSAttributedString`로 html 데이터를 파싱할 경우)

1. NSAttributedString 이 html 을 렌더링 하기 위해 WebKit 을 사용한다.
2. WebKit 을 사용하려면 main thread 로 전환해야 한다.
3. Background thread 에서 main thread 로 전환하기 위해 `performSelectorOnMainThread:withObject:waitUntilDone:` 을 호출한다.
4. Secondary thread 에서 `performSelectorOnMainThread:withObject:waitUntilDone:` 을 호출할 경우 명시적으로 run loop 를 시작해야 한다.
5. 명시적으로 run loop 를 실행하기 위해 `CFRunLoopRun` 을 호출한다.
6. 결론 (추측)
    1. Main thread 로 전환해 run loop 를 실행하는 과정에서 크래시 발생
    2. `RunLoop` 클래스는 thread-safe 하지 않기 때문에 `RunLoop` 메서드를 호출할 때는 같은 thread 문맥 내에서만 호출해야 한다. 다른 thread의 run loop 을 조작하면 크래시가 발생할 수 있음
    3. `CFRunLoopRun` 을 통해 시작한 run loop 는 `CFRunLoopStop` 을 호출하거나 run loop 내의 타이머 / 소스가 없을 때까지 무한정 실행되는데 이 과정에서 background thread 의 Run loop 가 정상적으로 종료되지 않음

# 2\. 기타 방법

1. `NSAttributedString`의 생성자가 main thread에서 호출되는 것이 보장되도록 `DispatchQueue.main.async` 내에서 호출

``` swift
extension NSAttributedString {
    static func attributedString(fromHtmlString string: String, completion: @escaping(NSAttributedString?) -> Void) {
        DispatchQueue.main.async {
            guard let attributedString: NSAttributedString = try? NSAttributedString(
                data: Data(string.utf8),
                options: [NSAttributedString.DocumentReadingOptionKey.documentType : NSAttributedString.DocumentType.html],
                documentAttributes: nil) else {
                // log error
                completion(nil)
                return
            }

            completion(attributedString)
        }
    }

    static func attributedString(fromHtmlString string: String, completion: @escaping(Result<NSAttributedString, Error>) -> Void) {
        DispatchQueue.main.async {
            do {
                let attributedString: NSAttributedString = try NSAttributedString(
                    data: Data(string.utf8),
                    options: [NSAttributedString.DocumentReadingOptionKey.documentType : NSAttributedString.DocumentType.html],
                    documentAttributes: nil)

                completion(.success(attributedString))
            } catch {
                completion(.failure(error))
            }
        }
    }
}
```

사용 예

``` swift
NSAttributedString.attributedString(fromHtmlString: html) { [weak self] result in
    switch result {
    case .success(let htmlAttributedString): self?.label.attributedText = htmlAttributedString
    case .failure(let error): break
    }
}
```

- - -

## 기타

### CFRunLoopRun 호출로 인한 동작 (SampleProject.zip)

* `viewDidLoad()`

``` swift
dispatchStuff()
for _ in 0..<10 {
     slowOperation()
//     parseHTML()
}
```

* `dispatchStuff()`

``` swift
func dispatchStuff() {
    for i in 0..<10 {
        let wait = Double(i) * 0.2
        DispatchQueue.main.asyncAfter(deadline: .now() + wait) {
            assert(Thread.isMainThread, "not main thread!")
            print("🔶 dispatched after \(wait) seconds")
        }
    }
}
```

* `slowOperation()`

``` swift
func slowOperation() {
    n += 1
    assert(Thread.isMainThread, "not main thread!")
    print("slowOperation \(n) START")
    var x = [0]
    // 10000일 경우 굉장히 느림
    for i in 0..<1000 {
        x.removeAll()
        for j in 0..<i {
            x.append(j)
        }
    }
    print("slowOperation \(n) END")
    print("")
}
```

* `parseHTML()`

``` swift
func parseHTML() {
        m += 1
        assert(Thread.isMainThread, "not main thread!")
        self.m += 1
        print("parseHTML \(self.m) START")

        let options = [NSAttributedString.DocumentReadingOptionKey.documentType: NSAttributedString.DocumentType.html]
        let attrString = try! NSAttributedString(data: Data(html.utf8), options: options, documentAttributes: nil)
        label.attributedText = attrString

        print("parseHTML \(self.m) END")
        print("")
    }
```

1. `slowOperation`을 실행할 때 결과

<img width="356" alt="image" src="https://user-images.githubusercontent.com/41438361/212314195-d0358053-5273-49a8-8ce3-a09e7b57b5d3.png">

2. `parseHTML`을 실행할 때 결과

`CFRunLoopRun` 실행으로 인해 메인 큐에 쌓인 dispatch 작업을 먼저 실행하는 모습

<img width="355" alt="image" src="https://user-images.githubusercontent.com/41438361/212314338-d9b00edf-1ae7-4cdb-90be-a0af5b282447.png">

3. `parseHTML` 메서드의 내부를 `Dispatch.main.async` 에서 `NSAttributedString`을 생성하도록 한 메서드로 교체해서 실행한 결과

<img width="366" alt="image" src="https://user-images.githubusercontent.com/41438361/212314508-b41d42fb-a4b2-46c3-a8b3-c62840a9a904.png">

* 참고
* [https://stackoverflow.com/questions/38712772/unable-to-reproduce-webkitlegacy-websafeforwarder-forwardinvocation-crash](https://stackoverflow.com/questions/38712772/unable-to-reproduce-webkitlegacy-websafeforwarder-forwardinvocation-crash)
* [https://github.com/ably/ably-cocoa/issues/667](https://github.com/ably/ably-cocoa/issues/667)
* [https://stackoverflow.com/questions/56154827/nsattributedstring-from-html-on-main-thread-behaves-as-if-multithreading](https://stackoverflow.com/questions/56154827/nsattributedstring-from-html-on-main-thread-behaves-as-if-multithreading)
* [https://web.archive.org/web/20150910101216/http://www.cocoabuilder.com/archive/cocoa/327998-does-initwithhtml-datausingencoding-documentattributes-run-an-event-loop.html](https://web.archive.org/web/20150910101216/http://www.cocoabuilder.com/archive/cocoa/327998-does-initwithhtml-datausingencoding-documentattributes-run-an-event-loop.html)
