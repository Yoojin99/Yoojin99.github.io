---  
layout: post  
title: "@Controller와 @RestController 차이"  
subtitle: ""  
categories: java
tags: java-spring controller restcontroller
comments: true  
header-img: img/dev/spring/spring.jpg
---  
  
> `@Controller와 @RestController의 차이는?`  

---

"Restful하게 만들어야 하는지 고려해주세요" 라는 말을 듣고 Rest에 대해서 공부하던 중 `@RestController`를 컨트롤러에 붙이면 해당 컨트롤러를
restful하게 만든다는 것을 듣게 되었다. 그렇다면 `@Controller`와 `@RestController`는 무슨 차이일까?

주된 차이는 **HTTP Response Body가 생성되는 방식**이다.

## 1. @Controller (Spring MVC Controller)

전통적인 Spring MVC 컨트롤러를 의미하는 `@Controller`는 주로 **View를 반환하기 위해 사용**한다.

### View

1. Client가 URI 형식으로 웹 서비스에 요청을 보낸다.
2. Mapping되는 Handler와 그 Type을 찾는 DispatcherServlet이 요청을 인터셉트한다.
3. Controller가 요청을 처리한 후에 응답을 DispatcherServlet으로 반환하고, DispatcherServelet은 View를 사용자에게 반환한다.

컨르롤러가 View를 반환하기 위해서는 ViewResolver가 사용되며, ViewResolver 설정에 맞게 View를 찾아 렌더링한다.

### Data

하지만 컨트롤러에서 Data를 반환해야 하는 경우도 있다. Spring MVC 컨트롤러에서는 **데이터를 반환하기 위해 `@ResponseBody` 어노테이션을 써줘야 한다.**

이를 통해 컨트롤러도 Json 형태로 데이터를 반환할 수 있다.

1. 클라이언트가 URI 형식으로웹 서비스에 요청을 보낸다.
2. Mapping되는 Handler와 그 Type을 찾는 DispatcherServlet이 요청을 인터셉트한다. (여기까지는 view와 같다.)
3. `@ResponseBody`를 사용해 클라이언트에게 json형태로 데이터를 반환한다.

예시는 아래와 같다.

```java
@Controller
@RequestMapping("/user")
public class UserController {

    private final UserService userService;

    @PostMapping("/info")
    public @ResponseBody User info(@RequestBody User user){
        return userService.retrieveUserInfo(user);
    }
    
    @GetMapping("/infoView")
    public String infoView(Model model, @RequestParam(value = "userName", required = true) String userName){
        User user = userService.retrieveUserInfo(userName);
        model.addAttribute("user", user);
        return "/user/userInfoView";
    }

}
```

위에서는 User를 json으로 반환하기 위해 `@ResponseBody` 어노테이션을 붙여줬다.

infoView에서는 View를 전달하고 있기 때문에 String을 반환값으로 설정했다.

## 2. @RestController (Spring Restful Controller)

`@RestController`는 Spring MVC Controller에 `@ResponseBody`가 추가된 것이다. 즉 각 메서드마다 `@ResponseBody`를 추가할 필요가 없어졌다.
**RestController의 주 목적은 Json 형태로 객체 데이터를 반환하는 것이다.**

1. 클라이언트가 URI 형식으로 웹 서비스에 요청을 보낸다.
2. Mapping되는 handler와 그 type을 찾는 DIspatcherServlet이 요청을 인터셉트한다.
3. RestController는 해당 요청을 처리하고 데이터를 반환한다.

예제는 아래와 같다.

```java
@RestController
@RequestMapping("/user")
public class UserController {

    @Resource(name = "userService")
    private UserService userService;

    @PostMapping(value = "/info1")
    public User info1(@RequestBody User user){
        return userService.retrieveUserInfo(user);
    }

    @PostMapping(value = "/info2")
    public ResponseEntity<User> info2(@RequestParam(value = "userName", required = true) String userName){
        User user = userService.retrieveUserInfo(userName);

        if(user == null){
            return ResponseEntity.noContent().build()
        }

        return ResponseEntity.ok(user)
    }

    @PostMapping(value = "/info3")
    public ResponseEntity<User> info3(@RequestParam(value = "userName", required = true) String userName){
        return Optional.ofNullable(userService.retrieveUserInfo(userName))
                .map(user -> ResponseEntity.ok(user))
                .orElse(ResponseEntity.noContent().build());
    }
}
```

`@RestController`가 데이터를 반환하기 위해서는 viewResolver 대신에 HttpMessageConverter가 동작한다. 여기에는 여러 converter가 등록되어 있고,
반환해야 하는 데이터에 따라 사용되는 converter가 달라진다. 예를 들어 단순 문자열인 경우에는 StringHttpMessageConverter, 객체일 때는 MappingJackson2HttpMessageConverter가 사용된다.

