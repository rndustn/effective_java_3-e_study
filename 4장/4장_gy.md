# 4장. 클래스와 인터페이스

[4장. 클래스와 인터페이스]() 

- [ ]  클래스와 인터페이스를 쓰기 편하고, 견고하며, 유연하게 만드는 방법을 다룬다.

# 아이템15. 클래스와 멤버의 접근 권한을 최소화하라.

## 📌정보은닉이란?

클래스 내부 데이터와 내부 구현 정보(메서드)를 하나의 클래스로 묶고 외부 컴포넌트로 부터 완벽히 숨겨 구현과 API를 완벽히 분리한다. 
오직 API를 통해서만 다른 컴포넌트와 소통하며 서로의 내부 동작방식에는 전혀 개의치 않는다.
→ 객체에 직접적인 접근을 막고 외부에서 내부의 정보에 직접접근하거나 변경할 수 없고, 객체가 제공하는 필드와 메소드를 통해서만 접근이 가능하다.

## 👍🏻정보은닉(캡슐화)의 장점

1. 시스템 개발속도 향상 : 여러 컴포넌트를 병렬로 개발 할 수 있기 때문이다.
2. 시스템 관리비용 절감 : 각 컴포넌트 더 빨리 파악하여 디버깅 할 수 있고, 다른 컴포넌트로 교체하는 부담도 적기 때문이다.
3. 성능 최적화에 도움 : 최적화 대상 컴포넌트를 정한 후 다른 컴포넌트에 영향없이 해당 컴포넌트만 최적화 할 수 있기 때문이다. 
4. 소프트웨어 재사용성 상승 : 외부에 거의 의존하지 않고 독자적으로 동작가능한 컴포넌트라면 다른 환경에서도 재사용 가능하기 때문이다.
5. 큰 시스템을 제작하는 난이도 감소 : 시스템 전체가 완성되지 않은 상태에서 개별 컴포턴트의 동작을 검증할 수 있기 때문이다.

## ✨정보은닉의 핵심 : 접근제한자의 활용

### 모든 클래스와 멤버의 접근성을 가능한 한 좁혀야 한다.

→ 소프트웨어가 올바로 동작하는 한 항상 낮은 접근 수준을 부여해야 한다는 뜻이다.

### 클래스의 접근제한자

1. 톱레벨 클래스(가장 바깥)와 인터페이스에 부여할 수 있는 접근 수준은 package-private와 public 두 가지이다.
    - **public** 선언 : 공개 API →  하위 호환을 위해 영원히 관리해줘야 한다.
    - **package-private** 선언 : 해당 패키지 안에서만 이용 → 확장, 수정, 교체, 제거에 용이
2. **한 클래스에서만 사용하는** package-private 톱레벨 클래스나 인터페이스는 이를 사용하는 클래스안에 **private static으로 선언한다.**
→ 톱레벨로 두면 같은 패키지의 모든 클래스가 접근할 수 있지만, private static을 중첩시키면 바깥 클래스 하나에서만 접근 할 수 있다.
3. public일 필요가 없는 클래스의 접근 수준을 package-private 톱레벨 클래스로 좁혀야 한다.
→ public 클래스는 그 패키지의 API인 반면, **package-private 톱레벨 클래스는 내부구현이다.**

### 멤버(필드, 메서드, 중첩클래스, 중첩인터페이스) 접근제한자의 종류

1. private : 멤버를 선언한 톱레벨 클래스에서만 접근할 수 있다.
2. package-private : 멤버가 소속된 패키지 안의 모든 클래스에서 접근할 수 있다.
접근 제한자를 명시하지 않았을 때 적용되는 패키지 접근수준이다.
package-private은 default 제어자랑 같은 의미로 사용된다.
3. protected : package-private의 접근 범위를 포함하며, 이 멤버를 선언한 클래스의 하위 클래서에도 접근할 수 있다.
4. public : 모든 곳에서 접근 할 수 있다.

## ❗접근제한자 구현시 주의할 점

private와 package-private 멤버는 모두 해당 클래스의 구현에 해당하므로 공개 API에 영향을 주지 않는다. 단, Serializable을 구현한 클래스에서는 의도치 않게 공개 API가 될 수 있다.

1. 클래스의 공개 API를 세심히 설계 후, 그 외의 모든 멤버는 private로 선언한다.
2. 같은 패키지의 다른 클래스가 접귾야하는 멤버는 package-private로 풀어준다.
3. public 클래스에서는 멤버의 접근 수준을 package-private에서 protected로 변경한 순간 접근가능 대상 범위가 엄청나게 넓어진다. →적을수록 좋다.

### 멤버 접근성을 좁히지 못하게 방해하는 제약

상위클래스의 메서드를 재정의할 때는 그 접근 수준을 상위 클래스에서보다 좁게 설정할 수 없다.

→ 리스코프 치환 원칙( 상위 클래스의 인스턴스는 하위 클래스의 인스턴스로 대체해 사용할 수 있어야 한다.)

단, 클래스가 인터페이스를 구현하는 경우에는 클래스의 메서드를 모두 public으로 선언해야 한다.

단지 테스트 목적으로 접근 범위를 넓히더라도 public 클래스의 private 멤버를 package-private(default)까지 풀어주는 것은 허용할 수 있지만, 그 이상(공개API가 되는)은 안 된다.

 

### public 클래스의 인스턴스 필드는 되도록 public이 아니어야 한다.

- 필드가 가변 객체(컬렉션이나 배열)를 참조하거나, final이 아닌 인스턴스 필드를 public으로 선언하면 그 필드를 담을 수 있는 값을 제한할 힘을 잃게된다. → 불변식을 보장할 수 없다.
- public 가변 필드를 갖는 클래스는 일반적으로 스레드 안전하지 않다.
- 상수라면 public static final 필드로 공개해도 된다.
→ 이런 필드는 반드시 기본 타입 값이나 불변객체를 참조해야 한다.
→ 가변 객체를 참조한다면 final이 아닌 필드에 적용되는 모든 불이익이 그대로 적용된다.
( 다른 객체를 참조하지는 못하지만, 참조된 객체 자체는 수정될 수 있기 때문 )

### 클래스에서 public static final 배열 필드를 두거나 이 필드를 반환하는 접근자 메서드를 제공해서는 안된다.

- 길이가 0이 아닌 배열은 모두 변경 가능하다.
- 클라이언트에서 배열의 내용을 수정할 수 있게된다.
→ 값이면 수정 불가능하겠지만 배열인 경우 값이 그냥 값이 아닌 주소값이 때문에 변경이 가능하다.

    ```jsx
    public static final Thing[] VALUES = {...};
    ```

🛠️**해결방안**

1. public 배열을 private로 만들고 public 불변리스트를 추가한다.

    ```jsx
    private static final Thing[] PRIVATE_VALUES = {...};
    public static final List<Thing> VALUES = 
    	Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
    ```

2. 배열을 private로 만들고 그 복사본을 반환하는 public 메서드를 추가한다. (방어적 복사)

    ```jsx
    private static final Thing[] PRIVATE_VALUES = {...};
    public static final Thing[] values() {
    	return PRIVATE_VALUES.clone();
    }
    ```

## **Java9의 Module System**

**패키지가 클래스들의 묶음이듯, 모듈은 패키지들의 묶음이다.**

