---  
layout: post  
title: "[SOLID] 의존 관계 역전 규칙(DIP), 의존성 주입(DI), 제어의 역전(IoC)"  
subtitle: ""  
categories: cs
tags: solid, dip, di, solid, ioc
comments: true  
header-img: 

---  
  
> `의존성 역전 규칙, SOLID의 의존 관계 역전 규칙, 의존성 주입, 제어의 에 대해 알아보자.`  

---

## 의존성 역전 규칙, DIP

![image](https://user-images.githubusercontent.com/41438361/120591256-cf6acc80-c476-11eb-9508-bae664e2376d.png)

의존 관계 역전 규칙은 SOLID 원칙의 마지막, Dependency Inversion Principle에 해당한다. 디자인 패턴의 전략 패턴에 익숙하다면 개념이 쉽게 다가올 것이다. 이 원칙은 **상위 모듈이 하위 모듈의 구현 내용에 의존하면 안되고,
상위 모듈과 하위 모듈 모두가 추상화된 내용에 의존해야 한다**라는 내용이다. 그러면 여기에서 상위 모듈과 하위 모듈이라는 게 대체 뭘까?

```java
class Programmer {
  private Americano _americano;
  
  public Programmer() {
    _americano = new Americano();
  }
  
  public void drink() {
    _americano.drink();
  }
}

class Americano {
  public void drink() {
    System.out.println("Drink Americano");
  }
}
```

여기 프로그래머와 아메리카노 클래스가 있다. 프로그래머는 커피를 달고 사므로 private 프로퍼티로 Americano를 가지고 있다. 또한 Programmer라는 클래스 내에서 `_americano.drink()`라고 Americano 클래스를 사용하고 있는데, 이렇게 한 클래스 내에서 다른 클래스를 사용할 때 **사용하는 클래스를 상위 모듈, 사용되는 클래스를 하위 모듈**이라고 한다.
여기에서는 Programmer 가 상위 모듈, Americano 가 하위 모듈이 된다. 이때 Americano는 Programmer에 의존 관계가 생기게 된다. 이렇게 상위 모듈인 클래스를 하위 모듈에서 의존하고 있으므로 의존 관계 역전 규칙을 위반하게 된다.
지금 당장은 이 코드가 아무 문제가 없어보일 수 있다.

만약 프로그래머가 아메리카노가 질려서 라떼도 마시고 싶다고 해보자. 그렇다면 위의 코드는 어떻게 바뀌어야 하는가? 

```java
class Programmer {
  private Americano _americano;
  private Latte _latte; // 커피 종류마다 추가해야 하나?
  
  public Programmer() {
    _americano = new Americano();
    _latte = new Latte(); // 프로그래머가 커피 한 잔만 마시고 싶은데 여러 잔의 커피 객체를 할당하게 될 수도 있다.
  }
  
  public void drink(string coffeeType) { // 아마 어떤 종류의 커피를 마실지 선택하게 해야 할 수도 있다.
    if "Americano".equals(coffeeType) 
      _americano.drink();
    else:
      _lattee.drinkLatte(); // 커피 종류마다 drink~ 메서드의 이름이 다르다면? 관리하기 어려울게 뻔하다.
  }
}

class Americano {
  public void drink() {
    System.out.println("Drink Americano");
  }
}

class Latte {
  public void drinkLatte() { 
    System.out.println("Drink Latte");
  }
}
```

이렇게 하위 모듈에서 상위 모듈에 의존하게 되면 아래의 문제들이 발생한다.

1. 의존 관계를 갖는 하위 모듈이 많아질 경우(라떼 말고도 카페모카, 아인슈페너 등등의 메뉴가 추가될 경우) 이를 상위 모듈에서 관리하기가 어렵다.(drink 메서드가 복잡해짐, 커피들의 마시는 메서드의 이름이 통일되어 있지 않음, 커피 종류마다 프로그래머 클래스에 멤버 변수 추가)
2. 하위 모듈에서 변경 사항이 있을 경우 이를 수정하기가 쉽지 않다.(라떼만 마시는 메서드 이름이 변할 경우 라떼, 프로그래머 클래스에서 사용되는 마시는 메서드 이름을 바꿔야 한다. 만약 모든 커피 클래스에 사이즈업 메서드가 추가된다면...?)

당장 생각나는 변경사항만 요정도인데도 이를 구현할 생각을 하니 귀찮기 짝이 없을 수가 없다. 이런 문제는 하위 모듈(커피)는 변할 수 있는, 자주 변하는 것인데 하위 모듈에서 상위 모듈에 의존하기 때문에 하위 모듈에 변경이 일어날 때마다 상위 모듈도 같이 수정해야 하기 떄문이다. 이는 코드의 **유지보수성**과 **재사용성**을 떨어뜨린다.

DIP는 이 문제를 해결해줄 수 있는 원칙이다. DIP는 **시스템 유지보수성**과 **재사용성**에 대한 원칙이다. 위에서처럼 하위 모듈에서 수정사항이 없으면 되지 않냐라고 생각할 수 있지만 그럴 일은 절대로 없다라고 생각하면 된다. 코드를
짤 때 수정을 하지 않는 경우는 거의 없고, 아마 대부분의 개발자는 이런 수정사항들에 유연하고 탄력적인 디자인을 적용하고 싶을 것이다. 변경 사항에 유연하게 대처하기 위해 아래와 같은 사항들이 지켜져야 한다.

1. 코드의 중요한, 핵심 부분은 바뀌지 않아야 한다. (위의 경우 커피 클래스들의 drink 메서드가 될 것이다.)
2. 변화가 자주 일어나는 코드에서의 의존 관계를 없앤다. (위의 경우 프로그래머 클래스에서 커피의 종류가 많아질 경우 바뀌는 부분들이 될 것이다.)

이를 구현하기 위해서는 DIP의 상위 모듈과 하위 모듈은 **추상화 된 것**에 의존해야 한다는 개념을 적용해야 한다. 즉 추상화 된 무언가가 필요하다. 추상화를 시켜야 하는 것은 **"자주 변경되는가?"**를 따져보면 된다.

위의 예제를 다시 보면 커피는 메뉴가 많아질 수도 있고, 그 안의 메서드들이 바뀔 수도 있었다. 즉 하위 모듈에 변경 사항이 일어나고, 그에 비해 프로그래머는 커피를 마시기만 하면 되니 변경 사항이 없었다. 따라서 이 커피들을 아래와 같이 추상화한다.

```java
interface Coffee { // 커피라는 인터페이스를 만든다. Swift에서는 프로토콜, C#같은 곳에서는 추상 클래스나 인터페이스가 될 것이다.
  public void drink();
}

class Americano implements Coffee {
  @Override
  public void drink() { 
    System.out.println("Drink Americano");
  }
}

class Latte implements Coffee {
  @Override
  public void drink() {
    System.out.println("Drink Americano");
  }
}

```

우선 이 커피들만 놓고 봤을 때 장점은 커피들에 메서드가 추가되거나 기존 메서드가 수정이 되는 경우 인터페이스에 이를 반영해서 커피를 구현하는 하위 클래스들에서 메서드를 통일 시켜 줄 수 있다는 것이다. 이제
이 추상화된 커피들을 이용해서 프로그래머가 커피를 골라 마셔보도록 하겠다.

```java
class Programmer {
  private Coffee _coffee;
  
  public Programmer(Coffee coffee) { // 1.
    _coffee = coffee;
  }
  
  public void drink() { // 2.
    _coffee.drink();
  }
}
```

추상화를 적용하기 전의 프로그래머 클래스 코드랑 크게 두 가지가 달라졌다.

1. 외부에서 커피를 주입받았다. (위처럼 외부에서 객체를 생성해서 넣어주는 것을 주입이라고 한다.) 이렇게 외부에서 의존성을 주입하는 것을 **의존성 주입(Dependency Injection)**이라고 한다.
2. drink 메서드에서 커피 종류를 확인하지 않아도 된다. 어떤 종류의 커피가 들어오든 `_coffee.drink()` 한 줄로 끝이 난다.

커피의 종류마다 같은 동작을 하는 메서드들도 통일되었고, 인터페이스로 인해 공통되는 동작을 관리하기도 쉬워졌다. 또한 
이제 커피의 메뉴를 무한정 추가시켜도 프로그래머의 클래스는 변경되지 않을 것이다. 그저 아래와 같이 외부에서 의존성 주입을 통해 커피 객체를 생성한 후 주입하면 된다.

```java
// Americano, Latte, ... , Einspener 등등 굉장히 많은 커피 메뉴가 있다고 해보자.

Programmer sleepyProgrammer = new Programmer(new Einspener()); // 의존성 주입
sleepyProgrammer.drink(); // 끝!
```

너무 보기도 좋아졌고 유지보수 하기도 쉬워졌다. 의존성 주입이라고 했는데 의존성 주입은 왜 의존성 주입이라고 하는가?

맨 처음에 의존성 관계가 생겼다고 했을 때 이를 말했던 코드는 `_americano = new Americano()`였다. 이는 다시 말해 프로그래머 클래스 안에 `Americano _americano = new Americano`로 
하위 모듈 객체를 가지게 된다는 것과 같은 의미라고 볼 수 있다. 이 코드로 인해 의존성이 생기게 된것인데, 바로 위의 코드를 다시 보자. 아래 코드는 바로 위에 나온 코드 똑같은 동작을 하는 코드다.

```java
Einspener es = new Einspener(); // dependency
Programmer sleepyProgrammer = new Programmer(es); // dependency 주입
sleepyProgrammer.drink(); 
```

프로그래머 클래스가 아닌 외부의 다른 곳에서 dependency를 만들고 이를 주입해서 의존성 주입이라고 하는 것이다.

### IoC

이제 DIP와 DI가 무엇인지도 알겠는데 IoC가 문제다. IoC는 우리말로 제어의 역전으로, 말 그대로 제어를 역전하는 것이다. 코드를 먼저 보자.

```java
Lamp lamp = new Lamp(new Bulb());
```

위 코드는 의존성 주입을 통해 전구를 램프에 끼워 램프를 생성한 것이다. 그런데 이를 실행시키는 주체가 client쪽이라고 생각해보자. 클라이언트 쪽에서는 램프 안에 조명으로 전구가 들어가는지, 혹은 led나 다른 무엇이 들어가는지
전혀 알 필요가 없고, 알아서는 안된다고 하자. 하지만 위의 코드에서는 클라이언트가 전구도 알아야 하고, 램프에 전구의 관계를 설정하는 상황이 되었다. 이런 코드는 클라이언트쪽에서 모든 것을 제어하고 있는 구조다.

IoC는 말 그대로 이 제어를 역전시키는 것이다. IoC에 대해 검색하다가, 어떤 분의 블로그에서 IoC는 "대신 해줌"이라는 비유를 보았는데 어울리는 말이라고 생각한다. IoC를 알려면 컨테이너의 개념을 알아야 한다.
컨테이너란 내가 작성한 코드의 처리과정을 위임받은 독립적인 존재다. 즉 설정만 잘 되어있다면 프로그래머가 작성한 코드를 컨테이너가 스스로 알아서 참조한 뒤 알아서 객체의 생성과 소멸을 관리해주는 것이다. 

![image](https://user-images.githubusercontent.com/41438361/120603554-21b3e980-c487-11eb-8489-a892a5cbbc64.png)

위의 그림에서 확인할 수 있듯이, IoC 컨테이너에 모든 관계 설정(위의 경우 램프와 전구)에 대한 책임을 위임한다.

![image](https://user-images.githubusercontent.com/41438361/120603300-d26db900-c486-11eb-9529-f769c925dd6d.png)

클라이언트 쪽에서 IoC 컨테이너에 필요한 객체(램프)를 요청하면 IoC 컨테이너는 전구를 생성하고 관계를 설정해서 이를 전달한다. 이제 클래스들은 다른 클래스에 대한 의존성이 모두 사라졌으므로 모든 클래스는 컴포넌트가 될 수 있고, 이는
클래스들이 전부 mock으로 대체될 수 있어 테스트하기 쉬워졌음을 의미한다. 



dddsdf


참고한 링크

* https://blog.ndepend.com/solid-design-the-dependency-inversion-principle-dip/
* https://pks424.tistory.com/entry/IoC-DI%EB%9E%80
* https://develogs.tistory.com/19
