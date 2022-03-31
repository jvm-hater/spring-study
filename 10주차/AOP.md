## AOP란?

AOP는 Aspect Oriented Programming의 약자로 관점 지향 프로그래밍을 뜻한다. 관점 지향은 어떤 로직을 기준으로 핵심적인 관점, 부가적인 관점으로 나누고, 그 관점을 기준으로 각각 모듈화하겠다는 의미이다. 여기서 모듈화란 어떤 공통된 로직이나 기능을 하나의 단위로 묶는 것을 말한다. 예를 들어, 핵심적인 관점은 비즈니스 로직이 되며, 부가적인 관점은 실행 시간 측정, 트랜잭션, 로깅 등이 될 수 있다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F3ad157fe-9a92-4c0d-bd06-a84c26203e14%2FUntitled.png?table=block&id=15257897-1eac-422d-a06d-0bdff151f682&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

위 그림과 같이 각 클래스에서 반복되는 기능이 생길 수 있는데, 이것을 흩어진 관심사라고 부르며, 이렇게 흩어진 관심사를 Aspect로 모듈화하고 핵심적인 비즈니스 로직에서 분리하여 재사용하겠다는 것이 AOP의 목적이다.

### OOP로도 반복을 줄일 수 있지 않은가?

비즈니스 기능과는 별개의 영역이지만, 필연적으로 대다수의 비즈니스 기능에 분포되어 있는 횡단 관심사를 추상화와 디자인 패턴을 사용하여, 기존 클래스에서 횡단 관심사와 핵심 관심사를 분리할 수 있다. 

예를 들어, 트랜잭션 기능(횡단 관심사)과 비즈니스 기능(핵심 관심사)이 하나의 클래스에 공존하고 있는 UserService 클래스가 있다고 가정하자. 이때 먼저 해야 할 일은 기존 클래스를 특수화하여 횡단 관심사와 핵심 관심사를 분리하는 것이다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F5ccaa4bd-b9aa-41f1-af5b-ceb66ea63712%2FUntitled.png?table=block&id=1e9d3f90-df44-4bf1-a8f3-70a03d31229b&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

설계적 측면으로 UserService 인터페이스 아래에 구현 클래스인 UserServiceImple(핵심 관심사), UserServiceTX(횡단 관심사)를 둘 수 있다.

```java
public interface UserService {

    void addUser(User user);

    void upgradeLevels();
}

public class UserServiceImple implements UserService {

    private UserRepository userRepository;

    public void upgradeLevels() {
        List<User> users = userRepository.findAll();
        for (User user : users) {
            upgradeLevel(user); // 이미 정의되어 있다고 가정
        }
    }

    public void addUser(User user) {
        userRepository.save(user);
    }
}

public class UserServiceTX implements UserService {

    private UserService userService;

    public void upgradeLevels() {
        // 트랜잭션 시작
        userService.upgradeLevels(); // 모든 기능을 구현 객체에 위임.
        // 트랜잭션 종료
    }
}
```

횡단 관심사는 트랜잭션 기능을 핵심 관심사 앞뒤에 배치를 하였고, 핵심 관심사는 유연한 확장을 위해 UserService 인터페이스에 위임한다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Ff928d128-ec98-4663-8a3f-fcabfb83635e%2FUntitled.png?table=block&id=d36ddb89-cd81-4bc1-b957-2d6b3d691d6f&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

그림으로 보면 위와 유사하다. 다만, 기능의 명세를 나타내는 UserService 인터페이스의 `upgradeLevels()` 는 구현 객체인 UserServiceImpl에 위임해 주어야 하므로 기존의 의존 관계를 다음과 같이 변경해야 한다.

```java
UserService -> UserServiceTX -> UserServiceImple
```

즉, UserService에 존재하는 UserServiceTX를 DI하고, UserServiceTX에 존재하는 UserServiceImple를 DI하여 의존 관계를 형성해야 한다. 의존 관계를 재정의함으로써 기존 프로세스에 따라 비즈니스 기능 호출 시, 트랜잭션이 결합한 기능이 호출된다.