- 자신이 속하는 패키지 중 공개(export)할 것들을 (관례상 [module-info.java](http://module-info.java) 파일에) 선언한다.
- protected 혹은 public 멤버라도 해당 패키지를 공개하지 않았다면 모듈 외부에서 접근할 수 없다.
- 모듈 시스템을 활요하면 클래스를 외부에 공개하지 않으면서도 같은 모듈을 이루는 패키지 사이에서는 자유롭게 공유할 수 있다.

암묵적 접근 수준 : public과 protected 수준과 같으나 모듈 내부로 한정

- 모듈 내부의 public: 모듈 내부의 모든 곳에서 접근할 수 있다.
- required의 public: 모듈에 종속하는 모듈의 모든 패키지 내의 클래스에 접근할 수 있다.
- export public: module-info.java에서 제공하는 모든 public에 접근할 수 있다.

## ✏️핵심 정리

- 프로그램 요소의 접근성은 가능한 한 최소한으로 하라
- 꼭 필요한 것만 골라 최소한의 public API를 설계하라
- public 클래스는 상수용 public static final 필드 외에는 어떠한 public 필드도 가져서는 안 된다.
- public static final 필드가 참조하는 객체가 불변인지 확인하라

# 아이템16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라.

```jsx
class Point{
  public double x;
  public double y;
}
```

- 이런 클래스는 데이터 필드에 직접 접근할 수 있어 캡슐화의 이점을 제공하지 못한다.
- API를 수정하지 않고는 내부 표현을 바꿀 수 없다.
- 불변식을 보장할 수 없다.
- 외부에서 필드에 접근할 때 부수 작업을 수행할 수 없다.
- public 필드를 사용하는 클라이언트가 생겨 내부 표현방식을 마음대로 바꿀 수 없다.

## 접근자와 변경자 메서드를 활용해 데이터를 캡슐화 한다.

→ 필드를 모두 private으로 바꾸고 public 접근자(getter)를 추가한다.

```jsx
class Point{
	private double x;
	private double y;   	
	public Point(double x, double y){
		this.x = x;
		this.y = y;
	}    
	public double getX() { return x; }
	public double getY() { return y; }
    
	public void setX(double x){ this.x = x; }
	public void setY(double y){ this.y = y; }
}
```

패키지 외부에서 접근할 수 있는 클래스라면 접근자를 제공함으로써 **클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 얻을 수 있다.**(getter, setter 혹은 또 다른 메서드를 통해 로직 추가가능)

## package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출한다 해도 문제가 없다.

→ 같은 패키지안에서 특정이유 때문에 사용하던가, 톱레벨 클래스에서만 접근 할 것이기 때문이다.

수정범위

- package-private : 패키지 바깥 코드를 손대지 않고 데이터 표현 방식을 바꿀 수 있다.
- private 중첩 : 이 클래스를 포함하는 외부 클래스까지로 제한한다.

### public 클래스의 불변필드

직접 노출할 때의 단점이 조금 줄어들지만 API를 변경하지 않고는 표현 방식변경이 불가하고 필드를 읽을 때 부수작업 수행불가의 단점이 여전하다. 단, 불변식은 보장할 수 있게 된다.

## ✏️핵심 정리

- public 클래스는 절대 가변 필드를 직접 노출해서는 안 된다.
- 불편 필드라면 노출해도 덜 위험하지만 완전히 안심할 수 없다.
- package-private 클래스나 private 중첩 클래스에서는 종종(불변이든 가변이든) 필드를 노출하는 편이 나을 때도 있다.

# 아이템17. 변경 가능성을 최소화하라.

## 📌불변 클래스란?

- **인스턴스의 내부 값을 수정할 수 없는 클래스**
- 자바의 불변 클래스 : String, Boxing Class, BigInteger, BigDecimal
- 불변 클래스는 가변 클래스보다 설계하고 사용하기 쉬우며, 오류가 생길 여지도 적도 훨씬 안전하다.

## ✔️불변 클래스 만들기 위한 다섯 가지 규칙

1. 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다. (Setter)
2. 클래스를 확장할 수 없도록 한다.
3. 모든 필드를 final로 선언한다.
    - 설계자의 의도 명확히 드러내는 방법이다.
    - 새로 생성된 인스턴스를 동기화 없이 다른 스레드로 건네도 문제없이 동작하게끔 보장한다.
4. 모든 필드를 private으로 선언한다.
    - 필드가 참조하는 가변 객체를 클라이언트가 직접 수정하는 일을 막아준다.
    - 기본 타입 필드나 불변 객체를 참조하는 필드를 public final로만 선언해도 불변 객체가 되지만, 다음 릴리즈에서 내부표현을 변경하지 못하므로 권하지 않음
5. 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.
    - 클라이언트에서 인스턴스 내에 가변 객체의 참조를 얻을 수 없게 해야한다.
    - 접근자 메서드가 그 필드를 그대로 반환하면 안된다. ( Collection에 대한 getter를 제공하면 안된다.)
    - 생성자, 접근자, readObject 메서드 모두에서 방어적 복사를 수행해야 한다.

## 함수형 프로그래밍

- 피연산자에 함수를 적용해 그 결과를 반환한다.
- 피연산자에 대한 연산을 하지만 피연산자 자체의 값은 그대로 변하지 않는다.
↔절차적 혹은 명령형 프로그래밍에서 피연산자인 자신을 수정해 자신의 상태가 변한다.
- 메서드 이름으로 (add같은) 동사 대신 (plus같은) 전치사를 사용하여 해당 메서드가 객체의 값을 변경하지 않는다는 사실을 강조한다.
- 코드에서 불변이 되는 영역의 비율이 높아진다.

## 👍🏻불변 객체의 장점

1. 생성된 시점의 상태를 파괴될 때까지 그대로 간직한다.
2. 모든 생성자가 클래스 불변식을 보장한다면 그 클래스를 사용하는 프로그래머가 다른 노력을 들이지 않더라도 영원히 불변으로 남는다.
3. 불변 객체는 근본적으로 스레드 안전하여 따로 동기화할 필요가 없다.
4. 불변 객체는 안심하고 공유할 수 있다. → 최대한 재활용해라
    - 자주 쓰이는 값들을 상수(public static final)로 제공한다.
    - 불변 클래스는 자주 사용되는 인스턴스를 캐싱하여 인스턴스 중복생성을 막아주는 정적 팩터리를 제공할 수 있다. (박싱된 기본 타입 클래스와 BigInteger)
    → 새로운 클래스 설계시 public 생성자 대신 정적팩터리 만들어두면 클라이언트 수정없이 필요에 따라 캐시 기능을 추가할 수 있다.
5. 불변 객체는 바뀌지 않으므로 방어적 복사가 필요없다.
clone 메서드나 복사 생성자를 제공하지 않는게 좋다.
6. 불변 객체는 자유롭게 공유할 수 있음은 물론, 불변 객체끼리는 내부 데이터를 공유할 수 있다.
7. 객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면 이점이 많다.
    - 구조가 복잡하더라도 불변식을 유지하기 수월하다.
    - Map의 Key
    - Set의 원소
8. 불변 객체는 그 자체로 실패 원자성을 제공한다.
실패 원자성 : 메서드에서 예외가 발생한 후에도 그 객체는 메서드 호출전 상태와 같은 유효한 상태를 가진다.

## 👎🏻불변 객체의 단점

값이 다르면 반드시 독립된 객체로 만들어야 한다. → 값의 가짓수가 많을수록 생성 비용증가

1. 백만 비트짜리 BigInteger에서 비트하나만 바꿔야 해도 새로운 인스턴스 생성 (O(N))
2. BigSet은 원하는 비트 하나만 상수 시간안에 바꿔주는 메서드를 제공한다. (O(1))
BigInteger와 같이 임의 길이의 비트 순열을 표현하지만, 가변이다. 

### 불변객체 단점에 대처하는 두가지 방법

1. 흔히 쓰일 다단계 연산들을 예측하여 기본 기능으로 제공한다.
→ 각 단계마다 객체를 생성하지 않아도 된다.
    - 다단계 연산 속도를 높여주는 **가변 동반 클래스(companion class)를 package-private**으로 두고 있다.
    - **가변 동반 클래스란?**

        불변 클래스와 거의 동일한 기능을 가지고 있지만 가변적인 클래스를 가변 동반 클래스라고 한다. 가변 동반 클래스는 주로 불변 클래스가 비즈니스 로직 연산 등 시간 복잡도가 높은 연산시 불필요한 클래스 생성을 막기 위해 내부적으로 사용한다.
        대표적으로, 불변 클래스인 String에게 가변 동반 클래스인 StringBuffer가 있다.

2. 예측이 안된다면, 가변 동반 클래스를 public으로 제공하는 게 최선이다.

## 불변 클래스 설계방법

1. **final** 클래스로 선언하여 상속을 막는다.
2. 모든 생성자를 **private** 혹은 **package-private**으로 만들고 public 정적 팩터리를 제공한다.

    ```jsx
    public Class Complex {
        private final double re;
        private final double im;

        private Complex(double re, double im) {
            this.re = re;
            this.im = im;
        }
        public static Complex valudOf(double re, double im) {
            return new Complex(re, im);
        }
        ...
    }
    ```

    - 외부에서 볼 수 없는 **package-private** 구현 클래스를 원하는 만큼 생성, 활용 가능
    - 패키지 외부의 클라이언트에서 본 이 불변객체는 사실상 **final**이다.
    - public이나 protected생성자가 없으니 다른 패키지에서 클래스 확장이 불가능하다.
3. 정적 팩터리 방식은 다수의 구현 클래스를 활용한 유연성을 제공한다.
4. 다음 릴리즈에서 객체 **캐싱**을 추가해 성능을 끌어 올릴 수 있다.

## ❗BigInteger와 BigDecimal 설계시 주의할 점

재정의 할 수 있도록 설계되어 신뢰할 수 없는 하위 클래스의 인스턴스라고 확인되면, 이 인수들을 가변이라 가정하고 **방어적으로 복사**해 사용해야 한다.

```jsx
public static BigInteger safeInstance(BigInteger val) {
    return val.getClass() == BigInteger.class ? val : new BigInteger(val.toByteArray());
}
```

## 불변 클래스의 규칙 중 "모든 필드가 final이고 어떤 메서드도 그 객체를 수정할 수 없다"→ 성능을 위한 완화

**"어떤 메서드도 객체의 상태 중 외부에 비치는 값을 변경할 수 없다."**

어떤 불변 클래스는 계산비용이 큰 값을 처음 쓰일 때 계산하여 **final이 아닌 필드에 캐시**해 놓는다.

→ 같은 값 재요청시 캐시 값 반환하여 계산비용 절감(항상 같은 결과 보장되기 때문)

실제로 String 클래스는 불변객체이지만, final이 아닌 필드 hash를 이용해 캐시하여 hashcode 재 연산 비용을 줄인다.

**직렬화할 경우 추가 주의점**

Serializable을 구현하는 불변 클래스의 내부에 가변 객체를 참조하는 필드가 있다면 readObject나 readResolve 메서드를 반드시 제공하거나, ObjectOutputStream.writeUnshared와 ObjectInputStream.readUnshared 메서드를 사용해야 한다. 그렇지 않으면 공격자가 이 클래스로부터 가변 인스턴스를 만들어 낼 수 있다.

## ✏️핵심 정리

1. **getter가 있다고 무조건 setter를 만들지 말자.**
2. **클래스는 꼭 필요한 경우가 아니라면 불변이어야 한다.**
→ 성능 때문에 어쩔 수 없다면 불변클래스와 쌍을 이루는 가변 동반 클래스를 public 클래스로 제공하자.
3. **불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이자.**

    → 꼭 변경해야 할 필드를 뺀 나머지 모두를 **final**로 선언하자.

4. **다른 합당한 이유가 없다면 모든 필드는 private final이어야 한다.**
5. **생성자는 불변식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야 한다.**
→ 생성자와 정적 팩터리 외에는 그 어떤 초기화 메서드도 public으로 제공하면 안 된다.

# 아이템18. 상속보다는 컴포지션을 사용하라.

상속은 코드를 재사용 하는 강력한 수단이지만, 잘못 사용하면 오류를 내기 쉬운 소프트웨어를 만든다. 이 장에서는 **클래스가 다른 클래스를 상속하는 구현 상속**의 경우를 다룬다. (인터페이스 상속과 무관)

## **메서드 호출과 달리 상속은 캡슐화를 깨뜨린다.**

- 상위 클래스의 구현에 따라 하위 클래스가 오작동 할 수 있다.
- 상위 클래스 설계자가 확장을 충분히 고려하고 문서화 해두지 않으면 상위 클래스가 변경 될 때 마다 하위 클래스를 수정해야 한다.

### 상속을 잘못 사용한 예

```java
public class InstrumentedHashSet<E> extends HashSet<E>{
    private int addCount = 0;

    public InstrumentedHashSet() {
    }

    public InstrumentedHashSet(int initialCapacity, float loadFactor) {
        super(initialCapacity, loadFactor);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(@NonNull Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}

InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
s.addAll(List.of("틱", "탁탁", "펑"));
```

getAddCount()를 호출하면 3을 반환할 것이라 예측하지만 6을 반환한다.

→ HashSet의 addAll은 각 원소를 add메서드를 호출해 추가하는데 이때 호출된 add는 InstrumentedHashSet에서 재정의한 add이다. 그래서 addAll에서 3더해지고 또 add에서 중복으로 더해져서 6이라는 값이 나온다.

### **해결방안**

1.  **하위 클래스에서 addAll 재정의하지 않는다.** 
→ 문제는 고칠 수 있지만, HashSet의 addAll이 add를 구현했음을 인지한 해법이다.
이처럼 자신의 다른 부분을 사용하는 '자기사용(self-use)'의 여부는 해당 클래스의 내부 구현방식으로 다음 릴리즈에서 변경, 삭제가 가능하다.
2. **addAll 메서드를 주어진 컬렉션을 순회하며 원소 하나당 add 메서드를 한번만 호출하도록 재정의 한다.**
    - 상위 클래스의 메서드 동작을 다시 구현하는게 어렵다.
    - 시간도 더 든다.
    - 자칫 오류를 내거나 성능을 떨어뜨릴 수 있다.
    - 하위 클래스에서 접근불가한 private 필드 사용하면 구현 자체가 불가능하다.
3. **다음릴리즈에서 상위클래스에 새로운 메서드를 추가한다.**
→ 상위클래스에 또 다른 원소 추가 메서드가 생성된다 가정하자. 
하위 클래스에서 재정의하지 못한 새로운 메서드를 사용해 허용되지 않은 원소를 추가할 수 있게된다.  (컬렉션에 Hashtable과  Vector를 포함시키자 이러한 보안구멍을 수정해야 했다.)
4. **클래스를 확장하더라도 메서드를 재정의하는 대신 새로운 메서드를 추가한다.**
    - 다음 릴리즈에서 상위 클래스에 새 메서드가 추가됐는데, 내가 하위 클래스에 추가한 메서드와 시그니처가 같고 반환 타입은 다르다면 컴파일 조차 되지 않는다.
    - 반환 타입마저 같다면 재정의한 꼴이라 또 문제가 된다.
    - 상위 클래스의 메서드가 요구하는 규약을 만족하지 못할 가능성이 크다.

## 📌컴포지션(Composition)이란?

: 기존 클래스를 확장하는 대신, 새로운 클래스를 만들고 **private 필드로 기존 클래스의 인스턴스르 참조**하게 한다.

기존 클래스가 새로운 클래스의 구성요소로 쓰인다는 뜻에서 이러한 설계를 컴포지션이라 한다.

## 📌전달(forwarding)이란?

: 새 클래스의 인스턴스 메서드들은 (private 필드로 참조하는) 기존 클래스의 대응하는 메서드를 호출해 그 결과를 반환한다.

새 클래스의 메서드들을 전달 메서드(forwarding method)라 한다.

**새로운 클래스는 기존 클래스의 내부 구현 방식의 영향에서 벗어나며, 심지어 기존 클래스에 새로운 메서드가 추가 되더라도 전혀 영향 받지 않는다.**

### ✔️**재사용할 수 있는 전달 클래스**

```java
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;
    public ForwardingSet(Set<E> s) { this.s = s; }

    public void clear()               { s.clear();            }
    public boolean contains(Object o) { return s.contains(o); }
    public boolean isEmpty()          { return s.isEmpty();   }
    public int size()                 { return s.size();      }
    public Iterator<E> iterator()     { return s.iterator();  }
    public boolean add(E e)           { return s.add(e);      }
    public boolean remove(Object o)   { return s.remove(o);   }
    public boolean containsAll(Collection<?> c)
                                   { return s.containsAll(c); }
    public boolean addAll(Collection<? extends E> c)
                                   { return s.addAll(c);      }
    public boolean removeAll(Collection<?> c)
                                   { return s.removeAll(c);   }
    public boolean retainAll(Collection<?> c)
                                   { return s.retainAll(c);   }
    public Object[] toArray()          { return s.toArray();  }
    public <T> T[] toArray(T[] a)      { return s.toArray(a); }
    @Override public boolean equals(Object o)
                                       { return s.equals(o);  }
    @Override public int hashCode()    { return s.hashCode(); }
    @Override public String toString() { return s.toString(); }
}
```

- InstrumentedSet은 HashSet의 모든 기능을 정의한 Set 인터페이스를 활용해 설계했다.
- Set 인터페이스를 구현하고 Set의 인터페이스를 인수로 받는 생성자를 하나 제공한다.
- 임의의 Set에 계측 기능을 덧씌워 새로운 Set으로 만드는 것이 이 클래스의 핵심이다.

### ✔️**재사용 클래스를 상속받아 구현된 래퍼 클래스**

```java
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {
        super(s);
    }

    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
    public int getAddCount() {
        return addCount;
    }
		public static void main(String[] args) {
        InstrumentedSet<String> s = new InstrumentedSet<>(new HashSet<>());
        s.addAll(List.of("틱", "탁탁", "펑"));
        System.out.println(s.getAddCount());
    }
}
```

상속 방식은 구체 클래스 각각을 따로 확장해야 하며, 지원하고 싶은 상위 클래스의 생성자 각각에 대응하는 생성자를 별도로 정의해줘야 한다. 

↔ 컴포지션 방식은 한번만 구현해두면 어떠한 Set 구현체라도 계측할 수 있으며, 기존 생성자들과도 함께 사용할 수 있다.

```java
Set<Instant> times = new InstrumentedSet<>(new TreeSet<>(cmp));
Set<E> s = new InstrumentedSet<>(new HashSet<>(INIT_CAPACITY));
```

- **래퍼 클래스** : 다른 Set 인스턴스를 감싸고(Wrap) 있다는 뜻에서 InstrumentedSet 같은 클래스를 래퍼 클래스라 한다.
- **데코레이터 패턴(Decorator pattern)** : 다른 Set에 계측 기능을 덧씌운다는 뜻
- **위임(delegation)** :  컴포지션과 전달의 조합의 넓은 의미 단, 래퍼 객체가 내부 객체에 자기 자신의 참조를 넘기는 경우만 해당한다.

### 👎🏻래퍼클래스의 단점

단점이 거의 없다.

단, 래퍼클래스가 콜백프레임워크와는 어울리지 않는다는 점만 주의하면 된다.

**Self문제** : 콜백 프레임워크에서는 자기 자신의 참조를 다른 객체에 넘겨서 다음 호출(콜백) 때 사용하도록한다. 내부 객체는 자신을 감싸고 있는 래퍼의 존재를 모르니 대신 자신(this)의 참조를 넘기고, 콜백 때는 래퍼가 아닌 내부 객체를 호출하게 된다.

## ✏️핵심정리

- 상속은 강력하지만 캡슐화를 해칠 수 있다.
- 상속은 상위 클래스와 하위 클래스가 순수한 is-a 관계일 때만 써야한다.
- 하위 클래스의 패키지가 상위 클래스와 다르고, 상위 클래스가 확장을 고려해 설계되지 않았다면  is-a 관계일 때도 안심 할 수 없다.
- 상속의 취약점을 피하려면 상속 대신 컴포지션과 전달을 사용하자.
- 래퍼 클래스는 하위 클래스보다 견고하고 강력하다.

# 아이템19. 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라

## 📄상속을 고려한 설계와 문서화

1. **상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지(자기사용) 문서로 남겨야 한다.**
    - 클래스의 API로 공개된 메서드에서 클래스 자신의 또 다른 메서드를 호출할 수도 있다.
    - 이때 호출되는 메서드가 **재정의 가능 메서드**라면 그 사실을 호출하는 메서드의 API설명에 적시해야 한다.
    - 호출 순서, 각각의 호출 결과가 이어지는 처리에 어떤영향을 주는지 담아야 한다.
    - '재정의 가능'이란 public과 protected 메서드 중 final이 아닌 모든 메서드를 뜻한다.
    - API문서의 메서드 설명 끝에서 'Implementation Requirements' 로 시작하는 구절은 메서드의 내부 동작 방식을 설명하는 곳이다.
    - **@impleSpec 태그**
        - 자바8에서 도입되어 자바9부터 본격 사용
        - 자바독으로 api 생성시 'Implementation Requirements' 로 대체된다.
        - 메서드 주석에 @impleSpec 태그를 붙여주면 자바독 도구가 생성해준다.
        - 태그 활성화하려면 -tag "implSpec:a:implementation Requirements:"를 지정해준다.
2. **클래스의 내부 동작 과정 중간에 끼어들 수 있는 훅(hook)을 잘 선별하여 protected메서드 형태로 공개해야 할 수도 있다.**
AbstractList의 **removeRange** 메소드는 이 리스트 혹은 이 리스트의 서브리스트에 정의된 **clear에 의해 호출된다**.
**public으로 공개된 API는 clear 메소드이고,  removeRange 메소드는 내부 동작 중간에 끼어드는 훅(hook)이다.**
결국 List 구현체의 최종 사용자는 removeRange 메소드에는 관심이 없다.
그럼에도 이를 protected로 제공한 이유는 **AbstractList를 상속한 클래스가 removeRange를 오버라이딩하여 clear 메소드의 동작을 개선할 수 있게 하기 위함이다.**

## 🔍상속용 클래스 설계시 protected로 노출할 메서드 정하는 방법

- **상속용 클래스를 시험하는 방법은 직접 하위 클래스를 만들어보는 것이 유일하다.**
protected 메서드 하나하나가 내부 구현에 해당하므로 그 수는 가능한 한 적어야 하면서도 너무 적게 노출해서 상속으로 얻는 이점마저 없애지 않도록 해야 한다.
- **상속용으로 설계한 클래스는 배포 전에 반드시 하위 클래스를 만들어 검증해야 한다.**

## ❗상속을 허용하는 클래스가 지켜야 할 제약

1. **상속용 클래스의 생성자는 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안 된다.**
    - 어기면 프로그램 오작동
    - 상위 클래스의 생성자가 하위 클래스의 생성자보다 먼저 실행되므로 **하위 클래스에서 재정의한 메서드가 하위 클래스의 생성자보다 먼저 호출된다.**
    - 이때 재정의한 메서드가 하위 클래스의 생성자에서 초기하는 값에 의존한다면 의도대로 동작하지 않는다.
    - 예시코드

        ```java
        public class Super {
          public Super() {
            overrideMe();
          }
          public void overrideMe() {  
          }
        }

        public final class Sub extends Super {
          private final Instant instant;

          Sub() {
            instant = Instant.now();
          }
          @Override public void overrideMe() {
            System.out.println(instant);
          }
          public static void main(String[] args) {
            Sub sub = new Sub();
            sub.overrideMe();
          }
        }
        ```

        첫 번째는 null을 출력, 두번째 instant  출력

        - 상위 클래스의 생성자는 하위 클래스의 생성자가 인스턴스 필드를 초기화하기도 전에 overrideMe를 호출

        private, final, static 메서드는 재정의가 불가능하니 생성자에서 안심하고 호출해도 된다.

2. **clone과 readObject 모두 직접적으로든 간접적으로든 재정의 가능메서드를 호출해서는 안 된다.**
    - clone과 readObject 메서드는 새로운 객체를 만든다. → 생성자와 비슷한 효과를 낸다.
    - readObject :하위 클래스의 상태가 미처 다 역직렬화되기 전에 오버라이딩 메서드부터 호출
    - clone : 복제본의 상태를 올바른 상태로 수정하기 전에 오버라이딩 메서드부터 호출 → clone이 잘못되면 원본 객체도 피해 입을 수 있다.
3. Serializable을 구현한 상속용 클래스가 readResolve나 writeReplace 메서드를 갖는다면 이 메서드들은 private이 아닌 protected로 선언해야 한다. → private로 선언하면 하위클래스에서 무시된다.
4. **상속용으로 설계하지 않은 클래스는 상속을 금지한다.**
    - 상속을 금지하는 방법 두가지
        1. 둘 중 더 쉬운 쪽은 클래스를 final로 선언한다.
        2. 모든 생성자를 private나 package-private로 선언하고 public 정적 팩터리를 만들어 준다.
        정적 팩터리는 내부에서 다양한 하위 클래스를 만들어 쓸 수 있는 유연성을 준다.
5. 상속을 꼭 허용해야 한다면
    1. 클래스 내부에서는 재정의 가능 메서드를 사용하지 않게 만들고 이 사실을 문서화 한다.
    → 재정의 가능 메서드를 호출하는 자기사용 코드를 완벽히 제거한다.
    → 메서드를 재정의해도 다른 메서드의 동작에 아무런 영향을 주지 않는다.
    2. 클래스의 동작을 유지하면서 재정의 가능 메서드를 사용하는 코드를 제거한다.
    → 재정의 가능 메서드는 자신의 본문 코드를 private 메서드로 옮기고 이 메서드를 호출하도록 수정한다.

## ✏️핵심정리

- 상속용 클래스 설계는 어렵다.
- 클래스 내부에서 스스로를 어떻게 사용하는지(자기사용 패턴) 모두 문서로 남겨야한다.
- 문서로 남긴 사항을 어기면 하위 클래스가 오작동 할 수 있다.
- 다른 이가 효율 좋은 하위클래스 만들 수 있도록 일부 메서드를 protected로 제공해야 할 수도 있다. (ex> AbstractList의 **removeRange** 메소드 → clear 성능 개선)
- 클래스 확장의 명확한 이유없으면 상속을 금지하는 것이 좋다.
- 상속금지는 클래스를 final로 선언하거나 생성자 모두를 외부에서 접근불가하도록 private 선언한다.

# 아이템20. 추상클래스보다는 인터페이스를 우선하라.

자바가 제공하는 다중 구현 메커니즘 인터페이스와 추상 클래스 두 가지이다.

**추상클래스와 인터페이스의 공통점**

자바8부터 인터페이스도 디폴트 메서드를 제공할 수 있게 되어 두 메커니즘 모두 인스턴스 메서드를 구현 형태로 제공할 수 있다.

## ✔️**추상클래스의 특징**

1. **단일상속**
2. 추상 클래스가 정의한 타입을 구현하는 클래스는 반드시 추상 클래스의 **하위 클래스**가 되야 한다. → 자바는 단일상속만 지원하니 새로운 타입 정의하는 데 큰 제약이 따른다.
3. 기존 클래스 위에 새로운 추상 클래스 끼워넣기 어렵다.
두 클래스가 같은 추상클래스를 확장하려면 그 추상클래스는 계층구조상 두 클래스의 공통 조상이어야 한다.→ 두 클래스가 관련없다면 계층구조에 혼란
4. 믹스인 정의 불가 : 기존 클래스에 덧씌울 수 없고, 클래스는 두 부모를 가질 수 없으며(다중상속 불가), 부모와 자식 계층에서 믹스인이 들어갈 합리적 위치가 없다.

## ✔️**인터페이스의 특징**

1. **다중상속**
2. 인터페이스가 선언한 메서드를 모두 정의하고 일반 규약을 잘 지킨 클래스라면 다른 어떤 클래스를 상속 했든 **같은 타입**으로 취급된다.
3. **기존 클래스에도 손쉽게 새로운 인터페이스를 구현해 넣을 수 있다.**
인터페이스가 요구하는 메서드 추가하고, 클래스 선언에 **implements** 구문 추가
4. **믹스인(mixin)** 정의에 안성맞춤이다.
믹스인 : 클래스가 구현할 수 있는 타입으로, 믹스인을 구현한 클래스에 원래의 '주된 타입' 외에 **추가적인 기능을 혼합한다.**
ex) Compareable은 자신을 구현한 클래스들의 순서를 정할 수 있다고 선언하는 인터페이스이다.
5. **계층구조가 없는 타입 프레임워크를 만들 수 있다.**
    - 인터페이스

    ```java
    public interface Singer{
      public void sing(String s);
    }
    public interface SongWriter{
      public void compose(int chartPosition);
    }
    public class People implements Singer, SongWriter{
      @Override
      public void sing(String s) {}
      @Override
      public void compose(int chartPosition) {}
    }
    //두 인터페이스를 확장하고 새로운 메소드까지 추가한 인터페이스 또한 정의할 수 있다.
    public interface SingerSongWriter extends Singer, SongWriter{
    	public void actSensitive();
    }
    ```

    - 계층구조가 불가한 추상클래스

    ```java
    public abstract class Singer {
        abstract void sing(String s);
    }

    public abstract class SongWriter {
        abstract void compose(int chartPosition);
    }

    public abstract class SingerSongWriter {
        abstract void actSensitive();
        abstract void Compose(int chartPosition);
        abstract void sing(String s);
    }
    ```

    다중 상속이 불가한 추상클래스는 새로운 추상클래스를 만들어 추상메서드를 추가해야 한다.

    → 가능한 조합 전부를 각각의 클래스로 정의한 고도비만 계층구조가 만들어진다. 속성이 n개라면 지원해야 할 조합의 수는 2^n개나 된다. (조합폭발)

6. **래퍼 클래스 관용구와 함께 사용하면 인터페이스는 기능을 향상시키는 안전하고 강력한 수단이 된다.**
타입을 추상 클래스로 정의하면 타입에 기능 추가방법은 상속 뿐이고, 래퍼클래스보다 활용도가 낮고 깨지기 쉽다.
7. **자바8부터 인터페이스는 디폴트 메서드 기능을 제공해 개발자들이 중복되는 메서드를 구현하지 않아도 된다.**
    - **디폴트 메서드의 제약**
        1. 제공할 때는 @implSpec 자바독 태그로 문서화 해야 한다.
        2. equals와 hashCode는 디폴트 메서드로 제공해서는 안 된다.
        3. 인터페이스는 인스턴스 필드를 가질 수 없다.
        4. public이 아닌 정적 멤버를 가질 수 없다. (단, private 정적 메서드는 예외)
        5. 본인이 만들지 않은 인터페이스에는 디폴트 메서드를 추가할 수 없다.
8. **자바8부터 Static메서드도 제공 한다.** 
    - 디폴트 메서드는 재정의 가능하고, 정적메서드는 **재정의 불가능**
    - 디폴트 메서드는 참조변수로 함수 호출 가능하고, 정적메서드는 반드시 **클래스명으로 호출**

## 추상 골격 구현(skeletal implementation) 클래스

디폴트 메서드에 제약이 있기 때문에 인터페이스와 추상골격구현 클래스를 함께 제공하여 인터페이스와 추상클래스의 장점을 모두 가져 갈 수 있다.

- 인터페이스 : 타입 정의, 디폴트 메서드 제공
- 골격 구현 클래스 : 나머지 메서드 구현

→ 단순히 골격 구현을 확장하는 것만으로 이 인터페이스를 구현하는데 필요한 일이 대부분 완료 됨

→**템플릿 메서드 패턴** 

관례상 네이밍 : Abstract[Interface명] >Interface , AbstractInterface
ex) AbstractCollection, AbstractSet, AbstractList, AbstractMap

