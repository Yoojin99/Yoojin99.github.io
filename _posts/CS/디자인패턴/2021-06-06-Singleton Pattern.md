---  
layout: post  
title: "Singleton Pattern, 싱글턴 패턴"  
subtitle: ""  
categories: cs
tags: cs-dp singleton
comments: true  
header-img: 

---  
  
> `Singleton Pattern은 세상에서 단 하나뿐인 특별한 객체를 만드는 디자인 패턴이다. 싱글턴 패턴이 무엇인지 알아보자.`  

---

# Singleton Pattern

**Singleton Pattern은 인스턴스가 하나 뿐인 객체를 만들게 하는 디자인 패턴이다.** 

인스턴스가 하나밖에 없기 때문에 클래스 다이어그램도 아래와 같이 클래스가 덩그러니 하나만 존재한다.

![image](https://user-images.githubusercontent.com/41438361/173220035-242a3694-32bc-49e8-a69b-214a289f8ae0.png)

간단해 보이지만, 클래스의 인스턴스가 오로지 한 개만 존재할 수 있는 방법을 생각하면 그리 간단하지 않다.

## Why

인스턴스가 하나만 필요한 이유나 필요한 상황은 무엇일까? **객체가 하나만 있어도 될 때다.** 

![image](https://user-images.githubusercontent.com/41438361/173220114-819ba919-8954-4880-8231-d511c5436346.png)

스레드 풀, 캐시, 대화상자, 사용자 설정, 프린터 같은 디바이스를 위한 디바이스 드라이버는 하나만 존재해도 되거나, 하나만 존재해야하는 경우에 해당한다. 이런 객체를 두 개 이상 만들면

1. **프로그램이 이상하게 동작한다.**
2. **자원을 불필요하게 많이 소모한다.**

와 같은 문제가 생길 수 있다.

싱글턴 패턴은 객체를 오직 하나만 만들어서 이런 문제를 예방한다. 또한 **전역 변수를 사용할 때와 같이 객체 인스턴스를 어디서든지 접근할 수 있게 한다.** 전역 변수의 경우 애플리케이션이 시작되는 시점에 객체가 생성되어 끝날때까지 사용이 되지 않는다면 자원을 불필요하게 소모만 하는 경우가 있겠지만 싱글턴 패턴을 사용하면 필요할 때만 객체를 만들 수 있다.

## How

클래스의 인스턴스가 하나만 존재하도록 만드려면 어떻게 해야 할까? Swift를 사용해서 볼 것이다.

### 객체 생성

우리는 클래스 인스턴스를 생성할 때 아래와 같이 클래스의 생성자를 이용해서 객체를 생성한다. 아래 코드는 객체를 여러개 생성할 수 있다.

```swift
class Singleton {
    // 생성자	
    init() {}
}

let singleton1 = Singleton()
let singleton2 = Singleton()
let singleton3 = Singleton()
```

### 클래스를 private하게?

만약 클래스 자체를 `private` 하게 만들면 해당 클래스를 정의하고 구현한 범위 내에서만 사용할 수 있기 때문에 아래와 같이 `singleton` 변수에 `private`이나 `fileprivate`를 붙이지 않으면 접근제어 관련된 컴파일 에러가 뜬다.

<img width="798" alt="image" src="https://user-images.githubusercontent.com/41438361/173220894-3d05ecf0-d79c-40de-81ee-c743ca5697ae.png">

클래스를 private하게 만들어도 인스턴스를 하나만 생성할 수 있는 건 아니다.

```swift
private class Singleton {
    // 생성자
    init() {}
}

private let singleton1 = Singleton()
private let singleton2 = Singleton()
private let singleton3 = Singleton()
```

### 생성자를 private하게

이니셜라이저, 생성자를 private하게 만들면 어떻게 될까?

<img width="794" alt="image" src="https://user-images.githubusercontent.com/41438361/173220989-79d25142-afa5-4fd7-af0b-8b276c17efb5.png">

클래스의 이니셜라이저가 private 접근 레벨 때문에 접근할 수 없다는 컴파일 에러가 발생한다. 따라서 저 클래스 밖에서는 이니셜라이저에 접근할 수 없기 때문에 **외부에서 인스턴스를 만들 수 없는 클래스가 됐다.**

위에서도 말했지만, `private` 접근 레벨을 설정하면 해당 이니셜라이저를 정의하고 구현한 클래스 `Singleton` 내에서만 이니셜라이저에 접근하고 호출할 수 있다.

### 클래스 내부에 인스턴스를 생성

코드를 먼저 보자.

```swift
class Singleton {
    // 생성자
    private init() {}
    
    public static func getInstance() -> Singleton {
        // return singleton instance
    }
}
```

위 코드에서는 `getInstance()` 라는 정적 메서드를 정의했다. 이 메서드는 Singleton 객체를 리턴하도록 정의됐다. 정적(타입) 메서드이기 때문에 외부에서 이 메서드를 호출하려면 `Singleton.getInstance()`와 같이 호출해야 한다. 

위 코드에서 코드를 조금 추가했다.

```swift
class Singleton {
    // 생성자
    private init() {}
    
    public static func getInstance() -> Singleton {
    	// private initializer에 접근할 수 있다.
        return Singleton()
    }
}

Singleton.getInstance()
```

`getInstance()`라는 정적(타입) 메서드에서 클래스의 private한 이니셜라이저를 호출해 인스턴스를 생성했다. 이렇게 하면 외부에서는 클래스의 인스턴스를 만들지 못하지만, 클래스 내부에서 인스턴스를 생성해서 반환할 수 있는 것이다.

### 객체를 하나만 만들기

이제 이 `getInstance` 정적 메서드를 사용해서 클래스의 인스턴스를 하나만 만들 수 있다.

```swift
class Singleton {
    // 유일한 인스턴스를 저장할 변수
    private static var uniqueInstance: Singleton?
    
    // private하기 때문에 클래스 내부에서만 접근 가능
    private init() {}
    
    public static func getInstance() -> Singleton {
    	// uniqueInstance가 nil(null)이면 인스턴스를 만들어 할당한다
        if uniqueInstance == nil {
            uniqueInstance = Singleton()
        }
        
	// 이 시점에서 uniqueInstance는 항상 nil(null)이 아니기 때문에 객체를 반환하면 된다.
        return uniqueInstance!
    }
}
```

유의할 점은, `Singleton`의 `uniqueInstance` 변수에는 실제로 `getInstance()` 메서드가 호출되기 전까지 아무런 값도 존재하지 않는다는 점이다. 이렇게 인스턴스가 필요한 상황이 오기 전까지 인스턴스를 생성하지 않는 것을 **lazy instantiation, lazy initialization**이라고 한다.

이제 `getInstance()` 메서드로 유일한 하나의 객체를 호출할 수 있다. 이 객체는 유일하기 때문에 한 애플리케이션의 어디서라도 똑같은 객체에 접근해 사용할 수 있다. 객체가 하나밖에 없기 때문에 스레드 풀과 같은 자원 풀을 관리하는데 유용하다. 

사실 위 코드도 안전하지 않다. 실수로 객체 인스턴스가 여러 개 생기면서 버그가 생길 수도 있고, 여러 곳에서 한 인스턴스에 동시 접근하기 때문에 예상치 못한 동작을 할 수도 있다. 

## 문제 개선

객체를 하나만 만들겠다고 위의 임시 싱글턴 패턴 코드를 사용하면 잘못해서 객체가 여러 개가 생길 수도 있다. 가령 멀티쓰레드 환경에서 `getInstance()` 코드를 실행했다고 생각해보자. 멀티쓰레드 환경에서는 쓰레드가 CPU를 번갈아 사용하면서 돌아가며 코드를 실행할 것이다.

<img width="964" alt="image" src="https://user-images.githubusercontent.com/41438361/173222100-08621efa-3554-4ca0-9335-087f015731ea.png">

이렇게 멀티쓰레드 환경에서는 문제가 생긴다. `getInstance()` **메서드를 동기화**시키면 멀티쓰레딩 문제가 해결된다. `getInstance()` 코드 영역에는 한 쓰레드만 접근할 수 있도록 하는 것이다. 

잠시 자바로 `getInstance()` 를 동기화하는 걸 보면 아래와 같이 하면 된다.

```java
public class Singleton {
	private static Singleton uniqueInstance;
	
	private Singleton() {}
	
	// synchronize method
	public static synchronized Singleton getInstacnce() {
		if (uniqueInstance == null) {
			uniqueInstance = new Singleton();
		}
		
		return uniqueInstance;
	}
}
```

위처럼 메서드 전체를 동기화 시키는 것도 방법이지만, 사실 **동기화가 필요한 부분은 메서드를 초반에 접근할 때 뿐이다.** 한 번 `uniqueInstance`에 객체가 할당되고 나서는 메서드를 동기화한 상태로 둘 필요가 없다. 따라서 오버헤드가 증가할 수 있는 것이다.

이 문제를 더 효율적으로 해결할 수 있는 방법을 보자.

### getInstance() 의 속도가 중요하지 않다면 그냥 둔다.

그냥 두는 것도 하나의 방법이다. 다만 메서드를 동기화하면 성능이 저하되기 때문에 만약 성능상 이슈가 있다면 다른 방법을 적용해야 한다.

### 인스턴스를 처읍에 만들어버린다.

애플리케이션에서 인스턴스를 처음에 생성해버리고 계속 그 인스턴스를 사용하는 방법이 있다. 다시 자바 코드를 보면 아래와 같다.

```java
public class Singleton {
	private static Singleton uniqueInstance = new Singleton();
	
	private Singleton() {}
	
	// synchronize method
	public static synchronized Singleton getInstacnce() {
		return uniqueInstance;
	}
}
```

## Swift는 문제 없다

위에서 멀티 쓰레딩 환경에서 발생할 수 있는 문제와 해결할 수 있는 방법을 봤다. 굳이 swift로 코드를 작성하지 않았던 이유는 swift는 위에서 봤던 이유를 신경쓰지 않을 수 있기 때문이다.

swift는 단순히 아래와 같이 코드를 작성하면 된다.

```swift
class Singleton {
    
    static let uniqueInstance = Singleton()
    
    // MARK: init
    private init() {}
}
```

[애플 공식문서](https://developer.apple.com/swift/blog/?id=7)를 보면 아래와 같이 나와있다.

<img width="887" alt="image" src="https://user-images.githubusercontent.com/41438361/173223022-d1f3439d-6b6c-4372-9755-6da096fa8ca8.png">

**Static 변수는 처음 접근되었을 때 lazy하게 초기화되고, 멀티쓰레드 환경에서도 thread-safe하게 동작한다.** 따라서 java와 같이 동기화를 하거나 double-checking locking과 같은 걸 할 필요가 없다.

이제 Apple 공식문서에서 Singleton Pattern을 어떻게 적용하는지도 문서화해서 읽어보면 좋을 것 같다. (https://developer.apple.com/documentation/swift/managing-a-shared-resource-using-a-singleton)

* 참조
* Head First Design Patterns (한빛 미디어)
