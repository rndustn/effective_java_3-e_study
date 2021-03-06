# 6장. 열거타입과 애너테이션

- [ ]  자바에는 특수한 목적의 참조타입이 두 가지 있다.
- [ ]  클래스의 일종인 열거타입(enum; 열거형)
- [ ]  인터페이스의 일종인 애너테이션(annotation)

# 아이템34. int 상수 대신 열거 타입을 사용하라

## 👎🏻정수 열거 패턴의 단점

```java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
```

- 타입 안전을 보장할 방법이 없으며 표현력도 좋지 않다.
- APPLE_FUJI == ORANGE_NAVEL; → true가 되어 컴파일러는 경고를 출력하지 않는다.
- 접두어를 사용해 이름 충돌을 방지하는 것이다.
- 평범한 상수 나열이라 컴파일하면 그 값이 클라이언트 파일에 그대로 새겨지므로 깨지기 쉽다. → **상수의 값이 바뀌면 클라이언트도 반드시 재 컴파일해야 한다.**
- 정수상수는 문자열로 출력하기가 까다롭다. 
System.out.println(APPLE_FUJI); → //0 출력 (APPLE_FUJI 같은 의미를 출력하기 어렵다는 의미)
- 같은 정수 열거 그룹에 속한 모든 상수를 한 바퀴 순회하는 방법도 마땅치 않다.
(enum은 values()메서드를 통해 상수를 배열 형태로 제공한다.)
- 문자열 열거패턴 : 정수대신 문자열 상수를 사용 → 더 나쁨!
상수의 의미를 출력할 수 있지만, 문자열값에 대한 하드코딩 실수→ 컴파일러 확인불가 → 런타임 버그생성, 문자열 비교에 따른 성능저하

## 📌열거 타입

### ✅열거 타입의 특징

- 열거 타입 자체는 클래스이다.
- 상수 하나당 자신의 인스턴스를 하나씩 만들어 **public static final 필드**로 공개한다.
- 열거 타입은 밖에서 접근할 수 있는 생성자를 제공하지 않으므로 사실상 final이다.
- 열거 타입은 근본적으로 **불변이라 모든 필드는 final**이어야 한다.
- 열거 타입 선언으로 만들어진 **인스턴스들은 딱 하나씩만 존재한다.**
- 열거 타입은 **인스턴스 통제클래스**이다. (정적 팩터리 방식 클래스)
    - 싱글턴 : 원소가 하나뿐인 열거타입
    - 열거타입 : 싱글턴을 일반화한 형태

### 👍🏻열거 타입의 장점

```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```

- 컴파일타임 **타입 안정성**을 제공한다.
→ 다른 열거 타입의 값끼리 Apple.FUJI == Orange.NAVEL 은 컴파일 에러가 발생한다.
- 열거 타입에는 **각자의 namespace가 있어서** 이름이 같은 상수도 공존한다.
- 새로운 상수를 추가하거나 순서를 바꿔도 **재컴파일 하지 않아도 된다.**
→ 공개되는 것이 필드의 이름 뿐이라, 상수 값이 컴파일 시점에 각인되지 않는다.
- 임의의 메서드나 필드를 추가할 수 있고 임의의 인터페이스를 구현하게 할 수도 있다.
Enum 자체는 Object 메소드들과 Comparable, Serializable을 구현해두었다.

    ```java
    public abstract class Enum<E extends Enum<E>>
            implements Comparable<E>, Serializable {
    ```

- 상수를 제거하더라도 제거한 상수를 참조하지 않는 클라이언트는 아무 영향이 없다.
제거된 상수를 참조하면 컴파일 에러가 발생한다.→ 정수열거는 못하는 기능

### 📎열거 타입 사용시 고려할 점

1. 일반클래스와 마찬가지로, 그 기능을 클라이언트에게 노출할 이유가 없다면 private 있으면 package-private로 선언한다.
2. 널리쓰이는 열거타입은 톱레벨 클래스, 특정 톱레벨 클래스에서만 쓰이면 멤버 클래스로 만든다.

## ✔️데이터와 메서드를 갖는 열거타입

- 각 상수와 연관된 데이터를 해당 상수 자체에 내재시키는 경우→ 메서드나 필드 추가

```java
public enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS(4.869e+24, 6.052e6),
    EARTH(5.975e+24, 6.378e6),
    MARS(6.419e+23, 3.393e6),
    JUPITER(1.899e+27, 7.149e7),
    SATURN(5.685e+26, 6.027e7),
    URANUS(8.683e+25, 2.556e7),
    NEPTUNE(1.024e+26, 2.477e7);

    private final double mass; // 질량 (단위: 킬로그램)
    private final double radius; // 반지름 (단위: 미터)
    private final double surfaceGravity; // 표면중력 (단위: m / s^2)

    // 중력 상수 (단위 : m^3 / kg s^2)
    private static final double G = 6.67300E-11;
		// 생성자
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius); //단순히 최적화위해 생성자에서 계산
    }

    public double getMass() { return mass;  }
    public double getRadius() { return radius; }
    public double surfaceGravity() { return surfaceGravity; }

    public double surfaceWeight(double mass) {
        return mass * surfaceGravity; // F = ma
    }
}
```

