## Java 비동기 처리
Spring @Async를 살펴 보기 전에 동기, 비동기, 멀티 스레드의 개념은 필수적이다. 해당 개념을 알고 있다고 가정하고, 순수 Java 비동기 처리 방식을 코드로 알아 보자. 만약 Java 스레드의 익숙하다면 해당 챕터는 건너뛰어도 무방하다.

```java
public class MessageService {

    public void print(String message) {
        System.out.println(message);
    }
}

public class Main {

    public static void main(String[] args) {
        MessageService messageService = new MessageService();

        for (int i = 1; i <= 100; i++) {
            messageService.print(i + "");
        }
    }
}
```

만약 message를 받아서 단순히 출력하는 기능을 동기 방식으로 만든다면 위와 같이 코드를 작성할 수 있다. 이를 멀티 스레딩 비동기 방식으로 바꾸면 다음과 같이 소스 코드를 작성할 수 있다.

```java
public class MessageService {

    public void print(String message) {
        new Thread(() -> System.out.println(message))
                .start();
    }
}

public class Main {

    public static void main(String[] args) {
        MessageService messageService = new MessageService();

        for (int i = 1; i <= 100; i++) {
            messageService.print(i + "");
        }
    }
}
```

하지만 이 방법은 Thread를 관리할 수 없어서 매우 위험하다. 가령, 동시에 10,000개의 호출이 이뤄진다면 아주 짧은 시간에 Thread를 10,000개 생성해야 한다. Thread를 생성하는 비용은 적지 않기 때문에 프로그램의 성능에 악영향을 미치며, 심지어는 OOM 에러가 발생할 수 있다. 따라서 Thread를 관리하기 위해서는 Thread Pool을 구현해야 하고, Java에서는 ExecutorService 클래스를 제공하고 있다.

```java
public class MessageService {

    private final ExecutorService executorService = Executors.newFixedThreadPool(10);

    public void print(String message) {
        executorService.submit(() -> System.out.println(message));
    }
}

public class Main {

    public static void main(String[] args) {
        MessageService messageService = new MessageService();

        for (int i = 1; i <= 100; i++) {
            messageService.print(i + "");
        }
    }
}
```

전체 스레드의 개수를 10개로 제한하고 있고, 우리가 원하는 멀티 스레딩 방식의 비동기 처리도 올바르게 할 수 있게 되었다. 하지만, 위 방식은 비동기 방식으로 처리하고 싶은 메소드마다 ExecutorService의 `submit()` 메소드를 적용해야 하므로 반복적인 수정 작업을 해야 한다. 즉, 처음에는 동기 로직으로 작성한 메소드를 비동기로 바꾸고 싶다면 메소드 자체의 로직 변경이 불가피하다는 것이다.

## Spring @Async

### 단순한 방법

```java
@EnableAsync
@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

}

@Service
public class MessageService {

    @Async
    public void print(String message) {
        System.out.println(message);
    }
}

@RequiredArgsConstructor
@RestController
public class MessageController {

    private final MessageService messageService;

    @GetMapping("/messages")
    @ResponseStatus(code = HttpStatus.OK)
    public void printMessage() {
        for (int i = 1; i <= 100; i++) {
            messageService.print(i + "");
        }
    }
}
```

@EnableAsync 어노테이션을 Application 클래스 위에 붙여 주고, 비동기 방식으로 처리하고 싶은 동기 로직의 메소드 위에 @Async 어노테이션을 붙이면 끝이다. 하지만 위 방식은 스레드를 관리하지 않는다는 문제가 있다. 왜냐하면 @Async의 기본 설정은 SimpleAsyncTaskExecutor를 사용하도록 되어 있는데, 이것은 스레드 풀이 아니고 단순히 스레드를 만들어내는 역할을 하기 때문이다.

### 스레드 풀을 사용하는 방법

우선 Application 클래스에서 @EnableAsync를 제거한다. Application 클래스에 @EnableAutoConfiguration 혹은 @SpringBootApplication 설정이 되어 있다면, 런타임 시 @Configuration이 설정된 SpringAsyncConfig 클래스(아래에서 생성할 예정)의 threadPoolTaskExecutor 빈 정보를 읽어 들이기 때문이다.

