# loC(Inversion of Control) - 제어의 역전

## loC란?

- IoC란 **Inversion of Control**의 줄임말이며, 해석하면 **제어의 역전**이라고 한다.
- 스프링 애플리케이션에서는 오브젝트(빈)의 생성과 관계설정, 사용, 제거 등의 작업을 애플리케이션 코드 대신 스프링 컨테이너가 담당한다.
- 이를 스프링 컨테이너가 코드 대신 오브젝트에 대한 제어권을 갖고 있다고 해서 IoC라고 부른다.
- 따라서, **스프링 컨테이너**를 **IoC 컨테이너**라고도 부른다.

## IoC 컨테이너란?

- 스프링에선 Ioc를 담당하는 컨테이너(loC 컨테이너)를 빈 팩토리,  DI 컨테이너, 애플리케이션 컨텍스트라고 부르기도 한다.
- 오브젝트의 생성과 오브젝트 사이의 런타임 관계를 설정하는 DI 관점으로 보면 컨테이너를 빈 팩토리 또는 DI 컨테이너라고 부른다.
- 그러나 스프링 컨테이너는 단순한 DI 작업보다 더 많은 일을 하는데, DI를 위한 빈 팩토리에 여러 가지 컨테이너 기능을 추가한 것을 애플리케이션 컨텍스트라고 부른다.
- 정리하자면, 애플리케이션 컨텍스트는 그 자체로 IoC와 DI 그 이상의 기능을 가졌다고 보면 된다.

## 빈 팩토리와 애플리케이션 컨텍스트

빈 팩토리와 애플리케이션 컨텍스트 관계를 간단히 살펴보면 아래와 같다.

