---  
layout: post  
title: "Singleton Pattern, 싱글톤 패턴"  
subtitle: ""  
categories: cs
tags: cs-dp singleton
comments: true  
header-img: 

---  
  
> `Singleton Pattern이 무엇인지 알아보자.`  

---


싱글톤 패턴은 이름에서도 대략 유추가 가능하듯이 객체를 딱 하나만 생성(객체의 인스턴스가 오직 1개만 생성)하여 생성된 객체를 프로그램 어디에서나 접근해 사용할 수 있게 하는 패턴이다.

![image](https://user-images.githubusercontent.com/41438361/120912324-13551000-c6c9-11eb-8ae8-2f8264f6f2a2.png)

개발할 때 전역적으로 객체를 하나만 생성해야 하는 경우가 있다. 만약 이를 제제하지 않는다면 객체들이 여러 개로 복제가 되어 여러 개가 생성될 텐데,
싱글톤 패턴을 사용하면 객체 생성을 한 번으로 제한해 객체들이 여러개가 생성되는 경우를 막을 수 있다. 

또한 클래스를 사용하는 여러 곳에서 인스턴스를 계속 생성해 불필요하게 메모리 낭비를 유발할 수 있다는 판단되는 경우에도 싱글톤 패턴을 사용할 수 있다.
한 번의 new 연산자를 통해서 고정된 메모리 영역을 사용하기 때문에 이후 해당 객체에 접근할 때 메모리 낭비를 방지할 수 있다. 또한 이미 생성된 객체를 활용하니 속도 측면에서도 이점이 있다.

다른 클래스 간에 데이터 공유를 쉽게 할 수도 있다. 싱글톤 인스턴스는 전역으로 사용하는 인스턴스이기 떄문에 다른 클래스의 인스턴스들이 접근해 사용할 수 있다. 하지만
이때 동시성 문제가 발생할 수 있으니 잘 설계해야 한다. 

### 싱글톤 패턴의 장점

1. 메모리 낭비 방지
2. 다른 클래스 간의 데이터 공유가 쉽다
3. 인스턴스가 한 개만 존재하는 것을 보장하기 때문에 개발 시 실수가 줄어든다.
4. 싱글톤 객체를 사용하지 않는 경우 인스턴스를 생성하지 않는다.
5. 싱글톤을 상속시킬 수 있다.

### 단점

1. 전역변수보다 사용하기 불편하다. 싱글톤 패턴을 구현하는 코드가 많이 필요하다.
2. 싱글톤의 역할이 커질수록 결합도가 높아져 객체 지향 설계 원칙에 어긋난다.
3. 멀티 쓰레드 환경에서 관리하기 어렵다.
4. 객체의 파괴 시점을 관리하기 어려울 수 있다. 
5. 테스트하기 어렵다. 싱글톤 인스턴스는 자원을 공유하고 있기 때문에 인스턴스의 상태를 초기화시켜줘야 한다.
 
### Singleton 사용하기

싱글톤 패턴에서는 생성자를 해당 싱글톤 패턴을 구현하는 클래스에서만 접근할 수 있게 해줘야 한다. 따라서 private으로 접근 제어자를 설정한다.
외부에서 접근할 수 없게 하는 이유는 해당 클래스를 외부에서 인스턴스화 하면 안되기 때문이다.

#### 이른 초기화

이른 초기화는 클래스가 호출될 때 인스턴스를 생성하는 방법이다. 이 경우 인스턴스를 사용하지 않아도 인스턴스를 생성해 효율성이 떨어진다. 
프로그램을 실행했을 때, 전역에서의 싱글톤 클래스의 객체가 생성되었는지 알 수 없기 때문에 이 객체에 접근할 때 문제가 생길 수 있다.

```java
public class Singleton {

    private static Singleton instance = new Singleton();
	
    private Singleton() {} //생성자를 private로
	
    public static Singleton getInstance() {
        return instance;
    }
}
```

#### 늦은 초기화

늦은 초기화는 인스턴스를 실제로 사용할 시점에 생성하는 방법이다. 인스턴스를 실제로 사용하지 않으면 생성하지 않기 때문에 이른 초기화보다 효율성이 좋지만
두 스레드가 동시에 싱글톤 인스턴스에 접근하고 생성이 안된 것을 확인해 생성한다면 중복으로 생성될 문제가 있다.

```java
public class Singleton {

    private static Singleton instance;
	
    private Singleton () {} //생성자를 private로
	
    public static Singleton getInstance() {
        if (instance == null){
            instance = new Singleton();
        }    
        
        return instance;
    }
}
```

#### 멀티 스레드 환경에서의 늦은 초기화

멀티 스레드 환경에서 늦은 초기화로 인해 중복으로 객체가 생성될 수 있는 문제는 synchronized를 사용해서 여러 쓰레드가
`getInstance()` 메서드에 동시에 접근하는 것을 막을 수 있다. 하지만 이때 많은 쓰레드에서 `getInstance()`메서드에 접근하면
정체현상이 일어나 성능 저하가 생긴다. 

이를 해결하기 위해 아래처럼 Double-Checked Locking 기법을 사용해서 synchronized 영역을 줄일 수 있다. 
첫 번째 if에서 인스턴스 존재 여부를 확인해 인스턴스가 생성되어 있지 않다면 두 번째 if에서 다시 검사할 때 synchronized로 동기화를 시킨다. 이후에
호출 될때는 인스턴스가 이미 생성되어 있기 때문에 synchronized에 접근하지 않는다.

```java
public class Singleton {

    private static Singleton instance;
	
    private Singleton () {} //생성자를 private로

    public static Singleton getInstance() {
        if (instance == null) {
            //synchroized를 활용하여 여러 인스턴스를 생성하는 것을 방지
            synchronized (Singleton.class) {
                if (instance == null)
                    instance = new Singleton();
                }
            }
        return instance;
    }
}
```