```java
@Configuration
@EnableAsync // Application이 아닌, Async 설정 클래스에 붙여야 함.
public class SpringAsyncConfig {

    @Bean(name = "threadPoolTaskExecutor")
    public Executor threadPoolTaskExecutor() {
        ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
        taskExecutor.setCorePoolSize(3); // 기본 스레드 수
        taskExecutor.setMaxPoolSize(30); // 최대 스레드 수
        taskExecutor.setQueueCapacity(100); // Queue 사이즈
        taskExecutor.setThreadNamePrefix("Executor-");
        return taskExecutor;
    }

}
```

core와 max 사이즈를 설정할 수 있다. 이때 최초 core 사이즈만큼 동작하다가 이 이상 작업을 처리할 수 없을 경우 max 사이즈만큼 스레드가 증가할 것이라고 예상하겠지만 그렇지 않다.

내부적으로 Integer.MAX_VALUE 사이즈의 LinkedBlockingQueue를 생성해서 core 사이즈만큼의 스레드에서 작업을 처리할 수 없을 경우 Queue에서 대기하게 된다. Queue가 꽉 차게 되면 그때 max 사이즈만큼 스레드를 생성해서 처리하게 된다.

이때 Queue 사이즈를 Integer.MAX_VALUE로 설정하기 부담스럽다면 queueCapacity를 설정해 줄 수 있다. 위와 같이 설정한다면, 최초 3개의 스레드에서 처리하다가 처리 속도가 밀릴 경우 작업을 100개 사이즈 Queue에 넣어 놓고,  그 이상의 요청이 들어오면 최대 30개의 스레드를 생성해서 작업을 처리하게 된다.

스레드 풀 설정이 완료되었다면, @Async 어노테이션이 붙은 메소드에서 위 빈의 이름을 붙이면 된다.

```java
@Service
public class MessageService {

    @Async("threadPoolTaskExecutor")
    public void print(String message) {
        System.out.println(message);
    }
}
```

만약에 스레드 풀의 종류를 여러 개 설정하고 싶다면, `threadPoolTaskExecutor()` 같은 빈 생성 메소드를 여러 개 만들고, @Async 설정할 때 원하는 스레드 풀 빈을 넣으면 된다.

### 리턴 타입 별 반환되는 형태

**리턴 값이 없는 경우**

비동기로 처리해야 하는 메소드가 처리 결과를 전달할 필요가 없는 경우이다. 이 경우에는 @Async 어노테이션의 리턴 타입을 void로 설정하면 된다.

**리턴 값이 있는 경우**

Future, ListenableFuture, CompletableFuture 타입을 리턴 타입으로 사용할 수 있다. 비동기 메소드의 반환 형태를 `new AsyncResult()` 로 묶으면 된다.

[Future]

```java
@Service
public class MessageService {

    @Async
    public Future<String> print(String message) throws InterruptedException {
        System.out.println("Task Start - " + message);
        Thread.sleep(3000);
        return new AsyncResult<>("jayon-" + message);
    }
}

@RequiredArgsConstructor
@RestController
public class MessageController {

    private final MessageService messageService;

    @GetMapping("/messages")
    @ResponseStatus(code = HttpStatus.OK)
    public void printMessage() throws ExecutionException, InterruptedException {
        for (int i = 1; i <= 5; i++) {
            Future<String> future = messageService.print(i + "");
            System.out.println(future.get());
        }
    }
}

// 실행 결과
Task Start - 1
jayon-1
Task Start - 2
jayon-2
Task Start - 3
jayon-3
Task Start - 4
jayon-4
Task Start - 5
jayon-5
```

`future.get()` 은 블로킹을 통해 요청 결과가 올 때까지 기다리는 역할을 한다. 그래서 비동기 블로킹 방식이 되어버려 성능이 좋지 않다. 보통 Future는 사용하지 않는다.

[ListenableFuture]

