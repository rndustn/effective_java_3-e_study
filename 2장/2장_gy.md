# 2장. 객체 생성과 파괴

## 아이템1. 생성자 대신 정적 팩터리 메서드를 고려하라.

### **정적팩토리 메서드: 객체 생성의 역할을 하는 클래스 메서드(객체를 생성하는 메서드를 만들고 Static으로 선언)**

-> 직접적으로 생성자를 통해 객체를 생성하는 것이 아닌 메서드를 통해 객체를 생성한다.

### **정적팩터리 메서드 장점**

1. 이름을 가질 수 있다.
    - 객체생성의 목적을 메서드 이름에 표현하여 가독성 상승
2. 호출될 때마다 객체를 생성할 필요가 없다.
    - 객체를 미리 생성해두거나 새로 생성된 객체를 캐싱하여 재활용
    - 생성자의 접근제한자를 private로 설정하여 객체 통제 가능 (instance-controlled class)
3. 반환 타임의 하위 타입 객체를 반환 할 수 있다.
    - 생성자와 달리 생성자 역할을 하는 정적 팩토리 메서드는 **반환값**을 가지고 있다.
    - 해당 클래스를 **상속받은 하위 객체를 반환**할 수 있다.
4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환 할 수 있다.
    - 매개변수에 따라 선택적인 하위클래스 객체를 반환(하위클래스이기만 하면 됨)
5. 정적 팩터리 메서드 작성 시점에는 실제 구현체 클래스가 없어도 된다.

### **정적팩터리 메서드 단점**

1. **상속이 불가능**하다.
    - 상속을 하려면 public 이나 protected 생성자가 필요하기 때문
    - 컴포지션(새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조), 불변타입으로 생성시 장점.
2. 프로그래머가 찾기 어렵다.
    - 생성자처럼 API 설명에 명확히 드러나지 않아 사용자가 정적 메서드를 찾아내야한다.
    - 명명 규약에 따라 메서드 이름을 지어야 극복 가능

### **명명방식 종류**

1. from: **매개변수 하나**를 받아서 해당 타입의 객체를 반환하는 형변환 메서드

    ex) Date d = Date.from(instant);

2. of: **여러 매개변수**를 받아 적합한 타입의 객체를 반환하는 집계 메서드

    ex) Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);

3. valueOf: from, of의 더 자세한 버전

    ex) BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);

4. instance 혹은 getInstance: 매개변수로 명시한 객체를 반환하지만, 같은 인스턴스임을 보장하지 않음(같을 수 있다)

    ex) StackWalker luke = StackWalker.getInstance(options);

5. create 혹은 newInstance: 4.번과 같지만 앞서 생성한 객체와는 다른 항상 새로운 객체를 생성해 반환.

    ex) Object newArray = Array.newInstance(classObject, arrayLen);

6. getType: getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩토리 메서드 정의할 때 사용.

    ex) FileStore fs = Files.getFileStore(path) -> Type은 반환할 객체타입지정

7. newType: newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩토리 메서드 정의할 때 사용.

    ex) BufferedReader br = Files.newBufferedReader(path);

8. type: getType과 newType의 간결한 버전

    ex) List<Complaint> litany = Collections.list(legachLitany);

## **아이템2. 생성자에 매개변수가 많다면 빌더를 고려하라.**

### **객체 생성 방법**

1. **점층적 생성자 패턴**: 필수 매개변수만 받는 생성자를 만들고 그 이후 선택 매개변수를 수를 늘여가며 생성자 추가
    - 사용하지 않는 매겨변수도 셋팅이 필요하며 개수가 많을 수록 가독성이 떨어진다.

        ex) NutritionFacts cocaCola = new NutritionFacts(240,8,100,0,34,27);

2. **자바빈즈 패턴**: 매개변수가 없는 생성자로 객체를 만든 후 Setter 메서드 사용
    - 객체 완전히 생성되기 전까지 일관성 및 스레드 안정성이 낮다.
3. **빌더패턴**: 클라이언트 코드에서 객체를 생성하는 대신 필수 매개변수만으로 생성자(또는 정적팩토리)를 호출해 빌더 객체를 만든 뒤, 빌더 객체에 정의된 선택 매개변수 메서드들을 호출하여 설정 후 build() 호출하여 객체를 생성한다.

    ex) NutritionFacts cocaCola = new NutritionFacts.Builder(240,8).calories(100).sodium(35).carbohydrate(27).build();

장점:

