---  
layout: post  
title: "Spring boot 패키지 구조"  
subtitle: ""  
categories: java
tags: java-spring controller service dao
comments: true  
header-img: img/dev/spring/spring.jpg
---  
  
> `spring boot는 대체 어떻게 돌아가는가? 어떤 구조로 돌아가는가?`  

---

spring boot 실습을 해봤지만 너무 빨리 지나간 터에 아직도 어떻게 내가 작성한 코드들이 돌아가는지 기본적인 구조도 이해하지 못했다.

그래서 검색하다가 찾은 블로그에서 나온 아주 간단한 예시로 대강의 구조를 파악하고 어떻게 코드를 작성, 구성해야 하는지 훑어보겠다.

일단 아래의 예제에서는 spring boot 내의 패키지 구조(Controller - service - dao)가 간략하게 나와있고, 여기에서 데이터 crud는 jpa를 사용해 처리한다.

## JPA?

**JPA(Java Persistence API)는 Java를 통해 데이터베이스와 같은 영속 계층을 처리하고자 하는 스펙이다.** JPA를 이해하기 위해서는 ORM(Object Relational Mapping)이라는 기술을 알아야 한다.

### ORM

ORM(Obejct Relational Mapping)은 단어에서 볼 수 있듯이 객체지향과 관련이 있다. ORM을 간단히 말하면 **객체지향 패러다임을 관계형 데이터베이스에 보존하는 기술**이라고 할 수 있다.
즉 객체지향 패러다임을 관계형 패러다임을 매핑해주는 개념이라고 할 수 있다.

정리했는데도 말이 어렵다. 객체지향 패러다임을 관계형 데이터베이스에 보존한다는 게 무슨 말인가? ORM의 시작은 '객체지향의 구조'가 '관계형 데이터베이스'와 유사하다는 점에서 시작됐다.

데이터 베이스를 보면 예를 들어 Member라는 테이블 안에 id, pw, name이라는 값들이 있을 것이고, 이를 클래스로 옮기면 Member 클래스 안에 id, pw, name이라는 변수가 존재할 수 있다.
이렇듯 테이블을 설계하는 것과 클래스를 설계하는 일은 매우 유사하다. 

클래스에서는 인스턴스를 생성해 이 공간에 데이터를 저장하는데, 테이블에서는 한 row마다 데이터를 저장한다. '관계'와 '참조'도 비슷하다. 관계형 데이터베이스는
테이블 사이의 관계를 통해 구조적인 데이터를 표현한다면, 객체지향에서는 '참조'를 통해 객체간의 관계를 표현한다. 이렇듯 객체지향과 관계형 데이터베이스는 유사한 특징을 가지고 있다.

ORM은 객체지향과 관계형 사이의 변환 기법을 의미하기 때문에 특정 언어에 국한되지 않는다. 

다시 JPA로 돌아가서, JPA는 ORM을 java 언어에 맞게 사용하는 '스펙'이다. 그래서 ORM이 JPA의 상위 개념이 된다.

이제 spring boot에서 패키지가 어떻게 구성되어있는지 보겠다. 

* Controller (사용자의 입력을 받아 처리/들어온 입력을 바탕으로 서비스단의 특정 서비스 요청, 뷰 보여주는 관리자의 역할)
  * HomeController.java
  * UserController.java
* repository
  * UserRepository.java
* entity
  * UserEntity.java
* service (dto/데이터 객체를 가지고 연산을 하거나 다른 처리 작업을 하는 애들)
  * UserService.java

보통 여기에 dto(service단에서 주고받기 위한 데이터 객체)를 추가하기도 한다. 엔티티 객체를 영속 계층 바깥쪽에서 계속 사용하기 보다는 DTO를 이용하는 것이 권장된다. 우편물 정도의 개념이라고 생각하면 된다.
* dto
  * UserDto.java
  
위의 구조를 보면 `User` -> `Controller`(요청 받음. 요청 받은 것을 토대로 서비스에게 특정 작업 요청/뷰 보여줌) -> (`Service` 단에서 데이터 연산 등 `repository`의 메서드를 활용해 데이터 crud를 한다. 이 과정에서 사용되는 객체가 dto/entity. 여기 과정은 생략될 수 있음) 

