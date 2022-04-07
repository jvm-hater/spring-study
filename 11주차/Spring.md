## Spring이란?

- 자바 엔터프라이즈 개발을 편하게 해주는 오픈소스 경량급 애플리케이션 프레임워크
    - 프레임워크
        - 개발할 때 설계 기본이 되는 뼈대나 구조/환경
    - 애플리케이션 프레임워크
        - 애플리케이션 개발의 전 과정을 빠르고 편리하며 효율적으로 진행하는데 일차적인 목표를 두는 프레임워크이다
    - 경량급
        - 스프링 자체가 아주 가볍다거나 작은 규모의 코드로 이뤄졌다는 뜻은 아니다.
        - 스프링이 처음 등장하던 시절의 자바 주류 기술이었던 예전의 EJB 같은 과도한 엔지니어링이 적용된 기술과 스프링을 대비시켜 설명하려고 사용됐던 표현이다.
    - 자바 엔터프라이즈 개발을 편하게
        - 로우레벨의 트랜잭션이나 상태 관리, 멀티스레딩, 리소스 풀링과 같은 복잡한 로우레벨의 API를 이해하지 못하더라도 아무런 문제 없이 애플리케이션을 개발할 수 있다.
        - 따라서 개발자들이 스프링이라는 프레임워크가 제공하는 기술이 아니라 자신이 작성하는 애플리케이션의 로직에 더 많은 관심과 시간을 쏟게 해준다.
        - 초기에 스프링의 기본 설정과 적용 기술만 잘 선택하고 준비해두면, 이후로 애플리케이션 개발 중에는 스프링과 관련된 코드나 API에 대해 개발자가 거의 신경 쓸 일이 없다.

## Spring 주요 특징

### **IoC(Inversion of Control, 제어의 역전)**

- 개발자는 JAVA 코딩시 new 연산자, 인터페이스 호출, 데이터 클래스 호출 방식으로 객체를 생성하고 소멸시킨다.
- IoC란 인스턴스 (객체)의 생성부터 소멸까지 객체 생명주기 관리를 개발자가 하는게 아닌 스프링 컨테이너가 대신 해주는 것을 말한다.
- 프로젝트의 규모가 커질수록 객체와 자원을 이용하는 방법이 더 복잡해지고 어디서 코드가 꼬일지 모르는 것을 Spring의 IoC는 자동으로 관리해준다.
- 즉, **제어권이 개발자가 아닌 IoC에게 있으며** IoC가 개발자의 코드를 호출하여 그 코드로 생명주기를 제어하는 것이다.

### **DI(Dependency Injection, 의존성 주입)**

- 프로그래밍에서 구성요소 간의 의존 관계가 소스코드 내부가 아닌 외부를 통해 정의되는 방식이다.
- 코드 재사용을 높여 재사용을 높여 소스코드를 다양한 곳에 사용할 수 있으며 모듈간의 결합도도 낮출 수 있다..

### **AOP(Aspect Object Programming, 관점 지향 프로그래밍)**

- 로깅, 트랜잭션, 보안 등 여러 모듈에서 공통적으로 사용하는 부가 기능을 분리하여 관리 할 수 있다.
- 즉, AOP는 여러 객체에 공통으로 적용할 수 있는 부가 기능을 구분함으로써 재사용성을 높여주는 프로그래밍 기법이다.

### **POJO(Plain Old Java Object) 방식**

- POJO는 Java EE를 사용하면서 해당 플랫폼에 종속되어 있는 무거운 객체들을 만드는 것에 반발하여 나타난 용어이다.
- 별도의 프레임 워크 없이 Java EE를 사용할 때에 비해 인터페이스를 직접 구현하거나 상속받을 필요가 없어 기존 라이브러리를 지원하기 용이하고, 객체가 가볍다.
- 즉, getter/setter를 가진 단순한 자바 오브젝트를 말한다.

## Spring 모듈

### Spring Core

- Spring 프레임워크의 근간이 되는요소. IoC(또는 DI) 기능을 지원하는 영역을 담당.
- BeanFactory를 기반으로 Bean 클래스들을 제어할 수 있는 기능을 지원

### Spring Context

- Spring Core 바로 위에 있으면서 Spring Core에서 지원하는 기능외에 추가적인 기능들과 좀 더 쉬운 개발이 가능하도록 지원
- 또한 JNDI, EJB등을 위한 Adaptor들을 포함

### Spring DAO

- 지금까지 우리들이 일반적으로 많이 사용해왔던 JDBC 기반하의 DAO개발을 좀 더 쉽고, 일관된 방법으로 개발하는 것이 가능하도록 지원
- Spring DAO를 이용할 경우 지금까지 개발하던 DAO보다 적은 코드와 쉬운 방법으로 DAO를 개발하는 것이 가능

### Spring ORM

- Object Relation Mapping 프레임워크인 Hibernate, IBatis, JDO와의 결합을 지원하기 위한 기능
- Spring ORM을 이용할 경우 Hibernate, IBatis, JDO 프레임워크와 쉽게 통합하는 것이 가능

### Spring AOP

- Spring 프레임워크에 Aspect Oriented Programming을 지원하는 기능이다.

### Spring Web

- Web Application 개발에 필요한 Web Application Context와 Multipart Request등의 기능을 지원

### Spring Web MVC

- Spring 프레임워크에서 독립적으로 Web UI Layer에 Model-View-Controller를 지원하기 위한 기능

## Spring boot

Spring Framework는 기능이 많은만큼 환경설정이 복잡한 편이다. 이에 어려움을 느끼는 사용자들을 위해 나온 것이 바로 Spring Boot이다. Spring Boot는 Spring를 사용하기 위한 설정의 많은 부분을 자동화하여 사용자가 편하게 Spring을 활용할 수 있도록 돕는다. 최소한의 설정으로 스프링의 여러 모듈을 사용할 수 있다.

### Spring boot의 장점

- Auto Configuration로 복잡한 설정 자동화
    - 프로젝트에서 필요한 bean들을 자동으로 등록 및 설정해준다.
    - starter를 통해 의존성을 통해 간단히 설정할 수 있다.
- 쉬운 의존성 관리
    - 스프링 프레임워크는 의존성들을 일일이 찾고, 호환되는 버전에 맞춰 의존성을 추가하는 번거로움이 존재한다.
    - Spring boot Starter를 통해 관련된 의존성을 한번에 추가할 수 있다.
- 내장 서블릿 컨테이너(=내장 서버) 제공
    - 스프링 프레임워크는 서버를 배포하기 위해 톰캣과 같은 별도의 외장 웹 서버를 설치하고, war 파일을 생성하여 배포해 주어야 했다.
    - 이러한 방식은 상당히 번거롭고 처리 속도도 느렸는데, 스프링 부트는 내장 웹 서버(톰캣 or 네티)을 가지고 있어서 별도의 작업 없이 빠르게 서버를 실행할 수 있도록 도와준다.

## 참고

- [https://12bme.tistory.com/157](https://12bme.tistory.com/157)
- [https://jerryjerryjerry.tistory.com/62](https://jerryjerryjerry.tistory.com/62)
- [https://goddaehee.tistory.com/156](https://goddaehee.tistory.com/156)
- [https://server-engineer.tistory.com/739](https://server-engineer.tistory.com/739)
- [https://ssoco.tistory.com/66](https://ssoco.tistory.com/66)
