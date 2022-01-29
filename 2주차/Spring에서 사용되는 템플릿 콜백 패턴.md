## 템플릿 콜백 패턴

템플릿 콜백 패턴은 전략 패턴의 변형된 형태이다. 전략 패턴은 변화되는 부분을 매번 클래스로 만들고 외부에서 구체 클래스를 주입해 주어야 한다. 반면 템플릿 콜백 패턴은 변화되는 부분을 독립된 클래스를 만드는 것이 아니라 익명 내부 클래스를 생성하여 이를 활용하므로 주입이 필요하지 않다.

- 템플릿
    - 정해져 있는 틀
- 콜백
    - 다른 코드의 인수로서 넘겨주는 실행 가능한 코드
    - 어떤 이벤트에 의해 호출되어지는 코드

## 예제 코드 With 전략 패턴

학생이 선생님께 공부를 배우고 있다. 이때 학생은 빨간색, 파란색, 검은색 펜을 사용하여 필기를 할 수 있지만, 선생님이 펜을 골라주어야 필기를 할 수 있는 상황이다.

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F6d83c3fa-c245-44ce-8873-73345d6573a1%2FUntitled.png?table=block&id=b20aa2bb-85cf-4f78-be4b-36d338a1f88a&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

펜을 고르는 전략을 Strategy, 전략을 수행하는 Student, 구체 클래스를 주입하는 Teacher로 구성되어 있다.

```java
public interface Strategy {
    
    public abstract void ChoosePen();
}

public class BlackPen implements Strategy {

    public void ChoosePen() {
        System.out.println("검은펜을 잡았습니다.");
    }
}

public class BluePen implements Strategy {

    public void ChoosePen() {
        System.out.println("파란펜을 잡았습니다.");
    }
}

public class RedPen implements Strategy {

    public void ChoosePen() {
        System.out.println("빨간펜을 잡았습니다.");
    }
}

public class Student {

    public void takeNotes(Strategy strategy) {

        System.out.println("=== 선생님이 펜을 주십니다. ===");
        strategy.ChoosePen();
        System.out.println("필기를 시작합니다.");
    }
}

public class Teacher {
    public static void main(String[] args) {
        Student student = new Student();

        BlackPen black = new BlackPen();
        RedPen red = new RedPen();
        BluePen blue = new BluePen();

        student.takeNotes(black);
        student.takeNotes(red);
        student.takeNotes(blue);
    }
}
```

전략 패턴은 런타임 도중 객체를 주입하는 DI를 사용함으로써 구현한다.

## 예제 코드 With 템플릿 콜백 패턴

템플릿 콜백 패턴은 전략 패턴과 달리 구체 클래스를 미리 만들어 놓지 않으며 익명 클래스를 사용한다. 그래서 위 전략 패턴 코드를 다음과 같이 바꿀 수 있다.

```java
public interface Strategy {

    public abstract void ChoosePen();
}

public class Student {

    public void takeNotes(Strategy strategy) {
        System.out.println("=== 선생님이 펜을 주십니다. ===");
        strategy.ChoosePen();
        System.out.println("필기를 시작합니다.");
    }
}

public class Teacher {
    public static void main(String[] args) {
        Student student = new Student();

        student.takeNotes(new Strategy() {
            public void ChoosePen() {
                System.out.println("검은펜을 잡았습니다.");
            }
        });

        student.takeNotes(new Strategy() {
            public void ChoosePen() {
                System.out.println("빨간펜을 잡았습니다.");
            }
        });

        student.takeNotes(new Strategy() {
            public void ChoosePen() {
                System.out.println("파란펜을 잡았습니다.");
            }
        });
    }
}
```

클라이언트 단에서 익명 클래스를 통해 전략을 그때 그때 정해 주면서 로직을 수행할 수 있다. 하지만 매번 익명 클래스를 만드는 것이 번거로우므로 이를 Student에 넣어보겠다.

```java
public class Student {
    public void takeNotes(String Pen) {
        System.out.println("=== 선생님께서 펜을 주십니다. ===");
        takePen(Pen).ChoosePen();
        System.out.println("필기를 시작합니다.");
    }

    private Strategy takePen(String Pen) {
        return new Strategy() {
            @Override
            public void ChoosePen() {
                System.out.println(Pen + "을 잡았습니다.");
            }
        };
    }
}

public class Teacher {
    public static void main(String[] args) {
        Student student = new Student();

        student.takeNotes("검정펜");
        student.takeNotes("빨간펜");
        student.takeNotes("파란펜");
        student.takeNotes("무지개 형관펜");
    }
}
```