```java
public abstract class UserService {
    
    @Autowired
    private UserServiceTX userServiceTX;

    public abstract upgradeLevels();

    public abstract addUser();

    // 부가 기능을 접목한 메소드
    public void upgradeLevelsWithTransaction() {
        userServiceTX.upgradeLevels();
    }
}

public class UserServiceTX implements UserService {

    @Autowired
    private UserServiceImple userServiceImple;

    public void upgradeLevels() {
        // 트랜잭션 시작
        userService.upgradeLevels(); // 모든 기능을 구현 객체에 위임.
        // 트랜잭션 종료
    }
}

public class UserServiceImple implements UserService {

    private UserRepository userRepository;

    public void upgradeLevels() {
        List<User> users = userRepository.findAll();
        for (User user : users) {
            upgradeLevel(user); // 이미 정의되어 있다고 가정
        }
    }

    public void addUser(User user) {
        userRepository.save(user);
    }
}   
```

위와 같이 간단한 템플릿 메소드 패턴으로 변경이 가능하다. 물론 데코레이터 패턴이나 프록시 패턴을 사용하여 추상 클래스가 아닌 인터페이스 단계로도 구현할 수 있다.

하지만 특정 메소드만 횡단 관심사를 적용하고 싶고, 아니면 트랜잭션 외에 다른 횡단 관심사를 추가로 적용하고 싶다면, 추상화의 본질적인 장점과는 다르게 많은 추상 클래스나 인터페이스가 생기게 되고 오히려 이를 관리하는데 큰 비용이 든다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F2d275bbc-714a-4a2b-aa38-46b55a034932%2FUntitled.png?table=block&id=a1a963ca-ad65-49cc-aa31-632b6d00d671&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

따라서 분리된 횡단 관심사는 따로 모듈 형태로 만들어서 설계하고 개발하는 것이 좋다. 결국 AOP는 OOP의 기술적인 한계를 극복하고자 고안된 기술이다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Ff34ab4d9-9ae6-4598-a9e1-708128211c1d%2FUntitled.png?table=block&id=e6b01626-5a79-40d4-9341-fe4b47da1c72&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

### AOP 주요 개념

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F2ed2b54f-ffc1-49c6-804a-2d20b0ce3700%2FUntitled.png?table=block&id=6cc2e2dd-2dbb-49fe-ba52-b73f46648269&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

**핵심 관심사 영역**

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F2275ecbd-9acf-49c4-a521-65129a981b80%2FUntitled.png?table=block&id=e40c574c-39e9-42c1-a730-4ed53cc1063b&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

- Target Object
    - 횡단 기능이 적용될 객체로, 핵심 기능을 들고 있다.
- JoinPoint
    - 타겟 객체 안에서 횡단 기능(Advice)이 적용될 수 있는 여러 위치를 뜻한다.
    

**횡단 관심사 영역**

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F63214bec-8ebe-4641-bfe2-a5782d983927%2FUntitled.png?table=block&id=a7f7327d-cfb0-461b-a7fd-d9cbf040f6e3&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

- Aspect
    - 횡단 관심사를 모듈화한 것이다.
- Advice
    - JoinPoint에 적용할 횡단 코드이다.
    - 부가 기능이라 생각해도 좋다.
- Pointcut
    - 여러 JoinPoint 중 실제적으로 Advice할 JoinPoint이다.
    - 따라서 Advice는 여러 JoinPoint 중에서 Pointcut의 표현식에 명시된 JoinPoint에서 실행된다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F58d9e15d-7a52-48fd-a46b-01e0239f4659%2FUntitled.png?table=block&id=8ca8028e-420c-41d4-8d22-fb35805928de&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

### AOP 특징

- 프록시 패턴 기반의 AOP 구현체이다.
    - 프록시 객체를 쓰는 이유는 기존 코드 변경 없이 접근 제어 및 부가 기능을 추가하기 위함.
- 스프링 빈에만 AOP를 적용할 수 있음.

### AOP 구현 방법

- 컴파일 시점에 코드에 공통 기능을 삽입하는 방법
- 클래스 로딩 시점에 바이트 코드에 공통 기능을 삽입하는 방법
- 런타임에 프록시 객체를 생성하여 공통 기능을 삽입하는 방법 (프록시 패턴)

## 프록시 패턴

