# 2장. 객체 생성과 파괴

1. 객체를 만들어야 할 때 만들지 말아야 할 때를 구분하는 법.
2. 올바른 객체 생성 방법과 불필요한 생성을 피하는 방법.
3. 제때 파괴됨을 보장하고 파괴 전에 수행해야 할 정리 작업을 관리하는 요령

---

[https://www.notion.so/2-83a9e156bf4d41d2bc6c04123f187160](https://www.notion.so/2-83a9e156bf4d41d2bc6c04123f187160)

# 아이템1. 생성자 대신 정적 팩터리 메서드를 고려하라.

클래스는 생성자와 별도로 **정적 팩터리 메서드(static factory method)**를 제공할 수 있다.

: 클래스의 인스턴스를 반환하는 단순한 정적 메서드 

ex)

```java
public static Boolean valueOf(boolean b){
	return b ? Boolean.TRUE : Boolean.FALSE;
}
```

### 정적 메소드가 생성자보다 좋은 장점 다섯가지

1. 이름을 가질 수 있다. = 반환될 객체의 특정을 쉽게 묘사할 수 있다.
2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다. = 플라이웨이트 패턴과 비슷한 기법

플라이웨이트 패턴은, 객체의 내부에서 참조하는 객체를 직접 만드는 것이 아니라, 없다면 만들고, 
만들어져 있다면 객체를 공유하는 식으로 객체를 구성하는 방법

 3.  반환 타입의 하위 타입 객체를 반환할 수 있는 능력 제공 = 엄청난 유연성 = 자바 8부터는 인터페이스가 정적 메서드를 가질 수 없다는 제한이 풀렸다.

 4.  입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

      = 반환타입의 하위 타입이만 하면 어떤 클래스의 객체를 반환하든 상관이 없다.

 5.   정적 패터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

      = 서비스 제공자 프레임워크를 만드는 근간이 된다.

**서비스 제공자 프레임워크**

: 제공자들이 구성하는 서비스 구현체다, 그리고 이 구현체들을 클라이언트에게 제공하는 역할을 프레임워크가 통제하여 클라이언트를 구현체로부터 분리해준다. (클라이언트는 제공자들을 몰라도 이용할 수 있다.)

ex) JDBD(Java Database Connectivity) 의 제공자 : mysql, oracle

**서비스 제공자 프레임워크의 3가지 핵심 컴포넌트** 

1. 구현체의 동작을 정의하는 **서비스 인터페이스(service interface)** 
2. 제공자가 구현체를 등록할 때 사용하는 **제공자 등록 api(provider registration API)**
3. 클라이언트가 서비스의 인스턴스를 얻을 때 사용하는 **서비스 접근 api (service access API)**

+ 종종 **서비스 제공자 인터페이스(service provider interface)**라는 네번째 컴포넌트가 쓰이는데 이 컴포넌트는 서비스 인스턴스를 생성하는 팩터리 객체를 설명해준다.

```
Ex. < JDBC >
서비스 인터페이스 : Connection
제공자 등록 API : DriverManager.registerDrive()
서비스 접근 API : DriverManager.getConnection()
서비스 제공자 인터페이스 : Driver
```

### 단점 다섯가지

1. 상속을 하려면 public 이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다. = 컬렉션 프레임워크의 유틸리티 구현클래스들을 상속할 수 없다.
2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.  = 문서에 설명이 잘없다.

