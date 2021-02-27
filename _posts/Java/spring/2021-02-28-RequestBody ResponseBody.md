---  
layout: post  
title: "@RequestBody @ResponseBody"  
subtitle: ""  
categories: java
tags: java-spring requestbody responsebody
comments: true  
header-img: img/dev/spring/spring.jpg
---  
  
> `@RequestBody와 @ResponseBody가 무엇인가?`  

---

Spring boot로 웹을 개발하면서 json형태의 데이터를 주고 받는 경우가 많이 생겼다. 보통 rest에서는 xml과 json을 이용해 데이터를 주고 받는 것이 일반적이다.

Spring MVC도 클라이언트에서 전송한 JSON/XML/기타 데이터를 컨트롤러에서 DOM 객체나 자바 객체로 변환해서 송수신할 수 있다.

`@RequestBody`와 `@ResponseBody`는 스프링에서 비동기 처리를 하는 경우 사용한다.

## 클라-서버의 비동기 통신 처리

웹에서 데이터를 가져오거나 전송하는 작업들은 클라리언트와 서버의 통신이 이루어지면서 일어난다. 여기서 중요한 것이 바로 요청(Request)와 응답(Response)이다.

즉 클라리언트 -> 서버 로 통신하는 메세지를 요청 메세지, 서버 -> 클라이언트 로 통신하는 메세지를 응답 메세지라고 한다. 

웹에서 화면 전환(refresh, F5) 없이 이루어지는 동작들은 대부분 비동기 통신으로 이루어진다. 비동기 통신을 하기 위해서는 클라가 서버로 요청 메세지의 본문에
데이터를 담아서 보내고, 마찬가지로 서버도 클라로 응답을 보내기  위해 데이터를 응답 메세지의 본문에 담아 보낸다. 이 본문이라는 것이 바로 Body로,
요청 본문(RequestBody)와 응답 본문(ResponseBody)가 여기서 나왔다.

본문에 담겨야 하는 데이터 형식은 일반적으로 JSON(JavaScript Object Notation)이다. JSON은 속성-값(attribute-value)쌍으로 이루어진 데이터 객체를
전달하기 위해 사람이 읽을 수 있는 텍스트를 사용하는 개방형 표준 포맷이다. 


**`@RequestBody`와 `@ResponseBody`는 각각 HTTP 요청 몸체를 자바 객체로 변환하고 자바 객체를 HTTP 응답 몸체로 변환하는데 사용한다.**

## `@RequestBody`

HTTP 요청 몸채를 자바 객체로 전달받는다. HTTP 요청의 body 내용을 자바 객체로 매핑하는 역할을 한다. 이때 HttpMessageConverter를 사용하는데
여러 컨버터가 존재한다. 

요청이 JSON으로 들어온 경우 요청 헤더(Request header)에 컨텐츠 타입(Content-Type)을 알려줘야 한다. 그래야 이 타입을 보고 JSON을 컨버팅 할 수 있는
컨버터를 사용해서 데이터를 자바 객체로 변환하게 되는 것이다.

예제는 아래와 같다.

```
@RestController
@RequestMapping("/api")
public class HttpMessageController {

  /**
  * @RequestBody를 통해 자바 객체로 변환할 때 HttpMessageConverter를 사용하여 
  * 헤더에 담긴 컨텐츠 타입을 보고 어떤 메시지 컨버터를 사용할 것인지 판단하여 
  * 요청 본문에 담긴 값을 자바 객체로 변환
  */
  @GetMapping
  public String create(@ReqeustBody Event event) {
    // 생략
    return "redirect:/api/list";
  }
  
}
```

Jackson2ObjectMapperBuilder는 스프링부트의 JacksonAutoConfiguration 클래스에서 자동으로 설정하기 때문에 별다른 설정 없이 JSON을 
자바 객체로 변환하는 ObjectMapper를 사용할 수 있다. 이 컨버터의 내부 구현을 보면 `autoDetectGettersSetters()`라는 메서드를 사용하는데,
JSON 타입으로 변환하기 위한 객체(DTO)에 getter, setter method가 존재해야 한다는 것이다.

스프링 부트의 pom.xml 파일을 보면 `spring-boot-starter-web`가 존재하는데, 이 안에 `jackson-databind` 의존성이 존재하는 것을 볼 수 있다.
이 의존성이 있어야 JSON 타입으로 변환이 된다. 


## `@ResposeBody`

자바 객체를 HTTP 응답 몸체로 전송한다. 자바 객체를 HTTP 요청의 body 내용으로 매핑하는 역할을 한다.


출처
https://webdevtechblog.com/reqeustbody%EC%99%80-responsebody-%EC%96%B8%EC%A0%9C-%EC%82%AC%EC%9A%A9%ED%95%A0%EA%B9%8C-2efcab364edb
