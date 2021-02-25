---  
layout: post  
title: "Spring boot Controller Test - JPA meta model오류 해결기"  
subtitle: ""  
categories: java
tags: java-spring test controller jpa error
comments: true  
header-img: img/dev/spring/test_code.gif
---  
  
> `Spring boot에서 controller test가 돌아가도록 세팅하느 방법?`  

---

## 오류 해결기

우선 나는 JUnit5를 이용해서 test했다. 그래서 `@RunWith(class명)` 과 같은 어노테이션을 사용하지 못했다.

처음 Controller를 test하기 위해 test 클래스를 생성하고, 아래와 같이 코드를 작성했다.

```java
import ...

@ActiveProfiles("test")
@WebMvcTest(MovieController.class)
class MovieControllerTest {
    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private MovieService movieService;

    @Test
    public void getMovie() throws Exception {

    }
}
```

참고로 `@ActiveProfiles("test")`는 application-test.properties 설정을 test를 돌릴 때 자동으로 적용하여 돌리기 위해 설정했다.
`@WebMvcTest()` 안에 테스트할 컨트롤러 클래스를 삽입해서 어떤 클래스를 테스트할 것인지 명시했다. 

### WebMvcTest

`@WebMvcTest` 어노테이션은 MVC를 위한 테스트를 지원하고, 컨트롤러가 예상대로 동작하는지 확인하기 위해 사용한다.

특징은 다음과 같다.

1. `@Controller, @ControllerAdvice, @JsonComponent, Converter, GenericConverter, Filter, HandlerInterceptor...`
의 내용마 스캔을 하게 해서 가벼운 테스팅을 가능하게 한다.
2. MockBean, MockMVC를 자동 구성하여 테스특 가능하도록 한다.
3. Spring Security의 테스트도 지원한다.

장점은 WebApplication 과 관련된 Bean들만 등록하므로 속도가 빠르다는 점, 단점으로는 요청부터 응답까지 모드 test를 mock을 기반으로 test하기 때문에
제대로 동작하지 않을 수 있다는 점이다.

다시 코드로 돌아가서, 처음에 저 코드를 실행했을 때는 아래와 같으 오류가 나왔다.

<img width="1239" alt="스크린샷 2021-02-26 오전 2 26 44" src="https://user-images.githubusercontent.com/41438361/109192073-2870f780-77da-11eb-9d87-27931b448ea2.png">

```
Caused by: java.lang.IllegalArgumentException: JPA metamodel must not be empty!
	at org.springframework.util.Assert.notEmpty(Assert.java:464)
	at org.springframework.data.jpa.mapping.JpaMetamodelMappingContext.<init>(JpaMetamodelMappingContext.java:58)
	at org.springframework.data.jpa.repository.config.JpaMetamodelMappingContextFactoryBean.createInstance(JpaMetamodelMappingContextFactoryBean.java:80)
	at org.springframework.data.jpa.repository.config.JpaMetamodelMappingContextFactoryBean.createInstance(JpaMetamodelMappingContextFactoryBean.java:44)
	at org.springframework.beans.factory.config.AbstractFactoryBean.afterPropertiesSet(AbstractFactoryBean.java:142)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.invokeInitMethods(AbstractAutowireCapableBeanFactory.java:1830)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1767)
	... 52 more
```

에러 로그가 굉장히 많이 찍혔는데, 그 중에서 가자 하단에 있는 걸 가져온 것이다. 많은 검색 끝에 원인은 프로젝트를 생성할 때 자동으로 생성되는
`~~~Application.java` 클래스에 `@EnableJpaAuditing` 어노테이션과 `@SpringBootApplication` 어노테이션을 함께 붙여서였다.

`@SpringBootApplication`가 붙은 클래스는 자동으로 모든 테스트들의 기본 설정을 적용된다.[참고](https://stackoverflow.com/questions/60606861/spring-boot-jpa-metamodel-must-not-be-empty-when-trying-to-run-junit-integrat)
따라서 위의 테스트 코드는 실행될 때 `@SpringBootApplication`가 붙은 나의 `~~~Application.java`를 로딩하는데, 테스트 코드에 붙은 `@WebMvcTest`같은 테스트 전용
어노테이션은 JPA 관련 bean 객체를 로딩하지 않아 `@EnableJpaAuditing`과 설정이 맞지 않는 문제가 발생한다. [참고](https://sup2is.tistory.com/107)


해결방법으로는 메인 클래스는 `@SpringBootApplication` 어노테이션만 갖고 있게 하고, `@EnableJpaAuditing`을 `@Configuration` 어노테이션을 갖는
설정 파일을 하나 따로 만들어서 거기로 옮기는 것이었다. [참고1](https://stackoverflow.com/questions/60606861/spring-boot-jpa-metamodel-must-not-be-empty-when-trying-to-run-junit-integrat) [참고2](https://stackoverflow.com/questions/51467132/spring-webmvctest-with-enablejpa-annotation)

그래서 다음과 같이 해결했다.

0. (모든 entity에 `@EntityListeners(AuditingEntityListener.class)`를 붙여 해당 클래스에 auditing 기능을 포함할 수 있게 했다. [참고](https://ibks-platform.tistory.com/283)
1. `~~~Application.java`에서 `@SpringBootApplication`은 놔두고 `@EnableJpaAuditing`을 삭제했다.
2. 임의의 패키지(폴더)에 `JpaAuditingConfiguration.java` 파일을 생성했다. 
  파일은 다음과 같이 만들었다.
  ```java
  import org.springframework.context.annotation.Configuration;
  import org.springframework.data.jpa.repository.config.EnableJpaAuditing;

  @Configuration
  @EnableJpaAuditing
  public class JpaAuditingConfiguration {
  }
  ```

위와 같이 `@EnableJpaAuditing`을 바로 다른 configuration 파일을 하나 생성해서 그리로 옮길 수 있다고 판단한 이유는 `@SpringBootApplication`이 
`@Component @Configuration @Repository @Service @Controller @RestController`의 어노테이션을 갖는 애들을 스캔해 bean으로 등록하기 때문에
다른 configuration 파일로 바로 어노테이션을 옮겨도 자동으로 스캔해서 인식할 수 있다고 생각했기 때문이다. [참고](https://seongmun-hong.github.io/springboot/Spring-boot-EnableAutoConfiguration)

하여간 이렇게 설정하고 위에 써놓은 기본적인 코드를 돌리니 이전과 같이 발생하던 에러가 더 이상 발생하지 않고 잘 실행된다. 이것때문에 거의 하루를 날렸는데
생각보다 간단하게 해결되어서 약가 허무하다..ㅎㅎ 그래도 덕분에 좀 더 깊이 있게 공부할 수 있어서 좋았다.