실제 기능을 수행하는 객체 대신 가상의 객체를 사용하여 로직의 흐름을 제어하는 디자인 패턴이다.

### 프록시 패턴의 특징

- 원래 하려던 기능을 수행하며 그 외의 부가적인 작업(로깅, 인증, 트랜잭션 등)을 별도로 수행할 수 있다.
- 비용이 많이 드는 연산(DB 쿼리, 대용량 텍스트 파일 등)을 실제로 필요한 시점까지 미룰 수 있다.

### 예제

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F201b1a0b-305e-4e40-9989-c2cc87672335%2FUntitled.png?table=block&id=0b897596-5bf0-47f6-8f60-89610e6bc3e2&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

Service와 Proxy는 동일한 인터페이스를 구현하며, Proxy는 메소드 수행 시 실제 객체(Service)의 메소드에 위임한다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F0a8ba953-760e-4959-87f8-c1da63f6c1f4%2FUntitled.png?table=block&id=cd019cda-9e5a-402d-877c-d3994d5d4238&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

위와 같이 구성된 도식을 구현해 보자. 아래 CashProxy를 CreditCard라고 생각하면 된다.

```java
public interface Payment {
    void pay(int amount);
}

public class Cash implements Payment {

    @Override
    public void pay(int amount){
        System.out.println(amount + " 현금 결제");
    }
}

public class CashProxy implements Payment {

    private final Payment cash = new Cash();

    @Override
    public void pay(int amount) {
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();

        cash.pay(amount);

        stopWatch.stop();
        System.out.println(stopWatch.prettyPrint());
    }
}

public class Store {

    private final Payment payment;

    public Store(Payment payment) {
        this.payment = payment;
    }

    public void butSomething(int amount){
        payment.pay(amount);
    }
}

class StoreTest {

    @Test
    public void cashProxy(){
        Payment payment = new CashProxy();
        Store store = new Store(payment);
        store.butSomething(100); // 실행 시간 측정 로직 출력 o
    }

    @Test
    public void cash(){
        Payment payment = new Cash();
        Store store = new Store(payment);
        store.butSomething(100); // 실행 시간 측정 로직 출력 x
    }
}
```

프록시 객체를 사용하면 부가 기능인 실행 시간이 측정되고, 일반 객체를 사용하면 실행 시간이 측정되지 않는다. 이러한 프록시 패턴을 이용하여 AOP를 구현할 수 있다.

## Spring AOP

### AOP 동작 원리

위에서 언급한 대로 AOP는 프록시 패턴을 사용한다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fcc3e6f99-ff30-473b-b062-c90a72f18bdb%2FUntitled.png?table=block&id=89653fc9-4f44-4b84-ae83-ab68782bc088&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

프록시는 타겟을 감싸서 타겟의 요청을 대신 받아주는 Wrapping 오브젝트이다. 클라이언트에서 타겟을 호출하게 되면 타겟이 아닌, 타겟을 감싸고 있는 프록시가 호출된다. 이때 프록시는 타겟 메소드 실행 전후로 부가 기능을 실행하도록 구성되어 있다.

다만, 프록시 패턴은 타겟 하나 하나마다 프록시 객체를 정의해야하므로 번거롭고 코드의 중복이 생긴다는 점이 있다. 그래서 Spring AOP에서는 런타임 시에 **JDK Dynamic Proxy** 또는 **CGLIB**를 활용하여 프록시를 생성해 준다. 참고로 이를 런타임 위빙(Runtime Weaving)이라고 부르며, 타겟 객체를 새로운 프록시 객체로 적용하는 과정을 의미한다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F70965547-1c7b-4a88-8668-749809c245a0%2FUntitled.png?table=block&id=c38988fe-e1af-42b6-ba6d-672b8f4f2ef5&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

Spring은 AOP 프록시를 생성하는 과정에서 자체 검증 로직을 통해 타겟의 인터페이스 유무를 판단한다. 이때 타겟이 하나 이상의 인터페이스를 구현하고 있는 클래스라면 JDK Dynamic Proxy를 사용하고, 그렇지 않으면 CGLIB의 방식으로 AOP 프록시를 생성해 준다.

### JDK Dynamic Proxy

