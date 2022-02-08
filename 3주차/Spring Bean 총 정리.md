
## 스프링 빈이란?

스프링 빈은 스프링 컨테이너에 의해 관리되는 자바 객체(POJO)를 의미한다.

### 스프링 컨테이너

스프링 컨테이너는 스프링 빈의 생명 주기를 관리하며, 생성된 스프링 빈들에게 추가적인 기능을 제공하는 역할을 한다. IoC와 DI의 원리가 스프링 컨테이너에 적용된다.

개발자는 new 연산자, 인터페이스 호출, 팩토리 호출 방식으로 객체를 생성하고 소멸하지만, 스프링 컨테이너를 사용하면 해당 역할을 대신해 준다. 즉, 제어 흐름을 외부에서 관리하게 된다. 또한, 객체들 간의 의존 관계를 스프링 컨테이너가 런타임 과정에서 알아서 만들어 준다.

## 스프링 빈 등록 방식

### Component Scan

컴포넌트 스캔은 @Component를 명시하여 빈을 추가하는 방법이다. 클래스 위에 @Component를 붙이면 스프링이 알아서 스프링 컨테이너에 빈을 등록한다.

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Indexed
public @interface Component {
```

참고로 @Component는 위와 같이 `ElementType.TYPE` 설정이 있으므로 Class 혹은 Interface에만 붙일 수 있다.

**컴포넌트 스캔의 대상**

@Component 외에 @Controller, @Service, @Repository, @Configuration는 @Component의 상속을 받고 있으므로 모두 컴포넌트 스캔의 대상이다.

- @Controller
    - 스프링 MVC 컨트롤러로 인식된다.
- @Repository
    - 스프링 데이터 접근 계층으로 인식하고 해당 계층에서 발생하는 예외는 모두 DataAccessException으로 변환한다.
- @Service
    - 특별한 처리는 하지 않으나, 개발자들이 핵심 비즈니스 계층을 인식하는데 도움을 준다.
- @Configuration
    - 스프링 설정 정보로 인식하고 스프링 빈이 싱글톤을 유지하도록 추가 처리를 한다. (물론 스프링 빈 스코프가 싱글톤이 아니라면 추가 처리를 하지 않음.)

### Java 코드로 등록

Java 코드로 빈을 등록할 수 있다. 클래스를 생성하고, 위에서 언급한 @Configuration 어노테이션을 활용한다.

```java
@Configuration
public class AppConfig {

    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    @Bean
    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }

}
```

이때, 빈을 등록하기 위해 인스턴스를 생성하는 메소드 위에 @Bean를 명시하면 된다.

```java
@Target({ElementType.METHOD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Bean {
```

참고로 @Bean은 위와 같이 `ElementType` 설정이 METHOD 혹은 ANNOTATION_TYPE이므로 메소드나 어노테이션에만 붙일 수 있다. 클래스에 붙일 수는 없다.

**Component Scan**

@Configuration에는 @Component가 있으므로 컴포넌트 스캔이 대상이 되어 자등 스캔을 통해 빈 등록이 가능하다.

**수동 등록**

거의 사용하지는 않지만, ApplicationContext를 호출하여 수동으로 설정 파일을 이용하여 빈을 수동 등록할 수도 있다.

```java
public class Main {

    public static void main(String[] args) {
        final ApplicationContext beanFactory = new AnnotationConfigApplicationContext(AppConfig.class);
        final AppConfig bean = beanFactory.getBean("appConfig", AppConfig.class);
    }
}
```

### @Bean vs @Component

- @Bean
    - 개발자가 컨트롤이 불가능한 **외부 라이브러리들을 Bean으로 등록하고 싶은 경우**
    에 사용된다.
    - 메소드 또는 어노테이션 단위에 붙일 수 있다.
- @Component
    - 개발자가 **직접 컨트롤이 가능한 클래스들의 경우**에 사용된다.
    - 클래스 또는 인터페이스 단위에 붙일 수 있다.

### @Configuration과 싱글톤

@Configuration은 @Bean에 추가 설정을 줘서 싱글톤으로 만들지 않는 이상 무조건 빈에 대해 싱글톤을 보장한다. 아래 코드를 살펴 보자.

```java
@Configuration
public class AppConfig {

    @Bean
    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }
    
    @Bean
    public OrderService orderService() {
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }
    
    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }
}
```

MemberService와 OrderService를 빈으로 등록할 때 모두 `memberRepository()` 메소드를 호출하는 것을 알 수 있다. 결과적으로 각각 다른 2개의 MemoryMemberRepository가 생성되어 싱글톤이 깨진다고 생각할 수 있다.

하지만 @Configuration은 클래스의 바이트 코드를 조작하는 라이브러리인 CGLIB를 사용하여 싱글톤을 보장한다. CGLIB는 프록시 객체의 일종으로 AppConfig가 빈으로 등록될 때, AppConfig 대신 AppConfig를 상속 받은 AppConfig$CGLIB 형태로 프록시 객체가 등록된다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fbda82b4c-e6e0-4ba7-986d-49325540dd17%2FUntitled.png?table=block&id=7b19b705-ce5c-429a-91a1-84d891e20ee0&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

위와 같이 이름은 appConfig가 되고, 실제 등록되는 스프링 빈은 CGLIB 클래스의 인스턴스가 등록된다. CGLIB는 대강 아래와 같이 구현이 되어 있다고 생각하면 편하다.

```java
@Bean
public MemberRepository memberRepository() {
    if(memorymemberRepository가 이미 스프링 컨테이너에 등록되어있으면?) {
        return 스프링 컨테이너에서 찾아서 반환;
    } else { // 스프링 컨테이너에 없으면
        기존로직을 호출해서 MemoryMemberRepository를 생성하고 스프링 컨테이너에 등록
        return 반환;
    }
}
```

@Bean이 등록된 메소드마다 이미 스프링 빈이 존재하면 존재하는 빈을 반환하고, 스프링 빈이 없으면 생성해서 스프링 빈으로 등록하고 반환하는 코드가 동적으로 만들어진다. 이 덕분에 싱글톤이 보장되는 것이다.

참고로 AppConfig$CGLIB는 AppConfig의 자식 타입이므로 AppConfig 타입으로 조회가 가능하다.

### Bean Lite Mode

Bean Lite Mode는 CGLIB를 이용하여 바이트 코드 조작을 하지 않는 방식을 의미한다. 즉, 스프링 빈의 싱글톤을 보장하지 않는다.

```java
@Component
public class AppConfig {

    @Bean
    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }
    
    @Bean
    public OrderService orderService() {
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }
    
    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }
}
```

Bean Lite Mode로 설정하려면 @Configuration이 아닌 @Component로 변경하면 된다. 이렇게 하면 `objectMapperLiteBean()` 메소드를 lite mode로 작동하여 매번 다른 객체를 반환해 줄 수 있다.

참고로 ApplicationContext를 사용해서 설정 파일 가지고 빈을 수동 등록한다면, @Component가 없어도 Bean Lite Mode가 동작한다.

## 스프링 빈 스코프

스프링에서 Singleton과 Prototype 빈 스코프를 제공하고 있으며, 스프링 MVC 웹 애플리케이션을 사용할 경우 웹 스코프를 제공한다.

### Singleton

- 싱글톤 빈은 스프링 컨테이너에서 한 번만 생성되며, 컨테이너가 사라질 때 제거된다.
- 생성된 하나의 인스턴스는 Spring Beans Cache에 저장되고, 해당 빈에 대한 요청과 참조가 있으며 캐시된 객체를 반환한다. 하나만 생성되기 때문에 동일 참조를 보장한다.
- 기본적으로 모든 빈은 스코프가 명시적으로 지정되지 않으면 싱글톤이다.
- 대상 클래스에 `@Scope(”singletone”)` 을 붙이면 된다.
- 싱글톤 타입으로 적합한 객체
    - 상태가 없는 공유 객체
    - 읽기 전용으로만 상태를 가진 객체
    - 쓰기가 가능한 상태를 지니면서도 사용 빈도가 매우 높은 객체
        - 단, 이때는 동기화 전략이 필요함.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F9526dd23-0dc3-4cc1-96ae-f86f7ae8441f%2FUntitled.png?table=block&id=13560a98-218a-4c24-b9ae-cb2c5da74677&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

### Prototype

- 프로토 타입 빈은 DI가 발생할 때마다 새로운 객체가 생성되어 주입된다.
- 빈 소멸에 스프링 컨테이너가 관여하지 않고, gc에 의해 빈이 제거된다.
- 대상 클래스에 `Scope("prototype")` 을 붙이면 된다.
- 프로토 타입으로 적합한 객체
    - 사용할 때마다 상태가 달라져야 하는 객체
    - 쓰기가 가능한 상태가 있는 객체

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F1a978377-4e42-4954-85ea-b86a160284d0%2FUntitled.png?table=block&id=6ff4cbe6-9404-4f61-9fa4-6e358d2a6459&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

### Singleton Bean과 Prototype Bean을 같이 사용할 때 생기는 문제

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F6ead2836-6117-423b-93f8-e1ad8c3fd15f%2FUntitled.png?table=block&id=e6a652c4-fa6c-4a47-ab6e-26a2111d7ddb&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

위와 같이 프로토 타입 객체가 싱글톤 객체를 가지고 있는 것은 문제가 되지 않는다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F50624e98-1691-4207-8b2e-6942616468e9%2FUntitled.png?table=block&id=b608e8e7-18d4-448d-bcd2-85cc25618c40&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

하지만, 위와 같이 싱글톤 객체가 프로토 타입 객체를 가지고 있는 경우에는 의도한 것과 다른 결과를 낼 수도 있다. 이미 싱글톤 빈으로 생성되는 시점에 프로토 타입 빈이 생성되어 들어오기 때문에 싱글톤 빈 내부의 프로토 타입 빈을 호출하게 되면 매번 같은 값을 가져 온다.

만약 싱글톤 빈 내부의 프토로 타입 빈을 사용할 때마다 다른 인스턴스를 받아오려면 어떻게 해야 할까?

**(1) Provider**

```java
@Component
class ClientBean {