- 열거 타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 된다.
- 모든 필드는 final
- 필드를 public으로 선언해도되지만, private로 두고 public 접근자 메서드 getter를 두는게 낫다.
- 상수들의 값을 배열에 담아 반환하는 정적 메서드인 values()를 제공한다.

    ```java
    public class WeightTable {
       public static void main(String[] args) {
          double earthWeight = Double.parseDouble(args[0]);
          double mass = earthWeight / Planet.EARTH.surfaceGravity();
          for (Planet p : Planet.values())
             System.out.printf("%s에서의 무게는 %f이다.%n", p, p.surfaceWeight(mass));
       }
    }
    ```

- 열거 타입 값의 toString() 메서드는 상수 이름을 문자열로 반환한다.
- valueOf(String)메서드 자동생성 : 상수 이름을 입력받아 그 이름에 해당하는 상수를 반환한다.
- fromString메서드 : toString이 반환하는 문자영을 해당 열거 타입 상수로 변환한다.
→ toString 메서드를 재정의할 때 함께 제공하는걸 고려해보자!

    ```java
    //열거타입 정적 필드의 생성 시점
    private static final Map<String, Operation> stringToEnum =
    						Stream.of(Operation.values())
                .collect(Collectors.toMap(Operation::toString, operation -> operation));

    //Optional로 반환하여 값이 존재하지않을 상황을 클라이언트에게 알린다.
    public static Optional<Operation> fromString(String symbol) {
        return Optional.ofNullable(stringToEnum.get(symbol));
    }
    ```

    Operation 상수가 stringToEnum 맵에 추가되는 시점: **열거 타입 상수 생성 후 정적 필드가 초기화 될 때** 

    →❗**열거 타입 상수는 생성자에서 자신의 인스턴스를 맵에 추가할 수 없다.→ 컴파일 오류**

    - 열거타입의 정적 필드 중 열거 타입의 **생성자에서 접근할 수 잇는 것은 상수 변수** 뿐이다.
    - 열거 타입 생성자가 실행되는 시점에는 정적 필드 초기화 전이다.
    **→ 열거 타입 생성자에서 정적 필드를 참조하려고 하면 컴파일 에러가 발생한다.**
    - 열거 타입 생성자에서 같은 열거 타입의 다른 상수에도 접근할 수 없다.
    (열거 타입의 인스턴스를 public static final으로 선언함. 다른 형제 상수도 static이므로 열거 타입 생성자에서 정적 필드에 접근할 수 없다는 제약이 적용된다.)

## ✔️상수별 메서드 구현

### 상수별 클래스 몸체와 데이터를 사용한 열거 타입

```java
public enum Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;

    Operation(String symbol) { this.symbol = symbol; }
		@Override
    public String toString() { return symbol; }
    public abstract double apply(double x, double y);    
}
```

- switch 문으로 해결할 수도 있지만, 새로운 상수가 추가된다면 case 문도 추가해야 한다.
- 열거 타입에 추상 메서드를 선언하고, 각 상수별 클래스 몸체(constant-specific class body)를 각 상수에 맞게 재정의하는 방법으로 개선할 수 있다.

### 👎🏻상수별 메서드 구현의 단점

- 상수별로 같은 처리를 하는 메서드가 있더라도 추상 메서드를 재정의 → 중복코드 발생
- 열거 타입 상수끼리 코드 공유가 어렵다.

## ✔️전략적 열거 타입 패턴

열거 타입 상수 일부가 같은 동작을 공유한다면 전략 열거 타입 패턴을 사용하자.

→ 추가하려는 메서드가 의미상 열거 타입에 속할 때

```java
enum PayrollDay {
    MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY),
    THURSDAY(WEEKDAY), FRIDAY(WEEKDAY),
    SATURDAY(WEEKEND), SUNDAY(WEEKEND);

    private final PayType payType;

    PayrollDay(PayType payType) { this.payType = payType; }

    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }
	//전략열거타입
    enum PayType {
        WEEKDAY {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked <= MINS_PER_SHIFT ? 0 :
                        (minsWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minsWorked, int payRate) {
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }

    public static void main(String[] args) {
        for (PayrollDay day : values())
            System.out.printf("%-10s%d%n", day, day.pay(8 * 60, 1));
    }
}
```

잔업수당 계산을 private중첩 열거 타입(PayType)으로 옮기고 PayrollDay 열거 타입의 생성자에서 이중 적당한 것을 선택한다.

## ✏️핵심정리