JDK Dynamic Proxy는 Java의 리플렉션 패키지에 존재하는 Proxy라는 클래스를 통해 생성된 프록시 객체를 의미한다. 리플렉션의 Proxy 클래스가 동적으로 프록시 객체를 생성해 주므로 JDK Dynamic Proxy라는 이름이 붙여졌다.

**프록시 객체 생성 과정**

```java
Object proxy = Proxy.newProxyInstance(ClassLoader       // 클래스로더
                                    , Class<?>[]        // 타겟의 인터페이스
                                    , InvocationHandler // 타겟의 정보가 포함된 Handler
              										  );
```

위 코드와 같이 단순히 리플렉션 Proxy 클래스의 `newProxyInstance()` 메소드를 사용하면 된다. 그리고 전달 받은 파라미터를 가지고 다음과 같이 프록시 객체를 생성한다. (자세한 예제는 하단 참조)

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F650f1dc3-4508-41af-8ce7-af5bdb3c2c3e%2FUntitled.png?table=block&id=9516f9fd-0ec7-4599-a0e0-9e2cc3029a0d&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

1. 타겟의 인터페이스에 대해 자체적인 검증 로직을 거치고, ProxyFactory에 의해 타겟의 인터페이스를 상속한 프록시 객체를 생성한다.
2. 프록시 객체에 InvocationHandler를 포함하여 하나의 객체로 변환한다.

위 과정에서 가장 핵심적인 부분은 인터페이스를 기준으로 프록시 객체를 생성한다는 것이다. 따라서 구현체는 인터페이스를 상속 받아야 하고, @Autowired를 통해 생성된 프록시 빈을 사용하기 위해서는 반드시 인터페이스의 타입으로 지정해야 한다.

```java
@Controller
public class UserController {

  @Autowired
  private MemberService memberService; // <- Runtime Exception 발생...
  ...
}

@Service
public class MemberService implements UserService {

  @Override
  public Map<String, Object> findUserId(Map<String, Object> params) {
    ...isLogic
    return params;
  }
}
```

가령 Spring AOP에 의해 UserService 프록시 빈이 만들어졌다고 하자. 이때, UserService가 아니라 MemberService를 DI 대상으로 선택하면 예외가 발생한다.

**내부 검증 로직**

프록시 패턴은 접근 제어 목적 및 사용자의 요청이 기존의 타겟을 그대로 바라볼 수 있도록 타겟에 대한 위임 코드를 프록시 객체에 작성하기 위해 사용된다. 이러한 위임 코드는 InvocationHandler에 작성해야 한다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F8e8d5715-d6ea-485c-9e46-5fa00743603a%2FUntitled.png?table=block&id=bfab1bd6-2d0a-48cc-a526-c5b033b2c2f1&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

사용자의 요청에 의해 Proxy 메소드가 호출되면, 내부적으로 invoke에 대한 내부 검증 로직이 일어난다.

```java
public Object invoke(Object proxy, Method proxyMethod, Object[] args) throws Throwable {
  Method targetMethod = null;

  // 주입된 타겟 객체에 대한 검증 코드
  if (!cachedMethodMap.containsKey(proxyMethod)) {
    targetMethod = target.getClass().getMethod(proxyMethod.getName(), proxyMethod.getParameterTypes());
    cachedMethodMap.put(proxyMethod, targetMethod);
  } else {
    targetMethod = cachedMethodMap.get(proxyMethod);
  }

  // 타겟의 메소드 실행
  Ojbect retVal = targetMethod.invoke(target, args);
  return retVal;
}
```

JDK Dynamic Proxy는 인터페이스에 대한 Proxy만 생성하기 때문에, 개발자가 타겟에 대한 정보를 잘 못 주입할 경우를 대비하여 내부적으로 타겟 객체에 관한 검증 코드를 형성하고 있다.

**장단점**

- 장점
    - 개발자가 직접 프록시 객체를 만들 필요가 없다.
- 단점
    - 프록시하려는 클래스는 반드시 인터페이스의 구현체여야한다.
    - 리플렉션을 활용하므로 성능이 떨어진다.

**이해를 돕기 위한 예제 코드**

위의 내용은 실제 Spring AOP에서 사용된 코드이므로 조금 이해하기 어려울 수 있다. 그래서 Spring AOP는 잠시 잊고 JDK Dynamic Proxy를 어떻게 사용하는지 살펴 보려고 한다.