    @Autowired
    private Provider<PrototypeBean> provider;	//javax.inject 하위 클래스로 import해야함

    public int logic() {
        PrototypeBean prototypeBean = provider.get();	// 컨테이너에 빈 요청
        prototypeBean.addCount();
        return prototypeBean.getCount();
    }
}
```

`logic()` 메소드를 호출할 때마다 다른 PrototypeBean 인스턴스가 호출된다. Provider는 자바 표준이라서 스프링에 독립적이라는 장점이 있다.

**(2) @Scope의 proxyMode 설정**

```java
@Component
@Scope(value = "prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class ProtoProxy {}

@Component
@AllArgsConstructor
public class ScopeWrapper {
    ...

    @Getter
    ProtoProxy protoProxy;
}
```

위와 같이 Protytpe에 proxyMode 설정을 추가한다. 프록시 적용 대상이 클래스면 TARGET_CLASS, 인터페이스면 INTERFACE를 선택한다.

```java
@Slf4j
@Service
@AllArgsConstructor
public class ScopeService {

    private final ApplicationContext ctx;

    public void scopeTest() {
        log.info("[============== Singleton getBean And getProxyPrototype ==============]");
        log.info("ScopeWrapper getBean Proxy Proto Case 1 : " + ctx.getBean(ScopeWrapper.class).getProtoProxy());
        log.info("ScopeWrapper getBean Proxy Proto Case 2 : " + ctx.getBean(ScopeWrapper.class).getProtoProxy());
    }
}
```

스프링 컨테이너에서 해당 프로토 타입 빈을 호출해 보자.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F036a4bb4-2607-4572-b4da-5793379ffa53%2FUntitled.png?table=block&id=435948a9-79d5-469c-a2f8-860b937faab5&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

서로 다른 주소로 되어 있는 것을 확인할 수 있다.

### 웹 스코프

- 기존의 스프링이 사용하는 싱글톤 스코프는 스프링 컨테이너의 시작부터 끝까지 함께하는 스코프이고, 프로토 타입 스코프는 생성과 의존 관계 주입 및 초기화까지만 진행하는 스코프였다.
- 웹 스코프는 웹 환경에서만 동작하는 스코프이며 프로토 타입과 다르게 특정 주기가 끝날 때까지 관리를 해 준다. 따라서 @PreDestory와 같은 소멸 콜백이 호출된다는 특징이 있다.
- 웹 스코프의 종류
    - Request
        - HTTP 요청 하나가 들어오고 나갈 때까지 유지되는 스코프
        - 각각의 HTTP 요청마다 별도의 빈 인스턴스가 생성되고 관리된다.
    - Session
        - HTTP Session과 동일한 생명 주기를 가지는 스코프
    - Application
        - 서블릿 컨텍스트와 동일한 생명 주기를 가지는 스코프
    - WebSocket
        - 웹 소켓과 동일한 생명 주기를 가지는 스코프

**Request**

웹 스코프 중에서도 Request 스코프를 자주 사용하므로 이 스코프만 짚고 넘어가려고 한다.

Request Scope가 동작하는 방식을 예시 상황을 통해 살펴 보자.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F200626e6-e942-4bd0-83dd-ee7b41991515%2FUntitled.png?table=block&id=35223d11-b3db-4fea-be33-80bedb14a6a0&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

우리가 MyLogger라는 로그 찍는 클래스를 Request Scope로 등록하였고, 한 클라이언트 A가 요청을 보냈다고 가정해 보자.

컨트롤러에서 myLogger 객체를 요청 받았다면, 스프링 컨테이너는 A 전용으로 사용할 수 있는 빈을 생성하여 컨트롤러에 주입해 준다. 그리고 로직이 진행되면서 서비스에서 다시 myLogger 객체가 필요해서 요청을 하게 되면 방금 A 전용으로 생성했던 빈을 그대로 활용해서 주입받을 수 있다. 이후 요청이 끝나면 Request 빈은 소멸된다.

만약 다른 클라이언트 B가 A와 동시에 요청을 보냈다고 가정해 보자.

클라이언트 B도 역시 컨트롤러와 서비스에서 각각 myLogger 객체가 필요한데, 이 때는 클라이언트 A에게 주입해 주었던 빈이 아닌 새로 생성해서 주게 된다. 따라서 Request Scope를 활용하면 디버깅하기 쉬운 로그 환경을 만들 수 있다.

## 스프링 빈의 생명 주기

### Singleton Bean

싱글톤 빈의 생명 주기는 다음과 같다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F4652ee99-92cb-414e-b097-55a6d3a8bf88%2FUntitled.png?table=block&id=96c607e0-d48f-404f-864c-dad4c78b8e38&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

온갖 메소드가 섞여 있어서 이해가 되지 않을텐데 아래 설명을 보면 이해가 될 것이다.

1. 스프링 컨테이너 생성
2. 스프링 빈 생성
3. 의존 관계 주입
4. 초기화 콜백
5. 사용
6. 소멸 전 콜백
7. 스프링 종료

스프링 컨테이너가 생성되고, 스프링 빈이 등록되어 의존 관계가 주입되는 사실은 익숙하지만, 초기화 및 소멸 전 콜백은 사용해 본 적이 없을 수도 있다. 해당 콜백의 사용법을 간단히 알아 보자.

```java
@Component
public class OrderServiceImpl implements OrderService {

    private final DiscountPolicy discountPolicy;

    public OrderServiceImpl(DiscountPolicy discountPolicy) {
        this.discountPolicy = discountPolicy;
    }

    @Override
    public Order createOrder(int age, String itemName, int itemPrice) {
        int discountPrice = discountPolicy.discount(age, itemPrice);
        return new Order(itemName, itemPrice, discountPrice);
    }
    
    @PostConstruct
    public void init() {
        System.out.println("초기화 콜백입니다.");
    }
    
    @PreDestroy
    public void close() {
        System.out.println("소멸 전 콜백입니다.");
    }
}
```

위 코드와 같이 @PostConstruct를 사용하면 초기화 콜백이 호출되고, @PreDestory를 호출하면 소멸 전 콜백이 호출된다.

### Prototype Bean

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F323533e9-9ff7-4ec8-8f8e-607cc5942064%2FUntitled.png?table=block&id=6a8c9902-1032-4f7a-84d9-8c20ec17a051&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

프로토 타입 빈은 스프링 컨테이너가 빈의 생성까지만 관리를 하고, 그 이후에는 제어권이 사라진다. 따라서 생명 주기가 아래처럼 단순해진다.

1. 스프링 컨테이너 생성
2. 스프링 빈 생성
3. 의존 관계 주입
4. 초기화 콜백
5. 사용
6. GC에 의해 수거

위에서 언급한 콜백도 초기화 콜백만 동작을 한다는 것을 기억해야 한다.

## 싱글톤 빈은 Thread-Safe한가?

싱글톤 빈 이야기를 하기 전에, 싱글톤 자체에 대해서 생각해 보자.

```java
public class Singleton {

    private static Singleton instance = new Singleton();
    
    private Singleton() {
    }

    public static Singleton getInstance() {
        return instance;
    }
}
```

싱글톤 패턴은 위와 같이 형태로, 인스턴스가 전체 애플리케이션 중에서 단 한 번만 초기화되어 애플리케이션이 종료될 때까지 메모리에 상주한다는 특징이 있다. 만약 싱글톤이 상태를 갖게 된다면 멀티 스레드 환경에서 동기화 문제가 발생할 수 있다는 사실은 다들 잘 알 것이다.

다시 스프링으로 돌아와서, 스프링 빈은 별다른 설정을 주지 않으면 싱글톤 빈이 된다. 그런데, 우리는 싱글톤 빈을 사용할 때 위와 같이 private 생성자, static 변수, static 메소드를 정의하지 않고도 싱글톤으로 잘 사용한다. 그래서 간혹 가다 개발자들이 싱글톤 빈은 상태를 가져도 Thread-Safe할 것이라는 착각을 하는 경우가 있다.

결론부터 말하자면, 스프링은 싱글톤 레지스트리를 통해 private 생성자, static 변수 등의 코드 없이 비즈니스 로직에 집중하고 테스트 코드에 용이한 싱글톤 객체를 제공해 주는 것 뿐이지, 동기화 문제는 개발자가 처리해야 한다. 만약에 싱글톤 빈이 상태를 갖게 되고, 아무런 동기화 처리를 하지 않는다면 멀티 스레드 환경에서 부작용이 발생할 수 있으니 주의해야 한다.

## 출처

- [https://gmlwjd9405.github.io/2018/11/10/spring-beans.html](https://gmlwjd9405.github.io/2018/11/10/spring-beans.html)
- [https://taes-k.github.io/2020/06/14/spring-bean-scope-lifecycle/](https://taes-k.github.io/2020/06/14/spring-bean-scope-lifecycle/)
- [https://chung-develop.tistory.com/64](https://chung-develop.tistory.com/64)
- [https://jojoldu.tistory.com/27](https://jojoldu.tistory.com/27)
- [https://cnu-jinseop.tistory.com/36](https://cnu-jinseop.tistory.com/36)
- [https://multifrontgarden.tistory.com/253](https://multifrontgarden.tistory.com/253)
- [https://steady-coding.tistory.com/459](https://steady-coding.tistory.com/459)
- [https://velog.io/@syleemk/Spring-Core-싱글톤-컨테이너](https://velog.io/@syleemk/Spring-Core-%EC%8B%B1%EA%B8%80%ED%86%A4-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88)

## 예상 면접 질문 및 답변

### 스프링 빈이란?

스프링 빈은 스프링 컨테이너에 의해 관리되는 자바 객체(POJO)를 의미한다.

### 스프링 컨테이너란?

스프링 컨테이너는 스프링 빈의 생명 주기를 관리하며, 생성된 스프링 빈들에게 추가적인 기능을 제공하는 역할을 한다. IoC와 DI의 원리가 스프링 컨테이너에 적용된다.

### 스프링 빈 등록 방식을 설명하라.

클래스 위에 @Component를 붙이거나, @Configuration이 붙은 설정 파일 내의 메소드 위에 @Bean을 붙인 후 ComponentScan을 이용하여 스프링 빈을 등록하는 방법이 있다.

### @Component vs @Bean

- @Bean
    - 개발자가 컨트롤이 불가능한 **외부 라이브러리들을 Bean으로 등록하고 싶은 경우**
    에 사용된다.
    - 메소드 또는 어노테이션 단위에 붙일 수 있다.
- @Component
    - 개발자가 **직접 컨트롤이 가능한 클래스들의 경우**에 사용된다.
    - 클래스 또는 인터페이스 단위에 붙일 수 있다.

### @Configuration은 어떻게 싱글톤 빈을 보장하는가?

클래스의 바이트 코드를 조작하는 CGLIB 라이브러리를 사용하여 싱글톤을 보장한다. CGLIB는 프록시 객체의 일종으로 설정 파일이 빈으로 등록될 때, 해당 설정 파일을 상속 받은 프록시 객체가 빈으로 등록된다. 그리고 설장 파일에서 @Bean이 붙은 메소드마다 이미 스프링 빈이 존재하면 존재하는 빈을 반환하고, 스프링 빈이 없으면 비로소 등록하고 반환하는 식으로 싱글톤을 보장한다.

### Bean Lite Mode가 무엇인가?

Bean Lite Mode는 CGLIB를 이용하여 바이트 코드 조작을 하지 않는 방식을 의미한다. 이때 스프링 빈에 대해 싱글톤을 보장하지 않는다.

### 스프링 빈 스코프에 대해 설명하라.

스프링에서 Singleton과 Prototype 빈 스코프를 제공하고 있으며, 스프링 MVC 웹 애플리케이션을 사용할 경우 웹 스코프를 제공한다.

### Singleton 빈이란?

- 싱글톤 빈은 스프링 컨테이너에서 한 번만 생성되며, 컨테이너가 사라질 때 제거된다.
- 생성된 하나의 인스턴스는 Spring Beans Cache에 저장되고, 해당 빈에 대한 요청과 참조가 있으며 캐시된 객체를 반환한다. 하나만 생성되기 때문에 동일 참조를 보장한다.
- 기본적으로 모든 빈은 스코프가 명시적으로 지정되지 않으면 싱글톤이다.

### Prototype 빈이란?

- 프로토 타입 빈은 DI가 발생할 때마다 새로운 객체가 생성되어 주입된다.
- 빈 소멸에 스프링 컨테이너가 관여하지 않고, gc에 의해 빈이 제거된다.

### 언제 Singleton 빈을 사용하고, 언제 Prototype 빈을 사용하는가?

- 싱글톤 타입으로 적합한 객체
    - 상태가 없는 공유 객체
    - 읽기 전용으로만 상태를 가진 객체
    - 쓰기가 가능한 상태를 지니면서도 사용 빈도가 매우 높은 객체
        - 단, 이때는 동기화 전략이 필요함.
- 프로토 타입으로 적합한 객체
    - 사용할 때마다 상태가 달라져야 하는 객체
    - 쓰기가 가능한 상태가 있는 객체

### Singleton 빈과 Prototype 빈과 같이 사용할 때 어떤 문제가 생길 수 있는가?

싱글톤 객체가 프로토 타입 객체를 가지고 있는 경우에는 의도한 것과 다른 결과를 낼 수도 있다. 이미 싱글톤 빈으로 생성되는 시점에 프로토 타입 빈이 생성되어 들어오기 때문에 싱글톤 빈 내부의 프로토 타입 빈을 호출하게 되면 매번 같은 값을 가져 온다.

### Singleton 빈 내부의 Prototype 빈을 사용할 때마다 다른 인스턴스를 받아오려면 어떻게 해야 하는가?

Java.inject의 Provider를 사용하거나, 빈 스코프 설정에서 프록스 모드를 TAGET_CLASS로 설정하면 된다.

### 웹 스코프가 무엇인가?

웹 스코프는 웹 환경에서만 동작하는 스코프이며 프로토 타입과 다르게 특정 주기가 끝날 때까지 관리를 해 준다. 따라서 @PreDestory와 같은 소멸 콜백이 호출된다는 특징이 있다.

### 웹 스코프의 종류는?

- Request
    - HTTP 요청 하나가 들어오고 나갈 때까지 유지되는 스코프
    - 각각의 HTTP 요청마다 별도의 빈 인스턴스가 생성되고 관리된다.
- Session
    - HTTP Session과 동일한 생명 주기를 가지는 스코프
- Application
    - 서블릿 컨텍스트와 동일한 생명 주기를 가지는 스코프
- WebSocket
    - 웹 소켓과 동일한 생명 주기를 가지는 스코프
    

### Singleton 빈의 생명 주기를 설명하라.

1. 스프링 컨테이너 생성
2. 스프링 빈 생성
3. 의존 관계 주입
4. 초기화 콜백
5. 사용
6. 소멸 전 콜백
7. 스프링 종료

### Prototype 빈의 생명 주기를 설명하라.

1. 스프링 컨테이너 생성
2. 스프링 빈 생성
3. 의존 관계 주입
4. 초기화 콜백
5. 사용
6. GC에 의해 수거

### 싱글톤 빈은 Thread-Safe한가?

결론부터 말하자면 Thread-Safe하지 않다. 스프링은 싱글톤 레지스트리를 통해 private 생성자, static 변수 등의 코드 없이 비즈니스 로직에 집중하고 테스트 코드에 용이한 싱글톤 객체를 제공해 주는 것 뿐이지, 동기화 문제는 개발자가 처리해야 한다. 만약에 싱글톤 빈이 상태를 갖게 되고, 아무런 동기화 처리를 하지 않는다면 멀티 스레드 환경에서 부작용이 발생할 수 있으니 주의해야 한다.