1. 각 변수가 어떤의미인지 알기 쉽다.
2. Setter 메소드가 없으므로 변경불가능 객체를 만들 수 있다.
3. 한번에 객체를 생성하므로 객체 일관성이 깨지지 않는다.
4. build() 함수가 잘못된 값이 입력되었는지 검증하게 할 수도 있다.
- 참고 URL

    [https://happy-playboy.tistory.com/entry/Item-2-%EC%83%9D%EC%84%B1%EC%9E%90%EC%97%90-%EB%A7%A4%EA%B0%9C%EB%B3%80%EC%88%98%EA%B0%80-%EB%A7%8E%EB%8B%A4%EB%A9%B4-%EB%B9%8C%EB%8D%94%EB%A5%BC-%EA%B3%A0%EB%A0%A4%ED%95%98%EB%9D%BC](https://happy-playboy.tistory.com/entry/Item-2-%EC%83%9D%EC%84%B1%EC%9E%90%EC%97%90-%EB%A7%A4%EA%B0%9C%EB%B3%80%EC%88%98%EA%B0%80-%EB%A7%8E%EB%8B%A4%EB%A9%B4-%EB%B9%8C%EB%8D%94%EB%A5%BC-%EA%B3%A0%EB%A0%A4%ED%95%98%EB%9D%BC)

## **아이템3. private 생성자나 열거 타입으로 싱글턴임을 보증하라.**

singleton : 인스턴스를 오직 하나만 생성할 수 있는 클래스

### **싱글턴 생성 방법**

- 생성자는 private로 감춰두고 유일한 인스턴스에 접근할 수 있는 수단으로 public static 멤버를 둔다는 점은 동일
1. **public static final**필드 방식의 싱글턴

    ```java
    public class Elvis{
    	public static final Elvis INSTANCE = new Elvis();
    	private Elvis(){ ... }
    }
    ```

    - private 생성자는 public static final 필드인 Elvis.INSTANCE를 초기화 할 때 한번만 호출된다.
    - 장점: 간결함, 해당 클래스가 싱글턴임을 API에 명백히 드러냄
2. **정적 펙터리 방식**의 싱글턴

    ```java
    public class Elvis{

    	private static final Elvis INSTANCE = new Elvis();
    	
    	private Elvis(){ ... }
    	
    	public static Elvis getInstance() { return INSTANCE; }
    	
    }
    ```

 Elvis.getInstance는 private로 선언된 항상 같은 객체를 반환한다.

- 장점
    - API 수정 없이 싱글턴이 아니게 변경할 수 있다.-> 호출하는 스레드별로 다른 객체 반환 가능
    - 제네릭 싱글턴 팩터리로 만들 수 있다.
    - 정적 펙터리 메서드 참조를 공급자로 사용할 수 있다.
    ex) Elvis::getInstance를 Supplier<Elvis> 로 사용

### **> 메서드 참조 종류**

1. 정적 메서드 참조: Integer의 parseInt를 Integer::parseInt로 사용 가능
2. 다양한 형식의 인스턴스 메서드 참조: String의 length 메서드를 String::length로 사용 가능
3. 기존 객체의 인스턴스 메서드 참조: Transaction 객체를 할당 받은 transaction 지역 변수가 있고, Transaction 객체에는 getValue 메서드가 있다면, transaction::getValue로 사용 가능

    **> Supplier<T>** : 매개값은 없고 리턴값이 있는 getXXX() 메소드를 가지고 있다.
    ex) Supplier<String> s = () -> "hello supplier";
    String result = s.get();

위 두 가지 방식으로 만든 싱글턴을 **직렬화** 할 경우 단순히 Serializable을 implements 한다면, 직렬화된 인스턴스를 역직렬화 할 때마다 새로운 인스턴스가 생성된다.

-> **더이상 싱글턴이 아니다.**

해결 방안 : 모든 인스턴스 필드를**transient**(일시적) 선언하고 **readResolve**메서드를 제공해야 한다.

```java
private Object readResolve(){
		return INSTANCE;
}
```

- 역직렬화 후 새로 생성된 객체를 인수로 이 메서드가 호출되고, 반환한 객체 참조가 새로 생성된 객체를 대신해 반환한다.
- 이 메서드는 역직렬화한 객체는 무시하고 클래스 초기화 때 만들어진 Elvis인스턴스를 반환한다.따라서 Elvis 인스턴스의 직렬화 형태는 실 데이터 필요 없으니 모든 인스턴스를 **transient**로 선언해야 한다.
1. **열거 타입 방식**의 싱글턴