```java
@Service
public class MessageService {

    @Async
    public ListenableFuture<String> print(String message) throws InterruptedException {
        System.out.println("Task Start - " + message);
        Thread.sleep(3000);
        return new AsyncResult<>("jayon-" + message);
    }
}

@RequiredArgsConstructor
@RestController
public class MessageController {

    private final MessageService messageService;

    @GetMapping("/messages")
    @ResponseStatus(code = HttpStatus.OK)
    public void printMessage() throws InterruptedException {
        for (int i = 1; i <= 5; i++) {
            ListenableFuture<String> listenableFuture = messageService.print(i + "");
            listenableFuture.addCallback(System.out::println, error -> System.out.println(error.getMessage()));
        }
    }
}

// 실행 결과
Task Start - 1
Task Start - 3
Task Start - 2
jayon-1
jayon-2
Task Start - 5
jayon-3
Task Start - 4
jayon-4
jayon-5
```

ListenableFuture은 콜백을 통해 논블로킹 방식으로 작업을 처리할 수 있다. `addCallback()` 메소드의 첫 번째 파라미터는 작업 완료 콜백 메소드, 두 번째 파라미터는 작업 실패 콜백 메소드를 정의하면 된다. 참고로, 스레드 풀의 core 스레드를 3개로 설정했으므로 “Task Start” 메시지가 처음에 3개 찍히는 것을 확인할 수 있다.

[CompletableFuture]

ListenableFuture만으로도 논블로킹 로직을 구현할 수 있지만, 콜백 안에 콜백이 필요할 경우 콜백 지옥이라고 불리는 매우 복잡한 코드를 유발하게 된다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F9f152db8-c015-43cd-bf13-85c594f5f218%2FUntitled.png?table=block&id=268ac0bc-ca7b-4bcb-b11b-ac611a5038b2&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

물론 이번 시간은 CompletableFuture을 자세하게 다루지 않을 것이므로 복잡한 콜백을 대처하는 코드가 궁금하다면 아래 출처의 링크를 참고하길 바란다.

```java
@Service
public class MessageService {

    @Async
    public CompletableFuture<String> print(String message) throws InterruptedException {
        System.out.println("Task Start - " + message);
        Thread.sleep(3000);
        return new AsyncResult<>("jayon-" + message).completable();
    }
}

@RequiredArgsConstructor
@RestController
public class MessageController {

    private final MessageService messageService;

    @GetMapping("/messages")
    @ResponseStatus(code = HttpStatus.OK)
    public void printMessage() throws InterruptedException {
        for (int i = 1; i <= 5; i++) {
            CompletableFuture<String> completableFuture = messageService.print(i + "");
            completableFuture
                    .thenAccept(System.out::println)
                    .exceptionally(error -> {
                        System.out.println(error.getMessage());
                        return null;
                    });
        }
    }
}
```

ListenableFuture의 콜백 정의보다 가독성이 좋아졌으며, 논블로킹 기능을 완벽하게 수행한다. 따라서 @Async를 사용할 때 리턴 값이 필요하다면 CompletableFuture를 사용할 것을 권장한다.

### @Async의 장점

개발자는 메소드를 동기 방식으로 작성하다가, 비동기 방식을 원한다면 단순히 @Async 어노테이션을 메소드 위에 붙여 주면 된다. 그래서 동기, 비동기에 대해 유지 보수가 좋은 코드를 만들 수 있다.

### @Async의 주의 사항

@Async 기능을 사용하기 위해서는 @EnableAsync 어노테이션을 선언하는데, 이때 별도의 설정을 하지 않으면 프록시 모드로 동작한다. 즉, @Async 어노테이션으로 동작하는 비동기 메소드는 모두 스프링 AOP의 제약 사항을 그대로 따르게 된다. 자세한 이유는 [해당 포스팅](https://steady-coding.tistory.com/608)을 참고하길 바란다.

- private 메소드에 @Async를 붙여도 AOP가 동작하지 않는다.
- 같은 객체 내의 메소드끼리 호출할 시 AOP가 동작하지 않는다.

## 출처

- [http://dveamer.github.io/java/SpringAsync.html](http://dveamer.github.io/java/SpringAsync.html)
- [https://brunch.co.kr/@springboot/401](https://brunch.co.kr/@springboot/401)
- [https://kapentaz.github.io/spring/Spring-ThreadPoolTaskExecutor-설정/#](https://kapentaz.github.io/spring/Spring-ThreadPoolTaskExecutor-%EC%84%A4%EC%A0%95/#)
- [https://www.hungrydiver.co.kr/bbs/detail/develop?id=2](https://www.hungrydiver.co.kr/bbs/detail/develop?id=2)