[정적팩터리 메서드에 흔히 사용하는 명명방식들 ](https://www.notion.so/170e56bed5194eed844da832c79cc866)

# 아이템2. 생성자에 매개변수가 많다면 빌더를 고려하라.

정적 팩터리와 생성자에는 똑같은 제약이 하나 있다.

선택적 매개변수가 많을 때 적절히 대응하기 어렵다는 점이다.

JAVA에서 정적 팩터리 메서드란 private 생성자를 통해 new를 통한 객체 생성을 감추고 static 메서드를 통해 객체 생성을 캡슐화하는 디자인 패턴을 말합니다.

**점층적 생정자 패턴** 

점층 적 생성자 패턴(변수가 하나씩 많아지는 패턴)은 물론, 매개변수 개수가 많아지면 코드를 작성하거나 읽기 어렵다.

**자바빈즈 패턴(JavaBeans pattern)**

매개변수가 없는 생성자로 객체를 만든 후, 세터 메서드를 호출해 원하는 매개변수 값을 설정하는 방식

→ 객체 하나를 만들려면 메서드 여러개를 호출해야 하고, 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태에 놓이게 된다. 자바 빈즈 패턴에서는 클래스를 불변으로 만들 수 없으며 스레드 안전성을 얻으려면 프로그래머가 추가 작업을 해줘야 한다.

⇒ 두 패턴을 대체할 대안 

### 빌더 패턴(Builder pattern)

필수 매개변수만으로 생성자(혹은 정적 팩터리)를 호출해 빌더 객체를 얻는다.

빌더는 생성할 클래스 안에 정적 멤버 클래스로 만들어두는게 보통이다.

```java
// 코드 2-3 빌더 패턴 - 점층적 생성자 패턴과 자바빈즈 패턴의 장점만 취했다. (17~18쪽)
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // 필수 매개변수
        private final int servingSize;
        private final int servings;

        // 선택 매개변수 - 기본값으로 초기화한다.
        private int calories      = 0;
        private int fat           = 0;
        private int sodium        = 0;
        private int carbohydrate  = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings    = servings;
        }

        public Builder calories(int val)
        { calories = val;      return this; }
        public Builder fat(int val)
        { fat = val;           return this; }
        public Builder sodium(int val)
        { sodium = val;        return this; }
        public Builder carbohydrate(int val)
        { carbohydrate = val;  return this; }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize  = builder.servingSize;
        servings     = builder.servings;
        calories     = builder.calories;
        fat          = builder.fat;
        sodium       = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }

    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
                .calories(100).sodium(35).carbohydrate(27).build();
    }
}
```

불변 : 어떠한 변경도 허용하지 않는다.
불변식 : 프로그램이 실행되는 동안, 혹은 정해진 기간 동안 반드시 만족해야 하는 조건을 말한다. 
다시 말해 변경을 허용할 수는 있으나 주어진 쪼건 내에서만 허용한다는 뜻이다.

장점 

- 빌더의 세터 메서드들은 빌더 자신을 반환하기 때문에 연쇄적으로 호출할 수 있다.  이러한 방식을 메서드 호출이 흐르듯 연결된다는 뜻으로 플루언트 API(fluent API) 또는 메서드 연쇄(method chaining)라 한다.
- 빌더 패턴은 상당히 유연하다. 빌더 하나로 여러객체를 순회하면서 만들 수 있고, 빌더에 넘기는 매개변수에 따라 다른객체를 만들 수도 있다.

단점

- 객체를 만들려면, 그에 앞서 빌더부터 만들어야 한다.
- 빌더 생성비용이 크지는 않지만 성능에 민감한 상황에서는 문제가 될 수 있다. 매개변수가 4개 이상은 되어야 값어치를 한다.

> 결론 : 생성자나 정적 팩터리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는게 더 낫다.  매개변수 중 다수가 필수가 아니거나 같은 타입이면 특히 더 그렇다. 
빌더는 점층적 생성자보다 클라이언트 코드를 읽고 쓰기가 훨씬 간결하고, 자바빈즈보다 훨씬 안전하다.

## 아이템 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라

싱글턴(singleton) : 인스턴스를 오직 하나만 생성할 수 있는 클래스

ex) 함수와 같은 무상태 객체, 설계상 유일해야하는 시스템 컴포넌트

싱글턴을 만드는 방법

1. 생성자는 private으로 숨기고, public static 멤버가 final 필드인 방식

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();

    private Elvis() { }

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }

    // 이 메서드는 보통 클래스 바깥(다른 클래스)에 작성해야 한다!
    public static void main(String[] args) {
        Elvis elvis = Elvis.INSTANCE;
        elvis.leaveTheBuilding();
    }
```

 2.  생성자는 private으로 숨기고, 정적 팩터리 메서드를 public static 멤버로 제공

```java
// 코드 3-2 정적 팩터리 방식의 싱글턴 (24쪽)
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { }
    public static Elvis getInstance() { return INSTANCE; }

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }

    // 이 메서드는 보통 클래스 바깥(다른 클래스)에 작성해야 한다!
    public static void main(String[] args) {
        Elvis elvis = Elvis.getInstance();
        elvis.leaveTheBuilding();
    }
}
```

 3.  **원소가 하나인 열거타입을 선언하는 것 ⇒ 가장 좋은 방법**

```java
public enum Elvis {
    INSTANCE;

    public void leaveTheBuilding() {
        System.out.println("하이.");
    }