- 필요한 원소를 컴파일타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자.
Ex) 태양계 행성, 한 주의 요일, 체스말
- 열거 타입에 정의된 상수 개수가 영원히 고정불변일 필요는 없다.
Ex) 메뉴 아이템, 연산 코드, 명령줄 플래그
- 열거 타입의 성능은 상수와 비슷하다.
- 각 상수를 특정 데이터와 연결짓거나 상수마다 다르게 동작할 때 열거 타입 사용하자.
- switch 문 보단 상수별 메서드 구현
- 열거 타입 상수 일부가 같은 동작을 공유한다면 전략 열거 타입 패턴을 사용하자.

# 아이템35. ordinal 메서드 대신 인스턴스 필드를 사용하라

- 대부분의 열거 타입 상수는 하나의 정숫값에 대응한다.
- ordinal 메서드 : 해당 상수가 열거 타입에서 몇 번째 위치인지를 반환한다.
- ordinal 메서드를 잘못 사용한 예

    ```java
    public enum Ensemble {
    	SOLO, DUET, TRIO, QUARTET, QUINTET,
    	SEXTET, SEPTET, OCTET, NONET, DECTET;

    	public int numberOfMusicians() { return ordinal() + 1; }
    }
    ```

    - 유지보수에 취약
    - 상수 선언을 바꾸는 순간 오작동한다.
    - 이미 사용중인 정수와 값이 같은 상수는 추가할 수 없다.
    - 중간에 값을 비울 수 없다 → 값을 비우기 위해 더미 상수를 추가해야 한다.
- **해결책** : **열거 타입 상수에 연결된 값은 ordinal 메서드로 얻지말고, 인스턴스 필드에 저장한다.**

    ```java
    public enum Ensemble {
    	SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    	SEXTET(6), SEPTET(7), OCTET(8), NONET(9), DECTET(10);

    	private final int numberOfMusicians;
    	Ensemble(int size) { this.numberOfMusicians = size; }
    	public int numberOfMusicians() { return numberOfMusicians; }
    }
    ```

- ordinal 메서드는 EnumSet과 EnumMap 같이 열거 타입 기반의 범용 자료구조에 쓸 목적으로만 사용해야 한다.

# 아이템36. 비트 필드 대신 EnumSet을 사용하라

- 열거한 값들이 주로 단독이 아닌 집합으로 사용될 경우, 예전에는 각 상수에 서로다른 2의 거듭제곱 값을 할당한 정수 열거 패턴을 사용했다.

    ```java
    public class Text {
        public static final int STYLE_BOLD =          1 << 0; //1
        public static final int STYLE_ITALIC =        1 << 1; //2
        public static final int STYLE_UNDERLINE =     1 << 2; //4
        public static final int STYLE_STRIKETHROUGH = 1 << 3; //8
        
        //매개 변수 styles는 0개 이상의 STYLE_ 상수를 비트별 OR 한 값이다.
        public void applyStyle(int styles) {...}
    }
    ```

- **비트별 OR를 사용해 여러 상수를 하나의 집합**으로 모을 수 있으며, 이렇게 만들어진 집합을 **비트 필드**라 한다.

    ```java
    text.applyStyles(STYLE_BOLD | STYLE_ITALIC); // 1 | 2 => 3
    ```

    비트별 연산으로 합집합, 교집합 같은 집합연산을 효율적으로 수행할 수 있다.

## 👎🏻비트 필드의 문제점

- 정수 열거 상수의 단점을 그대로 지닌다.
- 비트 필드 값이 그대로 출력되면 단순한 정수 열거 상수를 출력할 때 보다 해석하기 어렵다.
→ 어떤 값들이 OR되서 나온지 알기 어렵다.
- 비트 필드 하나에 녹아 있는 모든 원소를 순회하기 까다롭다.
- 최대 몇 비트가 필요한지를 API작성 시 미리 예측하여 적절한 타입을 선택해야 한다.
→ 보통은 int나 long, API수정 없이 비트의 수(32비트 or 64비트)를 더 늘릴 수 없기 때문이다.

## 📌EnumSet 클래스

- 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해준다.
- Set 인터페이스를 완벽히 구현한다.
- 타입 안전하다.
- 다른 어떤 Set 구현체와도 함께 사용할 수 있다.
- **EumSet의 내부는 비트 벡터로 구현되었다.**
- 원소가 총 64개 이하라면, 즉 대부분의 경우에 EnumSet 전체를 long변수 하나로 표현하여 **비트 필드와 성능이 비슷하다.**
- removeAll과 retainAll 같은 대량 작업은 비트를 효율적으로 처리할 수 있는 산술 연산을 써서 구현했다.

비트 필드를 EnumSet으로 대체한 예시

```java
public class Text {
    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }
    
    // 어떤 Set을 넘겨도 되나 EnumSet이 가장 좋다.
    // 이왕이면 Set Interface로 받는게 좋은 습관이다. 
		// -> 클라이언트가 다른 Set구현체를 넘기더라도 처리할 수 있다.
    public void applyStyles(Set<Style> styles) {...}
		// 사용 예
    public static void main(String[] args) {
        Text text = new Text();
        text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
    }
}
```

## ✏️핵심정리

