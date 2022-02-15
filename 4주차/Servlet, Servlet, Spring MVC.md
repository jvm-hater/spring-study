# Servlet, Servlet Container, Spring MVC 총 정리

## Servlet

서블릿은 클라이언트 요청을 처리하고, 그 결과를 다시 클라이언트에게 전송하는 Servlet 클래스의 구현 규칙을 지킨 자바 프로그램이다. 

이전의 웹 프로그램들은 클라이언트의 요청에 대한 응답으로 만들어진 페이지를 넘겨 주었으나, 현재는 동적인 페이지를 가공하기 위해서 웹 서버가 다른 곳에 도움을 요청한 후 가공된 페이지를 넘겨주게 된다. 이때 서블릿을 사용하게 되면 웹 페이지를 동적으로 생성하여 클라이언트에게 반환해 줄 수 있다.

### Servlet의 예시

```java
@WebServlet(name = "helloServlet", urlPatterns = "/hello")
public class HelloServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // 애플리케이션 로직
    }
}
```

- `urlPatterns("/hello")` 의 URL이 호출되면 서블릿 코드가 실행된다.
- HttpServletRequest를 통해 HTTP 요청 정보를 사용할 수 있다.
- HttpServletResponse를 통해 HTTP 응답 정보를 사용할 수 있다.

### Servlet의 동작 방식

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Feabc004c-2a0d-4bc4-b1df-50366002ad7f%2FUntitled.png?table=block&id=e3c51603-b3fb-4edf-83b3-7a4191e9cc9f&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=1700&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

- 사용자가 URL을 입력하면 요청이 서블릿 컨테이너로 전송된다.
- 요청을 전송 받은 서블릿 컨테이너는 Http Request, HttpResponse 객체를 생성한다.
- 사용자가 요청한 URL이 어느 서블릿에 대한 요청인지 찾는다. 위 예제에서는 helloServlet을 찾게 된다.
- 서블릿의 `service()` 메소드를 호출한 후 클라이언트의 GET, POST 여부에 따라 `doGet()`, `doPost()` 메소드를 호출한다.
- 동적 페이지를 생성한 후 HttpServletResponse 객체에 응답을 보낸다.
- 클라이언트에 최종 결과를 응답한 후 HttpServletRequest, HttpServletResponse 객체를 소멸한다.

### Servlet의 생명 주기

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fc4b5641c-c22d-498c-aabf-6c3b5aa4246a%2FUntitled.png?table=block&id=65c4b86b-e6c9-4e15-934b-4b2820a09956&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=1700&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

- 클라이언트 요청이 들어오면 서블릿 컨테이너는 서블릿이 메모리에 있는지 확인한다. 메모리에 없다면 `init()` 메소드를 호출하여 적재한다.
- 클라이언트 요청에 따라서 `service()` 메소드를 통해 요청에 대한 응답이 `doGet()`, `doPost()`로 분기한다.
- 서블릿 컨테이너가 서블릿에 종료 요청을 하면 `destory()` 메소드가 호출된다. 종료 시 처리해야 하는 작업은 `destory()` 메소드를 오버라이딩하여 구현하면된다. `destory()` 메소드가 끝난 서블릿 인스턴스는 GC에 의해 제거된다.

### 일반 자바 객체와 차이점

JVM에서 호출 방식은 서블릿과 일반 클래스 모두 같으나, 서블릿은 `main()` 메소드로 직접 호출되지 않고, 웹 컨테이너(Servlet Container)에 의해 실행된다. 컨테이너가 web.xml을 읽고, 서블릿 클래스를 클래스 로더에 등록하는 절차를 밟는다.

## Servlet Container

서블릿 컨테이너는 구현되어 있는 Servlet 클래스의 규칙에 맞게 서블릿 객체를 생성, 초기화, 호출, 종료하는 생명 주기를 관리한다. 서블릿 컨테이너는 클라이언트의 요청을 받고 응답할 수 있도록 웹 서버와 소켓으로 통신한다.

Tomcat은 웹 애플리케이션(WAS) 중 하나로, Servlet Container 기능을 제공하고 있다. 혹자는 Tomcat을 서블릿 컨테이너라고 부르긴 하지만, 엄밀히 말하면 내장 웹 서버 등의 부가 기능도 제공하므로 WAS라고 부르는 것이 좋다고 생각한다. (그래도 인터넷에서 Tomcat은 서블릿 컨테이너라고 많이 부르니, 적당히 이해하자.)

### 특징

**통신 지원**