```java
public enum Elvis{
	INSTANCE;
	public void leaveTheBuilding() { ... }
}
```

원소가 하나뿐인 열거타입으로 싱글턴을 만드는 것이 가장 좋은 방법이다.

단, 만들려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 사용할 수 없다. -> 열거 타입이 다른 인터페이스를 구현하도록 선언할 수는 있다.

- **싱글턴으로 클래스 생성 후 DAO 호출 시 고려 사항**

    정적(static) 멤버 필드에 @autowired 어노테이션으로 빈을 주입하는 방법
    [https://madplay.github.io/post/spring-framework-static-field-injection](https://madplay.github.io/post/spring-framework-static-field-injection)

    ```java
    @Component
    class Something {
    	public void sayHi() {
    		System.out.println("Hi");
    	}
    }

    class Someone {
    @Autowired
    public static SomeObject someObject;
    	public static void say() {
    		someObject.sayHi();	
    	}
    }

    public class InjectionTest {
    @Test
    public void method() {
    	// NullPointerException 발생
    	Someone.say();
    	}
    }
    ```

    Java 위 코드를 실행하면 NullPointerException이 발생한다. 이유는 Someone 클래스에서 @Autowired 되었을 것이라고 예상한 Something 객체가 Someone 클래스가 로드된 이후에 생성되기 때문에 Someone 클래스가 로드된 당시에는 Something 객체는 존재하지 않는다.

    그리고 핵심은 @Autowired는 스프링 프레임워크에 의해서 관리되어야 빈 주입을 해줄 수 있지만 현재 Someone 클래스는 그렇지 않다. 물론 관리 대상이라고 하더라도 클래스 로더에 의해 인스턴스화될 때 스프링 컨텍스트는 아직 로드되지 않아 정상적인 주입이 불가능하다.

    Someone 클래스의 객체가 생성될 때 주입받을 객체를 가져와 할당하면 된다.

    ```java
    class Someone {
    	public static SomeObject someObject;
    	
    	// 방법1) setter 메서드를 사용한다.
    	@Autowired
    	public void setSomeObject(SomeObject someObject) {
    		this.someObject = someObject;
    	}
    	
    	// 방법2) 생성자를 이용한다.
    	@Autowired
    	private Someone(SomeObject someObject) {
    		this.someObject = someObject;
    	}
    	
    	public static void say() {
    		someObject.sayHi();
    		}
    	}
    	
    	@Component
    	class Someone {
    		@Autowired
    		private SomeObject beanObject;
    		public static SomeObject someObject;
    		
    		@PostConstruct
    		private void initialize() {
    		this.someObject = beanObject;
    		}
    		
    		public static void say() {
    		someObject.sayHi();
    		}
    	}
    }
    ```

## **아이템4. 인스턴스화를 막으려거든 private 생성자를 사용하라.**

### **정적메서드와 정적 필드만을 담은 클래스 ( 유틸성 클래스 )**

java.lang.Math, java.util.Arrays 처럼 **기본타입** 값이나 배열관련 메서드를 모아두거나util.collectons 들과 같이 특정 인터페이스를 구현하는 개체를 생성해주는 **정적메서드(혹은 펙터리)를 모아둔 유틸성 클래스**
생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자(매개변수를 받지 않는 public 생성자)를 생성하고 사용자는 이 생성자가 자동생성된 것인지 구분할 수 없다.

추상클래스로 만든다면?
하위클래스를 만들어 인스턴스화가 가능하여 막을 수 없다.

### **해결방안 : private 생성자를 추가하여 막는다.**

```java
public class UtilityClass{
	//기본생성자가 만들어지는 것 방지 (인스턴스화 방지)
	private UtilityClass() {// 명시적 생성자가 private이므로 클래스 외부에서 접근 불가
		throw new AssertionError();
	}
}
```

## **아이템5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라.**

클래스들이 다른 클래스에 의존하는 경우 

(ex. 맞춤법검사기 클래스에서 사전 클래스라는 유틸리티 클래스를 사용한다.)

- 1. **정적 유틸리티 클래스** 2. **싱글턴**으로 구현하여 두 방법 모두 유연하지 않아 확장성이 없고 테스트가 어렵다.
(ex. 사전은 여러종류가 있는데 단 하나의 사전으로만 사용해야 한다.)
- final을 제거하고 다른 사전으로 교체하는 메서드는 멀티스레드 환경에서 사용할 수 없다.

그러므로, **사용하는 자원에 따라 동작이 달라지는 클래스에는 위 두 방법이 적합하지 않다.**

**해결방안**

### **의존 객체 주입: 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식**

맞춤법 검사기 클래스가 여러자원 인스턴스를 지원해야 하며, 클라이언트가 원하는 자원(사전)을 사용해야 하는 조건을 만족

```java
public class SpellChecker {

	private final Lexicon dictionary;
	
	// 여기서 의존성 주입을! (맞춤법 검사기를 생성할 때 의존객체인 사전을 주입)	
	public SpellChecker(Lexicon dictionary){	
		this.dictionary = Objects.requireNotNull(dictionary);	
	}	
	public static boolean isVaild(String word) {...}	
	public static List<String> suggestions(String typo) {...}	
}

// 인터페이스
interface Lexicon {}
// Lexicon을 상속 받아서 구현
public class myDictionary implements Lexicon {
	...
}

// 사용은 이렇게!
Lexicon dic = new myDictionary();
SpellChecker chk = new SpellChecker(dic);
chk.isVaild(word);
```

위 패턴의 응용 방법: **생성자에 자원 펙터리를 넘겨주는 방식**

> 팩터리란 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체(팩터리 메서드 패턴)

**Supplier<T> 인터페이스:** 팩터리를 표현함

한정적 와일드 카드 타입 (bounded wildcard type)으로 팩터리 타입 매개변수를 제한한다. (ITypeFactory.class)

클라이언트는 자신이 명시한 타입의 **하위 타입을 생성할 수 있는 팩터리를 넘길 수 있다.**

> IType의 하위 타입이어도 map에 put이 가능하다.

```java
public class IType {
	private static final int TYPE_Z = 0;
	private static final int TYPE_A = 1;
	private static final int TYPE_B = 2;
	
	final static Map<Integer, Supplier<? extends ITypeFactory>> map = **new** HashMap<>();
	
	static {
		map.put(TYPE_Z, ITypeFactory::new);
		map.put(TYPE_A, A::new);
		map.put(TYPE_B, B::new);
	}
}

class ITypeFactory {}
class A extends ITypeFactory {}
class B extends ITypeFactory {}
```

## **아이템6. 불필요한 객체 생성을 피하라.**

1. **객체의 재사용**: String 리터럴을 이용한 불필요 객체 생성 방지
똑같은 기능을 하는 객체를 매번 생성하는 것 보다 객체 하나를**재사용**하는 편이 더 나을 경우가 많고, 불변객체는 재사용을 보장하고 성능도 더 좋다.
> String s = "Effective Java"; //**String constant pool 영역**에 올라가 모든 코드가 똑같은 객체를**재사용**함을 보장한다.
(Java7 부터는 String constant pool 영역의 위치가 Perm > Heap으로 변경되어 **OOM**문제를 해결하고**GC**가 가능해졌다.)
> String s = new String("Effective Java"); // 매번**Heap영역**에 올라감, 동일한 문자열 값이라도**다른 인스턴스를 생성**한다.
2. **생성자 대신 정적 팩터리 메서드 사용:**Boolean(String)생성자 대신 Boolean.valueOf(String)팩터리 메서드 사용
3. **생성 비용이 비싼 객체는 캐싱하여 재사용**
ex) String.matches 메서드는 정규표현식용 pattern 인스턴스는 한번만 쓰고 버려져 GC대상이 된다.
Pattern 인스턴스는 입력받은 정규표현식에 해당하는 유한상태 머신(finite state machine)을 만들기 때문에 생성비용 높음
>해결: **Pattern 인스턴스를 클래스 초기화(정적 초기화) 과정에서 직접 생성해 캐싱한다. (상수로 선언하여 사용)**
    
    ```java
    public class RomanNumerals{
    	private static final Pattern ROMAN=Pattern.compile(	"^(?=.)M*(C[MD] | D?C{0,3})"+"(X[CL}|L?X{0,3})(I[XV]|V?I{0,3})$");
    	
    	staticbooleanisRomanNumeral(Strings){	
    	returnROMAN.matcher(s).matches();	
    	}
    }
    ```
  > isRomanNumeral메서드가 호출 될 때마다 Pattern 인스턴스 재사용
  초기화 된 후 해당 메서드를 한번도 호출하지 않으면 쓸데없이 초기화한 경우다 
  -> 이를 "**지연초기화**"(메서드가 처음 호출될 때 필드를 초기화 하는 방법)로 불필요한 초기화를 없앨 수 있으나 코드를 복잡하게 하거나 성능은 크게 개선되지 않을 때가 많기때문에 권장하는 방법이 아니다.