- 열거할 수 있는 타입을 한데 모아 집합 형태로 사용한다고 해도 비트 필드를 사용할 이유 없다.
- EnumSet클래스가 비트 필드 수준의 명료함과 성능을 제공한다.
- 열거 타입의 장점도 지닌다.
- 유일한 단점은 (자바9까지는 아직) 불변 EnumSet을 만들 수 없다.
→ 현재는 Collections.unmodifiableSet으로 EnumSet을 감싸 사용한다.

# 아이템37. ordinal 인덱싱 대신 EnumMap을 사용하라

## 👎🏻배열이나 리스트의 인덱스를 ordinal()로 얻는 경우 문제점

```java
//1.
Set<Plant>[] plant ByLisfeCycle = (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
for(int i =0; i < plantsByLifeCycle.length; i++)
  plantsByLifeCycle[i] = new HashSet<>();
for(Plant p : garden){
	//2.
  plantsByLifeCycle[p.lifeCycler.ordinal()].add(p);
}
for(int i =0; i<plantsByLifeCycle.length;i++)}{
  System.out.println("%s: %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
}
```

1. 배열은 제네릭과 호환되지 않아 비검사 형변환을 수행해야하고 컴파일되지 않는다.
2. 배열은 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야 한다.
3. 정수는 열거타입과 달리 타입안전하지 않기 때문에 정확한 정숫값을 사용하지 않으면 잘못된 동작 또는 ArrayIndexOutOfBoundsException을 던진다.

## 해결책: EnumMap을 사용해 데이터와 열거 타입을 매핑한다.

EnumMap: 열거 타입을 키로 사용

```java
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = 
																	new EnumMap<>(Plant.LifeCycle.class);
for (Plant.LifeCycle lc : Plant.LifeCycle.values()) {
    plantsByLifeCycle.put(lc, new HashSet<>());
}
for (Plant p : garden) {
    plantsByLifeCycle.get(p.lifeCycle).add(p);
}
System.out.println(plantsByLifeCycle);//EnumMap은 toString을 재정의
```

- 짧고 명료, 안전, 성능도 비슷
- 맵의 키인 열거타입이 toString(출력용 문자열) 제공하여 출력결과에 직접 레이블 달 필요없다.
- 배열 인덱스 계산 오류 가능성 없다.
- 성능비슷 이유: 내부에서 배열을 사용하고, 내부 구현 방식을 안으로 숨겨서 Map의 타입 안전성과 배열의 성능을 모두 얻어 낸다.
- EnumMap의 생성자가 받는 키 타입의 Class 객체는 한정적 타입 토큰이다. 
→ 제네릭의 타입 정보가 런타임시에 소거되기 때문에 런타임 제네릭 타입 정보를 제공하기 위해 키 타입의 Class 객체를 받는다.

## Stream 사용하여 Map 관리

collect() : 단순히 요소를 수집하는 기능 외컬렉션의 요소들을 그룹핑해서 Map객체를 생성하는 기능을 제공 →Collectors.groupingBy ()Map 객체 리턴

1. EnumMap 사용하지 않고 Stream만 사용

    ```java
    //HashMap을 이용한 데이터와 열거타입 매핑
    Arrays.stream(garden)
          .collect(groupingBy(p -> p.lifeCycle))
    ```

    - 반환 맵은 HashMap, Value는 ArrayList
    - EnumMap을 써서 얻은 공간과 성능이점이 사라진다.
2. EnumMap, Stream 같이 사용

    ```java
    //EnumMap을 이용해 데이터와 열거타입 매핑
    Arrays.stream(garden)
          .collect(groupingBy(p -> p.lifeCycle, () -> new EnumMap<>(LifeCycle.class),
                             toSet()));
    ```

    - 반환 맵은 EnumMap, Value는 HashSet

    Collectors.groupBy(맵의 키, 맵의 종류, value의 자료형)는 mapFactory 매개변수에 원하는 맵 구현체를 명시해 호출할 수 있다.

- EnumMap 열거 타입 상수 별로 하나씩 Key를 전부다 만든다. 
(garden에 데이터가 없어도 모든 키가 다 만들어 진다.)
- Stream을 이용한 방식에는존재하는 열거 타입 상수만 Key를 만든다. 
(garden에 있는 키만 만든다.)

## 중첩 EnumMap으로 데이터와 열거 타입 쌍 연결한다.

```java
public enum Phase {

    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT(SOLID, LIQUID),
        FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS),
        CONDENSE(GAS, SOLID),
        SUBLIME(SOLID, GAS),
        DEPOSIT(GAS, SOLID);

        private final Phase from;
        private final Phase to;

        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        public static final Map<Phase, Map<Phase, Transition>> m = Stream.of(values())
							    .collect(groupingBy(t -> t.from, //1.
																			() -> new EnumMap<>(Phase.class),//2.
		                     toMap(t -> t.to, //3. key-mapper
                               t -> t,  //4. value-mapper
                               (x, y) -> y, //5. merge-function
                               () -> new EnumMap<>(Phase.class)))); //6.

        public static Transition from (Phase from, Phase to) {
            return m.get(from).get(to);
        }
    }
```