    // 이 메서드는 보통 클래스 바깥(다른 클래스)에 작성해야 한다!
    public static void main(String[] args) {
        Elvis elvis = Elvis.INSTANCE;
        elvis.leaveTheBuilding();
    }
}
```

간결하고, 추가 노력 없이 직렬화 가능. 제2의 인스턴스가 생기는 일을 완벽히 만들어준다.

단, 만들려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다. 

(열거 타입이 다른 인터페이스를 구현하도록 선언할 수는 있다.)

## 아이템 4. 인스턴스화를 막으려거든 private 생정자를 사용하라

- 추상 클래스를 만드는 것으로는 인스턴스화를 막을 수 없다.

**인스턴스**는 추상화 개념 또는 클래스 객체, 컴퓨터 프로세스 등과 같은 템플릿이 실제 구현된 것이다. 
**인스턴스화**는 클래스로 부터 객체를 만드는 과정 ex) Car a = new Car();

인스턴스화를 막는 방법은 아주 간단하다. 컴파일러가 기본 생성자를 만드는 경우는 오직 명시된 생성자가 없을 때 뿐이니 private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있다.

이 방식은 상속을 불가능하게 하는 효과도 있다.

private으로 선언했으니 하위 클래스가 상위 클래스의 생성자에 접근할 길이 막혀버린다.

## 아이템 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다.

⇒ 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식을 사용하면 된다. (의존 객체 주입의 한 형태)

### 팩터리 메서드 패턴

의존 객체 주입 패턴의 쓸만한 변형으로, 생성자에 자원 팩터리를 넘겨주는 방식

팩터리란, 호출할 때 마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체를 말한다.

자바 8에서 소개한 Supplier<T> 인터페이스가 팩터리를 표현한 완벽한 예다

```java
Mosaic create<Supplier<? extends Tile> tileFactory){..}
```

클라이언트가 제공한 팩터리가 생성한 타일들로 구성된 모자이크를 만드는 메서드.

의존 객체 주입이 유연성과 테스트 용이성을 개선해주긴 하지만, 의존성이 수 천개나 되는 큰 프로젝트에서는 코드를 어지럽게 만들기도 한다.

⇒ 해결 책 : 의존 객체 주입 프레임워크 ex) 대거(Dagger),주스(Guice),스프링

> 결론 : 클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면  싱글턴과 정적 유틸리티 클래스는 사용하지 않는 것이 좋다. 이 자원들을 클래스가 직접 만들게 해서도 안된다.
대신 필요한 자원을(혹은 그 자원을 만들어주는 팩터리를) 생성자에 (혹은 정적 팩터리나 빌더에) 넘겨주자.
의존 객체 주입이라 하는 이 기법은 클래스의 유연성, 재 사용성, 테스트 용이성을 기막히게 개선해준다.

## 아이템 6. 불필요한 객체 생성을 피하라

똑같은 기능의 객체를 매번 생성하기보다는 객체 하나를 재사용하는 편이 나을 때가 많다. 재사용은 빠르고 세련되다.

### * String

```java
String s = new String(""); //실행될 때 마다 String 인스턴스를 새로만드는 안좋은 예
```

```java
String s = ""; //하나의 string 인스턴스를 사용. 나아가 같은 가상 머신안에서 똑같은 문자열 리터럴을 
               //사용하는 모든 코드가 같은 객체를 재 사용함이 보장된다.
```

### * Boolean

 Boolean(String) 생성자대신 Boolean.valueOf(String) 팩터리 메서드를 사용하는 것이 좋다.

### * String.mathes

String.mathes는 정규표현식으로 문자열 형태를 확인하는 가장 쉬운 방법이지만, 성능이 중요한 상황에서 반복해 사용하기엔 적합하지 않다. 

: 이 메서드가 내부에서 만드는 정규표현식용 pattern 인스턴스는, 한 번 쓰고 버려져서 곧바로 가지비 컬렉션 대상이 된다. Pattern은 입력받은 정귶현식에 해당하는 유한 상태 머신을 만들기때문에 인스턴스 생성 비용이 높다.

초기화(정적 초기화) 과정에서 직접 생성해 캐시해두고, 나중에 쓰이는 메서드가 호출될 때마다 이 인스턴스를 재사용한다.

```java
// 코드 6-2 private으로 선엄함으로써, 값비싼 객체를 재사용해 성능을 개선한다.
    private static final Pattern ROMAN = Pattern.compile(
            "^(?=.)M*(C[MD]|D?C{0,3})"
                    + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeralFast(String s) {
        return ROMAN.matcher(s).matches();
    }
```

### * 박싱된 기본타입

박싱된 기본타입보다는 기본타입을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의하자.

## 아이템 7. 다 쓴 객체 참조를 해제하라.

가비지 컬렉터가 회수가지 못하는 다 쓴 객체 참조를 해제하라.

### 스택(Stack)

스택은 자기 메모리를 직접 관리한다. 

스택이 켜졌다가 줄어들었을 때(pop) 스택에서 꺼내진 객체들은 가비지 컬렉터가 회수하지 않는다. 프로그램에서 그 객체들을 더 이상 사용하지 않더라도 말이다. 

이 스택이 그 객체들, 다 쓴 참조(obselete reference)를 여전히 가지고 있기 때문이다.

**다 쓴 참조**란, 문자 그대로 앞으로 다시 쓰지 않을 참조를 듯한다. (활성 영역 밖의 참조)

**활성 영역**이란, 인덱스가 size보다 작은 원소들을 말한다.

가비지 컬렉션 언어에서는 (의도치 않게 객체를 살려두는) 메모리 누스를 찾기가 아주 까다롭다. 객체 참조 하나를 살려두면 가비지 컬렉터는 그 개체뿐 아니라 그 객체가 참조하는 모든 객체를 회수해가지 못한다.

⇒ 해당 참조는 다 썻을 때 null 처리를 해서 참조해제를 해줘야 한다.

```java
    // 코드 7-2 제대로 구현한 pop 메서드 (37쪽)
    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
       Object result = elements[--size];
       elements[size] = null; // 다 쓴 참조 해제
       return result;
    }
