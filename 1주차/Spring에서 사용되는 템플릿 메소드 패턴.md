
## 템플릿 메소드 패턴

GOF의 디자인 패턴에 의하면, 템플릿 메소드 패턴을 아래와 같이 정의하고 있다.

```
알고리즘의 구조를 메소드에 정의하고, 
하위 클래스에서 알고리즘 구조의 변경없이 알고리즘을 재정의 하는 패턴이다. 
알고리즘이 단계별로 나누어 지거나, 
같은 역할을 하는 메소드이지만 여러 곳에서 다른 형태로 사용이 필요한 경우 유용한 패턴이다.
```

토비의 스프링에 의하면, 템플릿 메소드 패턴을 아래와 같이 정의하고 있다.

```
상속을 통해 슈퍼클래스의 기능을 확장할 때 사용하는 가장 대표적인 방법. 
변하지 않는 기능은 슈퍼클래스에 만들어두고 자주 변경되며 확장할 기능은 
서브클래스에서 만들도록 한다.
```

두 정의 모두 하위 클래스에서 사용되지만, 변하지 않는 기능을 상위 클래스에 저장해 놓고 확장할 기능은 서브 클래스에서 만들도록 설계한다는 내용을 담고 있다.

### 템플릿 메소드 패턴의 구조

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fbd8ab036-c988-41a5-9753-3ecb91d2ee3f%2FUntitled.png?table=block&id=475d0f68-efa0-4684-8694-1157f70ea066&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

AbstractClass는 템플릿 메소드를 정의하며, 하위 클래스에서 알맞게 확장할 수 있는 훅 메소드를 제공한다. 여기서, 템플릿 메소드는 일반 메소드와 훅 메소드를 이용한다. 그리고 ConcreateClass는 물려 받은 훅 메소드를 재정의하는 역할을 한다.

위 그림에서 템플릿 메소드는 훅 메소드만 사용하였지만, 실제로는 일반 메소드를 혼용하여 사용해도 무방하다.

### 템플릿 메소드 패턴의 예제

두 개의 라면을 끓이는 과정이 다음과 같다고 가정하자.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F55ae3913-1da7-4562-8938-b7f746b675c1%2FUntitled.png?table=block&id=e033a553-2e9d-4ef0-9966-2a0535c2a8d3&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

이를 생각나는 대로 코딩하면 아래처럼 작성할 수 있다.

```java
public class ShinRamen {
    public void boilWater() {
        System.out.println("물을 끓인다.");
    }

    public void putNoodles() {
        System.out.println("면을 넣는다.");
    }

    public void putEgg() {
        System.out.println("계란을 넣는다.");
    }

    public void waitForMinutes() {
        System.out.println("4분 기다린다.");
    }
}

public class RaccoonRamen {
    public void boilWater() {
        System.out.println("물을 끓인다.");
    }

    public void putNoodles() {
        System.out.println("면을 넣는다.");
    }

    public void putBeanSprouts() {
        System.out.println("콩나물을 넣는다.");
    }

    public void waitForMinutes() {
        System.out.println("5분 기다린다.");
    }
}
```

보면 알겠지만, 물을 끓이는 것과 면을 넣는 행위는 완전히 동일한 코드이다. 따라서, Ramen이라는 클래스를 정의하여 중복된 코드를 제거할 수 있다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fc6b113cb-ef39-421d-b433-56dd63fabad2%2FUntitled.png?table=block&id=a5dac5a1-8985-4b03-b88b-8ed2076b8cd0&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

단순히 동일한 메소드를 상위 클래스로 빼 냈지만, 여전히 '무언가'를 넣는 행위와 'x분' 기다린다는 행위가 유사하다. 템플릿 메소드 패턴은 이것도 추상화하는 것이 목표라고 할 수 있다.

```java
public abstract class Ramen {

    public void makeRamen() {
        boilWater();
        putNoodles();
        putExtra();
        waitForMinutes();
    }

    public void boilWater() {
        System.out.println("물을 끓인다.");
    }

    public void putNoodles() {
        System.out.println("면을 넣는다.");
    }

    public abstract void putExtra();

    public abstract void waitForMinutes();
}
```

Ramen 클래스를 추상 클래스로 선언한 다음, 추상화 할 메소드 2개를 추상 메소드로 선언한다. 여기서 `makeRamen()`은 템플릿 메소드이고, `putExtra()`와 `waitForMinutes()`는 훅 메소드라고 볼 수 있다.