- 기존에 2차원 배열을 사용하던것을 EnumMap 으로 사용하기 위해 java.util.stream 패키지의 Collectors의 메소드 중 groupingBy와 toMap을 사용하였다.
- groupingBy는 하나의 Key에 여러개의 Value를 가지는 Map을 반환한다.
- toMap은 하나의 Key에 하나의 Value를 가지는 Map을 반환한다.
1. groupingBy에서 Key 값을 전이(Transition)를 이전 상태(from phase)을 기준으로 바깥쪽 Map을 묶는다. (바깥 맵의 키)
2. 바깥 Map의 구현체(value의 자료형)
3. toMap에서 이후 상태(to phase)를 기준으로 안쪽 Map을 묶는다. (바깥 Map의 Value(Map으로), 안쪽 Map의 Key)
4. 안쪽 Map의 value
5. **(x,y) -> y** 부분은 mergeFunction 으로 만약 Key 값이 같은게 존재할때 Value를 기존값(x)으로 할지 새로운값(y)으로 갱신할지 정하는 부분이다. 이번 예제에선 중복되는 Key값이 존재하지 않으므로 실제로는 사용되지 않는다.
6. 안쪽 Map의 구현체(value의 자료형)

Stream Collect 관련 함수
toMap(Function<T, K> keyMapper, Function<T, U> valueMapper)
toMap : T를 K와 U로 매핑해서 K를 키로, U를 값으로 Map에 저장함을 뜻한다.
toSet: : collect로 HashSet 객체를 만든다. → 인자에 toSet()만 넣어주면 Set 자료형으로 만든다.

- 위 코드에 새로운 상태(Phase)가 추가되더라도 상태와 전이 목록에 각각 추가해주면 된다.

## ✏️핵심정리

- 배열의 인덱스를 얻을 때 ordinal을 쓰지 말고 EnumMap 사용하라.
- 다차원 관계는 EnumMap<..., EnumMap<...>>으로 표현하라.

# 아이템38. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

- 열거 타입은 타입 안전 열거 패턴보다 우수하다.
- 단점: 타입 안전 열거 패턴은 확장할 수 있으나, 열거 타입은 확장이 불가능하다.

## 👎🏻열거 타입을 확장하는 것은 지양해야 한다.

- 확장한 타입의 원소는 기반 타입의 원소로 취급하지만 그 반대는 성립하지 않는다.
- 기반타입과 확장된 타입들의 원소 모두를 순회할 방법이 마땅치 않다.
- 확장성을 높이려면 고려할 요소가 늘어나 설계와 구현이 복잡하다.
- 열거 타입 확장이 어울리는 쓰임의 예 : 연산코드(opcode)

## 📌확장방법 : 열거 타입이 임의의 인터페이스를 구현할 수 있다.

### 1. 연산 코드용 인터페이스를 정의하고 열거 타입이 이 인터페이스를 구현하게 한다.

- 이때 열거 타입이 그 인터페이스의 표준 구현체 역할을 한다.

```java
public interface Operation {
    double apply(double x, double y);
}
```

```java
public enum BasicOperation implements Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override public String toString() {
        return symbol;
    }
}
```

- 열거 타입인 BasicOperation 는 확장 불가, 인터페이스인 Operation는 확장 가능
- 이 인터페이스를 연산의 타입으로 사용하면 된다.
- Operation을 구현한 또 다른 열거 타입을 정의해 기본 타입인 BasicOperation 을 대체할 수 있다.
- 지수연산(EXP) , 나머지연산(REMAINDER) 추가 예시

    ```java
    public enum ExtendedOperation implements Operation {
        EXP("^") {
            public double apply(double x, double y) {
                return Math.pow(x, y);
            }
        },
        REMAINDER("%") {
            public double apply(double x, double y) {
                return x % y;
            }
        };
        private final String symbol;
        ExtendedOperation(String symbol) {
            this.symbol = symbol;
        }
        @Override public String toString() {
            return symbol;
        }
    }
    ```

- 새로 작성한 연산은 기존 연산을 쓰던 곳이면 어디든 쓸 수 있다. (Operation 만 impl하면 된다)

### 2. 기본 열거 타입 대신 확장된 열거 타입을 넘겨 확장된 열거 타입의 원소 모두를 사용할 수 있다.

```java
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
		//class리터럴을 넘겨 확장된 연산들이 무엇인지 알려준다. (class리터럴-> 한정적타입토큰)
    test(ExtendedOperation.class, x, y);
}
```

```java
// 열거 타입의 Class 객체를 이용해 확장된 열거 타입의 모든 원소를 사용하는 예
private static <T extends Enum<T> & Operation> void test(
  Class<T> opEnumType, double x, double y) {
  for (Operation op : opEnumType.getEnumConstants())
    System.out.printf("%f %s %f = %f%n",
                      x, op, y, op.apply(x, y));
}
```

