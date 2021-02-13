---  
layout: post  
title: "Spring - Controller"  
subtitle: ""  
categories: java
tags: java-spring
comments: true  
header-img: img/dev/spring/spring.jpg
---  
  
> `controller는 무엇인가?`  

---

## MVC

Controller를 얘기하기 전에 간단히 MVC를 짚고 넘어가겠다.

**MVC는 하나의 디자인 패턴이다.** 그 중에서도 스프링 MVC는 **스프링이 제공하는 웹 어플리케이션 구축 전용 MVC 프레임워크**다.

* 모델 : 비즈니스 규칙을 표현
* 뷰 : 프레젠테이션을 표현
* 컨트롤러 : 위 두개를 분리하기 위해 둘 사이에 배치된 인터페이스다.

## Controller

Controller는 주로 사용자의 요청을 처리하고, 결과를 모델 객체로 지정된 뷰로 넘겨준다.

### Annotation

#### 1. `@Controller`

이 어노테이션은 Controller의 역할을 수행한다고 명시하는 것이다.(특정 클래스를 Controller로 사용하겠다고 spring framework에 알리는 것이다.)
필요한 비즈니스 로직을 호출해 전달할 모델과 이동할 뷰 정보를 DispatherServlet에 반환한다.
Bean으로 등록해주며, `@Component`의 구체화 된 어노테이션이다.

#### 2. `@RequestMapping`

요청에 대해 어떤 Controller, 어떤 메소드가 처리할 지 매핑하기 위한 어노테이션이다. 클래스/메서드 선언부에 위 어노테이션과 함께 url을 명시한다.

RequestMapping의 속성으로는 다음과 같은 것들이 있다.

1. `value(String[])` : URL 값
  ```java
  @RequestMapping(value="/login")
  @RequestMapping("/login")
  ```
2. `method(RequestMethod[])` : Http Request 메소드 값
  ```java
  @RequestMapping(value="/login", method=@RequestMethod.GET)
  @RequestMapping(value="/login", method=@RequestMethod.POST)
  ```

  Spring4.3 이후로는 다음과 같이 쓸 수 있다고 한다.

  `@GetMapping("/login")` == `@RequestMapping("/login", method=@RequestMethod.GET)`

  `@PostMapping("/login")` == `@RequestMapping("/login", method=@RequestMethod.POST)`

3. `params(String[])` : Http Request 파라미터

  * `@RequestParam` : 사용자가 원하는 매개변수에 값을 매핑하기 위해 사용한다.
    
    ```java
    @PostMapping("/member")
    public String member(@RequestParam String name, @RequestParam Int age)
    ```
    
    여기서 `@RequestParam`은 생략이 가능하다. 사용자가 입력한 key값과 매개변수의 이름을 비교해 값을 넣어주기 때문이다. 따라서 아래와 코드와 동일하다.
    
    ```java
    @PostMapping("/member")
    public String member(String name, Int age)
    ```
    
  * `@PathVariable` : url 경로를 변수화하여 사용할 수 있게 해준다.
  
    ```java
    @RequestMapping("/member/{name}/{age}")
    public String member(@PathVariable("name") String name, @PathVariable("age") String age
    ```
    
    위의 코드는 RequestMapping의 `{name}`과 PathVariale의 String name을 매핑해준다.
    
4. `consumes(String[])` : Reuqest body에 담는 타입을 제한할 수 있다.
  ```java
  @PostMapping("/login", consumes="application/json")
  ```
  
  위의 경우 application/json이 존재해야 처리가 된다.
  
  
### 예제

#### `@ResponseBody`로 객체 정보 전달

View가 아닌 data를 반환해야 할 경우 `@ResponseBody`를 사용하면 된다. String, map, json 등을 전달할 수 있다.

Controller의 코드를 아래와 같이 설정한다.

```java
...

@Controller
public class TestController {
  @RequestMapping(value="/home")
  public String home() {
    return "index.html";
  }
  
  @ResponseBody
  @RequestMapping("/valueTest")
  public String valueTest() {
      String value = "test string";
      return value;
  }
}
```

그리고 view page를 다음과 같이 설정한다.

```html
<!DOCTYPE html> 
<html lang="en"> 
  <head> 
    <meta charset="UTF-8"> 
      <title>Index</title> 
      <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script> 
      <script> 
        $.ajax({ 
          type: "GET", 
          url: "/valueTest", 
          success: (data) => { console.log(data); $('#contents').html(data); 
          } 
        }); 
      </script> 
    </head> 
    <body> 
      <h1>Hello World!</h1> 
      <div id="contents"> 
      </div> 
    </body> 
  </html>
```

서버를 다시 시작하면 값을 전달받은 것을 확인할 수 있다.

![image](https://user-images.githubusercontent.com/41438361/107851012-f494ea00-6e49-11eb-9a8a-d7f1a551f4d8.png)



출처 
  
https://goddaehee.tistory.com/203