참고로 Controller는 식당 알바생 정도라고 생각하면 된다. 서비스 단이 할 일(데이터 연산/처리)까지 Controller단에 다 넣어버리면 식당 알바생에게 나가서 주문도 받고 요리도 하고 청소도 하고 다 하라는 셈이 된다. 따라서 비즈니스 로직과 같은 서비스 단에서 할 작업들을 Controller에게 맡기지 말자.

## UserEntity

데이터베이스 테이블의 **스키마**라고 할 수 있다.

```java
@Entity
@Table(name = "user")
@Getter
@Setter
public class UserEntity {
  @Id
  private String userId;
  
  private String password;
  
  private String userNm;
}
```

Spring Data JPA를 이용해서 개발하기 위해 필요한 종류의 코드는 두 가지이다.

1. JPA를 통해 관리하게 되는 객체(Entity Object를 위한 **Entity Class**)
2. 엔티티 객체들을 처리하는 기능을 가진 **Repository** (엔티티를 가지고 crud 하는 메서드 가진 애들)

레포지토리는 뒤에서 보기로 하고, 위에 나온 코드의 여러 어노테이션에 대해 좀 더 자세히 알아보도록 하겠다.

### `@Entity`

엔티티 클래스는 Spring Data JPA에서는 반드시 `@Entity`라는 어노테이션을 추가해야 한다. 이 어노테이션은 해당 클래스가 엔티티를 위한 클래스이고,
이 클래스의 인스턴스들이 JPA로 관리된는 엔티티 객체라는 것을 의미한다.

또한 이 어노테이션이 붙은 클래스는 옵션에 따라 자동으로 테이블 생성이 가능하다.

### `@Table`

말 그대로 데이터베이스상에서 엔티티 클래스를 어떤 테이블로 생성할 것인지에 대한 정보를 담는다. 위 코드 같은 경우는 생성되는 테이블 이름은 'user'가 된다.
테이블의 이름뿐만 아니라 인덱스 등을 생성하는 기능도 지원한다.

### `@Id`, `@GeneratedValue`

엔티티 어노테이션이 붙은 클래스는 PK(Primary Key)에 해당하는 특정 필드를 `@Id`로 지정해야 한다. 이 `@Id`가 사용자가 입력하는 값을 사용하는 게 아니면
자동으로 생성되는 번호를 사용하기 위해 `@GeneratedValue`라는 어노테이션을 활용한다.

키 생성 전략은 다음과 같다.

* AUTO(default) : JPA 구현체(스프링 부트에서는 Hibernate)가 생성 방식을 결정
* IDENTITY : 사용하는 데이터베이스가 키 생성을 결정(MySQL/MariDB의 경우 auto increment 채용)
* SEQUENCE : 데이터베이스의 sequencefmf dldydgotj zlfmf todtjd
* TABLE : 키 생성 전용 테이블을 생성해서 키 생성

### `@Column`

만약 추가적인 필드가 필요할 경우, 이 어노테이션을 이용해서 다양한 속성을 지정할 수 있다. 주로 nullable, name, length등을 이용해서 데이터베이스 칼럼에 필요한 정보를 제공한다.

```java
@Column(length=200, nullable=false)
private String memoText;
```

### `@Getter`, `@Builder`

Lombok의 `@Getter`를 이용해서 Getter 메서드를 생성하고 `@Builder`를 이용해서 객체를 생성할 수 있게 처리한다. `@Builder`를 사용하기 위해서는
`@AllArgsConstructor`와 `@NoArgsConstructor`를 항상 같이 처리해야 컴파일 에러가 발생하지 않는다.

Lombok얘기가 나와서 잠깐 lombok이 뭔지도 보고 넘어가겠다. 

기본적으로 웹 애플리케이션에서 사용하는 VO 객체는 DB 테이블의 column과 같은 이름의 private 변수를 가지고, getter, setter 메소드를 정의한 후 toString 메소드를 정의한다.
하지만 프로젝트가 커질수록 이런 변수들, 메소드들이 굉장히 양이 많아지면서 유지보수가 힘들어지는 문제가 생긴다.

이런 문제를 해결한 라이브러리가 lombok이고, 위의 문제들을 자동으로 처리해준다.

## UserRepository

JpaRepository<T, ID>인터페이스를 상속받는다. T에는 미리 만든 엔티티를 넣고, ID에는 타입을 넣는다. 이렇게 설정하면 UserRepository 인터페이스로
기본적으로 미리 만들어진 데이터베이스 API를 사용할 수 있게 된다. 미리 생성된 API를 사용함으로써 직접 쿼리를 작성할 일이 줄어들게 된다.