```java
public interface Person {

    void speak(String message);
}

public class Jayon implements Person {

    @Override
    public void speak(String message) {
        System.out.println(message);
    }
}

public class MyInvocationHandler implements InvocationHandler {

    private Person target;

    public MyInvocationHandler(Person target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("BEFORE");
        method.invoke(target, args);
        System.out.println("AFTER");
        return null;
    }
}

public class Main {

    public static void main(String[] args) {
        Jayon jayon = new Jayon();
        MyInvocationHandler myInvocationHandler = new MyInvocationHandler(jayon);
        Person person = (Person) Proxy.newProxyInstance(
            Person.class.getClassLoader(),
            new Class[]{Person.class},
            myInvocationHandler);
        person.speak("JDK Dynamic Proxy"); // 프록시 객체
        jayon.speak("제이온"); // 일반 객체
    }
}
```

Person 인터페이스의 구현체인 Jayon을 정의하고, 간단한 핸들러를 정의한다. 해당 핸들러는 원본 객체의 메소드를 호출하기 전후에 로그를 찍도록 하였다. `newProxyInstance()` 메소드를 사용하여 Person 인터페이스의 프록시 객체를 만들 수 있다. 이제 프록시 객체와 일반 객체를 실행했을 때 결과를 살펴 보자.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fe6a03251-2dc1-40b4-b012-b79c6affcae5%2FUntitled.png?table=block&id=0094c0b8-1855-4d38-babd-6c944f5dc52f&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

예상한 대로 프록시 객체는 메소드 호출 전후에 간단한 로그가 찍히고, 일반 객체는 로그 없이 메소드 호출만 되는 것을 확인할 수 있다.

만약 위 코드에서 인터페이스가 아니라 구현체를 대상으로 프록시를 만드려고 하면 에러가 발생한다.

```java
public class Main {

    public static void main(String[] args) {
        Jayon jayon = new Jayon();
        MyInvocationHandler myInvocationHandler = new MyInvocationHandler(jayon);
        Person person = (Jayon) Proxy.newProxyInstance(
            Jayon.class.getClassLoader(),
            new Class[]{Jayon.class},
            myInvocationHandler);
        person.speak("JDK Dynamic Proxy");
        jayon.speak("제이온");
    }
}
```

위와 같이 `newProxyInstance()` 인자로 인터페이스의 구현체인 Jayon을 주면 된다. 실행해 보자.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fbe25b002-1ccf-4a30-b40d-c5deddfeb4e4%2FUntitled.png?table=block&id=80bd50c1-1a15-43ed-bdc2-83b5b37b729d&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

Jayon 객체는 인터페이스가 아니라는 런타임 예외가 발생하는 것을 확인할 수 있다.

### CGLIB

CGLIB는 Code Generator Libray의 약자로, 클래스의 바이트 코드를 조작하여 프록시 객체를 생성해 주는 라이브러리다. CGLIB를 사용하면 인터페이스가 아닌 타겟 클래스에 대해서도 프록시 객체를 만들어 줄 수 있고, 이 과정에서 Enhancer라는 클래스를 활용한다.

**프록시 객체 생성 과정**

먼저 의존성을 추가해 주자. 아래는 gradle의 예시인데, maven을 사용해도 좋다.

```java
implementation group: 'cglib', name: 'cglib', version: '3.2.4'
```

그리고 아래와 같이 Enhancer를 사용하여 프록시 객체를 생성할 수 있다. (자세한 예제는 하단 참조)

```java
Enhancer enhancer = new Enhancer();
enhancer.setSuperclass(MemberService.class); // 타겟 클래스
enhancer.setCallback(MethodInterceptor); // Handler
Object proxy = enhancer.create(); // Proxy 생성
```

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Ff016d2e6-fd93-45a9-93a1-cc365393a64a%2FUntitled.png?table=block&id=ec93731a-47c2-4252-8119-652e36a2585d&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