![image](https://user-images.githubusercontent.com/55661631/152982520-a34e8ad1-1d4c-4701-b734-c633e4506215.png)

**빈 팩토리**

- 스프링 컨테이너의 최상위 인터페이스다.
- 스프링 빈을 관리하고 조회하는 역할을 담당한다.
- 대표적으로 `getBean()` 메소드를 제공한다.

**애플리케이션 컨텍스트**

```java
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, 
											HierarchicalBeanFactory, MessageSource, ApplicationEventPublisher, 
											ResourcePatternResolver {
```

- 애플리케이션 컨텍스트는 빈 팩토리 기능을 모두 상속받아서 제공한다.
    - 위의 인터페이스 정의를 보면, `ListableBeanFactory`, `HierarchicalBeanFactory` 인터페이스를 상속하고 있는데, 이 두 개의 이넡페이스 모두 `BeanFactory` 인터페이스의 서브 인터페이스다.
- 위 인터페이스 정의를 보면 `BeanFactory` 이외의 다른 인터페이스들도 상속하고 있는 것을 볼 수 있다. 따라서, 애플리케이션 컨텍스트는 다음과 같은 기능들도 제공한다.
    - **메시지 소스를 활용한 국제화 기능(MessageSource)**
        - 한국에서 들어오면 한국어로, 영어권에서 들어오면 영어로 출력
    - **환경변수(EnvironmentCapable)**
        - 로컬, 개발, 운영등을 구분해서 처리
    - **애플리케이션 이벤트(ApplicationEventPublisher)**
        - 이벤트를 발행하고 구독하는 모델을 편리하게 지원
    - **편리한 리소스 조회(ResourcePatternResolver)**
        - 파일, 클래스패스, 외부 등에서 리소스를 편리하게 조회

## 설정 메타 정보

IoC 컨테이너의 가장 기초적인 역할은 오브젝트를 생성하고 이를 관리하는 것이다. **스프링 컨테이너가 관리하는 이런 오브젝트는 빈이라고 부른다.**  설정 메타정보는 바로 이 빈을 어떻게 만들고 어떻게 동작하게 할 것인가에 관한 정보다.

스프링 컨테이너는 자바 코드, XML, Groovy 등 다양한 형식의 설정 정보를 받아드릴 수 있게 유연하게 설계되어 있다.

![image](https://user-images.githubusercontent.com/55661631/152982543-8d90eee9-e6a3-4081-a4d8-0b7c0c8b186a.png)

**애노테이션 기반 자바 코드 설정**

```java
@Configuration
public class AppConfig {

		@Bean
		public MemberService memberService() {
				return new MemberServiceImpl(memberRepository());
		}
}
```

- @Configuration : 1개 이상의 빈을 제공하는 클래스의 경우 반드시 명시해야 한다.
- @Bean : 클래스를 빈으로 등록할 때 사용한다.

**XML 기반의 스프링 빈 설정**

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
			 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
			 xsi:schemaLocation="http://www.springframework.org/schema/beans http://
www.springframework.org/schema/beans/spring-beans.xsd">

	 <bean id="memberService" class="hello.core.member.MemberServiceImpl">
			 <constructor-arg name="memberRepository" ref="memberRepository"/>
	 </bean>
</beans>
```

- XML 기반의 설정 파일을 보면 자바 코드로 된 설정 파일과 거의 비슷하다는 것을 알 수 있다.
- XML 기반으로 설정하는 것은 최근에 잘 사용하지 않는다.

## **스프링 빈 설정 메타 정보 - BeanDefinition**

- 스프링은 어떻게 이런 다양한 형식을 지원하는 것일까? 그 중심에는 `BeanDefinition`이라는 추상화가 있다.
- 쉽게 말하자면, XML을 읽어서 BeanDefinition을 만들고, 자바 코드를 읽어서 BeanDefinition을 만든다. 따라서 스프링 컨테이너는 자바 코드인지, XML인지 몰라도 되고 오직 BeanDefinition만 알면 된다.
- BeanDefinition을 빈 설정 메타 정보라 하는데, `@Bean`과 `<bean>` 당 각각 하나씩 메타 정보가 생성된다.

코드 레벨로 조금 더 깊이 있게 들어가보자.

![image](https://user-images.githubusercontent.com/55661631/152982595-dc5d3d39-c515-4a84-9caa-96e77fc6d878.png)

- AnnotationConfigApplicationContext는 AnnotatedBeanDefinitionReader를 사용해서
`AppConfig.class`를 읽고 BeanDefinition을 생성한다.
- GenericXmlApplicationContext는 XmlBeanDefinitionReader를 사용해서 `appConfig.xml` 설정
정보를 읽고 BeanDefinition을 생성한다.
- 새로운 형식의 설정 정보가 추가되면, XxxBeanDefinitionReader를 만들어서 BeanDefinition을
생성하면 된다.

# ****DI(Dependency Injection) - 의존관계 주입****

## Dependency(의존관계)란?

“A가 B를 의존한다”는 굉장히 추상적인 표현이지만, 토비의 스프링에서는 다음과 같이 정의한다.

> *의존 대상 B가 변하면, 그것이 A에 영향을 미친다.*
*- 이일민, 토비의 스프링 3.1, 에이콘(2012), p113*
> 

즉, B의 기능이 추가 또는 변경되거나 형식이 바뀌면 그 영향이 A에 미친다.

간단한 예시와 함께 설명을 계속하겠다.

```java
class BurgerChef {
    private HamBurgerRecipe hamBurgerRecipe;

    public BurgerChef() {
        hamBurgerRecipe = new HamBurgerRecipe();        
    }
}
```

위 코드의 경우, 햄버거 레시피가 변화하게 되었을 때, 변화된 레시피에 따라서 `BurgerChef` 클래스를 수정해야 한다. 레시피의 변화가 요리사의 행위에 영향을 미쳤기 때문에. **“요리사는 레시피에 의존한다”**고 말할 수 있다.

## Dependency를 인터페이스로 추상화

위 예시를 보면, `BurgerChef`는 `HamburgerRecipe`만 의존할 수 있는 구조로 되어있다. 더 다양한 햄버거 레시피를 의존할 수 있게 구현하려면 **인터페이스로 추상화**해야 한다.

```java
class BurgerChef {
    private BurgerRecipe burgerRecipe;

    public BurgerChef() {
        burgerRecipe = new HamBurgerRecipe();
        //burgerRecipe = new CheeseBurgerRecipe();
        //burgerRecipe = new ChickenBurgerRecipe();
    }
}

interface BugerRecipe {
    newBurger();
} 

class HamBurgerRecipe implements BurgerRecipe {
    public Burger newBurger() {
        return new HamBerger();
    }
}
```

위 코드에서 볼 수 있듯이, 다양한 버거 레시피에 의존할 수 있는 `BurgerChef`가 되었다. **이처럼 의존관계를 인터페이스로 추상화하게 되면, 더 다양한 의존관계를 맺을 수가 있고, 실제 구현 클래스와의 관계가 느슨해지고, 결합도가 낮아진다.**

## DI(**의존관계** 주입)란?

의존관계가 무엇인지에 대해, 그리고 다양한 의존관계를 위해 인터페이스로 추상화함을 알아봤다. 그럼 DI은 무엇일까?

지금까지의 구현에서는 `BurgerChef` 내부적으로 의존관계인 `BurgerRecipe`가 어떤 값을 가질지 직접 정하고 있다. 그러나 DI는 어떤 햄버거 레시피를 만들지를 버거 가게 사장님이 정하는 상황이라고 할 수 있다. 즉, `BurgerChef`가 의존하고 있는 `BurgerRecipe`를 외부(사장님)에서 결정하고 주입하는 것이다.

```java
class BurgerChef {
    private BurgerRecipe burgerRecipe;

    public BurgerChef(BurgerRecipe bugerRecipe) {
        this.burgerRecipe = bugerRecipe;
    }
}

//의존관계를 외부에서 주입 -> DI
new BurgerChef(new HamBurgerRecipe());
new BurgerChef(new CheeseBurgerRecipe());
new BurgerChef(new ChickenBurgerRecipe());
```

이처럼 그 의존관계를 외부(스프링 컨테이너)에서 결정하고 주입하는 것이 **DI(의존관계 주입)**이다.

![image](https://user-images.githubusercontent.com/55661631/152982636-1129bd12-60cf-49ef-a2c4-c1f7339b33d8.png)

토비의 스프링에서는 다음의 세 가지 조건을 충족하는 작업을 의존관계 주입이라 말한다.

> *1. 클래스 모델이나 코드에는 런타임 시점의 의존관계가 드러나지 않는다. 그러기 위해서는 인터페이스만 의존하고 있어야 한다.

2. 런타임 시점의 의존관계는 컨테이너나 팩토리 같은 제3의 존재가 결정한다.

3. 의존관계는 사용할 오브젝트에 대한 레퍼런스를 외부에서 제공(주입)해줌으로써 만들어진다.

- 이일민, 토비의 스프링 3.1, 에이콘(2012), p114*
> 

## DI(**의존관계** 주입) 구현 방법

스프링에서 의존성을 주입하는 방법은 아래와 같이 세 가지가 있다.

- 필드 주입
- 수정자 주입
- 생성자 주입

### 필드 주입

변수 선언부에 `@Autowired` 어노테이션을 붙인다.

```java
@Service
public class BurgerService {

    @Autowired
    private BurgerRecipe burgerRecipe;
}
```

**장점**

- 사용하기 편하다.

**단점**

- 단일 책임(SRP)의 원칙 위반
    - `@Autowired` 선언만 하면 되기 때문에 의존성을 주입하기 쉽다.
    - 따라서, 하나의 클래스가 많은 책임을 갖게 될 가능성이 높다.
- 의존성이 숨는다.
    - 생성자 주입에 비해 의존관계를 한눈에 파악하기 어렵다.
- DI 컨테이너의 결합성과 테스트 용이성
    - DI 컨테이너에만 의존한다. 따라서 스프링 컨테이너가 없는 환경(테스트)에서는 의존관계 주입을 할 수가 없다.
- 불변성을 보장할 수 없다.
    - `final`을 선언할 수 없다.
- 순환 참조가 발생할 수 있다.
    - 자세한 내용은 아래에서 따로 다루겠다.

### 수정자 주입

수정자 주입은 선택적인 의존성을 사용할 때 유용하다.

```java
@Service
public class BurgerService {

    private BurgerRecipe burgerRecipe;

		@Autowired
    public void setBurgerRecipe(BurgerRecipe burgerRecipe) {
        this.burgerRecipe = burgerRecipe;
    }
}
```

**장점**

- 선택적인 의존성을 사용할 수 있다.

**단점**

- 선택적인 의존성을 사용할 수 있다는 것은 `BurgerService`에 모든 구현체를 주입하지 않아도 `burgerRecipe` 객체가 생성이 가능하다는 것이고 이 말은 `burgerRecipe`의 모든 메소드를 호출할 수 있다는 뜻이다. 따라서 주입받지 않은 구현체를 사용하는 메소드에서 NPE가 발생한다.

### 생성자 주입

아래 처럼 생성자에 `@Autowired` 어노테이션을 붙여 의존성을 주입받을 수 있다(더 다양한 방법이 있다).

```java
@Service
public class BurgerService {

    private BurgerRecipe burgerRecipe;

		@Autowired
    public BurgerRecipe(BurgerRecipe burgerRecipe) {
        this.burgerRecipe = burgerRecipe;
    }
}
```

**장점**

- 의존관계를 모두 주입하지 않은 경우에 객체를 생성할 수 없다.
    - NPE가 발생하지 않는다.
- 불변성을 보장할 수 있다.
    - `final` 키워드를 사용할 수 있다.
- 순환 참조를 알 수 있다.
    - 생성자 주입은 컴파일 단계에서 순환 참조를 잡아 낼 수 있다.
- 의존성을 주입하기 번거롭고 생성자 인자가 많아지면 코드가 길어져 위기감을 느낄 수 있다.
    - 이를 바탕으로 SRP 원칙을 생각하게 되고, 리팩토링을 하게 된다.
    

**이러한 장점으로 스프링 4.x 공식문서에서는 생성자 주입을 권장한다.**

## 순환 참조

순환 참조란 서로 다른 여러 빈들이 서로를 참조하고 있음을 의미한다. 즉, 아래처럼 A는 B에서 필요한데 B는 또 A에서 필요한 상태를 말한다.

```
Bean A → Bean B → Bean A
```

CourseService에서 StudentService에 의존하고, StudentService가 CourseService에 의존하고 있는 예시를 보면서 설명을 하겠다.

### 필드 주입인 경우

```java
@Service
public class CourseServiceImpl implements CourseService {

    @Autowired
    private StudentService studentService;

    @Override
    public void courseMethod() {
        studentService.studentMethod();
    }
}

@Service
public class StudentServiceImpl implements StudentService {

    @Autowired
    private CourseService courseService;

    @Override
    public void studentMethod() {
        courseService.courseMethod();
    }
}
```

이 상황은 StudentServiceImple의 `studentMethod()`는 CourseServiceImpl의 `courseMethod()`를 호출하고, CourseServiceImpl의 `courseMethod()`는 StudentServiceImple의 `studentMethod()` 를 호출하고 있는 상황이다. 서로서로 주거니 받거니 호출을 반복하면서 끊임없이 호출하다가 결국 StackOverflowError를 발생시키고 죽는다.

**이처럼 필드 주입이나 수정자 주입은 객체 생성 후 비즈니스 로직 상에서 순환 참조가 일어나기 때문에 컴파일 단계에서 순환 참조를 잡아낼 수 없다.**

### 생성자 주입인 경우

```java
@Service
public class CourseServiceImpl implements CourseService {

		private final StudentService studentService;

    @Autowired
    public CourseServiceImpl(StudentService studentService) {
        this.studentService = studentService;
    }

    @Override
    public void courseMethod() {
        studentService.studentMethod();
    }
}

@Service
public class StudentServiceImpl implements StudentService {

    private final CourseService courseService;

    @Autowired
    public StudentServiceImpl(CourseService courseService) {
        this.courseService = courseService;
    }

    @Override
    public void studentMethod() {
        courseService.courseMethod();
    }
}
```

생성자 주입일 경우에는 애플리케이션 실행이 잘 될까? 실행해보면 아래와 같은 로그가 찍히면서 애플리케이션 실행이 실패한다.

**생성자 주입은 스프링 컨테이너가 빈을 생성하는 시점에 순환 참조를 확인하기 때문에 컴파일 단계에서 순환 참조를 잡아낼 수 있다.**

## @Autowired

### @Autowired란?

의존관계 주입(DI)를 할 때 사용하는 어노테이션이며, 의존 관계의 타입에 해당하는 빈을 찾아 주입하는 역할을 한다.

쉽게 말하자면, 스프링 서버가 올라갈 때 애플리케이션 컨텍스트가 @Bean이나 @Service, @Controller 등 어노테이션을 이용하여 등록한 스프링 빈을 생성하고, @Autowired 어노테이션이 붙은 위치에 의존성 주입을 수행하게 된다.

그럼 @Autowired 어노테이션에 붙은 위치에 어떻게 의존성을 주입하는걸까?

우선 @Autowired 어노테이션 코드를 살펴보자.

```java
/**
* Note that actual injection is performed through a BeanPostProcessor which in turn means
* that you cannot use @Autowired to inject references into BeanPostProcessor or 
* BeanFactoryPostProcessor types. Please consult the javadoc for the 
* AutowiredAnnotationBeanPostProcessor class (which, by default, checks for the presence 
* of this annotation).
* Since:
* 2.5
* See Also:
* AutowiredAnnotationBeanPostProcessor, Qualifier, Value
* Author:
* Juergen Hoeller, Mark Fisher, Sam Brannen
*/
@Target({ElementType.CONSTRUCTOR, ElementType.METHOD, ElementType.PARAMETER, ElementType.FIELD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Autowired {
	boolean required() default true;

}
```

- @Target
    - 생성자와 필드, 메소드에 적용 가능하다.
- @Retention
    - 컴파일 이후(런타임 시) JVM에 의해 참조가 가능하다. 런타임 시 이 어노테이션의 정보를 리플렉션으로 얻을 수 있다.

위 코드 상단의 주석을 살펴보면 `Note that actual injection is performed through a BeanPostProcessor`라는 내용을 찾을 수 있다. 실제 타깃에 Autowired가 붙은 빈을 주입하는 것은 BeanPostProcessor이라는 말이고, 그 아래를 계속 살펴보면 구현체는 AutowiredAnnotationBeanPostProcessor인 것을 확인할 수 있다.

![image](https://user-images.githubusercontent.com/55661631/154046543-4f36f93c-c0b8-4cd5-9f40-006beef3d917.png)

### AutowiredAnnotationBeanPostProcessor 클래스

위에서 AutowiredAnnotationBeanPostProcessor 클래스가 실제 타깃에 빈을 주입하는 역할을 한다고 했다. 

이 클래스에서 주의 깊게 살펴볼 부분은 `processInjection()` 메소드이다.

**proccessInjection() 메소드**

```java
public void processInjection(Object bean) throws BeanCreationException {
		Class<?> clazz = bean.getClass();
		InjectionMetadata metadata = findAutowiringMetadata(clazz.getName(), clazz, null);
		try {
				metadata.inject(bean, null, null);
		}
		catch (BeanCreationException ex) {
				throw ex;
		}
		catch (Throwable ex) {
				throw new BeanCreationException(
						"Injection of autowired dependencies failed for class [" + clazz + "]", ex);
		}
}
```

이 메소드는 @Autowired로 어노테이티드된 필드나 메서드에 대해서 객체를 주입하는 역할을 수행한다. InjectionMetadata 클래스의 `inject()` 메소드를 호출하여 객체를 주입하게 된다.

**inject()**

```java
public void inject(Object target, @Nullable String beanName, @Nullable PropertyValues pvs) throws Throwable {
		Collection<InjectedElement> checkedElements = this.checkedElements;
		Collection<InjectedElement> elementsToIterate = (checkedElements != null ? checkedElements : this.injectedElements);
		if (!elementsToIterate.isEmpty()) {
				for (InjectedElement element : elementsToIterate) {
					element.inject(target, beanName, pvs); //아래 inject() 메소드 호출
				}
		}
}

protected void inject(Object target, @Nullable String requestingBeanName, @Nullable PropertyValues pvs)
		throws Throwable {
	
		if (this.isField) {
				Field field = (Field) this.member;
				ReflectionUtils.makeAccessible(field);
				field.set(target, getResourceToInject(target, requestingBeanName));
		}
		else {
				if (checkPropertySkipping(pvs)) {
						return;
				}
				try {
						Method method = (Method) this.member;
						ReflectionUtils.makeAccessible(method);
						method.invoke(target, getResourceToInject(target, requestingBeanName));
				}
				catch (InvocationTargetException ex) {
						throw ex.getTargetException();
				}
		}
}
```

이 메소드가 객체를 주입할 때 ReflectionUtils 클래스를 사용하는 것을 볼 수 있다. 즉, @Autowired는 리플렉션을 통해 수행된다.

<aside>
💡 **리플렉션이란?**
리플렉션은 구체적인 클래스 타입을 알지 못해도, 그 클래스의 메서드, 타입, 변수들에 접근할 수 있도록 해주는 자바 API

</aside>

# 참고

- 토비의 스프링 3.1
- [https://tecoble.techcourse.co.kr/post/2021-04-27-dependency-injection/](https://tecoble.techcourse.co.kr/post/2021-04-27-dependency-injection/)
- [https://velog.io/@gillog/Spring-DIDependency-Injection-세-가지-방법](https://velog.io/@gillog/Spring-DIDependency-Injection-%EC%84%B8-%EA%B0%80%EC%A7%80-%EB%B0%A9%EB%B2%95)
- [https://jurogrammer.tistory.com/79](https://jurogrammer.tistory.com/79)
- [https://yaboong.github.io/spring/2019/08/29/why-field-injection-is-bad/](https://yaboong.github.io/spring/2019/08/29/why-field-injection-is-bad/)
- [https://keichee.tistory.com/446](https://keichee.tistory.com/446)
- [https://kellis.tistory.com/58](https://kellis.tistory.com/58)
- [https://beststar-1.tistory.com/40](https://beststar-1.tistory.com/40)
- [https://jwchung.github.io/DI는-IoC를-사용하지-않아도-된다](https://jwchung.github.io/DI%EB%8A%94-IoC%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EC%A7%80-%EC%95%8A%EC%95%84%EB%8F%84-%EB%90%9C%EB%8B%A4)

# 예상 면접 질문 및 답변

### Q. IoC 컨테이너란?

스프링 애플리케이션에서는 객체(빈)의 생성과 관계설정, 사용, 제거 등의 작업을 애플리케이션 코드 대신 스프링 컨테이너가 담당하는데, 이를 IoC 컨테이너라고 한다.

### Q. IoC 컨테이너의 장점은?

스프링 애플리케이션의 객체(빈)을 IoC 컨테이터가 관리해줌으로써 개발자의 부담이 줄고 비즈니스 로직에 더욱 집중할 수 있다는 장점이 있다.

### Q. DI란?

DI는 객체(빈)들 간의 의존관계를 외부에서 결정하고 주입하는 것이다.

`BurgerChef` 클래스와 `BurgerRecipe` 인터페이스를 예시로 설명하자.

### Q. DI의 장점은?

- 의존성이 줄어든다.
    - 의존한다는 것은 그 의존대상의 변화에 취약하다는 것이다.(대상이 변화하였을 때, 이에 맞게 수정해야함)
    - DI로 구현하게 되었을 때, 주입받는 대상이 변하더라도 그 구현 자체를 수정할 일이 없거나 줄어들게된다.
- 재사용성이 높은 코드가 된다.
    - 기존에 BurgerChef 내부에서만 사용되었던 BurgerRecipe을 별도로 구분하여 구현하면, 다른 클래스에서 재사용할 수가 있다.
- 테스트하기 좋은 코드가 된다.
    - BurgerRecipe의 테스트를 BurgerChef 테스트와 분리하여 진행할 수 있다.
- 가독성이 높아진다.
    - BurgerRecipe의 기능들을 별도로 분리하게 되어 자연스레 가동성이 높아진다.

### Q. DI의 종류는?

DI는 생성자 삽입, 수정자 삽입, 필드 주입이 있다.

생성자 주입은 생성자 호출시점에 딱 1번만 호출되는 것을 보장하며 불변, 필수 의존관계에 사용한다.

수정자 주입은 선택, 변경 가능성이 있는 의존관계에 사용되며 빈을 선택적으로 주입이 가능하다.

필드 주입은 외부에서 변경이 불가능하여 테스트 하기 힘들다. DI 프레임워크 없이는 작동하기 힘들며, 주로 애플리케이션과 관계없는 테스트코드나 @Configuration 같은 스프링 설정 목적으로 사용한다.

### Q. 순환 참조가 무엇이고 언제 발생하는가?

순환 참조란 서로 다른 여러 빈들이 서로를 참조하고 있음을 의미한다. 필드 주입이나 수정자 주입은 객체 생성 후 비즈니스 로직 상에서 순환 참조가 일어나기 때문에 컴파일 단계에서 순환 참조를 잡아낼 수 없다. 반면에 생성자 주입을 사용하면 스프링 컨테이너가 빈을 생성하는 시점에 순환 참조를 확인하기 때문에 컴파일 단계에서 순환 참조를 잡아낼 수 있다

### Q. 생성자 주입을 사용해야 하는 이유는?

- 의존관계를 모두 주입하지 않은 경우에 객체를 생성할 수 없기 때문에 NPE가 발생하지 않는다.
- `final` 키워드를 사용할 수 있어 불변성을 보장할 수 있다.
- 생성자 주입은 컴파일 단계에서 순환 참조를 잡아 낼 수 있다.
- 의존성을 주입하기 번거롭고 생성자 인자가 많아지면 코드가 길어져 위기감을 느낄 수 있다. 이를 바탕으로 SRP 원칙을 생각하게 되고, 리팩토링을 하게 된다.
- DI 컨테이너 없이 직접 의존성을 주입할 수 있다.

### Q. Spring IoC/DI의 동작 과정은?

IoC(제어의 역전)은 프로그램의 제어 흐름을 직접 제어하는 것이 아니라 외부에서 관리하는 것으로 코드의 최종 호출은 개발자가 제어하는 것이 아닌 프레임워크의 내부에서 결정된 대로 이루어진다.

DI(의존관계 주입)은 스프링 프레임워크에서 지원하는 IoC의 형태로 객체(빈) 사이의 의존관계를 빈 설정 정보를 바탕으로 DI 컨테이너가 자동으로 연결한다.

### Q. Autowiring 동작 과정은?

스프링 서버가 올라갈 때 스프링 컨텍스트가 @Bean이나 @Service, @Controller 등 어노테이션을 이용하여 등록한 스프링 빈을 생성하고, @Autowired 어노테이션이 붙은 위치 또는 생성자, 수정자를 통해 주입한다.

### Q. DI와 IoC의 차이는?

DI는 의존관계를 어떻게 가질 것인가에 대한 문제고, IoC는 누가 소프트웨어의 제어권을 갖고 있느냐의 문제다. IoC 컨테이너가 빈을 생성할 때 빈들간의 의존관계를 DI를 통해 해결한다.

DI는 IoC 사용을 필수로 요구하지 않는다는 점을 주의해야 한다.