4. **어댑터(뷰) 패턴 사용**
어댑터(뷰)는 실제작업은 뒷단 객체에 위임하고, 자신은 제2의 인터페이스 역할을 해주는 객체다. 뒷단 객체 하나당 어댑터하나만 만들어지면 된다.
ex) Map 인터페이스의 keySet메서드 : Map 객체 안의 키 전부를 담은 Set 뷰를 반환 -> 호출 될 때마다 같은 객체 반환
즉, 모두 같은 Map 객체를 대변하기 때문에 반환된 Set 객체 중 하나를 수정하면 모든 객체가 변경된다.
5. **Auto boxing(오토박싱) 사용 주의 : 기본타입과 그에 대응하는 박싱된 기본 타입의 구분을 흐려주지만, 완전히 없애주는 것은 아니다.**
> 박싱된 기본 타입(Long)보다는 기본타입(long)을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의해야 한다.
6. **단순히 객체 생성을 피하고자 본인만의 객체 풀(Pool)을 만들지 않기**
> 요즘 JVM의 GC가 최적화 되어있어 생성 비용이 워낙 비싼 데이터베이스 연결이나 아주 무거운 객체가 아닌 경우에는 pool 생성 비추

아이템 50 : 새로운 객체를 만들어야 한다면 기존 객체를 재사용하지 마라 : 방어적 복사 -> 버그와 보안으로 이어짐