서블릿과 웹 서버가 통신할 수 있는 손 쉬운 방법을 제공한다. 만약 해당 유저의 이름 값을 FORM을 통해 입력받는다고 가정해 보자. 그러면 아래와 같은 수많은 작업이 필요하다.

![https://blog.kakaocdn.net/dn/dnW7IQ/btq7GAsXvzO/yzsPyK9nUMdNi43yQaoLqk/img.png](https://www.notion.so/image/https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdnW7IQ%2Fbtq7GAsXvzO%2FyzsPyK9nUMdNi43yQaoLqk%2Fimg.png?table=block&id=e1aaf14e-4757-4f26-90e9-e7aeb3e92649&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=1700&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

FORM 인증을 하면 HTTP 메시지가 전송되는데 그것을 읽어 들이기 위해 여러 가지 과정을 거쳐야 하고 응답하기 위해서도 또 번거로운 과정들을 거쳐야 한다.

서블릿 컨테이너는 개발자가 비즈니스 로직에 집중할 수 있도록 해당 과정을 모두 자동으로 해 준다. 우리는 단순히 HTTP 요청 메시지로 생성된 request를 읽어서 비즈니스 로직을 수행하고 response를 반환하면 된다.

**서블릿의 생명 주기를 관리**

서블릿 클래스를 로딩해 인스턴스화하고, 서블릿의 초기화 메소드를 호출하고, 요청이 들어오면 적절한 서블릿 메소드를 호출하는 작업을 서블릿 컨테이너가 한다. 서블릿의 사용이 끝난 시점에는 가비지 컬렉션을 진행해 제거한다.

**멀티 스레딩 관리**

서블릿 컨테이너는 요청이 올 때마다 새로운 자바 스레드를 하나 생성하여 다중 처리하고, 실행이 끝나면 자동으로 종료된다.

**선언적 보안 관리**

보안 관련 설정을 배포 서술자라는 xml 문서를 활용하여 관리하므로 개발자가 보안 설정을 바꾸더라도 자바 코드에 영향이 가지 않는다.

**JSP 지원**

JSP 코드를 자바 코드로 변환해 준다.

### 동작 과정

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F2dffdd5e-e114-47e1-a28e-8a0e130b35d6%2FUntitled.png?table=block&id=ad85d212-4100-449a-ad4e-36830858e0d0&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=1700&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

1. 웹 브라우저에서 웹 서버에 HTTP 요청을 보내면, 웹 서버는 받은 HTTP 요청을 WAS의 Web Server로 전달한다.
2. WAS의 웹 서버는 HTTP 요청을 서블릿 컨테이너에 전달한다.
3. 서블릿 컨테이너는 HTTP 요청 처리에 필요한 서블릿 인스턴스가 힙 메모리 영역에 있는지 확인한다. 존재하지 않는다면, 서블릿 인스턴스를 생성하고 해당 서블릿 인스턴스의 `init()` 메소드를 호출하여 서블릿 인스턴스를 초기화한다.
4. 서블릿 컨테이너는 서블릿 인스턴스의 `service()` 메소드를 호출하여 HTTP 요청을 처리하고, WAS의 웹 서버에게 처리 결과를 전달한다.
5. WAS의 웹 서버는 HTTP 응답을 앞 단에 위치한 웹 서버에게 전달하고, 앞 단의 웹 서버는 받은 HTTP 응답을 웹 브라우저에게 전달한다.

### 멀티 스레딩 부가 설명

위에서 서블릿 컨테이너는 요청이 올 때마다 새로운 자바 스레드를 하나씩 생성한다고 하였다. 여기서 유의할 점은 요청이 올 때마다 해당 서블릿의 스레드를 생성하는 것이지, 서블릿 인스턴스 자체를 새로 생성하는 것이 아니다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F97b574cf-e3a8-4862-bd05-d587e9aba28b%2FUntitled.png?table=block&id=a18f408b-6bd3-4a54-afe1-61012d1cf6d4&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=1700&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

서블릿 인스턴스는 싱글톤으로 생성되며, Thread-Safe하지 않기 때문에 서블릿은 무상태 혹은 읽기 전용 상태, 동기화 처리된 구조로 설계되어야 한다.

그렇다면, 서블릿 컨테이너는 사용자 요청이 올 때마다 무한정 스레드를 생성할까? 그렇지 않다. 내부에 스레드 풀이 있으므로 전체 스레드 수를 조절한다.

## Spring Container

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Ffba1fd42-d95d-463d-a32e-5a6d9dc92074%2FUntitled.png?table=block&id=82c579e8-8b92-4484-b071-00310640a99a&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=1700&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

스프링 컨테이너는 Bean 생명 주기를 관리한다. Bean을 관리하기 위해 IoC가 이용되며 BeanFactor 객체가 바로 IoC 컨테이너(=DI 컨테이너, 스프링 컨테이너)에 해당되며, 이 IoC 컨테이너를 상속하면서 부가 기능을 추가한 것이 ApplicationContext이다.

Spring MVC 역시 서블릿 컨테이너가 관리하고 있는 거대한 서블릿이라고 생각하면 된다. 그래서 서블릿 없이 Spring MVC만 있으면 된다고 하는 것은 비즈니스 로직을 Spring을 통해 처리한다는 것이지, 서블릿이 필요 없다는 뜻이 아니다.

그림을 보면, 스프링 컨테이너는 서블릿 컨테이너 안에 존재하는 것을 확인할 수 있다. 즉, 스프링 컨테이너는 서블릿 컨테이너와 독립적인 존재가 아니며, 서블릿 컨테이너가 Spring Bean에 접근하려면 스프링 컨테이너를 거쳐야 한다.

## Spring MVC

Spring MVC란 프론트 컨트롤러 패턴에 기초한 웹 MVC 프레임워크이다. Spring 프레임워크의 하위 모듈이며, Model, View, Controller를 명확하게 분리하여 매우 유연하고 확장성이 좋다는 특징이 있다.

### Spring MVC가 없던 시절

Spring MVC가 없던 과거에는, URL마다 서블릿을 생성하고 Web.xml로 서블릿을 관리했다. URL마다 서블릿이 필요하다 보니, 매번 서블릿 인스턴스를 만들어야했다. 또한, 각 서블릿마다 공통 기능을 하는 코드들이 중복해서 발생하여 유지 보수 하기 어려웠다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Ff0b81489-53d5-4b3a-b931-f9b0091efde4%2FUntitled.png?table=block&id=101b875c-f5c4-4b2b-9ef1-deaaff28c34e&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=1700&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

### 프론트 컨트롤러 패턴

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F92ef2a5f-5d77-4ed3-9dab-418b67af8c20%2FUntitled.png?table=block&id=446db194-1038-4200-9674-b4dc90a50446&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=1700&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

프론트 컨트롤러 패턴은 모든 요청을 프론트 컨트롤러(하나의 서블릿)에게 보내고, 프론트 컨트롤러는 각 요청에 맞는 컨트롤러를 찾아서 호출하는 역할을 한다. 그래서 공통 기능은 프론트 컨트롤러에서 처리하고, 서로 다른 코드들만 각 컨트롤러에서 처리하도록 할 수 있다.

### DispatcherServlet

DispatcherServlet은 표현 계층 전면에서 HTTP 프로토콜을 통해 들어오는 모든 요청을 중앙 집중식으로 처리하는 프론트 컨트롤러이다. DispatcherServlet은 Spring MVC의 핵심 요소 중 하나로, 클라이언트로부터 어떤 요청이 들어오면 서블릿 컨테이너가 요청을 받는다. 이후 공통 작업을 DipatcherServlet에 처리하고, 이외 작업은 적절한 세부 컨트롤러로 위임한다.

### Spring MVC의 동작 흐름

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fb2cee0d0-d963-4772-b113-bc52ee10627a%2FUntitled.png?table=block&id=25b7bc0c-0b8f-41b6-bfb3-14e4fdf4cb00&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=1700&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

1. DispatcherServlet으로 클라이언트의 웹 요청이 들어온다.
2. 웹 요청을 Handler Mapping에 위임하여 해당 요청을 처리할 Handler(Controller)를 탐색한다.
3. 찾은 Handler를 실행할 수 있는 HandlerAdapter를 탐색한다.
4. 찾은 Handler Adapter를 사용해서 Handler의 메소드를 실행한다.
5. Handler의 반환 값은 Model과 View이다.
6. View 이름을 ViewResolver에게 전달하고, ViewResolver는 해당하는 View 객체를 전달한다.
7. DispatcherServlet은 View에게 Model을 전달하고 화면 표시를 요청한다. 이때, Model이 null이면 View를 그대로 사용하고, 그렇지 않으면 View에 Model 데이터를 렌더링한다.
8. 최종적으로 DispatcherServlet은 View 결과(HttpServletResponse)를 클라이언트에게 반환한다.

위 흐름은 @Controller 기준이며, @RestController의 경우 6번과 7번 과정이 생략된다. 즉, ViewResolver를 타지 않고 반환 값에 알맞는 MessageConverter를 찾아 응답 본문을 작성한다.

## Controller 1개는 어떻게 수십 만 개의 요청을 처리하는가?

톰캣은 기본 값으로 200개의 작업 스레드가 존재한다. 톰캣은 하나의 프로세스에서 동작하고, 내부적으로 스레드 풀을 만들어서 HTTP 요청이 오면 스레드 풀에서 스레드 하나를 가져온다. 따라서 여러 요청이 오면, 각각 스레드를 생성하여 하나의 컨트롤러에 요청을 한다. 그러면, 컨트롤러는 요청에 맞게 로직을 수행하여 적절한 데이터를 반환한다.

이때 Controller 1개가 어떻게 수십 만 개의 요청을 처리하는지 궁금할 수가 있다. 동기화를 배운 사람이라면, Controller 객체에 lock을 거는 방식으로 동시성 이슈를 막아야한다고 생각할 수 있다. 하지만, 일반적으로 Controller는 싱글톤 빈으로 등록되고 무상태, 읽기 전용 상태, 혹은 동기화 처리된 상태로 설계가 된다. 그래서 여러 스레드가 동시에 하나의 Controller에 접근해도 Thread-Safe하므로 아무 상관이 없다.

물론 Controller가 프로토타입 빈이라면, 매번 새로운 인스턴스가 생성되므로, 이 때는 요청 당 하나의 Controller 객체가 대응한다.

## 출처

- [https://tecoble.techcourse.co.kr/post/2021-05-23-servlet-servletcontainer/](https://tecoble.techcourse.co.kr/post/2021-05-23-servlet-servletcontainer/)
- [https://medium.com/jiwon-bae/web-servlet-servlet-container-b6c3a4c6549f](https://medium.com/jiwon-bae/web-servlet-servlet-container-b6c3a4c6549f)
- [https://jojoldu.tistory.com/28](https://jojoldu.tistory.com/28)
- [https://devmoony.tistory.com/102](https://devmoony.tistory.com/102)
- [https://tecoble.techcourse.co.kr/post/2021-06-25-dispatcherservlet-part-1/](https://tecoble.techcourse.co.kr/post/2021-06-25-dispatcherservlet-part-1/)

## 예상 면접 질문 및 답변

### Servlet이란?

서블릿은 클라이언트 요청을 처리하고, 그 결과를 다시 클라이언트에게 전송하는 Servlet 클래스의 구현 규칙을 지킨 자바프로그램이다. 서블릿을 사용하게 되면 웹 페이지를 동적으로 생성하여 클라이언트에게 반환해 줄 수 있다.

### Servlet의 동작 방식을 설명하라.

- 사용자가 URL을 입력하면 요청이 서블릿 컨테이너로 전송된다.
- 요청을 전송 받은 서블릿 컨테이너는 Http Request, HttpResponse 객체를 생성한다.
- 사용자가 요청한 URL이 어느 서블릿에 대한 요청인지 찾는다. 위 예제에서는 helloServlet을 찾게 된다.
- 서블릿의 `service()` 메소드를 호출한 후 클라이언트의 GET, POST 여부에 따라 `doGet()`, `doPost()` 메소드를 호출한다.
- 동적 페이지를 생성한 후 HttpServletResponse 객체에 응답을 보낸다.
- 클라이언트에 최종 결과를 응답한 후 HttpServletRequest, HttpServletResponse 객체를 소멸한다.

### Servlet의 생명 주기를 설명하라.

- 클라이언트 요청이 들어오면 서블릿 컨테이너는 서블릿이 메모리에 있는지 확인한다. 메모리에 없다면 `init()` 메소드를 호출하여 적재한다.
- 클라이언트 요청에 따라서 `service()` 메소드를 통해 요청에 대한 응답이 `doGet()`, `doPost()`로 분기한다.
- 서블릿 컨테이너가 서블릿에 종료 요청을 하면 `destory()` 메소드가 호출된다. 종료 시 처리해야 하는 작업은 `destory()` 메소드를 오버라이딩하여 구현하면된다. `destory()` 메소드가 끝난 서블릿 인스턴스는 GC에 의해 제거된다.

### Servlet과 일반 자바 객체는 무슨 차이가 있는가?

JVM에서 호출 방식은 서블릿과 일반 클래스 모두 같으나, 서블릿은 `main()` 메소드로 직접 호출되지 않고, 웹 컨테이너(Servlet Container)에 의해 실행된다. 

### Servlet Container란?

서블릿 컨테이너는 구현되어 있는 Servlet 클래스의 규칙에 맞게 서블릿 객체를 생성, 초기화, 호출, 종료하는 생명 주기를 관리한다.

### Servlet Container의 특징을 설명하라.

- 개발자가 비즈니스 로직에 집중할 수 있도록 HTTP 요청 메시지 파싱, Content-Type 확인, HTTP 응답 메시지 생성 등 작업을 대신 처리 해준다.
- 서블릿의 생명 주기를 관리한다.
- 요청이 올 때마다 자바 스레드 하나를 생성하여 멀티 스레딩 처리를 한다.

### Servlet Container의 동작 과정을 설명하라.

1. 웹 브라우저에서 웹 서버에 HTTP 요청을 보내면, 웹 서버는 받은 HTTP 요청을 WAS의 Web Server로 전달한다.
2. WAS의 웹 서버는 HTTP 요청을 서블릿 컨테이너에 전달한다.
3. 서블릿 컨테이너는 HTTP 요청 처리에 필요한 서블릿 인스턴스가 힙 메모리 영역에 있는지 확인한다. 존재하지 않는다면, 서블릿 인스턴스를 생성하고 해당 서블릿 인스턴스의 `init()` 메소드를 호출하여 서블릿 인스턴스를 초기화한다.
4. 서블릿 컨테이너는 서블릿 인스턴스의 `service()` 메소드를 호출하여 HTTP 요청을 처리하고, WAS의 웹 서버에게 처리 결과를 전달한다.
5. WAS의 웹 서버는 HTTP 응답을 앞 단에 위치한 웹 서버에게 전달하고, 앞 단의 웹 서버는 받은 HTTP 응답을 웹 브라우저에게 전달한다.

### Spring MVC란?

Spring MVC란 프론트 컨트롤러 패턴에 기초한 웹 MVC 프레임워크이다. Spring 프레임워크의 하위 모듈이며, Model, View, Controller를 명확하게 분리하여 매우 유연하고 확장성이 좋다는 특징이 있다.

### 프론트 컨트롤러 패턴이란?

프론트 컨트롤러 패턴은 모든 요청을 프론트 컨트롤러(하나의 서블릿)에게 보내고, 프론트 컨트롤러는 각 요청에 맞는 컨트롤러를 찾아서 호출하는 역할을 한다. 그래서 공통 기능은 프론트 컨트롤러에서 처리하고, 서로 다른 코드들만 각 컨트롤러에서 처리하도록 할 수 있다.

### DispatcherServlet이란?

DispatcherServlet은 표현 계층 전면에서 HTTP 프로토콜을 통해 들어오는 모든 요청을 중앙 집중식으로 처리하는 프론트 컨트롤러이다. DispatcherServlet은 Spring MVC의 핵심 요소 중 하나로, 클라이언트로부터 어떤 요청이 들어오면 서블릿 컨테이너가 요청을 받는다. 이후 공통 작업을 DipatcherServlet에 처리하고, 이외 작업은 적절한 세부 컨트롤러로 위임한다.

### Spring MVC의 흐름을 DispatcherServlet 위주로 설명하라.

1. DispatcherServlet으로 클라이언트의 웹 요청이 들어온다.
2. 웹 요청을 Handler Mapping에 위임하여 해당 요청을 처리할 Handler(Controller)를 탐색한다.
3. 찾은 Handler를 실행할 수 있는 HandlerAdapter를 탐색한다.
4. 찾은 Handler Adapter를 사용해서 Handler의 메소드를 실행한다.
5. Handler의 반환 값은 Model과 View이다.
6. View 이름을 ViewResolver에게 전달하고, ViewResolver는 해당하는 View 객체를 전달한다.
7. DispatcherServlet은 View에게 Model을 전달하고 화면 표시를 요청한다. 이때, Model이 null이면 View를 그대로 사용하고, 그렇지 않으면 View에 Model 데이터를 렌더링한다.
8. 최종적으로 DispatcherServlet은 View 결과(HttpServletResponse)를 클라이언트에게 반환한다.

### Controller 1개는 어떻게 수십 만 개의 요청을 처리하는가?

톰캣은 하나의 프로세스에서 동작하고, 내부적으로 스레드 풀을 만들어서 HTTP 요청이 오면 스레드 풀에서 스레드 하나를 가져온다. 따라서 여러 요청이 오면, 각각 스레드를 생성하여 하나의 컨트롤러에 요청을 한다. 그러면, 컨트롤러는 요청에 맞게 로직을 수행하여 적절한 데이터를 반환한다. 이때 컨트롤러가 Thread-Safe하게 설계되었고, 싱글톤 빈으로 생성되었다면 안전하게 1개의 Controller 객체만으로 다중 요청을 처리할 수 있다.