```

> 일반적으로 자기 메모리를 직접 관리하는 클래스라면 프로그래머는 항시 메모리 누수에 주의해야 한다. 원소를 다 사용한 즉시 그 원소가 참조한 객체들을 다 null 처리를 해줘야 한다.

### 캐시

캐시 역시 메모리 누수를 일으키는 주범이다.

캐시 외부에서 키(key)를 참조하는 동안만(값이 아니다) 엔트리가 살아있는 캐시가 필요한 상황이라면 **WeakHashMap**을 사용해 캐시를 만들자.

캐시에 새 엔트리를 추가할 때 부수 작업으로 수행하는 방법이 있다.

LinkedHashMap은 remove EldesEntry 메서드를 써서 후자의 방식으로 처리한다.

### 리스너 혹은 콜백

클라이언트가 콜백을 등록만하고 명확히 해지하지 않는다면, 뭔가 조치를 해주지 않는 한 콜백은 계속 쌓여갈 것이다. 이럴 때 콜백은 약한 참조(weak reference)로 저장하면 가비지 컬렉터가 즉시 수거해간다.

예를 들어 WeakHashMap에 키로 저장하면 된다.

## 아이템 8. finalize와 cleaner 사용을 피하라

finalize은 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요하다.

cleaner은 finalizer보다는 덜 위험하지만, 여전히 예측할 수 없고, 느리고, 일반적으로는 불필요하다.

둘은 즉시 수행된다는 보장도 없다.

C++의 파괴자 개념에 이것들을 쓰는 개발자들이 있는데 안쓰는게 좋다.

자바에서는 try-with-resources와 try-finally를 사용해 해결한다. 

## 아이템 9. try-finally 보다는 try-with-resources를 사용하라.

자바 라이브러리 에는 close 메서드를 호출해 직접 닫아줘야 하는 자원이 많다.

ex) InputStream, OutputStream, java.sql.Connection 등

```java
public class TopLine {
    // 코드 9-1 try-finally - 더 이상 자원을 회수하는 최선의 방책이 아니다! (47쪽)
    static String firstLineOfFile(String path) throws IOException {
        BufferedReader br = new BufferedReader(new FileReader(path));
        try {
            return br.readLine();
        } finally {
            br.close();
        }
    }
```

try-finally는 readLine에서 문제가 생긴다면 예외를 던지고, 같은 이유로 close메서드도 실패할 것이다.

이런 상황이라면 두 번째 예외가 첫 번째 예외를 완전히 집어 삼켜 버린다.

대체. try-catch-resources - try 괄호 안에 리소스가 들어감

```java
// 코드 9-5 try-with-resources를 catch 절과 함께 쓰는 모습 (49쪽)
    static String firstLineOfFile(String path, String defaultVal) {
        try (BufferedReader br = new BufferedReader(
                new FileReader(path))) {
            return br.readLine();
        } catch (IOException e) {
            return defaultVal;
        }
    }
```

> 꼭 회수해야 하는 자원을 다룰 때는 try-finally 말고, try-wirh-resources를 사용하자.
예외는 없다. 코드는 더 짧고 분명해지고, 만들어지는 예외 정보도 훨씬 유용하다. try-finally로 작성하면 실용적이지 못한 만큼 코드가 지저분해지는 경우라도,
try-with-resources로는 정확하고 쉽게 자원을 회수할 수 있다.