- T extends Enum<T> & Operation : Class 객체가 열거타입인 동시에 Operation의 하위타입
→ 열거타입이어야 원소를 순회할 수 있고, Operation이어야 원소가 뜻하는 연산을 수행한다.

### 3. Class 객체 대신 한정적 와일드카드 타입인 Collection<? extends Operation>을 넘긴다.

- 컬렉션 인스턴스를 이용해 확장된 열거 타입의 모든 원소를 사용한다.

```java
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(Arrays.asList(ExtendedOperation.values()), x, y);
}
```

```java
private static void test(Collection<? extends Operation> opSet,
                         double x, double y) {
    for (Operation op : opSet)
        System.out.printf("%f %s %f = %f%n",
                x, op, y, op.apply(x, y));
}
```

- 코드가 덜 복잡하다.
- test메서드가 유연해진다 → 여러 구현 타입의 연산을 조합해 호출할 수 있다.

## 👎🏻인터페이스를 이용해 확장 가능한 열거타입 방법의 문제점

- 열거 타입끼리 구현을 상속할 수 없다.
- 디폴트 구현(의존하는 부분없을때)을 이용해 인터페이스에 추가하는 방법을 사용할 수 없다.
→ Operation에는 연산기호를 저장하고 찾는로직이 BasicOperation과 ExtendedOperation 모두에 들어가야 하기 때문
- 이 경우 공유하는 기능이 많다면 그 부분을 별도의 도우미 클래스나 정적 도우미 메서드로 분리하는 방식으로 코드 중복을 없애야 한다.

## ✏️핵심정리

- 열거 타입 자체는 확장할 수 없지만, 인터페이스와 그 인터페이스를 구현하는 기본 열거 타입을 함께 사용해 같은 효과를 낼 수 있다.
- 클라이언트는 인터페이스를 구현해 자신만의 열거타입을 만들 수 있다.
- API가 (기본 열거 타입을 직접 명시하지 않고) 인터페이스 기반으로 작성되었다면 기본 열거 타입의 인스턴스가 쓰이는 모든 곳을 새로 확장한 열거 타입의 인스턴스로 대체해 사용할 수 있다.

# 아이템39. 명명 패턴보다 애너테이션을 사용하라

명명패턴 ex) JUnit v3까지 테스트 메서드 이름을 test로 시작.

## 👎🏻명명패턴의 단점

- 오타가 나면 안된다. ex) tset~ 라고 오타내면 무시
- 올바른 프로그램 요소에서만 사용되라라 보증할 방법이 없다. 
ex) 메서드가 아닌 클래스를 testClassName하면 무시
- 프로그램 요소를 매개변수로 전달할 마땅한 방법이 없다.

## 명명패턴의 단점 해결: 애너테이션

- Test라는 이름의 애너테이션 정의 과정: 자동으로 수행되며 예외발생시 실패 처리

```java
/**
* 테스트 메서드임을 선언하는 애너테이션이다.
* 매개변수 없는 정적메서드 전용이다.
*/
@Retention(RetentionPolicy.RUNTIME) //Test가 런타임에도 유지되어야한다는 표시
																		//생략하면 테스트 도구는 @Test인식불가
@Target(ElementType.METHOD)//@Test가 만드시 메서드 선언에서만 사용되어야 한다.
public @interface Test{
}
```

## 1. 메타 애너테이션(meta-annotation): 애너테이션 선언에 다는 애너테이션

ex) @Retention, @Target

## 2 .마커 애너케이션(marker-annotation): 아무 매개변수 없이 단순히 대상에 마킹한다.

ex) @Test

- Test이름에 오타를 내거나 메서드 선언 외의 프로그램 요소에 달면 컴파일 오류
- 위와같이 주석으로 해당 애너테이션의 설명을 적어둘 수 있지만 주석처럼 동작을 강제 할 수 는 없기 때문에 클래스의 의미에 직접적인 영향을 주지는 않는다.
- 이 애너테이션이 관심 있는 프로그램에게 추가정보를 제공할 뿐이다.

```java
public class Sample {
    @Test public static void m1(){} //성공  
    public static void m2(){} //실행되지 않는다.
    @Test public static void m3(){ //실패
        throw new RuntimeException("실패");
    }
    @Test public void m4(){} //잘못 사용한 예 //static method가 아니다.
}
```

- 대상 코드의 의미는 그대로 둔 채 그 애너테이션 프로세서에서 해당 요구사항을 충족시켜 구현을 해야한다. → **마커 애너테이션을 처리하는 프로그램**