아이템 6 : 불필요한 객체 생성을 피하라 -> 코드형태 성능에 영향

## **아이템7. 다 쓴 객체 참조를 해제하라.**

### **메모리 누수 현상 원인**

1. **메모리를 직접 관리하는 클래스** 
가비지 컬렉션을 통해 소멸 대상이 되는 객체가 되기 위해서는 어떠한 reference 변수에서 가르키지 않아야 한다.
다 쓴 객체에 대한 참조를 해제하지 않으면 가비지 컬렉션의 대상이 되지 않아 계속 메모리가 할당 되는 **메모리 누수** 현상이 발생 된다.
ex) **Stack**
Stack이 자원을 pop 하기만 한다고 GC 대상이 되지는 않는다.
일반적으로 구현하는 pop 의 로직에는 stack size 를 -1 해줄 뿐이기 때문이다.
pop 로직에서**객체 참조를 직접 null 처리**해주어야 한다.
2. **캐시**
객체 참조를 캐시에 넣고 정리하지 않으면 계속 쌓이게 되어 캐시의 역할을 하지 못한다.
    - **WeakHashMap 사용하여 캐시 생성**
    일반적인 HashMap 은 사용여부에 관계없이 Key - Value 쌍을 지우지 않는다.
    WeakHashMap 은 특정 key 값이 더이상 사용되지 않는다고 판단되면 해당 Key - Value 쌍을 삭제해준다.
    **캐시의 경우 시간이 지남에 따라 사용되지 않으면 그 가치를 떨어트리는 방법을 사용한다.**
    > Scheduled ThreadPoolExecutor -> 백그라운드 스레드 사용
    > LinkedHashMap.removeEldestEntry -> 새로운 엔트리 추가 될 때 사용된지 가장 오래된 엔트리 삭제
3. **리스너(listener) 혹은 콜백(callback)**
클라이언트가 리스너와 콜백을 등록만하고 해지하지 않으면 계속 쌓인다.
4. **약한 참조(Weak reference)로 저장하여 GC 대상이 되도록 한다.**

## **아이템8. finalizer와 cleaner 사용을 피하라.**

finalizer는 예측할 수 없고, 상황에 따라 위험할 수 있어 불필요하다.

( 오작동, 낮은 성능, 이식성 문제의 원인)

cleaner는 finalizer 보다 덜 위험하지만, 여전히 예측할 수 없고, 느리고, 일반적으로 불필요하다.

### **finalizer와 cleaner 사용을 피해야 하는 이유**

