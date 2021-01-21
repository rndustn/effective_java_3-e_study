# 6장. 열거 타입과 어노테이션

자바의 특수한 목적의 참조 타입 두가지

- 클래스의 일종인 열거타입
- 인터페이스 일종인 어노테이션

# 아이템 34. int 상수 대신 열거 타입을 사용ㅎ라

열거타입은 일정 개수의 상수 값을 정의한 다음, 그 외의 값은 허용하지 않는 타입이다.

정수 열거 패턴

```java
public static final int APPLE_FUGI = 0;
public static final int APPLE_PIPPIN = 1;
```

→ 단점이 많다. → 정수대신 문자열 열거패턴으로 변경하려하면 이것은 더 단점이 많다.

### 대체안. 열거타입

```java
public enum Apple {FUJI, PIPPIN}
```

- 열거 타입 자체는 클래스이며, 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개한다.
- 열거타입은 밖에서 접근할 수 있는 생성자를 제공하지 않으므로 사실상 final이다.
- 클라이언트가 인스턴스를 직접 생성하거나 확장할 수 없으니 열거타입 선언으로 만들어진 인스턴스들은 딱 하나씩만 존재함이 보장된다.
- 다시 말해 열거타입은 인스턴스 통제된다.
- 싱글턴은 원소가 하나뿐인 열거타입이라 할 수 있고, 거꾸로 열거타입은 싱글턴을 일반화한 형태라고 볼 수 있다.
- 열거타입은 컴파일 타임 타입 안정성을 제공한다.

- 열거 타입 상수 각각을 특정데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 된다. 열거타입은 근본적으로 불변이라 모든 필드는 final이여야 한다.
- 필드를 public으로 선언해도 되지만, private 으로 두고 별도의 public 접근자 메서드를 두는게 낫다.
- 열거타입은 자신안에 정의된 상수들을 값을 배열에 담아 반환하는 정적 메서드인 values를 제공한다.

### 상수 별 메서드를 구현을 활용한 열거타입

```java
public enum Operation {
PLUS {public double apply(double x, double y) { return x + y;}},
MINUS {public double apply(double x, double y) { return x - y;}}

public abstract double apply(double x, double y);
}
```