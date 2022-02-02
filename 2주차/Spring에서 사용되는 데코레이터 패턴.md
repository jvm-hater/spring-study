## 데코레이터 패턴이란?

데코레이터 패턴(Decorator Pattenr)은 주어진 상황 및 용도에 따라 **어떤 객체에 책임(기능)을 동적으로 추가하는 패턴**을 말한다. 데코레이터라는 말 그대로 장식이라고 생각하시면 편하다. 기본 기능을 가지고 있는 클래스를 하나 만들어주고 추가할 수 있는 기능들을 추가하기 편하도록 설계하는 방식이다.

![image](https://user-images.githubusercontent.com/55661631/152149530-5826e066-c828-42d1-b56f-ad1baf8a4aff.png)

- **Component**
    - ConcreteComponent와 Decorator가 구현할 인터페이스다. 두 객체를 동등하게 다루기 위해존재한다.
- **ConcreteComponent**
    - ConcreteDecorator에게 Decorate를 받을 객체다. 기능 추가를 받을 **기본 객체**이다.
- **Decorator**
    - Decorate를 할 객체의 추상 클래스다. ConcreteComponent를 Decorate 할 객체는 이 객체를 상속받는다.
- **ConcreteDecorator**
    - Decorator를 상속받는 객체이다. ConcreteComponent를 Decorate  해주는 역할을 한다.

## 데코레이터 패턴을 사용하는 이유

크리스마스 트리에 원하는 장식품을 추가할 수 있는 프로그램을 만든다고 가정하겠다.

**Bad Case 1**

```java
abstract class ChristmasTree{
  
	getDecorations();
}

class ChristmasTreeWithLights extends ChristmasTree {
	...
	getDecorations()
}

class ChristmasTreeWithFlowers extends ChristmasTree{
	...
	getDecorations()
}
```

크리스마스 트리를 정의하는 ChristmasTree 추상 클래스를 생성하였다. 모든 크리스마스 트리는 이 추상 클래스를 상속받아 `decorate()` 메소드를 구체화한다.

그러나 이 방식은 장식품이 추가될 때마다 클래스 생성하는 단점이 있다. 만약, 크리스마스 트리에 공을 추가하고 싶으면 `ChristmasTreeWithBalls` 클래스를 생성해야 한다.

**Bad Case 2**

```java
abstract class ChristmasTree{

	light
	flower
	ball
    
	getDecorations()
	setLight()
	setFlower()
	setBall() 
}

class defaultChristmasTree extends ChristmasTree{
	...
	getDecorations()
}
```

`ChristmasTree` 추상 클래스를 위와 같이 정의하면 더 이상 `~WithLights`와 같은 클래스를 생성하지 않아도 된다. 그러나 여전히 문제는 남아있다.

1. 장식품 종류가 많아지면 새로운 메소드를 추가해야 한다.
2. 모든 서브 클래스가 필요없는 메소드도 상속받는다.
3. ‘전구 두 개 추가'와 같은 기능을 사용할 수 없다.

## 데코레이션 패턴 구현

위와 같이 크리스마스 트리에 원하는 장식품을 추가할 수 있는 프로그램을 만든다고 가정하겠다.

**Component**

크리스마스 트리와 장식을 추상화 해보자.

```java
public interface ChristmasTree {

    String getDecorations();
}
```

**ConcreteComponent**

기본 크리스마스 트리를 다음과 같이 구체화할 수 있다.

```java
public class DefaultChristmasTree implements ChristmasTree {

    @Override
    public String getDecorations() {
        return "크리스마스 트리 ";
    }
}
```

**Decorator**

크리스마스 트리에 올라갈 장식은 다음과 같이 추상화할 수 있다.

```java
abstract public class TreeDecorator implements ChristmasTree {

    private ChristmasTree christmasTree;

    public TreeDecorator(ChristmasTree christmasTree) {
        this.christmasTree = christmasTree;
    }

    @Override
    public String getDecorations() {
        return christmasTree.getDecorations();
    }
}
```

**ConcreteDecorator**

크리스마스 트리에 올릴 전구를 구체화 해보자.

```java
public class Lights extends TreeDecorator {

    public Lights(ChristmasTree christmasTree) {
        super(christmasTree);
    }

    @Override
    public String getDecorations() {
        return super.getDecorations() + "전구 ";
    }
}
```

마찬가지로 장식용 꽃도 구체화 하자.

```java
public class Flowers extends TreeDecorator {

    public Flowers(ChristmasTree christmasTree) {
        super(christmasTree);
    }

    @Override
    public String getDecorations() {
        return super.getDecorations() + "꽃 ";
    }
}
```

**Client**

이제 클라이언트 단에서 크리스마스 트리를 만들어보자.

```java
public class Main {

    public static void main(String[] args) {
				//기본 트리
        ChristmasTree tree = new DefaultChristmasTree();
        System.out.println(tree.getDecorations());

				//기본 트리 + 전구
        ChristmasTree treeWithLights = new Lights(new DefaultChristmasTree());
        System.out.println(treeWithLights.getDecorations());
				
				//기본 트리 + 전구 + 꽃
        ChristmasTree treeWithLightsAndFlowers = new Flowers(new Lights(new DefaultChristmasTree()));
        System.out.println(treeWithLightsAndFlowers.getDecorations());
    }
}
```

실행하면 다음과 같은 결과를 볼 수 있다.

```java
크리스마스 트리 
크리스마스 트리 전구 
크리스마스 트리 전구 꽃
```

클라이언트 코드 부분을 살펴보면, 기본 객체인 크리스마스 트리에 장식 추가를 `new Lights(new DefaultChristmasTree())`와 같이 동적인 방식으로 하고 있다.

이게 가능한 이유는

- **Decorator 객체의 생성자로 Component를 받음으로써 Decorator를 이어 붙일 수 있고,**
- **super를 통해 넘어오는 Component 의 operation(`decorate()`) 을 먼저 수행하기 때문이다.**

이후 추가적인 장식(기능)을 만들고 싶으면 `TreeDecorator`를 상속받아 위와 같은 방식으로 구현하면 된다. 즉 새로운 기능을 유연하게 만들고 추가할 수 있다.

## 데코레이터 패턴의 장단점

**장점**

- 기존 코드를 수정하지 않고도 데코레이터 패턴을 통해 행동을 확장시킬 수 있다.
- 구성과 위임을 통해서 실행 중에 새로운 행동을 추가할 수 있다.

**단점**

- 의미없는 객체들이 너무 많이 추가될 수 있다.
- 데코레이터를 너무 많이 사용하면 코드가 필요 이상으로 복잡해질 수 있다.

## 참고

- [https://coding-factory.tistory.com/713](https://coding-factory.tistory.com/713)
- [https://velog.io/@hanna2100/디자인패턴-3.-데코레이터-패턴-개념과-예제-decorator-pattern](https://velog.io/@hanna2100/%EB%94%94%EC%9E%90%EC%9D%B8%ED%8C%A8%ED%84%B4-3.-%EB%8D%B0%EC%BD%94%EB%A0%88%EC%9D%B4%ED%84%B0-%ED%8C%A8%ED%84%B4-%EA%B0%9C%EB%85%90%EA%B3%BC-%EC%98%88%EC%A0%9C-decorator-pattern)
- [https://dailyheumsi.tistory.com/198](https://dailyheumsi.tistory.com/198)
- [https://www.baeldung.com/java-decorator-pattern](https://www.baeldung.com/java-decorator-pattern)

## 예상 면접 질문 및 답변

### Q. 데코레이터 패턴이란?

데코레이터 패턴은 주어진 상황 및 용도에 따라 어떤 객체에 **책임(기능)을 동적으로 추가하는 패턴**을 말한다. 쉽게 말하자면, 기본 기능을 가지고 있는 클래스를 하나 만들어주고 추가할 수 있는 기능들을 추가하기 편하도록 설계하는 방식이다.

### Q. 데코레이터 패턴을 코드로 작성하시오.

본문 참고

### Q. 데코레이터 패턴의 장점은?

- 기존 코드를 수정하지 않고도 데코레이터 패턴을 통해 행동을 확장시킬 수 있다.
- 구성과 위임을 통해서 실행 중에 새로운 행동을 추가할 수 있다.

### Q. 데코레이터 패턴의 단점은?

- 의미없는 객체들이 너무 많이 추가될 수 있다.
- 데코레이터를 너무 많이 사용하면 코드가 필요 이상으로 복잡해질 수 있다.