장점 : 추상클래스처럼 구현을 도와주는 동시에, 추상 클래스로 타입을 정의할 때의 제약에서 자유롭다.

- AbstractList 골격 구현을 사용해 List 구현체를 반환하는 정적 팩터리 메서드

    ```java
    public class IntArrays {
        static List<Integer> intArrayAsList(int[] a) {
            Objects.requireNonNull(a);
            return new AbstractList<Integer>() {//익명클래스(아이템 24)
                @Override
                public Integer get(int i) {
                    return a[i];  // 오토박싱(아이템 6)
                }
                @Override
                public Integer set(int i, Integer val) {
                    int oldVal = a[i];
                    a[i] = val;     // 오토언박싱
                    return oldVal;  // 오토박싱
                }
                @Override
                public int size() {
                    return a.length;
                }
            };
        }

        public static void main(String[] args) {
            int[] a = new int[10];
            for (int i = 0; i < a.length; i++)
                a[i] = i;
            List<Integer> list = intArrayAsList(a);
            Collections.shuffle(list);
            System.out.println(list);
        }
    }
    ```

## 시뮬레이트한 다중상속(simulated multiple inheritance)

: 골격 구현 클래스 우회적 이용 방법

인터페이스를 구현한 클래스에서 해당 골격 구현을 확장한 private 내부 클래스를 정의하고, 각 메서드 호출을 내부 클래스의 인스턴스에 전달 한다.