CGLIB는 타겟의 클래스를 상속 받아서 위 그림처럼 프록시를 생성해 준다. 이 과정에서 CGLIB는 타겟 클래스에 포함된 모든 메소드를 재정의하고, 타겟 클래스에 대한 바이트 코드를 조작하여 프록시를 생성한다. 따라서 CGLIB를 적용할 클래스는 final 메소드가 들어있거나, final 클래스면 안 된다. 더불어, private 접근자로 된 메소드도 상속이 불가하므로 적용되지 않는다.

**장단점**

- 장점
    - 인터페이스 없이 단순 클래스만으로도 프록시 객체를 동적으로 생성해 줄 수 있다.
    - 리플렉션이 아닌 바이트 조작을 사용하며, 타겟에 대한 정보를 알고 있기 때문에 JDK Dynamic Proxy에 비해 성능이 좋다.
- 단점
    - 의존성을 추가해야 한다. (Spring 3.2 이후 버전의 경우 Spring Core 패키지에 포함되어 있음)
    - default 생성자가 필요하다. (현재는 objenesis 라이브러리를 통해 해결)
    - 타겟의 생성자가 두 번 호출된다. (현재는 objenesis 라이브러리를 통해 해결)

**이해를 돕기 위한 예제 코드**

CGLIB 라이브러리를 사용하여 프록시 객체를 만들어 보자. 위에서 사용한 예제 코드를 거의 그대로 가져오되, Handler만 수정하자. 기존 InvocationHandler 대신 CGLIB 라이브러리 소속인 MethodInterceptor를 사용하면 된다.

```java
class MyMethodInterceptor implements MethodInterceptor {

    private final Person target;

    public MyMethodInterceptor(Person target) {
        this.target = target;
    }

    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        System.out.println("BEFORE");
        method.invoke(target, args);
        System.out.println("AFTER");
        return null;
    }
}
```

그리고 메인 메소드는 다음과 같이 구성한다.

```java
public class Main {

    public static void main(String[] args) throws IOException {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(Jayon.class);
        enhancer.setCallback(new MyMethodInterceptor(new Jayon()));
        Jayon jayon = (Jayon) enhancer.create();
        jayon.speak("CGLIB");
    }
}
```

Enhancer 객체에 타겟 클래스와 핸들러 정보를 넘겨 주고 `create()` 메소드를 실행하면 바이트 코드를 조작한 프록시 객체를 얻어올 수 있다. 이제 위 코드를 실행해 보자.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Faeb2d5e7-d219-42ee-836a-d7804befa741%2FUntitled.png?table=block&id=63069349-3265-49dd-b3ba-7b7e803a29a3&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

정상적으로 프록시를 거쳐서 로직이 수행된 것을 확인할 수 있다. CGLIB는 인터페이스가 없는 일반 클래스에 대해서 프록시를 만들어준다는 것이 큰 특징이다.

### JDK Dynamic Proxy와 CGLIB의 성능 차이

CGIB는 타겟에 대한 정보를 직접적으로 제공 받고, 타겟 클래스에 대한 바이트 코드를 조작하여 프록시를 생성하므로 리플렉션을 사용하는 JDK Dynamic Proxy에 비해 성능이 좋다. 
또한 CGLIB는 메소드가 처음 호출되었을 때 동적으로 타겟 클래스의 바이트 코드를 조작하고, 이후 호출 시엔 조작된 바이트 코드를 재사용한다.

### Spring AOP와 JDK Dynamic Proxy, CGLIB

위에서 이야기 한 JDK Dynamic Proxy와 CGLIB를 AOP 개념으로 바라 보자.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fc66d11cf-1a95-49d8-8eac-3b3261f8a8e6%2FUntitled.png?table=block&id=097f680c-30ed-4adc-a028-2191ecb7edae&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

JDK Dynamic Proxy에서 InvocationHandler, CGLIB에서 MethodInterceptor는 Spring AOP에서 JoinPoint라는 개념과 일치한다. 그리고 위에서 Person은 PointCut이라는 개념과 일치한다. 

마지막으로 프록시 로직이 실행되는 JDK Dynamic Proxy에 `invoke()` 메소드, CGLIB에서 `intercept()` 메소드는 Advice라는 개념과 일치한다.

### Spring AOP 주의 사항