1. **즉시 수행된다는 보장이 없다.**
실행되기까지 얼마나 걸릴지 알 수 없기 때문에(GC 알고리즘에 달려있다.) **제때 실행되어야 하는 작업은 절대 할 수 없다.**
2. **수행 여부조차 보장되지 않기 때문에 상태를 영구적으로 수정하는 작업에서는 절대 finalizer와 cleaner에 의존해서는 안된다.**
ex) DB 같은 공유 자원의 영구 락 해제를 맡겨 놓으면 분산 시스템 자체가 서서히 멈출 것이다.
3. **finalizer 동작 중 발생한 예외는 무시되며, 처리할 작업이 남아있더라도 그 순간 종료된다.**
잡지 못한 예외 때문에 해당 객체는 마무리가 덜 된 상태로 남을 수 있고, 다른 스레드가 이러한 훼손된 객체를 사용할 수도 있다.
보통 잡지 못한 예외가 스레드를 중단 시키고 스택 추적 내역을 출력하지만 finalizer는 경고조차 출력하지 않는다. 그나마 cleaner는 자신의 스레드 통제하기 때문에 이런 문제는 없다.
4. **finalizer와 cleaner는 GC의 효율을 떨어트리기 때문에 심각한 성능 문제도 동반한다.**
5. **finalizer를 사용한 클래스는 finalizer 공격에 노출되어 심각한 보안 문제를 일으킬 수 있다.**
생성자나 직렬화 과정에서 예외가 발생하면 finalizer가 수행되는데, 이 생성되다 만 객체에서 finalizer를 악의적으로 오바라이딩한 하위 클래스의 finalizer가 수행될 수 있다. 심지어 이 finalizer를 정적 필드에 할당하면 GC에 의해 수거되지도 않는다.
> 해결책: **finalizer를 final로 선언하여 오버라이딩을 막는다.**

### **대안: AutoCloseable 구현**

종료해야 할 자원(파일이나 스레드)을 담고 있는 클래스에 AutoCloseable을 implements하고 인스턴스를 다 사용하면 close 메서드를 호출하도록 하면 된다.

> 예외가 발생할 경우 제대로 종료되도록 **try-with-resources**를 사용한다. (try-catch문을 추가하면 더 좋다.)
ex) close메서드 호출여부를 필드로 저장하고, 객체가 닫힌 후 호출되면 IllegalStateException을 던진다.

### **finalizer와 cleaner의 사용처**

1. **close메서드를 호출하지 않는 것에 대비한 안전망 역할**(AutoCloseable 구현하지 않았을 때) -> 자원 회수를 아예 하지 않는 것보다 늦게라도 하는 것이 낫다.
FileInputStream, FileOutputStream, ThreadPoolExecutor 는 안전망 역할의 finalizer를 제공한다.
2. **네이티브 피어(Native peer)와 연결된 객체에서 사용**
네이티브 피어: 일반 자바객체가 네이티브 메서드를 통해 기능을 위임한 객체이며, **자바 객체가 아니므로 GC의 대상이 되지 못해 자원 회수가 불가능**하다.
> 네이티브 피어가 중요하지 않은 자원일 경우만 사용하고, 즉시 회수해야 하며 **close() 메서드**를 사용해야 한다.

## **아이템9. try-finally보다는 try-with-resources를 사용하라.**

자바에서는 close()메서드를 호출하여 직접 닫아줘야 하는 자원이 많다.
ex) InputStream, OutputStream, java.sql.Connection 등

### **try-finally**

단일 자원일 경우 상관없지만, 둘 이상의 자원일 경우 코드가 지저분하다.
예외는 try블록 뿐만아니라 finally에서도 발생하는데, 기기에 물리적인 문제가 생기면 try에서도 예외, finally 블록에서도 예외를 발생시켜, 두번째 예외가 첫번째 예외를 덮어 이전의 예외를 찾을 수 없다.

```java
InputStream in = new FileInputStream(str1);
try {
	OutputStream out = new FileOutputStream(str2);
	try {
		...
	}finally{
		out.close();
	}
} finally {
	in.close();
}
```

### **try-with-resources**

```java
try (InputStream in = new FileInputStream(str1);
		OutputStream out = new FileOutputStream(str2)) {
		...
}
```

이 구조를 사용하기 위해서는 단순히 void를 반환하는 close 메서드 정의한 **AutoCloseable** 인터페이스 구현이 필요하다.

**try 내 작업이 종료되면 자동으로 자원 회수** (try-catch문을 추가하면 더 좋다.)

try-with-resources 구문은try블록과 finally블록(close())의 양쪽 호출에서 예외가 발생하면, **close()에서 발생한 예외는 숨겨지고 try블록에서 발생한 예외가 기록된다.**

> 숨겨진 예외들은 버려지지 않고, 스택 추적내역에 표시된다. (Throwable에 추가된 getSuppressed 메서드 이용)