- 시뮬레이트한 다중상속의 예시

    ```java
    public interface Vending {
        void start();
        void chooseProduct();
        void stop();
        void process();
    }

    public abstract class AbstractVending implements Vending {
        @Override
        public void start() {
            System.out.println("vending start");
        }

        @Override
        public void stop() {
            System.out.println("stop vending");
        }

        @Override
        public void process() {
            start();
            chooseProduct();
            stop();
        }
    }
    /* 추상 골격 클래스를 확장 할 수 있는 경우 */
    public class BaverageVending extends AbstractVending implements Vending {
        @Override
        public void chooseProduct() {
            System.out.println("choose menu");
            System.out.println("coke");
            System.out.println("energy drink");
        }
    }

    /*제조사 VendingManufacturer 클래스 상속받아야해서 
    추상 골격 클래스를 확장 할 수 없는 경우 
    골격 구현을 확장한 private 내부 클래스를 정의하고,
    각 메서드 호출을 내부 클래스의 인스턴스에 전달*/

    public class VendingManufacturer {
        public void printManufacturerName() {
            System.out.println("Made By JavaBom");
        }
    }

    public class SnackVending extends VendingManufacturer implements Vending {
        InnerAbstractVending innerAbstractVending = new InnerAbstractVending();

        @Override
        public void start() {
            innerAbstractVending.start();
        }

        @Override
        public void chooseProduct() {
            innerAbstractVending.chooseProduct();
        }

        @Override
        public void stop() {
            innerAbstractVending.stop();
        }

        @Override
        public void process() {
            printManufacturerName();
            innerAbstractVending.process();
        }

        private class InnerAbstractVending extends AbstractVending {
            @Override
            public void chooseProduct() {
                System.out.println("choose product");
                System.out.println("chocolate");
                System.out.println("cracker");
            }
        }
    }
    ```