- 타겟 클래스의 자기 호출(inner method invoke)은 AOP가 동작하지 않는다.
    - 클라이언트는 프록시를 타겟 클래스라고 생각하고, 프록시 메소드를 호출하게 된다.
    - 프록시는 클라이언트로부터 요청을 받으면 타겟 클래스의 메소드로 위임하고, 경우에 따라 부가 작업을 추가한다.
    - 즉 프록시는 **클라이언트가 타겟 클래스를 호출하는 과정에만 동작**한다.
    - 타겟 클래스가 자기 자신의 메소드를 호출할 때는 AOP가 적용되지 않고 대상 객체를 직접 호출하게 된다.
- 프록시를 만들 때, Default로 CGLIB 라이브러리를 사용하여 클래스를 상속하므로 private 메소드에서 AOP가 동작하지 않는다.
    - 마찬가지로 final 메소드나 final 클래스에도 AOP가 적용되지 않는다.

## 출처

- [https://engkimbs.tistory.com/746](https://engkimbs.tistory.com/746)
- [https://dong-co.tistory.com/84](https://dong-co.tistory.com/84)
- [https://tecoble.techcourse.co.kr/post/2021-06-25-aop-transaction/](https://tecoble.techcourse.co.kr/post/2021-06-25-aop-transaction/)
- [https://sa1341.github.io/2019/05/25/스프링-AOP-개념-및-Proxy를-이용한-구동원리/](https://sa1341.github.io/2019/05/25/%EC%8A%A4%ED%94%84%EB%A7%81-AOP-%EA%B0%9C%EB%85%90-%EB%B0%8F-Proxy%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EA%B5%AC%EB%8F%99%EC%9B%90%EB%A6%AC/)
- [https://velog.io/@hanblueblue/Spring-Proxy-1-Java-Dynamic-Proxy-vs.-CGLIB](https://velog.io/@hanblueblue/Spring-Proxy-1-Java-Dynamic-Proxy-vs.-CGLIB)
- [https://jypthemiracle.medium.com/spring-transaction-관리에-대한-메모-f391fd2885b4](https://jypthemiracle.medium.com/spring-transaction-%EA%B4%80%EB%A6%AC%EC%97%90-%EB%8C%80%ED%95%9C-%EB%A9%94%EB%AA%A8-f391fd2885b4)
- [https://huisam.tistory.com/entry/springAOP](https://huisam.tistory.com/entry/springAOP)
- [https://gmoon92.github.io/spring/aop/2019/01/15/aspect-oriented-programming-concept.html](https://gmoon92.github.io/spring/aop/2019/01/15/aspect-oriented-programming-concept.html)

## 예상 면접 질문 및 답변

### AOP란?

AOP는 Aspect Oriented Programming의 약자로 관점 지향 프로그래밍을 뜻한다. 관점 지향은 어떤 로직을 기준으로 핵심적인 관점, 부가적인 관점으로 나누고, 그 관점을 기준으로 각각 모듈화하겠다는 의미이다. 여기서 모듈화란 어떤 공통된 로직이나 기능을 하나의 단위로 묶는 것을 말한다. 예를 들어, 핵심적인 관점은 비즈니스 로직이 되며, 부가적인 관점은 실행 시간 측정, 트랜잭션, 로깅 등이 될 수 있다.

### AOP의 장점은?

각 클래스에서 로깅, 트랜잭션과 같은 반복되는 기능이 생길 수 있는데, 이것을 흩어진 관심사라고 부르며, 이렇게 흩어진 관심사를 Aspect로 모듈화하고 핵심적인 비즈니스 로직에서 분리하면 재사용할 수 있고 OOP를 더욱 잘 구현할 수 있다.

### 프록시 패턴과 특징을 설명하라.

프록시 패턴은 실제 기능을 수행하는 객체 대신 가상의 객체를 사용하여 로직의 흐름을 제어하는 디자인 패턴이다. 

- 특징
    - 원래 하려던 기능을 수행하며 그 외의 부가적인 작업(로깅, 인증, 트랜잭션 등)을 별도로 수행할 수 있다.
    - 비용이 많이 드는 연산(DB 쿼리, 대용량 텍스트 파일 등)을 실제로 필요한 시점까지 미룰 수 있다.

### Spring AOP 동작 원리를 설명하라.

프록시는 타겟을 감싸서 타겟의 요청을 대신 받아주는 Wrapping 오브젝트이다. 클라이언트에서 타겟을 호출하게 되면 타겟이 아닌, 타겟을 감싸고 있는 프록시가 호출된다. 이때 프록시는 타겟 메소드 실행 전후로 부가 기능을 실행하도록 구성되어 있다.

다만, 프록시 패턴은 타겟 하나 하나마다 프록시 객체를 정의해야하므로 번거롭고 코드의 중복이 생긴다는 점이 있다. 그래서 Spring AOP에서는 런타임 시에 **JDK Dynamic Proxy** 또는 **CGLIB**를 활용하여 프록시를 생성해 준다.

### JDK Dynamic Proxy란?

JDK Dynamic Proxy는 Java의 리플렉션 패키지에 존재하는 Proxy라는 클래스를 통해 생성된 프록시 객체를 의미한다. 리플렉션의 Proxy 클래스가 동적으로 프록시 객체를 생성해 주므로 JDK Dynamic Proxy라는 이름이 붙여졌다.

### JDK Dynamic Proxy의 동작 과정을 설명하라.

1. 타겟의 인터페이스에 대해 자체적인 검증 로직을 거치고, ProxyFactory에 의해 타겟의 인터페이스를 상속한 프록시 객체를 생성한다.
2. 프록시 객체에 InvocationHandler를 포함하여 하나의 객체로 변환한다.

### JDK Dynamic Proxy의 장단점을 설명하라.

- 장점
    - 개발자가 직접 프록시 객체를 만들 필요가 없다.
- 단점
    - 프록시하려는 클래스는 반드시 인터페이스의 구현체여야한다.
    - 리플렉션을 활용하므로 성능이 떨어진다.

### CGLIB란?

CGLIB는 Code Generator Libray의 약자로, 클래스의 바이트 코드를 조작하여 프록시 객체를 생성해 주는 라이브러리다. CGLIB를 사용하면 인터페이스가 아닌 타겟 클래스에 대해서도 프록시 객체를 만들어 줄 수 있고, 이 과정에서 Enhancer라는 클래스를 활용한다.

### CGLIB의 동작 과정을 설명하라.

CGLIB는 타겟 클래스의 상속을 받아서 프록시를 생성한다. 이때 타겟 클래스에 포함된 모든 메소드를 재정의하고, 타겟 클래스에 대한 바이트 코드를 조작하여 프록시를 생성한다. 

### CGLIB의 장단점을 설명하라.

- 장점
    - 인터페이스 없이 단순 클래스만으로도 프록시 객체를 동적으로 생성해 줄 수 있다.
    - 리플렉션이 아닌 바이트 조작을 사용하며, 타겟에 대한 정보를 알고 있기 때문에 JDK Dynamic Proxy에 비해 성능이 좋다.
- 단점
    - 의존성을 추가해야 한다. (Spring 3.2 이후 버전의 경우 Spring Core 패키지에 포함되어 있음)
    - default 생성자가 필요하다. (현재는 objenesis 라이브러리를 통해 해결)
    - 타겟의 생성자가 두 번 호출된다. (현재는 objenesis 라이브러리를 통해 해결)

### Spring AOP를 사용할 때 주의할 점이 있는가?

- 타겟 클래스의 자기 호출(inner method invoke)은 AOP가 동작하지 않는다.
    - 클라이언트는 프록시를 타겟 클래스라고 생각하고, 프록시 메소드를 호출하게 된다.
    - 프록시는 클라이언트로부터 요청을 받으면 타겟 클래스의 메소드로 위임하고, 경우에 따라 부가 작업을 추가한다.
    - 즉 프록시는 **클라이언트가 타겟 클래스를 호출하는 과정에만 동작**한다.
    - 타겟 클래스가 자기 자신의 메소드를 호출할 때는 AOP가 적용되지 않고 대상 객체를 직접 호출하게 된다.
- 프록시를 만들 때, Default로 CGLIB 라이브러리를 사용하여 클래스를 상속하므로 private 메소드에서 AOP가 동작하지 않는다.
    - 마찬가지로 final 메소드나 final 클래스에도 AOP가 적용되지 않는다.