```java
public class ShinRamen extends Ramen {

    @Override
    public void putExtra() {
        System.out.println("계란을 넣는다.");
    }

    @Override
    public void waitForMinutes() {
        System.out.println("4분 기다린다.");
    }
}

public class RaccoonRamen extends Ramen {

    @Override
    public void putExtra() {
        System.out.println("콩나물을 넣는다.");
    }

    @Override
    public void waitForMinutes() {
        System.out.println("5분 기다린다.");
    }
}
```

이후 위와 같이 하위 클래스는 각자의 행위를 재정의하면 된다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fc91a7b28-c974-4c46-96a4-39b3102b2bca%2FUntitled.png?table=block&id=cbe92c84-b82b-4aff-890a-196b272e8009&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

따라서 결과적으로 위와 같은 구조를 Ramen 클래스가 갖게 되고, `ramen.makeRamen()` 을 호출하면 신라면이든 너구리든 라면을 끓이는 행위를 다형성 있게 구현할 수 있다. 이렇듯 템플릿 메소드 패턴은 같은 역할을 하는 메소드이지만 여러 곳에서 다른 형태로 사용이 필요한 경우 효과적으로 적용할 수 있다.

## Spring에서 템플릿 메소드 패턴을 적용한 사례

Spring에서 템플릿 메소드 패턴은 DispatcherServlet에서 사용되고 있다. Spring의 DispatcherServlet은 Http 요청에 대해서 처리하는데, DispatcherServlet은 Servlet을 상속 받은 것으로 구현되어 있다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F585f20e7-2761-4d5f-9cb6-2187c88298a5%2FUntitled.png?table=block&id=23e0bdec-dcde-4303-bf1d-d9a30c224fe7&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

정확히 DispatcherServlet은 FrameworkServlet을 상속받은 것을 알 수 있다.

한편 DispatcherServlet의 `doService()` 라는 메소드가 있는데, 이 메소드는 실질적으로 Http 요청에 대해서 처리하는 역할을 한다. 그리고 이 메소드가 바로 템플릿 메소드 패턴으로 구현되어 있다.

```java
public abstract class FrameworkServlet extends HttpServletBean implements ApplicationContextAware {
    ...중략...
    protected final void processRequest(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        ...중략...

        initContextHolders(request, localeContext, requestAttributes);

        try {
            doService(request, response); // template method pattern 이용
        }
        ...중략...
    }
    ...중략...

    protected abstract void doService(HttpServletRequest request, HttpServletResponse response) throws Exception; // subClass에게 위임

    ...중략...
}

public class DispatcherServlet extends FrameworkServlet {

    ...

    @Override
    protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
        logRequest(request);

        ...중략...

    }

    ...
}
```

FrameworkServlet에서 `doService()` 에 대한 구현은 하위 클래스에게 위임하고 있고, DispatcherServlet은 `doService()` 를 구체화하고 있다. 즉, 우리가 DispatcherServlet을 구현하여 `processRequest()` 를 호출하면 frameworkServlet의 `processRequest()` 의 로직을 수행하다가 `doService(request, response)` 를 만나면 DispatcherServlet의 로직을 수행하는 것이다.

## 출처

- [https://steady-coding.tistory.com/384#](https://steady-coding.tistory.com/384#)
- [https://sabarada.tistory.com/19](https://sabarada.tistory.com/19)

## 예상 면접 질문 및 답변

### 템플릿 메소드 패턴이란?

알고리즘의 구조를 메소드에 정의하고, 하위 클래스에서 알고리즘 구조의 변경 없이 알고리즘을 재정의 하는 패턴이다.

### 스프링에서 템플릿 메소드 패턴은 어디서 사용되는가?

Spring에서 템플릿 메소드 패턴은 FrameworkServlet의 상속을 받은 DispatcherServlet의 `doService()` 에서 사용되고 있다.

FrameworkServlet은 `processRequest()` 라는 템플릿 메소드가 정의되어 있는데, 이 메소드 안에 `doService()` 가 추상 메소드로 구현되어 있다. 즉, `processRequest()` 를 호출하면 일반적인 로직을 수행하다가 `doService()` 를 만나면 DispatcherServlet의 `doService()` 를 호출하는 것이다.