## 골격 구현 작성법

1. 인터페이스를 잘 살펴 다른 메서드들의 구현에 사용되는 **기반 메서드들을 선정**한다.
→ 골격구현에서 **추상 메서드**가 될 것이다.
2. **기반 메서드들을 사용해 직접 구현할 수 있는 메서드를 모두 디폴트 메서드로 제공**한다.
→ 단, equals와 hashCode 같은 **Object의 메서드는 디폴트 메서드로 제공하면 안 된다.**
3. 만약, 인터페이스의 메서드 모두가 기반 메서드나 디폴트 메서드가 된다면 골격 구현 클래스를 별도로 만들 필요가 없다.
4. 기반, 디폴트 메서드로 만들지 못한 메서드가 남아 있다면, 이 인터페이스를 구현하는 골격 구현 클래스를 하나 만들어 남은 메서드들을 작성해 넣는다.
5. 골격 구현 클래스에는 필요하면 public이 아닌 필드와 메서드를 추가해도 된다.
6. 골격 구현은 기본적으로 상속해서 사용하는 걸 가정하므로 설계 및 문서화 지침을 모두 따라야 한다.
- 골격 구현 클래스 예시 : Map.Entry 인터페이스
    - setValue, equals, hashCode, toString 메소드는 골격 구현 클래스가 구현해놨다
    - getKey, getValue는 확실한 기반 메서드, setValue도 선택적 포함할 수 있다.
    - equals와 hashCode 동작 정의→ Object 메서드는 디폴트 제공 X → 골격구현 클래스에 구현
    - toString도 기반 메서드 사용하여 구현

    ```java
    public abstract class AbstractMapEntry<K,V> implements Map.Entry<K,V> {

        @Override public V setValue(V value) {
            throw new UnsupportedOperationException();
        }
        
        @Override public boolean equals(Object o) {
            if (o == this) {
                return true;
            }
            if (!(o instanceof Map.Entry)) {
                return false;
            }
            Map.Entry<?,?> e = (Map.Entry) o;
            return Objects.equals(e.getKey(),   getKey()) && Objects.equals(e.getValue(), getValue());
        }

        @Override public int hashCode() {
            return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());
        }

        @Override public String toString() {
            return getKey() + "=" + getValue();
        }
    }
    ```