```java
public class RunTests {
  public static void main(String[] args) {
    int tests = 0;
    int passed = 0;
    Class<?> testClass = Class.forName(args[0]);
    for (Method m : testClass.getDeclaredMethods()) {
      if (m.isAnnotationPresent(Test.class)) {//실행할 메서드를 찾아주는 메서드
        test++;
        try {
            m.invoke(null);
            passed++;
        } catch (InvocationTargetException wrappedExc) {//테스트 예외발생-> @Test 잘못사용
            Throwable exc = wrappedExc.getCause();//실패정보 추출
            System.out.println(m + " 실패: " + exc);
        } catch (Exception e) {
            System.out.println("잘못 사용한 @Test: " + m);
        }
      }
    }    
    System.out.printf("성공: %d, 실패: %d%n", passed, tests-passed);
  }
}
```

## 3. 매개변수 하나를 받는 애너테이션 타입

```java
/**
* 명시한 예외를 던져야만, 성공하는 테스트케이스 애너테이션
*/
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest{
    //한정적 와일드카드를 통해 Throwable을 상속한 모든 타입을 지정		
    Class<? extends Throwable> values(); //-> 모든 예외 타입을 수용한다.
}
```

예제 프로그램: 한정적 타입 토큰 활용(class리터럴을 매개변수로 사용)

```java
public class Sample2 {
    @ExceptionTest(ArithmeticException.class)
    public static void m1() {
        int i = 0;
        i = i / i; //divide by zero. ArithmeticException 예외를 발생시킴 -> 성공
    }
    @ExceptionTest(ArithmeticException.class)
    public static void m2() {
        int[] a = new int[0];
        int i = a[1]; //IndexOutOfBoundsException발생->ArithmeticException가 아니므로 실패
    }
    @ExceptionTest(ArithmeticException.class)
    public static void m3() {} // 아무 Exception도 발생하지 않음 -> 실패
}
```

```java
if(m.isAnnotaionPresent(ExceptionTest.class)) {
    test++;
    try {
        m.invoke(null);
        System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
    } catch (InvocationTargetException wrappedExc) {
      Throwable exc = wrappedExc.getCause();
      Class<? extends Throwable> excType = m.getAnnotation(ExceptionTest.class).value();
																					//ArithmeticException.class가 리턴된다.
			if(!excType.isInstance(exc)) {
	        System.out.printf("테스트 %s 실패: 기대한 예외 %s, 발생한 예외 %s%n"
					, m, excType.getName(), exc);
	    }    
	}
}
```

- 애너테이션 매개변수의 값을 추출하여 테스트 메서드가 올바른 예외를 던지는지 확인한다.
- 테스트 프로그램이 문제 없이 컴파일 되면 애터네이션 매개변수가 가리키는 예외가 올바른 타입
- 해당 예외의 클래스 파일이 컴파일타임에는 존재하고, 런타임에는 존재하지 않는 경우: 테스트러너가 TypeNotPresentException 던진다.

## 4. 배열 매개변수를 받는 애너테이션 타입

- 예외를 여러 개 명시하고 그중 하나가 발생하면 성공하게 만든다.

```java
/**
* 명시한 예외를 던져야만, 성공하는 테스트케이스 애너테이션
*/
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest{
    //한정적 와일드카드를 통해 Throwable을 상속한 모든 타입을 지정
    Class<? extends Throwable>[] values(); 
}
```

예시코드

```java
class Sample {
    @ExceptionTest({IndexOutOfBoundsException.class, NullPointerException.class})
    public static void doublyBad() {
        List<String> list = new ArrayList<>();
        list.addAll(1, null);
    }
}
```

## 5. @Repeatable 메타애너테이션→배열 매개변수 대신 사용(자바8)

하나의 프로그램 요소에 여러번 달 수 있다.

### 주의할 점

1. @Repeatable을 단 애너테이션을 반환하는 '컨테이너 애너테이션'을 하나 더 정의하고, @Repeatable에 이 컨테이너 애너테이션의 class객체를 매개변수로 전달해야 한다.

    ```java
    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.METHOD)
    @Repeatable(ExceptionContainer.class)
    public @interface ExceptionTest {
        Class<? extends Throwable> value();
    }
    ```

2. 컨테이너 애너테이션은 내부 애너테이션 타입의 배열을 반환하는 value메서드를 정의해야 한다.
3. 컨테이너 애너테이션 타입에는 보존정책(@Retention)과 적용대상(@Target)을 명시해야한다.
→ 그렇지 않으면 컴파일 불가

    ```java

    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.METHOD)
    public @interface ExceptionContainer {
        ExceptionTest[] value();
    }
    ```

### 반복가능 애너테이션 적용한 예

```java
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class)
public static void m1() {...}
```

- ❗반복 가능 애너테이션을 하나만 적용한 경우와 여러 개를 적용한 경우를 구분하기 때문에 주의해야한다. → 구분하기 위해 해당 '컨테이너' 애너테이션 타입이 적용된다.
- getAnnotaionByType메서드 : 구분 X
- isAnnotationPresent메서드 : 구분 O
    1.  반복 가능 애너테이션(ExceptionTest)을 여러 번 단 다음 반복 가능 애너테이션이 있는지를isAnnotationPresent()로 검사한다면 false를 반환한다.
    이 경우 컨테이너 애너테이션(ExceptionTestContainer)을 인식하기 때문이다.
    2. 반대로, 컨테이너 애너테이션이 달렸는지 검사하면 한 번만 단 메서드는 무시한다.
