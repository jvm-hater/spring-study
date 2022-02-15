## 필터(Filter)

### 필터(Filter)란?

필터(Filter)는 J2EE표준 스펙 기능으로 디스패처 서블릿(Dispatcher Servlet)에 요청이 전달되기 전/후에 url 패턴에 맞는 모든 요청에 대해 부가작업을 처리할 수 있는 기능을 제공한다. 즉, 스프링 컨테이너가 아닌 톰캣과 같은 웹 컨테이너에 의해 관리가 되므로 디스패처 서블릿으로 가기 전에 요청을 처리하는 것이다.

![image](https://user-images.githubusercontent.com/55661631/154046882-3ce7570e-4829-42ab-86ce-e6f2a5d117b9.png)

### 필터(Filter) 구현

필터를 추가하기 위해서는 javax.servlet의 Filter 인터페이스를 구현해야 하며 이는 다음의 3가지 메소드를 가지고 있다.

- init 메소드
    - init 메소드는 필터 객체를 초기화하고 서비스에 추가하기 위한 메소드이다. 웹 컨테이너가 1회 init 메소드를 호출하여 필터 객체를 초기화하면 이후의 요청들은 doFilter 메소드를 통해 처리된다.
- doFilter 메소드
    - doFilter 메소드는 url-pattern에 맞는 모든 HTTP 요청이 디스패처 서블릿으로 전달되기 전에 웹 컨테이너에 의해 실행되는 메소드이다. doFilter의 파라미터로는 FilterChain이 있는데, FilterChain의 doFilter 통해 다음 대상으로 요청을 전달하게 된다. chain.doFilter 전/후에 우리가 필요한 처리 과정을 넣어줌으로써 원하는 처리를 진행할 수 있다.
- destroy 메소드
    - destroy 메소드는 필터 객체를 서비스에서 제거하고 사용하는 자원을 반환하기 위한 메소드이다. 이는 웹 컨테이너에 의해 1번 호출되며 이후에는 이제 doFilter에 의해 처리되지 않는다.

**예제 코드**

```java
@Order(1)
@Component
public class CustomFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException { }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
        throws IOException, ServletException {
        chain.doFilter(request, response);
    }

    @Override
    public void destroy() { }
}
```

- @Component : Filter를 사용하려면 빈으로 등록해야 한다.
- @Order : 필터 순서를 정하고 싶을 때 사용한다.

 **예제 코드**

```java
@Configuration
public class CustomRegistrationBean {

    @Bean
    public FilterRegistrationBean<CustomFilter> customFilterBean() {
        FilterRegistrationBean<CustomFilter> registration = new FilterRegistrationBean<>();
        registration.setFilter(new CustomFilter());
        registration.setOrder(1);
        registration.addUrlPatterns("/api/*");
        return registration;
    }
}
```

만약 특정 URI에만 필터를 동작하게 하려면 FilterRegistrationBean를 사용해 필터를 등록하면 된다.

### 필터(Filter) 용도

필터에서는 스프링과 무관하게 전역적으로 처리해야 하는 작업들을 처리할 수 있다.

- 보안 관련 공통 작업
    - 필터는 웹 컨테이너에서 동작하기 때문에 보안 검사(XXS 방어 등)를 하여 올바른 요청이 아닐 경우 차단할 수 있다. 스프링 컨테이너까지 요청이 전달되지 못하고 차단되므로 안정성을 더욱 높일 수 있다.
- 모든 요청에 대한 로깅
- 이미지/데이터 압축 및 문자열 인코딩
    - 필터는 이미지나 데이터의 압축이나 문자열 인코딩과 같이 웹 애플리케이션에 전반적으로 사용되는 기능을 구현할 수 있다.
- ServletRequest 커스터마이징
    - HttpServletRequest는 body의 내용을 한 번만 읽을 수 있다. 따라서 Filter나 Interceptor에서는 body를 읽을 수 없다(IOException 발생). body를 로깅하기 위해 커스텀한 ServletRequest를 생성할 수 있다.
    

## 인터셉터(Interceptor)

### 인터셉터(Interceptor)란?

인터셉터(Interceptor)는 J2EE 표준 스펙인 필터(Filter)와 달리 Spring이 제공하는 기술로써, 디스패처 서블릿(Dispatcher Servlet)이 컨트롤러를 호출하기 전과 후에 요청과 응답을 참조하거나 가공할 수 있는 기능을 제공한다. 즉, 웹 컨테이너에서 동작하는 필터와 달리 인터셉터는 스프링 컨텍스트에서 동작을 하는 것이다.

![image](https://user-images.githubusercontent.com/55661631/154046913-adb99b52-a675-4701-a8e9-748f13836a38.png)

디스패처 서블릿은 핸들러 매핑을 통해 적절한 컨트롤러를 찾도록 요청하는데, 그 결과로 실행 체인(HandlerExecutionChain)을 돌려준다. 그래서 이 실행 체인은 1개 이상의 인터셉터가 등록되어 있다면 순차적으로 인터셉터들을 거쳐 컨트롤러가 실행되도록 하고, 인터셉터가 없다면 바로 컨트롤러를 실행한다.

### 인터셉터(Interceptor) 구현

인터셉터를 추가하기 위해서는 org.springframework.web.servlet.HandlerInterceptor 인터페이스를 구현해야 하며, 이는 다음의 3가지 메소드를 가지고 있다.

- preHandle 메소드
    - preHandle 메소드는 컨트롤러가 호출되기 전에 실행된다. 그렇기 때문에 컨트롤러 이전에 처리해야 하는 전처리 작업이나 요청 정보를 가공하거나 추가하는 경우에 사용할 수 있다.
    - preHandle 메소드의 3번째 파라미터인 handler 파라미터는 @RequestMapping이 붙은 메소드의 정보를 추상화한 객체이다.
    - preHandle 메소드의 반환 타입은 boolean인데 반환값이 true이면 다음 단계로 진행이 되지만, false라면 작업을 중단하여 이후의 작업(다음 인터셉터 또는 컨트롤러)은 진행되지 않는다.
- postHandle 메소드
    - postHandle 메소드는 컨트롤러를 호출된 후에 실행된다. 그렇기 때문에 컨트롤러 이후에 처리해야 하는 후처리 작업이 있을 때 사용할 수 있다.
- afterCompletion 메소드
    - afterCompletion 메소드는 이름 그대로 모든 뷰에서 최종 결과를 생성하는 일을 포함해 모든 작업이 완료된 후에 실행된다.

**예제 코드**

```java
@Component
public class CustomInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
        throws Exception {
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
        ModelAndView modelAndView) throws Exception {
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
        throws Exception {
    }
}

@Configuration
public class WebMvcConfiguration implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new CustomInterceptor());
        registry.addInterceptor(new LoggingInterceptor());
    }
}
```

- 생성한 인터셉터를 빈으로 등록한다.
- WebMvcConfigurer 인터페이스의 `addInterceptors()` 메소드에 생성한 인터셉터를 등록한다.
- 인터셉터는 InterceptorRegistry에 등록한 순서대로 동작한다.

### 인터셉터(Interceptor) 용도

인터셉터는 클라이언트의 요청과 관련되어 전역적으로 처리해야 하는 작업들을 처리할 수 있다. 

물론 필터에서도 충분히 클라이언트의 요청과 관련된 작업을 처리할 수 있다. 그러나 웹 컨테이너에서 동작하는 필터는 `@ControllerAdvice`와 `@ExceptionHandler`를 사용해서 예외 처리를 할 수 없다. **따라서 작성해야할 전후처리 로직에서 예외 처리를 하고 싶다면 인터셉터를 사용하자.**

- 인증/인가 등과 같은 공통 작업
    - 대표적으로 인증이나 인가와 같이 클라이언트 요청과 관련된 작업들을 컨트롤러로 넘어가기 전에 검사할 수 있다.
- API 호출에 대한 로깅
    - 전달 받는 HttpServletRequest, HttpServletResponse 객체를 통해 클라이언트의 정보를 기록할 수 있다.
- Controller로 넘겨주는 데이터 가공
    - 전달 받는 HttpServletRequest, HttpServletResponse 객체를 가공해여 컨트롤러에 넘겨줄 수 있다.
- AOP 흉내
    - preHandle 메소드의 3번째 파라미터인 HandlerMethod로 실행될 메소드의 시그니처 등 추가적인 정보를 파악해서 로직 실행 여부를 판단할 수 있다.

## 정리

### 필터(Filter)와 인터셉터(Interceptor) 차이점

- 필터 웹 컨텍스트에서 실행되고 인터셉터는 스프링 컨텍스트에서 실행된다(실행 시점이 다르다).
- 필터는 Servlet의 전후를 다룰 수 있고, 인터셉터는 컨트롤러의 전후를 다룰 수 있다.
- 필터는 스프링과 무관하게 전역적으로 처리해야 하는 작업을 하는 것이 좋고, 인터셉터는 클라이언트의 요청과 관련되서 전역적으로 처리해야 하는 작업을 하는 것이 좋다.
- 그 이유는 `@ControllerAdvice`와 `@ExceptionHandler`를 사용해 예외 처리를 할 수 있느냐 없느냐의 차이다.

## Argument Resolver

### Argument Resolver란?

ArgumentResolver는 어떠한 요청이 컨트롤러에 들어왔을 때, 요청에 들어온 값으로부터 원하는 객체를 만들어내는 일을 간접적으로 해줄 수 있다.

### Argument Resolver 용도

예를 들어, JWT 토큰과 함께 요청이 들어왔다고 가정하자. 우리는 이 토큰이 유효한 토큰인지 검증을 거친 후에 토큰에 저장된 id를 꺼내 LoginMember라는 객체로 만들어내는 과정이 필요하다.

이런 경우에 Argument Resolver를 사용하지 않는다면 토큰을 검증하고 LoginMember 객체로 변환하는 과정을 모든 컨트롤러마다 구현해야 한다. 그럼 사용자 검증이 필요한 컨트롤러에 중복 코드가 생기고, 컨트롤러의 책임이 증가하는 문제가 생긴다.

이러한 문제는 Argument Resolver를 사용하여 해결할 수 있다.

### Argument Resolver 구현

ArgumentResolver는 HandlerMethodArgumentResolver를 구현함으로써 사용할 수 있다.  그리고 이 인터페이스는 아래 두 메소드를 구현하도록 명시하고 있다.

```java
boolean supportsParameter(MethodParameter parameter);

@Nullable
Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer, NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception;
```

- supportsParameter 메소드
    - ArgumentResolver가 실행되길 원하는 Parameter의 앞에 특정 어노테이션을 생성해 붙인다. supportsParameter 메소드는 요청받은 메서드의 인자에 원하는 어노테이션이 붙어있는지 확인하고 원하는 어노테이션을 포함하고 있으면 true를 반환한다.
- resolveArgument 메소드
    - supportsParameter에서 true를 받은 경우, 즉, 특정 어노테이션이 붙어있는 어느 메서드가 있는 경우 parameter가 원하는 형태로 정보를 바인딩하여 반환하는 메서드이다.

이렇게 ArgumentResolver를 사용했을 때 컨트롤러의 구현은 아래와 같다.

```java
@GetMapping("/me")
public ResponseEntity<MemberResponse> findMemberOfMine(@AuthenticationPrincipal LoginMember loginMember) {
    MemberResponse memberResponse = memberService.findMember(loginMember.getId());
    return ResponseEntity.ok().body(memberResponse);
}
```

### Argument Resolver와 인터셉터(Interceptor) 차이점

- ArgumentResolver는 어떠한 요청이 컨트롤러에 들어왔을 때, 요청에 들어온 값으로부터 원하는 객체를 반환한다.
- 반면에 인터셉터는 위 처럼 객체를 반환할 필요가 없다.
- 참고로 인터셉터가 실행된 후에 Argument Resolver가 실행된다.

## 참고

- [https://mangkyu.tistory.com/173](https://mangkyu.tistory.com/173)
- [https://junhyunny.github.io/spring-boot/filter-interceptor-and-aop/](https://junhyunny.github.io/spring-boot/filter-interceptor-and-aop/)
- [https://supawer0728.github.io/2018/04/04/spring-filter-interceptor/](https://supawer0728.github.io/2018/04/04/spring-filter-interceptor/)
- [https://tecoble.techcourse.co.kr/post/2021-05-24-spring-interceptor/](https://tecoble.techcourse.co.kr/post/2021-05-24-spring-interceptor/)
- [https://baeldung-cn.com/spring-mvc-handlerinterceptor-vs-filter](https://baeldung-cn.com/spring-mvc-handlerinterceptor-vs-filter)
- [https://www.baeldung.com/spring-boot-add-filter](https://www.baeldung.com/spring-boot-add-filter)

## 예상 면접 질문 및 답변

### Q. 필터란?

필터는 디스패처 서블릿에 요청이 전달되기 전/후에 url 패턴에 맞는 모든 요청에 대해 부가작업을 처리할 수 있는 기능을 제공하며 톰캣과 같은 웹 컨텍스트에서 동작한다.

### Q. 필터의 사용 시기는?

필터에서는 스프링과 무관하게 전역적으로 처리해야 하는 작업들을 처리할 수 있다. 대표적으로 보안 관련 공통 작업, 모든 요청에 대한 로깅, 이미지/데이터 압축 및 문자열 인코딩, ServletRequest 커스터마이징 작업을 처리한다.

### Q. 인터셉터란?

인터셉터는 디스패처 서블릿이 컨트롤러를 호출하기 전과 후에 요청과 응답을 참조하거나 가공할 수 있는 기능을 제공하며, 즉, 스프링 컨텍스트에서 동작한다.

### Q. 인터셉터의 사용 시기는?

인터셉터는 클라이언트의 요청과 관련되어 전역적으로 처리해야 하는 작업들을 처리할 수 있다. 대표적으로 인증/인가 등과 같은 공통 작업, API 호출에 대한 로깅, Controller로 넘겨주는 데이터 가공 작업을 처리한다.

### Q. 필터와 인터셉터의 차이는?

필터 웹 컨텍스트에서 실행되고 인터셉터는 스프링 컨텍스트에서 실행되며, 필터는 Servlet의 전후를 다룰 수 있고, 인터셉터는 컨트롤러의 전후를 다룰 수 있다.따라서, 필터는 스프링과 무관하게 전역적으로 처리해야 하는 작업을 하는 것이 좋고, 인터셉터는 클라이언트의 요청과 관련되서 전역적으로 처리해야 하는 작업을 하는 것이 좋다.

### Q. Argument Resolver란?

ArgumentResolver는 어떠한 요청이 컨트롤러에 들어왔을 때, 요청에 들어온 값으로부터 원하는 객체를 만들어내는 일을 간접적으로 해준다.

### Q. Argument Resolver의 사용 시기는?

JWT 토큰과 함께 요청이 들어왔을 때, 유효한 토큰인지 검증을 거친 후에 토큰에 저장된 id를 꺼내 LoginMember라는 객체로 만들어내는 작업을 할 때 사용할 수 있다.

### Q. 인터셉터와 Argument Resolver의 차이는?

ArgumentResolver는 어떠한 요청이 컨트롤러에 들어왔을 때, 요청에 들어온 값으로부터 원하는 객체를 반환한다. 반면에 인터셉터는 위 처럼 객체를 반환할 필요가 없으며, 인터셉터가 실행된 후에 Argument Resolver가 실행된다.