익명 클래스를 Student 객체 내부로 옮기고 변하는 최소한의 부분만 클라이언트에서 주입하도록 수정함으로써 코드가 간결해진 것을 알 수 있다.

## 템플릿 콜백 패턴의 장단점

- 장점
    - 전략 패턴은 객체와 전략 객체 간의 의존성을 설정해 주는 팩토리 객체가 필요한데, 템플릿 콜백 패턴은 팩토리 객체 없이 해당 객체를 사용하는 메소드에서 인터페이스의 전략을 선택한다. 그래서 수동 DI, 메소드 수준의 DI라고 부른다. 이를 통해 모든 전략마다 팩토리 객체를 일일이 만들 필요가 없다.
    - 외부에서 어떤 전략을 사용하는지 감추고 중요한 부분에 집중할 수 있다.
- 단점
    - 인터페이스를 사용하긴 하지만, 실제 사용할 클래스를 직접 선언할 경우 결합도가 증가한다. (위 예제는 아님)

## Spring에서 템플릿 콜백 패턴을 적용한 사례

Spring에서 템플릿 콜백 패턴은 JdbcTemplate에서 사용되고 있다. 모든 코드를 분석하지는 않았으나 `update()` 메소드에서 템블릭 콜백 패턴이 쓰인 것을 확인하였다. `update()` 만 간단히 살펴 보자.

```java
@Override
public int update(final String sql) throws DataAccessException {
    // ...
    // 내부의 콜백 객체가 정의되어 있다.
    class UpdateStatementCallback implements StatementCallback<Integer>, SqlProvider {
        @Override
        public Integer doInStatement(Statement stmt) throws SQLException {
            int rows = stmt.executeUpdate(sql);
            if (logger.isTraceEnabled()) {
                logger.trace("SQL update affected " + rows + " rows");
            }
            return rows;
        }
        @Override
        public String getSql() {
            return sql;
        }
    }

    // 템플릿 메소드 파라미터로, 이 콜백 객체를 전달한다.
    return updateCount(execute(new UpdateStatementCallback()));
}

// 클라이언트
public class UserDao {
    // ...

    public void deleteAll() {
        final String query = "delete from users";
        jdbcTemplate.update(query);
    }

    // ...
}
```

클라이언트 단에서 sql문을 JdbcTemplate의 CRUD 메소드로 보내면 `update()` 메소드 내부의 정의된 익명 클래스(콜백 객체)를 사용하고 있다.

## 출처

- [https://western-sky.tistory.com/41](https://western-sky.tistory.com/41)
- [https://limkydev.tistory.com/85](https://limkydev.tistory.com/85)
- [https://niceman.tistory.com/133](https://niceman.tistory.com/133)
- [https://siyoon210.tistory.com/131](https://siyoon210.tistory.com/131)
- [https://gurumee92.tistory.com/211](https://gurumee92.tistory.com/211)

## 예상 면접 질문 및 답변

### 템플릿 콜백 패턴이란?

템플릿 콜백 패턴은 전략 패턴의 변형된 형태이다. 전략 패턴은 변화되는 부분을 매번 클래스로 만들고 외부에서 구체 클래스를 주입해 주어야 한다. 반면 템플릿 콜백 패턴은 변화되는 부분을 독립된 클래스를 만드는 것이 아니라 익명 내부 클래스를 생성하여 이를 활용하므로 주입이 필요하지 않다.

### 템플릿 콜백 패턴의 장단점

- 장점
    - 전략 패턴은 객체와 전략 객체 간의 의존성을 설정해 주는 팩토리 객체가 필요한데, 템플릿 콜백 패턴은 팩토리 객체 없이 해당 객체를 사용하는 메소드에서 인터페이스의 전략을 선택한다. 그래서 수동 DI, 메소드 수준의 DI라고 부른다. 이를 통해 모든 전략마다 팩토리 객체를 일일이 만들 필요가 없다.
    - 외부에서 어떤 전략을 사용하는지 감추고 중요한 부분에 집중할 수 있다.
- 단점
    - 인터페이스를 사용하긴 하지만, 실제 사용할 클래스를 직접 선언할 경우 결합도가 증가할 수 있다.

### Spring에서 템플릿 콜백 패턴은 어디서 쓰이는가?

JdbcTemplate의 `update()` 메소드에서 내부의 익명 객체를 정의하여 사용하고 있다.