## 단순 구현(simple implementation)

: 골격 구현의 작은 변종이며, 쉽게 말해 동작하는 가장 단순한 구현이다.

단순 구현도 골격 구현과 같이 상속을 위해 인터페이스를 구현하지만, **추상 클래스가 아니다.**

단순 구현은 그대로 써도 되고 필요에 맞게 확장해도 된다.

ex) AbstractMap.SimpleEntry

## ✏️핵심정리

- 일반적으로 다중 구현용 타입은 인터페이스가 가장 적합하다.
- 복잡한 인터페이스라면 인터페이스 + 골격 구현 클래스 고려하자.
- 골격 구현은 '가능한 한' 인터페이스의 디폴트 메서드로 제공하여 그 인터페이스를 구현한 모든곳에서 활용하는 것이 좋다.
- '가능한 한'의 이유는 인터페이스 구현상 제약 때문에 골격 구현을 추상 클래스로 제공하는 경우가 더 흔하기 때문이다.

# 아이템21. 인터페이스는 구현하는 쪽을 생각해 설계하라.

## 인터페이스에 디폴트 메서드 추가

자바8 전에는 기존 구현체를 깨뜨리지 않고는 인터페이스에 메서드 추가 불가

→ 자바8 부터 디폴트 메서드가 추가 되어 기존 인터페이스에 메서드를 추가 할 수 있다.

디폴트 메서드를 선언하면, 그 인터페이스를 구현한 후 **디폴트 메서드를 재정의 하지 않은 모든 클래스에서 디폴트 구현이 쓰이게 된다.**

**생각할 수 있는 모든 상황에서 불변식을 해치지 않는 디폴트 메서드를 작성하기는 어렵다.**

예시) 자바8의 Collection 인터페이스에 추가 된 removeIf 메서드 → Predicate가 true를 반환하는 모든 원소를 제거한다.

```java
default boolean removeIf(Predicate<? super E> filter) {
  Objects.requireNonNull(filter);
  boolean result = false;
  for (Iterator<E> it = iterator(); it.hasNext();) {
    if (filter.test(it.next())) {
      it.remove();
      result = true;
    }
  }
  return result;
}
```

위 코드는 범용적으로 구현하였지만, 현존하는 모든 Collection 구현체와 잘 어우러지는 것은 아니다.

예시) org.apache.commons.collections.collection.**SynchronizedCollecion**

- java.util의 Collection.synchronizedCollection 정적 팩터리 메서드가 반환하는 클래스와 비슷
- 아파치버전은(컬렉션 대신) 클라이언트가 제공한 객체로 락을 거는 능력을 추가로 제공한다.
- 모든 메서드에서 주어진 락 객체로 동기화한 후 내부 컬렉션 객체에 기능을 위임하는 래퍼 클래스이다.
- 이 클래스를 자바8과 함께 사용하면 removeIf()가 자바8 이전 버전까지는 동기적으로 구현이 되어있지않아 default 메서드를 부르게 되어 동기적으로 작동하지 않을 것이다.
- removeIf의 구현은 (재정의 되지 않으면) 동기화에 관해 아무것도 모르므로 락 객체를 사용할 수 없다.
- 멀티스레드 환경에서 한 스레드가 removeIf를 호출하면 ConcurrentModificationException이 발생하거나 예기치 못한 결과가 나온다.
- 해결방안 : removeIf를 재정의하고, 이를 호출하는 다른 메서드들은 디폴트 구현을 호출하기 전에 동기화 하도록 한다.

## ❗인터페이스 디폴트메서드 추가시 주의사항

- **기존 인터페이스에 디폴트 메서드 추가는 꼭 필요하지 않으면 피해야 한다.**
- 디폴트 메서드는 컴파일에 성공하더라도 기존 구현체에 런타임 오류를 일으킬 수 있다.
- **디폴트 메서드는 인터페이스로 부터 메서드를 제거하거나 기존 메서드를 수정하는 용도가 아니다.**
- 인터페이스를 릴리스한 후라도 결함을 수정하는게 가능한 경우도 있겠지만, 그 가능성에 기대서는 안 된다.

# 아이템22. 인터페이스는 타입을 정의하는 용도로만 사용하라.

**인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할을 한다.**

→ 클래스가 자신의 인스턴스로 무엇을 할 수 있는지 클라이언트에게 얘기해주는 용도로만 사용해야 한다.

## ❗상수 인터페이스 안티패턴 - 사용금지!

메서드 없이, 상수를 뜻하는 static final 필드로만 가득 찬 인터페이스를 말한다.

그리고 이 상수를 사용하려는 클래스에서는 정규화 된 이름을 쓰는 걸 피하고자 상수 인터페이스를 구현한다.

```java
public interface PhysicalConstants {
    // 아보가드로 수 (1/몰)
    static final double AVOGADROS_NUMBER   = 6.022_140_857e23;
    // 볼츠만 상수 (J/K)
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
    // 전자 질량 (kg)
    static final double ELECTRON_MASS      = 9.109_383_56e-31;
}
```

**상수 인터페이스 안티패턴은 인터페이스를 잘못 사용한 예다.**

## ✔️상수 인터페이스를 사용하면 안 되는 이유

- 클래스 내부에서 사용하는 상수는 외부 인터페이스가 아니라 내부 구현에 해당한다.
- 상수 인터페이스 구현 == 내부 구현을 클래스의 API로 노출하는 행위
- 클라이언트 코드가 내부 구현에 해당하는 이 상수들에 종속되게 한다.
- 다음 릴리스에서 이 상수들을 더 이상 쓰지 않아도 바이너리 호환성을 위해 여전히 상수 인터페이스를 구현해야 한다.
- final이 아닌 클래스가 상수 인터페이스를 구현한다면 모든 하위 클래스의 nameSpace가 그 인터페이스가 정의한 상수들로 오염된다.

## ✔️상수를 공개해야 하는 경우

1. **특정 클래스나 인터페이스와 강하게 연관된 상수라면 그 클래스나 인터페이스 자체에 추가한다.**
예시) 모든 숫자 기본 타입의 박싱 클래스 Integer, Double에 선언 된 MIN_VALUE, MAX_VALUE
2. **열거 타입으로 나타내기 적합한 상수라면 열거 타입으로 공개하면 된다. (Enum클래스 활용)**
3. 1, 2의 경우가 아니라면 **인스턴스화할 수 없는 유틸리티 클래스에 담아 공개하자.**

    ```java
    public class PhysicalConstants {
      private PhysicalConstants() {}  //인스턴스화 방지

      public static final double AVOGADROS_NUMBER = 6.022_140_857e23;
      public static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
      public static final double ELECTRON_MASS = 9.109_383_56e-31;
    }
    ```

*유틸리티 클래스에 정의된 상수를 클라이언트에서 사용하려면 클래스 이름까지 명시해야 한다.
→ PhysicalConstants .AVOGADROS_NUMBER 
→ 유틸리티 클래스의 상수 사용빈도가 높다면, 정적 임포트(static import)하여 클래스 이름 생략가능하다.

```java
import static effective_java.item_22.PhysicalConstants.*;
public class Test {    
    double atoms(double mols) {
        return AVOGADROS_NUMBER * mols;
    }
}
```

## ✏️핵심정리

인터페이스는 타입을 정의하는 용도로만 사용해야 하며, 상수 공개용 수단으로 사용하지 말자.

# 아이템23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라

## 📌태그 달린 클래스

: 두 가지 이상의 기능을 가지고 있으며, 그중 현재 어떤 기능을 갖고 있는지 나타내는 태그 필드가 있는 클래스