- 따로따로 확인해야한다.

    ```java
    ...
    if (method.isAnnotationPresent(MadExceptionTest.class)
                        || method.isAnnotationPresent(MadExceptionContainer.class)) {
    ...
    ```

## ✏️핵심정리

- 반복 가능 애너테이션을 사용해 하나의 프로그램 요소에 같은 애너테이션을 여러번 달 때의 코드 가독성을 높일 수 있다.
- 하지만, 선언과 처리부에 대한 코드양이 늘어나며 처리코드가 복잡해 오류가 날 가능성이 커진다.
- 애너테이션으로 할 수 있는 일을 명명 패턴으로 처리할 이유는 없다.

# 아이템40. @Override 애너테이션을 일관되게 사용하라

- 메서드 선언에만 달 수 있다.
- 상위 타입의 메서드를 재정의했음을 뜻한다.
- 일관된 사용 → 버그예방
- Overriding을 Overloading로 잘못 작성하면 컴파일러가 바로 알려준다.
- Java 8 부터 Default 메서드의 사용이 가능해 지면서, 클래스뿐 아니라 인터페이스의 메서드를 재정의 할 때도 사용할 수 있다.
- 구현하려는 인터페이스에 Default 메서드가 없음을 안다면 @Override를 생략해 코드를 조금 깔끔히 유지해도 좋다.
- 웬만하면 추상클래스나 인터페이스에서는 상위 클래스나 상위 인터페이스를 재정의하는 모든 메서드에 @Override를 다는것이 좋다.(실수 했을 때 컴파일러가 알려준다.)

## @Override 달지 않아도 되는 예외

구체 클래스에서 상위 클래스의 추상메서드를 재정의할 경우

→ 아직 구현하지 않은 추상 메서드가 있다면 컴파일러가 알려준다.

# 아이템41. 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라

## 마커인터페이스(maker Interface)

아무 메서드도 담고 있지 않고, 단지 자신을 구현하는 클래스가 특정 속성을 가짐을 표시해주는 인터페이스

ex) Serializable : 자신을 구현한 클래스의 인스턴스는 ObjectOutputStream을 통해 write할 수 있다고 알려준다.

## 마커인터페이스의 장점

1. 마커 인터페이스는 이를 구현한 클래스의 인스턴스들을 구분하는 타입으로 쓸 수 있다.
    - 마커애너테이션은 불가
    - 인터페이스가 어엿한 타입이므로 런타임에야 발견될 오류를 **컴파일타임에** 잡을 수 있다.
    ↔ 애너테이션 : 직렬화 할 수 없는 객체를 넘겨도 **런타임에야 발견 가능함**
    - ex) 직렬화는 Serializable 마커 인터페이스를 보고 그 대상이 직렬화 가능 타입인지 확인한다.
2. 적용 대상을 더 정밀하게 지정할 수 있다.
    - 적용 대상(@Target)을 **ElementType.TYPE**으로 선언한 애너테이션은 모든타입(클래스, 인터페이스, 열거타입, 애너테이션)에 달 수 있어 **세밀하게 제한하지 못한다.**
    - **마킹하고 싶은 특정 클래스에서만 마커인터페이스를 구현하여 적용할 수 있다.
    →** 마킹된 타입은 자동으로 그 인터페이스의 하위타입임을 보장한다.
3. 객체의 특정 부분을 불변식으로 규정하거나, 그 타입의 인스턴스는 다른 클래스의 특정 메서드가 처리할 수 있다는 사실을 명시하는 용도로 사용할 수 있다
(Serializeable 인터페이스가 ObjectOutputStream이 처리할 수 있는 인스턴스이듯이)

## 마커애너테이션의 장점

1. 거대한 애너테이션 시스템의 지원을 받는다. → 일관성 유지
2. 클래스와 인터페이스 외의 프로그램 요소(**모듈, 패키지, 필드 지역변수** 등)에 마킹해야할 때 인터페이스는 사용하지 못하니 애너테이션을 사용해야 한다.
→ 클래스와 인터페이스만이 인터페이스를 구현하거나 확장할 수 있기 때문

## ✏️핵심정리

1. 새로 추가하는 메서드 없이 단지 타입정의가 목적이라면 마커 인터페이스를 선택하자.
2. 마커를 클래스나 인터페이스에 적용할 때 **마킹된 객체를 매개변수로 받는 메서드를 작성**한다면 **마커 인터페이스**를 써야한다.
→ 마커인터페이스를 메서드의 매개변수 타입으로 사용하여 컴파일 타임에 오류를 잡을 수 있다.
3. @Target(ElementType.TYPE)인 마커 애너테이션을 작성하고 있다면,마커 애너테이션을 정말 사용해야 하는지? 마커 인터페이스를 사용할 수 있는지 생각해보고 웬만하면 마커 인터페이스를 사용하도록 하자