```java
@Repository
public interface UserRepository extends JpaRepository<UserEntity, String> {
  
}
```

위의 UserRepository 클래스 안에 crud 작업을 조건에 맞게 처리하는 쿼리문을 다양하게 구성해 넣을 수 있다. 지금 아무 메서드도 없는 이유는 밑의 Service에서 보겠지만, 메서드 이름만으로도 JPQL 쿼리문을 실행시킬 수 있기 때문이다. 만약 복잡한 쿼리문을 따로 생성해서 실행하고 싶다면 여기에 추가적인 메서드를 작성하면 된다.

Spring Data JPA는 JPA의 구현체인 Hibernate를 이용하기 위한 여러 API를 제공한다. 그 중에서 가장 많이 사용되는 것이 JpaRepository라는 인터페이스이다.
Spring Data JPA에는 여러 종류의 인터페이스의 기능을 통해 JPA 관련 작업을 별도의 코드 없이 처리할 수 있게 하는데, 그게 위에서 언급한 대로 직접
쿼리를 작성할 일이 줄어들게 된다는 말이다. 예를 들어 CRUD 작업, 페이징, 정렬과 같은 처리도 인터페이스의 메서드를 호출하는 형태로 처리할 수 있다.

참고로 JpaRepository의 상속 구조는 다음과 같다.

`JpaRepository` -> `PagingAndSortRepository` -> `CrudRepository` -> `Repository`

일반적인 기능만을 사용할 때는 CrudRepository를 사용하는 것이 좋지만 틀벽하지 않는 한 JpaReposiotry를 사용한다.

JpaRepository는 인터페이스고, 이를 상속하는 것만으로도 모든 처리를 끝낼 수 있다. Spring Data JPA는 인터페이스 선언만으로도 자동으로 스프링의 빈으로 등록된다.

## UserService

getUser는 아이디로 User 테이블을 조회해서 정보를 리턴하고, setUser는 User 테이블에 데이터를 저장하는 기능을 수행한다. 서비스 단에서는 repository에 접근해서 데이터의 crud 작업을 
상황에 맞게 실행하거나, 데이터를 처리하는 역할을 한다.

```java
@Service
public class UserService {
  @Autowired
  private UserRepository userRepository;
  
  public UserEntity getUser(String userId, String password) {
    return userRepository.findById(userId).get; 
  }
  
  public UserEntity setUser(UserEntity user) {
    return userRepository.save(user);
  }
}
```

웹 어플리케이션을 제작할 때는 HttpServletRequest나 HttpServletResponse를 서비스 계층으로 전달하지 않는 것을 원칙으로 한다.
유사하게 엔티티 객체가 JPA에서 사용하는 객체이므로 JPA 외에서 사용하지 않는 것이 권장된다.(엔티티 객체를 서비스단에서 사용하지 말고 DTO를 사용하라는 소리다.)

## UserController

데이터를 어떤 형태의 자료구조로 받을지는 코더의 자유다. 여기에서는 별다른 데이터 처리를 하지 않기 위해 Entity로 받았다.
아래와 같이 코드를 작성하면 클라이언트에서 body안에 userId, password, userNm 필드를 보내면 받을 수 있다.

```java
@RestController
@RequestMapping("user")
public class UserController {
  @Autowired
  private UserService userService;
  
  @PostMapping("signin")
  public UserEntity signin(@RequestBody UserEntity reqBody) {
    String userId = reqBody.getUserId();
    String password = reqBody.getPassword();
    UserEntity user = userService.getUser(userId, password); // 서비스 단에 작업 처리를 요청해서 데이터를 받았다.
    return user;
  }
  
  @PostMapping("signup")
  @ResponseStatus(HttpStatus.CREATED)
  public UserEntity signup(@RequestBody UserEntity reqBody) {
    UserEntity user = userService.setUser(reqBody);
    return user;
  }
}
```


출처
* lombok : https://medium.com/@dlaudtjr07/spring-boot-lombok-%EA%B0%9C%EB%85%90-%EB%B0%8F-%EC%84%A4%EC%B9%98-71f9dbbc2f42
* 코드로 배우는 스프링 부트 웹 프로젝트 책 by 구멍가게 코딩단
* https://gofnrk.tistory.com/16