- 태그 달린 클래스의 예시

    ```java
    class Figure {
        enum Shape { RECTANGLE, CIRCLE };

        final Shape shape; // 태그 필드 - 현재 모양을 나타낸다.

        // 다음 필드들은 모양이 사각형(RECTANGLE)일 때만 쓰인다.
        double length;
        double width;

        // 다음 필드느 모양이 원(CIRCLE)일 때만 쓰인다.
        double radius;

        // 원용 생성자
        Figure(double radius) {
            shape = Shape.CIRCLE;
            this.radius = radius;
        }

        // 사각형용 생성자
        Figure(double length, double width) {
            shape = Shape.RECTANGLE;
            this.length = length;
            this.width = width;
        }

        double area() {
            switch(shape) {
                case RECTANGLE:
                    return length * width;
                case CIRCLE:
                    return Math.PI * (radius * radius);
                default:
                    throw new AssertionError(shape);
            }
        }
    }
    ```

## 👎🏻태그 달린 클래스의 단점

- 열거타입(Enum) 선언, 태그 필드, switch문 등 쓸데없는 코드가 많다.
- 여러 구현이 한 클래스에 혼합되어 있어서 가독성이 나쁘다.
- 다른 기능을 위한 코드도 항상 함께 하니 메모리도 많이 사용한다.
- 필드들을 final로 선언하려면 해당 기능에 쓰이지 않는 필드들까지 생성자에 초기화해야 한다.
→ 쓰지 않는 필드를 초기화하는 코드가 생긴다.
- 또 다른 기능을 추가하려면 코드를 수정해야 한다.
→ 모든 switch문을 찾아 새 기능에 대한 코드 추가
- 인스턴스의 타입만으로는 현재 나타내는 기능을 알 수 없다.
- 장황하고, 오류를 내기 쉽고, 비효율적이다.

## 📌서브타이핑(subtyping) - 클래스 계층구조

: 클래스 계층 구조를 활용하여 타입 하나로 다양한 기능의 객체를 표현하는 수단

## ✔️태그달린 클래스를 클래스 계층구조로 변경하는 법

1. 계층구조의 루트(root)가 될 추상클래스를 정의한다.
2. 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상메서드로 선언한다.
ex) Figure클래스의 area 메서드
3. 태크 값에 상관없이 동작이 일정한 메서드들을 루트 클래스에 일반 메서드로 추가한다.
4. 모든 하위 클래스에서 공통으로 사용하는 데이터 필드들도 전부 루트 클래스로 올린다.
5. 루트 클래스를 확장한 구체 클래스를 기능별로 하나씩 정의한다.
ex) Figure를 확장한 Circle 클래스와 Rectangle 클래스 생성
6. 각 하위 클래스에는 각자의 기능에 해당하는 데이터 필드들을 넣는다.
ex) Circle에는 radius, Rectangle에는 length, width
7. 루트 클래스가 정의한 추상 메서드를 각자의 기능에 맞게 구현한다.
- 클래스 계층구조로 변경한 예시

    ```java
    abstract class Figure {
        abstract double area();
    }

    class Circle extends Figure {
        final double radius;
        Circle(double radius) { this.radius = radius; }
        @Override double area() { return Math.PI * (radius * radius); }
    }

    class Rectangle extends Figure {
        final double length;
        final double width;
        Rectangle(double length, double width) {
            this.length = length;
            this.width = width;
        }
        @Override double area() { return length * width; }
    }
    ```

### 👍🏻클래스 계층구조의 장점

- 태그 달린 클래스의 단점을 모두 해결하였다.
- 간결하고 명확하다.
- 각 기능을 독립된 클래스에 담아 관련 없던 데이터 필드를 모두 제거했다.
- 살아 남은 필드들은 모두 final이다.
- 각 클래스의 생성자가 모든 필드를 남김없이 초기화 하고 추상 메서드를 구현했는지 컴파일러가 확인해준다.
- 실수로 빼먹은 case문 때문에 런타임 오류가 발생할리 없다.
- 루트 클래스의 코드 수저없이 독립적으로 계층구조를 확장할 수 있다.
- 타입이 기능별로 따로 존재하여 변수의 기능을 명시하거나 제한할 수 있고, 또 특정 기능만 매개변수로 받을 수 있다.
- 타입 사이의 자연스러운 계층관계를 반영할 수 있어 유연성은 물론 컴파일타임 타입 검사 능력을 높여준다
- 정사각형 추가시 간단하게 Rectangle을 확장하여 계층구조 활용 할 수 있다.

    ```java
    class Square extends Rectangle {
        Square(double side) {
            super(side, side);
        }
    }
    ```

# 아이템24. 멤버 클래스는 되도록 static으로 만들라.

## 📌중첩 클래스(nested class)

: 다른 클래스 안에 정의된 클래스이다.

자신을 감싼 바깥 클래스에서만 쓰여야 하며, 그 외의 쓰임새가 있다면 톱레벨 클래스로 만들어야한다.

중첩 클래스의 종류 네 가지

1. 정적 멤버 클래스
2. 비정적 멤버 클래스
3. 익명 클래스
4. 지역 클래스

## ✔️정적 멤버 클래스

- 다른 클래스 안에 선언되고, **바깥 클래스의 private 멤버에도 접근할 수 있다**는 점만 제외하고 일반 클래스와 동일하다.
- 다른 정적 멤버와 똑같은 접근 규칙을 적용받는다.
→ private로 선언하면 바깥 클래스에서만 접근 할 수 있다.
- 바깥 클래스와 함께 쓰일 때만 유용한 public 도우미로 쓰인다.
→ Calculator.Operation.PLUS, Calculator.Operation.MINUS
    - 계산기 클래스 예시 코드
    Operation 열거 타입은 Calculator 클래스의 public 정적 멤버 클래스가 된다.

        ```java
        public class Calculator {
        	public int calculate(int num1, int num2, Operation operation){
        			int result = 0;	
        			switch(operation){
        				case Operation.PLUS :
        					result = num1 + num2;
        				break;
        				case Operation.MINUS :
        					result = num1 - num2;
        				break;
        			}
        			return result;
        	}
           public static enum Operation {
        			PLUS, MINUS
        	}
        }
        public static void main(String[] args){
        	int num1 = 5, int num2 = 6;
        	// 바깥 클래스가 생성되지 않아도 접근 가능
        	// Calculator calculator = new Calculaotr(); <- 이거안해도 됨. 
        	Calculator.Operation operation = Calculator.Operation.PLUS; 
        	Calculator.calculate(num1, num2, operation);
        }
        ```

## ✔️비정적 멤버 클래스

### 📎특징

- 정적 멤버클래스와 구문상 차이는 static이 붙어 있는가의 차이지만, 의미상으로는 비정적 멤버 클래스의 인스턴스는 **바깥 클래스의 인스턴스와 암묵적으로 연결 된다**는 점이다.
- 비정적 멤버 클래스의 인스턴스 메서드에서 **정규화 된 this**( 클래스명.this 형태로 바깥 클래스의 이름을 명시하는 용법)를 사용해 **바깥 인스턴스의 메서드를 호출하거나 바깥 인스턴스의 참조를 가져 올 수 있다.**
- **정적 멤버 클래스와 비정적 멤버 클래스의 차이 예시**

    ```java
    class A {
        int a = 10;
        public void run() {
            System.out.println("Run A");
            B.run();
            C c = new C();
            c.run();
        }
        // 정적 멤버 클래스
        public static class B {
            public static void run() {
                System.out.println("Run B");
            }
        }
        // 비정적 멤버 클래스
        public class C {
            public void run() {
    				// **바깥 인스턴스의 메서드를 호출하거나 바깥 인스턴스의 참조를 가져 올 수 있다.**
                System.out.println("Run C: " + A.this.a);
            }
        }
    }
    ```

    ```java
    public class Example {
        public static void main(String[] args) {
            // 정적 멤버 클래스는 이렇게 외부에서 접근 가능하다.
            A.B.run();
            A a = new A(); // 비정적은 바깥 클래스 없이는 생성 불가하기 때문에 이거 필요
            a.run();
            A.C c = a.new C();
            c.run();
        }
    }
    ```

    ```java
    // 출력 결과
    Run B
    Run A
    Run B
    Run C: 10
    Run C: 10
    ```

- 만약 중첩 클래스의 인스턴스가 **바깥 인스턴스와 독립적으로 존재 할 수 있다면, 정적 멤버 클래스로 만들어야 한다.** → 비정적 멤버 클래스는 바깥 인스턴스 없이는 생성 할 수 없기 때문
- **멤버 클래스가 인스턴스화 될 때** 비정적 멤버 클래스의 인스턴스와 바깥 인스턴스 사이의 관계가 확립되며, 변경 불가하다.

    **멤버클래스 인스턴스화 방법**
    1. 보통 바깥 클래스의 인스턴스 메서드에서 호출할 때 자동으로 생성
    2. OuterClass.new InnerMemberClass(args) 호출해 수동으로 생성
    → 위 예시 중 A.C c = a.new C(); 
    **→** 이 관계 정보는 비정적 멤버 클래스의 인스턴스 안에 만들어져 **메모리 공간차지, 생성시간도 더 걸린다.**

### 📎쓰임새

어댑터를 정의할 때 자주 쓰인다.

즉, 어떤 클래스의 인스턴스를 감싸 마치 다른 클래스의 인스턴스처럼 보이게 하는 뷰로 사용한다.

- Map 인터페이스의 구현체들은 보통(keySet, entrySet, values 메서드가 반환하는) 자신의 컬렉션 뷰를 구현할 때 비정적 멤버 클래스를 사용

    ![4%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8F%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A2%E1%84%89%E1%85%B3%E1%84%8B%E1%85%AA%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%90%E1%85%A5%E1%84%91%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20a8f9f7d38c164b62ba51f8831d59eb34/Untitled.png](4%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8F%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A2%E1%84%89%E1%85%B3%E1%84%8B%E1%85%AA%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%90%E1%85%A5%E1%84%91%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20a8f9f7d38c164b62ba51f8831d59eb34/Untitled.png)

- Set, List 같은 다른 컬렉션 인터페이스들도 자신의 반복자를 구현할 때 비정적 멤버 클래스 사용

    ```java
    // 비-정적 멤버 클래스의 흔한 쓰임 - 자신의 반복자 구현 
    class MySet<E> extends AbstractSet<E> {
       ... // 생략
       public Iterator<E> iterator() {
           return new MyIterator(); 
       }
       private class MyIterator implements Iterator<E> {
        ... //    
       } 
    }
    ```

멤버 크래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여서 정적 멤버 클래스로 만들어라.

### 👎🏻단점

- static을 생략하면 **숨은 외부 참조**를 갖게 되고, 이 **참조 저장을 위한 시간과 공간이 소비**된다.
- GC가 바깥 클래스의 인스턴스를 수거하지 못하는 **메모리 누수**가 생길 수 있다.
- 참조가 눈에 보이지 않아 문제의 원인을 찾기 어렵다.

## private 정적 멤버클래스

**바깥 클래스가 표현하는 객체의 구성요소**를 나타낼 때 사용한다.

- (ex. Map 구현 클래스의 Entry 객체)

    Map 구현체는 각각의 key-value 쌍을 표현하는 엔트리 객체를 가진다.

    모든 엔트리가 Map과 연관되어 있지만 엔트리의 메서드들(getKey, getValue, SetValue)은 맵을 직접 사용하지 않기 때문에 비정적은 낭비다.

멤버 클래스가 공개된 클래스의 public이나 protected 멤버라면 정적여부는 더 중요하다.

멤버 클래스 역시 공개 API가 되어 향후 static을 붙이면 하위 호환성이 깨진다.

## ✔️익명 클래스(Anonymous class)

### 📎특징

- 이름이 없으며, 바깥 클래스의 멤버도 아니다.
- 멤버와 달리, 쓰이는 시점에 선언과 동시에 인스턴스가 생성된다.
- 코드의 어디서든 만들 수 있다.
- 오직 비정적인 문맥에서 사용될 때만 바깥 클래스의 인스턴스를 참조할 수 있다.
- 정적 문맥에서라도 상수 변수 이외의 static 멤버는 가질 수 없다.
- 상수 표현을 위해 초기화된 final 기본 타입과 문자열 필드만 가질 수 있다.

### 📎제약사항

- 선언한 지점에만 인스턴스를 만들 수 있다.
- instanceof 검사나 클래스의 이름이 필요한 작업은 수행할 수 없다.
- 여러개의 인터페이스를 구현할 수 없다.
- 인터페이스를 구현하는 동시에 다른 클래스를 상속할 수 없다.
- 익명클래스를 사용하는 클라이언트는 그 익명클래스가 상위 타입에서 상속한 멤버 외에는 호출할 수 없다.
- 익명 클래스는 짧지 않으면 가독성이 떨어진다.
- 자바8 람다 지원 전에는 작은 함수 객체나, 처리 객체를 만드는데 주로 사용했지만 현재 람다를 사용한다.
- 주 쓰임은 정적 팩터리 메서드를 구현 할 때다.

```java
Thread th = new Thread() { // 익명 클래스
    final int value = 5;
    public void run() {
        System.out.println("Hello Thread: " + value);
    }
};
```

## ✔️지역 클래스

- 네 가지 중첩클래스 중 사용 빈도가 가장 낮다.
- 지역 변수를 선언할 수 있는 곳이면 어디든 선언할 수 있고, 유효범위도 지역변수와 같다.
- 멤버클래스처럼 이름이 있고 반복사용이 가능하다.
- 익명클래스처럼 비정적 문맥에서 사용될 때만 바깥 인스턴스를 참조 할 수 있다.
- static 멤버는 가질 수 없다.
- 가독성을 위해 짧게 작성해야 한다.

```java
public class LocalExample {
    private int number;

    public LocalExample(int number) {
        this.number = number;
    }

    public void foo() {
        // 지역변수처럼 선언해서 사용할 수 있다.
        class LocalClass {
            private String name;

            public LocalClass(String name) {
                this.name = name;
            }

            public void print() {
                // 비정적 문맥에선 바깥 인스턴스를 참조 할 수 있다.
                System.out.println(number + name);
            }

        }

        LocalClass localClass = new LocalClass("local");

        localClass.print();
    }
}
```

## ✏️핵심정리

- 메서드 밖에서도 사용해야 하거나 메서드 안에 정의하기엔 너무 길다면 멤버 클래스로 만든다.
- 멤버 클래스의 인스턴스 각각이 바깥 인스턴스를 참조한다면 비정적, 그렇지않으면 정적
- 중첩 클래스가 한 메서드 안에서만 쓰이면서, 그 인스턴스를 생성하는 지점이 단 한 곳이고, 해당 타입으로 쓰기에 적합한 클래스나 인터페이스가(자료형) 이미 있다면 익명클래스로 만들고, 아니면 지역 클래스로 만든다.

# 아이템25. 톱레벨 클래스는 한 파일에 하나만 담으라.

소스파일 하나에 톱레벨 클래스를 여러 개 선언하더라도 자바 컴파일러는 문제가 없다.

하지만 이 방법은 **한 클래스를 여러 가지로 정의할 수 있으며, 그중 어느 것을 사용할지는 어느 소스 파일을 먼저 컴파일하느냐에 따라 달라진다** → 심각한 위험!

```java
// Main.java
public class Main {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
}
```

```java
// Utensil.java
class Utensil {
    static final String NAME = "pan";
}

class Dessert {
    static final String NAME = "cake";
}
```

// [Utensil.java](http://utensil.java) 클래스 파일까지만 만들고 main()호출하면 pancake를 출력한다.

```java
// Dessert.java
class Utensil {
    static final String NAME = "pot";
}

class Dessert {
    static final String NAME = "pie";
}
```

[//Utensil.java와](//utensil.java와) 같은 두 클래스를 담은 [Dessert.java](http://dessert.java) 파일을 생성하면 컴파일 순서에 따라 동작이 달라진다.

1. javac [Main.java](http://main.java) , [Dessert.java](http://dessert.java) : 컴파일 오류 발생, Utensil과 Dessert클래스 중복정의를 알려준다.
→ Main.java 컴파일 → 그 안의 [Utensil.java](http://utensil.java) → [Desert.java](http://desert.java)
2. javac [Main.java](http://main.java) 나 javac Main.java [Utensil.java](http://utensil.java) : pancake 출력
3. javac [Dessert.java](http://dessert.java) [Main.java](http://main.java) : potpie 출력

## 📍해결책

- 톱레벨 클래스들(Utensil과 Dessert)를 **서로 다른 소스 파일로 분리한다.**
- 다른 클래스에 딸린 부차적인 클래스라면 정적 멤버 클래스(아이템 24)로 만드는 쪽이 더 좋다.
→ 가독성 좋고, private로 선언하면 접근 범위도 최소로 관리할 수 있다.
정적 멤버 클래스로 바꾼 코드

    ```java
    public class Main {
        public static void main(String[] args) {
            System.out.println(Utensil.NAME + Dessert.NAME);
        }
        
        private static class Utensil {
        	static final String NAME = "pan";
        }

        private static class Dessert {
            static final String NAME = "cake";
        }
    }
    ```

## ✏️핵심정리

- 소스 파일 하나에는 반드시 톱레벨 클래스(혹은 톱레벨 인터페이스)를 하나만 담자.
- 이 규칙을 지키면 컴파일러가 한 클래스에 대한 정의를 여러 개 만들어 내는 일은 사라진다.
- 소스 파일을 어떤 순서로 컴파일하든 바이너리 파일이나 프로그램의 동작이 달라지는 경우는 없다.