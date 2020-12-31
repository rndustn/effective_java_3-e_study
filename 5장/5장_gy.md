# 5장. 제네릭

- [ ]  제네릭은 자바5 부터 사용할 수 있다.
- [ ]  제네릭 지원 전에는 컬렉션에서 객체를 꺼낼 때 마다 형변환을 해야 했다.
- [ ]  제네릭을 사용하면 컬렉션이 담을 수 있는 타입을 컴파일러에 알려 주고 컴파일러는 알아서 형변환 코드를 추가한다.
- [ ]  더 안전하고 명확한 프로그램을 만들어 준다.
- [ ]  꼭 컬렉션이 아니라도 이러한 이점을 누릴 수 있으나 코드가 복잡해지는 단점이 있다.

# 아이템26. 로 타입은 사용하지 말라

- **제네릭 클래스 / 제네릭 인터페이스** : 클래스와 인터페이스 선언에 타입 매개변수가 쓰인다.
ex) List 인터페이스는 원소와 타입을 나타내는 타입 매개변수 E를 받는다.  List<E>
- **제네릭 타입(generic type)** : 제네릭 클래스와 제네릭 인터페이스를 통틀어 의미한다.
- **매개변수화 타입(parameterized type) :** 각각의 제네릭 타입은 매개변수화 타입을 정의한다.
표현법 : 클래스(or 인터페이스)이름<타입 매개변수> ex) List<String>
- **로 타입(raw type)** : 제네릭 타입에서 타입 매개변수를 전혀 사용하지 않을 때를 말한다.
→ 제네릭 타입을 하나 정의하면 그에 딸린 로 타입도 함께 정의된다.
ex) List<E>의 로 타입은 List다.
로 타입은 제네릭 타입 정보가 전부 지워진 것처럼 동작하는데, 제네릭이 도래 전 코드와 호환되기 위한 대책이지만 좋은 예는 아니다.

## 👎🏻로 타입의 문제점

- **컴파일 타임에 타입 정보를 알지 못 한다.**

    ```java
    private final Collection stamps = ...;
    stamps.add(new Coin(...));
    // unchecked call "경고"를 호출하지만 컴파일도 되고 실행도 됩니다.
    ```

    → add한 Coin 객체를 꺼내 stamp 변수에 할당하는 런타임 순간에야 ClassCastException을 발생한다.

**문제점 해결방법 : 제네릭 활용**

- 매개변수화된 컬렉션 타입을 사용하여 타입 안정성을 확보한다.

    ```java
    private final Collection<Stamp> stamps = ...;
    stamps.add(new Coin()); // 컴파일 오류 발생
    ```

## ❗로 타입은 절대로 사용하지 말자.

### ✔️로 타입을 사용하는 이유 - 마이그레이션 호환성

- 제네릭 도입 이전 코드를 모두 수용하면서 제네릭을 사용하는 새로운 코드와의 **호환성**을 위해서 사용한다.
→ 로 타입을 사용하는 메서드에 매개변수화 타입의 인스턴스를 넘겨도 동작 가능하도록 **로 타입을 지원하고 제네릭 구현에는 소거(erasure) 방식**을 사용.

### ✔️로 타입을 사용하면 안 되는 이유 - 제네릭의 안정성과 표현력을 모두 잃는다.

- List<Object>같은 매개변수화 타입을 사용할 때와 달리 List 같은 **로 타입을 사용하면 타입 안정성을 잃게 된다. → 컴파일 단계에서 오류를 찾을 수 없기 때문에**
- List 같은 로 타입은 사용해서는 안 되나, List<Object>처럼 임의 객체를 허용하는 매개변수화 타입은 괜찮다.
- **로 타입 List** : 제네릭 타입에서 완전히 발을 뺀 것
- **매개변수화 타입 List<Object>** : 모든 타입을 허용한다고 컴파일러에게 명확히 의사 전달한 것
- **제네릭 하위타입 규칙** : 매개변수로 List를 받는 메서드에 List<String>은 넘길 수 있지만, List<Object>를 받는 메서드에는 넘길 수 없다.
→ **List<String>은 로타입인 List의 하위 타입이지만, List<Object>의 하위 타입은 아니다.**

## 📎비한정적 와일드카드 타입(unbounded wildcard type)

제네릭 타입을 쓰고 싶지만 **실제 타입 매개변수를 지정하지 않으려면 <?>** 를 사용하자.

<?> : 어떤 타입이라도 담을 수 있는 가장 범용적인 매개변수화 타입이다.

```java
static int numElementsInCommon(Set<?> s1, Set<?> s2){...}
```

### ✔️비한정적 와일드카드 타입과 로타입의 차이

1. **로 타입**
    - 불안전하다.
    - 아무 원소나 넣을 수 있어 타입 불변식 훼손하기 쉽다.
2. **비한정적 와일트 카드 타입**
    - 안전하다.
    - **Collection<?>에는 (null 외에는) 어떤 원소도 넣을 수 없다.**
    - 다른 원소 넣으려 하면 컴파일 오류
    - 어떤 원소도 못들어가니 컬렉션에서 꺼낼 수 있는 객체의 타입도 알 수 없다.
- 예시 코드

    ```java
    public class TypeTest {
        private static void addtoObjList(final List<Object> list, final Object o) {
            list.add(o);
        }
        private static void addToWildList(final List<?> list, final Object o) {
            // null 외에 허용되지 않는다
            list.add(o);
        }
        private static <T> void addToGenericList(final List<T> list, final T o) {
            list.add(o);
        }
        public static void main(String[] args) {
            List<String> list = new ArrayList<>();
            String s = "kimtaeng";

            // 아무 타입이나 Okay! 하지만 메서드에서 오류 null아니면 오류
            addToWildList(list, s);

            // List<Object> 이므로 incompatible types 오류
            addtoObjList(list, s);
            
            // Okay!
            addToGenericList(list, s);
        }
    }

    public class TypeTest2 {
        public static void main(String[] args) {
            List raw = new ArrayList<String>(); // Okay!
            List<?> wildcard = new ArrayList<String>(); // Okay!
            List<Object> generic = new ArrayList<String>();//컴파일오류>**제네릭하위타입규칙**
                
            raw.add("Hello"); // Okay! 하지만 raw타입은~
            wildcard.add("Hello"); // 컴파일 오류
            wildcard.size(); // Okay! 제네릭 타입에 의존성 없음
            wildcard.clear(); // Okay! 제네릭 타입에 의존성 없음
        }
    }
    ```

## 📍로 타입 사용의 예외

### ✔️**class 리터럴에는 로 타입을 써야 한다.**

- 자바 명세는 class 리터럴에 매개변수화 타입을 사용하지 못하게 했다.(배열과 기본타입은 허용)
- List.class, String[].class, int.class는 허용 한다.
- List<String>.class, List<?>.class는 허용하지 않는다.

### ✔️instanceof 연산자

- 런타임에는 제네릭 타입 정보가 지워지므로 instanceof 연산자는 비한정적 와일드카드 타입 외의 매개변수화 타입에는 적용할 수 없다.
- 로 타입이든 비한정적 와일드카드 타입이든 instanceof는 완전히 똑같이 동작한다.
→ 표현이 지저분하니 로타입 사용 깔끔!
- 제네릭 타입에 instanceof 사용하는 올바른 예

```java
// o의 타입이 Set인지 확인한 다음, 와일드카드 타입으로 형변환해야 한다.
// 여기서 로 타입인 Set이 아닌 와일드카드 타입으로 변환함에 주의!
if( o instanceof Set) { //로 타입
    Set<?> s = (Set<?>) o; // 와일드카드 타입으로 형변환해야 검사형변환으로 컴파일경고 안뜸
}
```

## ✏️핵심정리

- 로 타입을 사용하면 런타임 예외가 발생할 수 있으니 사용하면 안 된다.
- 로 타입은 제네릭 도입 이전 코드와의 호환성을 위해 제공될 뿐이다.
- Set<Object>는 어떤 타입의 객체도 저장할 수 있는 매개변수화 타입
- Set<?>는 모종의 타입 객체만 저장할 수 있는 와일드카드 타입
- Set은 제네릭 타입에서 발 뺀 로 타입이다.
- Set<Object>, Set<?>은 안전하고 Set은 안전하지 않다.

# 아이템27. 비검사 경고를 제거하라

제네릭을 사용하기 시작하면 수 많은 컴파일러 경고를 볼 수 있다.

- 비검사 형변환 경고
- 비검사 메서드 호출 경고
- 비검사 매개변수화 가변인수 타입 경고
- 비검사 변환 경고

## ✔️**가능한 모든 비검사 경고를 제거하자 → 타입 안정성 보장**

컴파일 명령줄 인수에 -Xlint:unchecked 옵션을 추가하면 자세한 코멘트가 보이고 컴파일러가 알려준대로 수정하면 경고가 사라진다.

```java
Set<Lark> exaltation = new HashSet();

Venery.java:4: warning: [unchecked] unchecked conversion
    Set<Lark> exaltation = new HashSet();
                            ^
required: Set<Lark>
found:    HashSet
```

자바7부터 제네릭 타입추론이 가능하여 (<>)연산자로 변경하면 경고가 사라진다.

```java
Set<Lark> exaltation = new HashSet<>();
```

## ✔️ @SuppressWarnings("unchecked")

경고를 제거할 수 없지만 타입 안전하다고 확신하면 annotation을 달아 경고를 숨기자.

단, 타입 안전검증 없이 경고를 숨기면 경고없이 컴파일은 되겠지만, 런타임 오류가 발생할 수 있다.

반대로, 안전검증된 경고를 숨기지 않고 두면, 거짓 경고 속 진짜 문제되는 경고가 묻힐 수 있다.

### 📍@SuppressWarnings 애너테이션은 항상 가능한 한 좁은 범위에 적용하자.

@SuppressWarnings 애너테이션은 개별 지역변수 선언부터 클래스 전체까지 달 수 있다.

단, 자칫 심각한 경고를 놓칠 수 있으니 변수 선언, 아주 짧은 메서드, 혹은 생성자에만 적용하고 **절대로 클래스에는 적용해서는 안 된다.**

### 📍@SuppressWarnings 사용할 땐 그 경고를 무시해도 안전한 이유를 주석으로 남겨야 한다.

다른사람의 코드 이해를 도와 잘못 수정하여 타입 안정성을 잃는 상황을 줄여 준다.

## ✏️핵심정리

- 비검사 경고는 중요하니 무시하지 말자.
- 모든 비검사 경고는 런타임에 ClassCastException을 일으킬 수 있는 잠재적 가능성을 뜻하므로 최대한 제거하자.
- 경고를 없앨 수 없다면, 코드의 타입안정성을 증명하고 가능한 한 범위를 좁혀 @SuppressWarnings("unchecked") 애너테이션으로 경고를 숨기자.
- 그 다음 경고를 숨기는 이유를 주석으로 남겨야 한다.

# 아이템28. 배열보다는 리스트를 사용하라.

## 📎제네릭과 배열의 차이

### ✔️배열

- 공변이다.
- Sub가 Super의 하위 타입이라면 배열Sub[]는 배열Super[]의 하위 타입이 된다.→ 함께 변한다.
- 타입 오류를 **런타임**에서 알 수 있다.

    ```java
    Object[] objectArray = new Long[1];
    objectArray[0] = "타입이 달라 넣을 수 없다" // ArrayStoreException
    ```

- **실체화(reify) 된다.**
→ 배열은 런타임에도 자신이 담기로 한 원소의 타입을 알고 확인한다.

### ✔️제네릭

- 불공변이다.
- 서로 다른 Type1과 Type2가 있을 때, List<Type1>은 List<Type2>의 하위/상위 타입도 아니다.
- 타입 오류를 **컴파일**할 때 바로 알 수 있다.

    ```java
    List<Object> ol = new ArrayList<Long>();    // 호환되지 않는 타입이다.
    ol.add("타입이 달라 넣을 수 없다.")
    ```

- **타입 정보가 런타임에는 소거(erasure) 된다.
→** 원소 타입을 컴파일타임에만 검사하며 런타임에는 알 수 없다.

## 📍제네릭 배열은 생성이 불가하다.

배열은 **제네릭 타입**(new List<E>[]), **매개변수화 타입**(new List<String>[]), **타입 매개변수**(new E[])로 작성하면 컴파일 때 제네릭 배열 생성오류를 발생한다.

```java
Object[] objects = new List<String>[1]; // 컴파일 에러 발생
```

→ **타입 안전하지 않기 때문에 컴파일러가 자동 생성한 형변환 코드에서 런타임 ClassCastException을 발생 할 수 있다.**

## 📎실체화 불가 타입

- E, List<E>, List<String> 같은 타입
- **실체화되지 않아서 런타임에는 컴파일타임보다 타입 정보를 적게 가지는 타입**
- 소거 메커니즘 때문에 매개변수화 타입 중 실체화될 수 있는 타입은 List<?>와 Map<?,?>같은 비한정적 와일드카드 타입 뿐이다. → 만들 수 있지만 유용하지 않음

## 👎🏻배열을 제네릭으로 만들 수 없어 생기는 불편함

- 제네릭 컬렉션에서는 자신의 원소 타입을 담은 배열반환이 불가하다.

    ```java
    List<T> genericList = new ArrayList<T>();
    E[] genericArray = genericList.toArray(); // 컴파일 에러 발생
    ```

- 제네릭 타입과 가변인수 메서드를 함께 쓰면 해석이 어려운 경고가 뜬다.
→ 가변인수 메서드를 호출할 때마다 **가변인수 매개변수를 담을 배열이 하나 생성**되는데, 이때 그 **배열의 원소가 실체화 불가 타입이면 경고**가 발생한다. → @SafeVarargs애너테이션으로 대처

## ✅배열대신 List를 사용하자.

배열로 형변환할 때 제네릭 배열 생성 오류나 비검사 형변환 경고가 뜨는 경우 대부분 배열인 E[] 대신 컬렉션인 List<E>를 사용하면 해결된다.

→ 코드는 복잡해지나, 타입 안정성과 상호운용성이 좋아진다.

```java
public class Chooser {
	private final Object[] choiceArray;

	public Chooser(Collection choices) {
		choiceArray = choices.toArray();
	}

	public Object choose() {
		Random random = ThreadLocalRandom().current();
		return choiceArray[random.nextInt(choiceArray.length)];
	}
}
```

→ choose 메서드를 호출할 때마다 반환된 Object를 원하는 타입으로 형변환해야 한다.

```java
public class Chooser<T> {
	private final T[] choiceArray;

	public Chooser(Collection<T> choices) {
		choiceArray = (T[]) choices.toArray(); // T가 무슨 타입인지 알 수 없어서 경고 발생
	}

	public T choose() {
		Random random = ThreadLocalRandom().current();
		return choiceArray[random.nextInt(choiceArray.length)];
	}
}
```

→ T가 무슨 타입인지 알 수 없어 컴파일러는 형변환이 런타임에 안전하지 않다는 경고를 띄운다.

해결방법

1. 코드가 안전하다고 확신하면 애너테이션을 달아 경고를 숨긴다.
2. **배열대신 리스트를 쓴다.** 

```java
public class Chooser<T> {
	private final List<T> choiceList;

	public Chooser(Collection<T> choices) {
		choiceList = new ArrayList<>(choices);
	}

	public T choose() {
		Random random = ThreadLocalRandom().current();
		return choiceList.get(random.nextInt(choiceList.size()));
	}
}
```

→ 경고도 제거 되고 ClassCastException도 발생하지 않는다.

## ✏️핵심정리

- 배열은 공변이고 실체화 되는 반면, 제네릭은 불공변이고 타입 정보가 소거된다.
- 배열은 런타임에 타입 안전하지만, 컴파일타임에는 불안전하다. 제네릭은 반대다.
- 배열과 제네릭을 섞어 사용하다 컴파일 오류나 경고가 발생하면 배열을 리스트로 대체해 본다.

# 아이템29.이왕이면 제네릭 타입으로 만들라

Object 기반 스택

스택에서 꺼낸 객체를 형변환해야 해서 런타임 오류가 날 수 있다. → 제네릭 타입이어야 한다.

```java
public class Stack{
	private Object[] elements;
	...
	public Stack(){
	    elements = new Object[DEFAULT_INITIAL_CAPACITY];
	}
...
}
```

## 🔍제네릭 클래스로 만드는 방법

### ✔️클래스 선언에 타입 매개변수를 추가한다.

- 스택이 담을 원소의 타입 하나를 추가한다. 이때 타입 이름으로는 보통 E를 사용한다. Stack<E>
- 그 다음 코드에 쓰인 Object를 적절한 타입 매개변수로 바꾼다.
- 컴파일 되지 않는다.
→ E와 같은 실체화 불가 타입으로는 배열을 만들 수 없다.

    ```java
    public class Stack<E>{
    	private E[] elements;
    	...
    	public Stack(){
    	    elements = new E[DEFAULT_INITIAL_CAPACITY]; //generic array creation 오류
    	}
    ...
    }
    ```

### 🔑해결책

### ✔️**제네릭 배열 생성을 금지하는 제약을 대놓고 우회하는 방법**

1. **Object 배열을 생성한 다음 제네릭 배열로 형변환 한다. (배열타입을 E[]로)**

    ```java
    public class Stack<E> {
        private E[] elements;
        private int size = 0;
        private static final int DEFAULT_INITIAL_CAPACITY = 16;
       
    	 @SuppressWarnings("unchecked")
        public Stack(){
          elements = (E[]) new Object [DEFAULT_INITIAL_CAPACITY]}
    		}
        
        public void push(E e) {
        	ensureCapacity();
            elements[size ++] = e;
        }
        
        public E pop() {
        	if (size == 0)
            	throw new EmptyStackException();
            return elements[--size];
        }
        
        // 나머지 메서드는 그대로다

    }
    ```

    → 컴파일러는 오류 대신 타입 안전하지 않다고 경고를 내보낸다.

2. 비검사 형변환이 프로그램의 **타입 안정성을 해치치 않음을 확인한다.**
    - 배열 elements는 private에 저장되고, 클라이언트로 반환되거나 다른 메서드에 전달되는 일이 없다
    - push메서드를 통해 배열에 저장되는 원소의 타입은 항상 E다.
    - 비검사 형변환은 확실히 안전하다.
    - 하지만, 이 배열의 런타임 타입은 E[]가 아닌 Object[]다.
3. 비검사 형변환이 안전하면 범위를 최소로 좁혀 **@SuppressWarnings** 애너테이션으로 경고를 숨긴다.

👍🏻장점

- 가독성이 좋다.
- 오직 E타입 인스턴스만 받음을 선언
- 코드가 더 짧다.
- 형변환을 배열 생성 시 단 한번만 해주면 된다.
- 현업에서 더 선호 한다.

👎🏻단점

- E가 Object가 아닌 한 배열의 런타임 타입이 컴파일타임 타입과 달라 힙 오염을 일으킨다.

### ✔️**elements 필드의 타입을 E[]에서 Object[]로 바꾸는 방법**

```java
Object[] elements = new Object[10];
E result = (E) elements[--size]; // 형변환이 필요한 시점에 명시적 형변환
```

1. (E) 형변환 안하면 타입 불안정 오류가 뜨고, 형변환하면 E는 실체화 불가 타입이므로 컴파일러는 런타임에 이뤄지는 형변환이 안전한지 증명할 방법이 없다.
2. pop 메서드 전체에 경고를 숨기지 말고, 비검사 경고를 적절히 숨긴다.

    ```java
    public E pop(){
                // 비검사 경고를 적절히 숨긴다.
          if(size == 0){
               throw new EmptyStackException();
          }
     
          @SuppressWarnings("unchecked")
          E result = (E) elements[--size];
     
          elements[size] = null; // 다 쓴 참조 해제
          return result;
    }
    ```

👍🏻장점

- 힙 오염이 발생하지 않는다.

👎🏻단점

- 배열에서 원소를 읽을 때마다 형변환을 해줘야 한다.

## 배열보다 리스트를 우선하라 아이템28. 의 참고사항

- 제네릭 타입 안에서 리스트 사용이 항상 가능하진 않고, 꼭 더 좋은 것은 아니다.
- 자바가 리스트를 기본타입으로 제공하지 않기에 ArrayList같은 제네릭 타입도 기본타입인 배열을 사용해 구현해야 한다.
- HashMap 같은 제네릭 타입은 성능을 높일 목적으로 배열을 사용한다.

## 제네릭타입의 타입 매개변수

- 아무런 제약을 두지 않는다.
Stack<Object>, Stack<int[]>, Stack<List<String>>, Stack 등 어떤 참조타입도 가능
- 단, **기본 타입은 사용할 수 없다.**
Stack<int>, Stack<double> 불가 → Stack<Integer>, Stack<Double> 박싱된 기본타입으로 우회가능

## ✏️핵심정리

- 클라이언트에서 직접 형변환해야 하는 타입보다 제네릭 타입이 더 안전하고 쓰기 편하다.
- 새로운 타입을 설계할 때는 형변환 없이도 사용할 수 있도록 한다. → 제네릭 타입으로 생성

# 아이템30. 이왕이면 제네릭 메서드로 만들라

매개변수화 타입을 받는 정적 유틸리티 메서드는 보통 제네릭이다.

Collections의 알고리즘메서드 (binarySearch, sort 등)는 모두 제네릭이다.

- 로 타입 사용하여 경고가 발생하는 아래 코드

    ```java
    public static Set union(Set s1, Set s2){
            Set result = new HashSet(s1);
            result.addAll(s2);
            return result;
        }
    ```

## 해결책 : 제네릭 메서드로 만들자

1. 메서드를 타입 안전하게 만들어야 한다.
2. 메서드 선언에서의 세 집합(입력2개, 반환1개)의 원소 타입을 타입 매개변수로 명시한다.
3. 메서드 안에서도 이 타입 매개변수만 사용하게 수정한다.
**(타입 매개변수들을 선언하는) 타입 매개변수 목록은 메서드의 제한자와 반환 타입 사이에서 온다.** → 예시에서 타입 매개변수 목록은 <E>, 반환타입은 Set<E>
- 수정 된 제네릭 메서드 → **타입 안전, 쓰기도 쉽다, 사용 시 직접 형변환하지 않아도 된다.**

    ```java
    public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
        Set<E> result = new HashSet<>(s1);
        result.addAll(s2);
        return result;
    }
    ```

- 위의 단순한 제네릭 메서드 활용

    ```java
    public static void main( final String[] args ){
        Set<String> guys = Set.of("톰", "딕", "해리");
        Set<String> stooges = Set.of("래리", "모에", "컬리");
        Set<String> aflCio = union(guys, stooges);
        System.out.println(aflCio);
    }
    ```

    union 메서드는 집합3개(입력2개, 반환1개)의 타입이 모두 같아야 한다.

    → 한정적 와일드카드 타입을 사용하면 유연하게 개선할 수 있다.(타입이 달라도 된다.)

## 불변 객체를 여러 타입으로 활용- 제네릭 싱글턴 팩터리

- 제네릭은 런타임에 타입 정보가 소거되어 하나의 객체를 어떤 타입으로든 매개변수화 가능하다.
- 요청한 타입 매개변수에 맞게 매번 그 객체의 타입을 바꿔주는 정적 팩터리가 필요하다.
- 이 패턴을 **제네릭 싱글턴 팩터리**라 한다.
- 제네릭 싱글턴 팩터리 패턴의 예시)  항등함수 (identity function)
→ 항등함수 객체는 상태가 없으니 요청할 때마다 새로 생성하는 것은 낭비다 → 싱글턴

```java
private static UnaryOperator<Object> IDENTITY_FN = (t) ->t;
    
@SuppressWarnings("unchecked")
public static <T> UnaryOperator<T> identityFunction(){
    return (UnaryOperator<T>) IDENTITY_FN;
}
```

## 재귀적 타입 한정(recursive type bound)

자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정할 수 있다.

주로 타입의 자연적 순서를 정하는 Comparable인터페이스와 함께 쓰인다.

```java
public interface Comparable<T> {
  public int compareTo(T o);
}
```

타입 매개변수 T는 Comparable<T>를 구현한 타입이 비교할 수 있는 원소의 타입을 정의한다.

거의 모든 타입은 자신과 같은 타입의 원소와만 비교할 수 있다.

String은 Comparable<String>을 구현, Integer는 Comparable<Integer>를 구현

### 재귀적 타입 한정을 이용해 상호 비교할 수 있다.

```java
public static <E extends Comparable<E>> E max(Comparable<E> c);
```

타입 한정인 <E extends Comparable<E>>는 **"모든 타입 E는 자기자신과 같은 타입의 원소와 비교 할 수 있다."**

→ 재귀적 타입 한정은 E로 나타내는 제네릭 타입이 compareTo 메서드를 사용가능 하게 해야 하므로, compareTo를 구현한 클래스만 E 타입매개변수로 사용하라

## ✏️핵심정리

- 제네릭 타입과 마찬가지로, 클라이언트에서 입력 매개변수와 반환값을 명시적으로 형변환해야 하는 메서드보다 제네릭 메서드가 더 안전하며 사용하기도 쉽다.

# 아이템31. 한정적 와일드카드를 사용해 API 유연성을 높이라

📍매개변수화 타입(제네릭)은 불공변이다. → 이보다 유연성을 높이기 위해 와일드 카드 사용하자

- 기존 불공변 방식의 예시 코드(아이템29 public class Stack<E> ) 에 pushAll 메서드 추가
    1. **와일드카드 타입을 사용하지 않는 pushAll 메서드**

    ```java
    // 매개변수의 원소들을 스택에 넣는다.
    public void pushAll(Iterable<E> src) {
        for (E e : src) {
            push(e);
        }
    }
    ```

    ```java
    Stack<Number> numberStack = new Stack<>();
    Iterable<Integer> integers = ...;
    numberStack.pushAll(integers);
    ```

    → 컴파일은 되지만 Iterable src의 원소 타입이 스택의 원소 타입과 일치할 경우만 정상작동 한다.
    Stack<Number>로 선언 후 integers 이 Integer 타입인 pushAll(integers)을 호출하면 **불공변이기때문에 오류가 발생한다.**

    2. **와일드카드 타입을 사용하지 않는 popAll 메서드**

    ```java
    // 모든 원소를 매개변수로 전달받은 컬렉션에 옮긴다.
    public void popAll(Collection<E> dst) {
        while(!isEmpty()) {
            dst.add(pop());
        }
    }
    ```

    ```java
    Stack<Number> numberStack = new Stack<>();
    Collection<Object> objects = Arrays.asList(new Object());
    // incompatible types...
    numberStack.popAll(objects);
    ```

    → push와 마찬가지로 컴파일되지만, Collection<Object>는 Collection<Number>의 하위타입이 아니다라는 오류가 발생한다.

## ✔️한정적 와일드카드 타입

유연성을 극대화하려면 원소의 생산자나 소비자용 입력매개변수에 와일드카드 타입을 사용하라.
**PECS : Producer-Extends, Consumer-Super**

### 1. 생산자(producer)매개변수에 와일드카드 타입 적용

```java
// class Integer extends Number ...
public void pushAll(Iterable<? extends E> src) {
    for (E e : src) {
        push(e);
    }
}
```

매개변수는 'E의 Iterable'이 아니라 **'E의 하위 타입의 Iterable'**이다.

Number 클래스를 상속하는 Integer, Long, Double 등의 타입 요소를 가질 수 있게 된다.

### 2. 소비자(consumer)매개변수에 와일드카드 타입 적용

```java
// E의 상위 타입의 Collection이어야 한다.
public void popAll(Collection<? super E> dst) {
    while(!isEmpty()) {
        dst.add(pop());
    }
}
```

매개변수의 타입이 'E Collection'이 아니라 **'E의 상위 타입의 Collection'**이어야 한다. (모든 타입은 자기 자신의 상위 타입이다.)

## ✔️와일드카드 타입 적용의 예

1. T생산자 매개변수에 와일드카드 타입 적용(아이템28)

    ```java
    public Chooser(Collection<? extends T> choices)
    ```

    → Chooser<Number>의 생성자에 List<Integer>를 넘길 수 있다. 
    : Number의 하위타입의 Collection List

2. Set 컬렉션의 union메서드(아이템30 30-2 )

    ```java
    public static <E> Set<E> union(Set<E> s1, Set<E> s2){}
    //와일드카드 적용 -> **<E> Set<E> 반환타입은 와일드 사용하면 안 된다!**
    public static **<E> Set<E>** union(Set<? extends E> s1, Set<? extends E> s2){}
    public static void main(String[] args) {
        // Set.of 메서드는 java 9 이상부터 지원
        Set<Double> doubleSet = Set.of(1.0, 2.1);
        Set<Integer> integerSet = Set.of(1, 2);
        Set<Number> unionSet = union(doubleSet, integerSet);
    }
    ```

    → 입력2, 반환1의 타입이 모두 달라도 가능해진다.

    **❗반환타입에는 한정적 와일드카드 타입을 사용하면 클라이언트 코드에서도 와일드카드 타입을 써야하기 때문에 사용하지 않는다.**

    자바8이전 버전을 사용하면 타입추론이 어렵기 때문에 명시적 타입인수를 사용해야 한다.

    ```java
    Set<Double> doubleSet = new HashSet<>(Arrays.asList(1.0, 2.1));
    Set<Integer> integerSet = new HashSet<>(Arrays.asList(1, 2));
    Set<Number> unionSet = Union.<Number>union(doubleSet, integerSet);
    ```

3. max 메서드 (아이템30 30-7 )

    ```java
    public static <E extends Comparable<E>> E max(List<E> list)
    //와일드카드 타입 적용
    public static <E extends Comparable<? super E>> E max(List<? extends E> list)
    ```

    - 입력 매개변수에서는 E 인스턴스를 생산하므로 List<? extends E>
    - Comparable<E>는 E 인스턴스를 소비하고 <E extends Comparable<? super E>> 그리고 선후 관계를 뜻하는 정수를 생산한다.
    - **Comparable, Comparator 모두 <? super E>를 사용하는 편이 낫다.**
    - Comparable, Comparator를 직접 구현하지 않고, 직접 구현한 다른 타입을 확장한 타입을 지원하기 위해 와일드카드가 필요하다.
    ScheduledFuture는 Delayed의 하위 인터페이스이고, Delayed는 Comparable<Delayed>를 확장했다.

        ```java
        public static void main(String[] args) {
            List<ScheduledFuture<?>> scheduledFutureList = ...
            // incompatible types...
            RecursiveTypeBound.max(scheduledFutureList);
        }
        ```

        ```java
        // ScheduledFuture interface
        public interface ScheduledFuture<V> extends Delayed, Future<V> {}
        // Delayed interface
        public interface Delayed extends Comparable<Delayed> {}
        // Comrable interface
        public interface Comparable<T> {}
        ```

4. 타입 매개변수와 와일드카드 사이에 공통되는 부분으로 둘다 사용 가능한 경우

    ```java
    class swapTest {
        // 방법1) 비한정적 타입 매개변수
        public static **<E>** void typeArgSwap(**List<E> list**, int i, int j) {
            list.set(i, list.set(j, list.get(i)));
        }
        // 방법2) 비한정적 와일드카드 -> **List<?>에는 null 외에 어떤 값도 넣을 수 없다.**
        public static void wildcardSwap(**List<?> list**, int i, int j) {
    				list.set(i, list.set(j, list.get(i)));        
    				//swapHelper(list, i, j);
        }
    		//방법2의 null 문제 
    		//→ 와일드카드 타입의 실제 타입을 알려주는 메서드를 private 도우미 메서드로 작성한다.
    		//실제 타입을 알아내려면 제네릭메서드여야 한다.
        private static <E> void swapHelper(List<E> list, int i, int j) {
            list.set(i, list.set(j, list.get(i)));
        }
    }
    ```

    public API라면 방법2가 타입 매개변수를 신경쓰지 않아도 되지만, **List<?>에는 null 외에 어떤 값도 넣을 수 없다.**

    기본규칙

    - 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드카드로 대체하라.
    - 비한정적 타입 매개변수 → 비한정적 와일드카드
    - 한정적 타입 매개변수 → 한정적 와일드카드

    ## ✏️핵심정리

    - 와일드카드 타입을 적용하면 API가 훨씬 유연해진다.
    - PECS공식 : Producer-Extends, Consumer-Super
    - Comparable, Comparator는 모두 소비자이다. Comparable/Comparator<? super E>

    # 아이템32. 제네릭과 가변인수를 함께 쓸 때는 신중하라

    ## 가변인수

    - 가변인수(varargs)메서드와 제네릭은 자바5때 추가됬다.
    - 가변인수: 메서드에 넘기는 인수의 개수를 클라이언트가 조절할 수 있다.
    static void test(List<String>... param) { }

    ### 가변인수의 허점

    - 가변인수 메서드 호출 시 **가변인수를 담기 위한 배열이 자동생성** 된다.
    → **내부로 감춰야 했을 이 배열이 클라이언트에 노출된다.**
    - **varargs 매개변수에 제네릭이나 매개변수화 타입이 포함되면 알기 어려운 컴파일 경고 발생한다.**
    - 실제화 불가 타입은 런타임에는 컴파일타임보다 타입관련 정보를 적게 담고 있다(소거)
    → 거의 모든 제네릭과 매개변수화 타입은 실체화 불가 → **가변인수 메서드 호출에서 varargs 매개변수가 실체화 불가타입으로 추론되면 힙오염 경고 발생**

    ## 제네릭과 varargs를 혼용하면 타입 안정성이 깨진다.

    - 매개변수화 타입의 변수가 타입이 다른 객체를 참조하면 힙 오염이 발생한다.
    - 다른 타입 객체를 참조하는 상황에서 컴파일러는 자동 생성한 형변환이 실패하고 ClassCastException을 던진다.

        ```java
        static void dangerous(List<String>... stringLists) {
          List<Integer> intList = List.of(42);
          //varargs는 내부적으로 배열이고
          //배열은 공변이기 때문에 List<String>타입은 Object의 하위클래스로 인식되어
          //Object[]에 참조 될 수 있다.
          Object[] objects = stringLists;
          Object[0] = intList; //힙 오염 발생 
        	//내부적으로는 List<String> 타입이지만, 
        	//런타임에는 제네릭 타입이 소거되므로 같은 List로만 인식되어 할당이 가능하다.
          String s = stringLists[0].get(0); // ClassCastException 형변환 숨어있다.
        }
        public static void main(String[] args) {
            dangerous(List.of("There be dragons!"));
        }
        ```

    - **제네릭 varargs 배열 매개변수에 값을 저장하는 것은 안전하지 않다.**

## @SafeVarargs 애너테이션

: 메서드 작성자가 그 메서드가 타입 안전함을 보장하는 장치다.

- 자바7에서 @SafeVarargs 추가되기 전에는 사용자들은 경고를 무시하거나 호출부마다 @SuppressWarnings("unchecked")애너테이션을 달아 숨겼다.
- **@SafeVarargs는 재정의할 수 없는 메서드에만 달아야 한다.** 
→ 재정의한 메서드도 안전할지 보장할 수 없기 때문이다.
→ 자바8에서는 정적메서드와 final인스턴스 메서드에만 붙일 수 있고, 자바9부터 private 인스턴스 메서드에도 허용된다.

## 타입 안전 조건

- varargs 매개변수 배열에 아무것도 저장하지 않는다. (그 매개변수들을 덮어쓰지 않기)
- 그 배열(혹은 복제본)을 신뢰할 수 없는 코드에 노출하지 않는다. (그 배열의 참조 노출하지 않기)
- 즉, 이 varargs매개변수 배열이 호출자로부터 그 메서드로 인수들만 전달하면 안전하다.

## 타입 불안전 예시

### 자신의 제네릭 매개변수 배열의 참조를 노출한다.

```java
public class PickTwo {
    // 코드 32-2 자신의 제네릭 매개변수 배열의 참조를 노출한다. - 안전하지 않다!
    static <T> T[] toArray(T... args) {
        return args;
    }

    static <T> T[] pickTwo(T a, T b, T c) {
        switch(ThreadLocalRandom.current().nextInt(3)) {
            case 0: return toArray(a, b);
            case 1: return toArray(a, c);
            case 2: return toArray(b, c);
        }
        throw new AssertionError(); // 도달할 수 없다.
    }

    public static void main(String[] args) { 
        String[] attributes = pickTwo("좋은", "빠른", "저렴한");//ClassCastException
        System.out.println(Arrays.toString(attributes));
    }
}
```

컴파일러는 toArray에 넘길 T 인스턴스 2개를 담을 varargs 매개변수 배열을 만드는 코드를 생성하고 반환되는 배열의 타입은 Object[]이다.

그대로 pickTwo를 호출한 클라이언트까지 전달된다. 즉, pickTwo는 항상 Object[] 타입 배열을 반환한다.

 String[] attributes = pickTwo("좋은", "빠른", "저렴한"); → String[] 형변환 자동생성

**Object[]는 String[]의 하위타입이 아니므로 형변환 에러가 발생한다.** 

**제네릭 varargs 매개변수 배열에 다른 메서드가 접근하도록 허용하면 안전하지 않다.**

📍안전한 두가지 예외 케이스

1. @SafeVarargs로 제대로 애노테이트된 또 다른 varargs 메서드에 넘기는 경우
2. 그저 이 배열 내용의 일부 함수를 호출만 하는(varargs를 받지 않는) 일반 메서드에 넘기는 경우

## 제네릭 varargs 매개변수를 안전하게 사용하는 메서드

```java
@SafeVarargs
static <T> List<T> flatten(List<? extends T>... lists) {
  List<T> result = new ArrayList<>();
  for(List<? extends T> list : lists) {
    result.addAll(list);
  }
  return result;
}
```

varargs 배열을 직접 노출 시키지 않고, T타입의 제네릭 타입을 사용하였기 때문 ClassCastException 또한 발생할 일이 없다.

## varargs (실체는 배열) 매개변수를 List 매개변수로 바꾼다.

@SafeVarargs 사용이 유일한 답은 아니다.

```java
//아이템28 적용
static <T> List<T> flatten(List<List<? extends T>> lists) { //배열-> list로 변경
  List<T> result = new ArrayList<>();
  for (List<? extends T> list : lists) {
    result.addAll(list);
  }
  return result;
}
public static void main(String[] args) {
    List<Integer> flatList = flatten(List.of(
            List.of(1, 2), List.of(3, 4, 5), List.of(6,7)));
    System.out.println(flatList);
}
```

- @SafeVarargs 애너테이션이 달린 **정적 팩터리 메서드인 List.of**를 활용하여 컴파일러가 타입 안정성을 검사하게 할 수 있다.
- @SafeVarargs 애너테이션을 우리가 직접 달지 않아도 되며 실수로 안전하다고 판단할 걱정도 없다.
- 단점이라면 클라이언트 코드가 살짝 지저분해지고 속도가 조금 느려질 수 있다는 정도다.
- varargs 메서드를 안전하게 작성하는 게 불가능한 상황에 적용( 앞의 PickTwo 메서드 수정)

```java
public class PickTwoList {

    static <T> List<T> pickTwo(T a, T b, T c) {
        switch(ThreadLocalRandom.current().nextInt(3)) {
            case 0: return List.of(a, b);
            case 1: return List.of(a, c);
            case 2: return List.of(b, c);
        }
        throw new AssertionError();
    }

    public static void main(String[] args) {
        List<String> attributes = pickTwo("좋은", "빠른", "저렴한");

        for(String str : attributes){
            System.out.println(str);
        }
    }
}
```

## ✏️핵심정리

- 가변인수 기능은 배열을 노출하여 추상화가 완벽하지 못하고, 배열과 제네릭의 타입 규칙이 다르 때문에 가변인수와 제네릭의 궁합이 좋지 않다.
- 제네릭 varargs 매개변수는 타입안전하지 않지만, 허용된다.
- 메서드에 제네릭(혹은 매개변수화된) varargs 매개변수를 사용하고자 한다면, 먼저 그 메서드가 타입 안전한지 확인한 다음 @SafeVarargs애너테이션을 달아 경고를 없애준다.
- varargs 매개변수는 단순히 파라미터를 받아와 메서드의 생산자(T타입의 객체를 제공하는 용도)로만 사용한다.
- varargs에 아무것도 저장하면 안된다.
- varargs배열을 외부에 리턴하거나 노출하지 말자
- 웬만하면 다시 컬렉션(List)에 담아 리턴하는 안전한 방식을 사용한다.

# 아이템33. 타입 안전 이종 컨테이너를 고려하라

## 컨테이너 정의

객체를 저장하는 배열(array)의 '크기가 한번 정해지면 바꿀 수 없다' 제약을 해결하기 위해 
java.util 라이브러리에는 컨테이너(container) 클래스 들이 있으며, 그것의 기본 타입들은 List, Set, Queue, Map 이다.

Set<E>, Map<K,V> : 매개변수화 되는 대상은 (원소가 아닌) 컨테이너 자신이다. 따라서 하나의 컨테이너에서(클래스 레벨에서) **매개변수화 될 수 있는 타입의 수가 제한된다.**

## 타입 안전 이종 컨테이너

타입의 수에 제약없이, 특정 타입 외에 다양한 타입을 지원해야하는 경우 사용한다.

### 설계 방식

- 컨테이너 대신 키를 매개변수화한 다음, 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공하면 된다.
- 제네릭 타입 시스템이 "값의 타입 == 키의 타입" 보장해준다.

### 타입 안전 이종 컨테이너 패턴 - API

```java
public class Favorites {
	public <T> void putFavorite(Class<T> type, T instance);
	public <T> T getFavorite(Class<T> type);
}
```

각 타입의 Class 객체를 매개변수화한 키 역할로 사용한다. → class의 클래스가 제네릭이기 때문에 동작한다.

class 리터럴의 타입은 Class가 아닌 Class<T>다. 

타입 토큰(type token) : 컴파일타임 타입 정보와 런타임 타입 정보를 알아내기 위해 메서드들이 주고받는 class 리터럴

(ex. Integer.class, String.class 는 class 리터럴이며 타입토큰은 Class<Integer>, Class<String>)

### 타입 안전 이종 컨테이너 패턴 - 클라이언트

```java
public static void main(String[] args) {
	Favorites f = new Favorites();
	
	f.putFavorite(String.class, "Java");
	f.putFavorite(Integer.class, 0xcafebabe);
	f.putFavorite(Class.class, Favorites.class);

	String favoriteString = f.getFavorite(String.class);
	int favoriteInteger = f.getFavorite(Integer.class);
	Class<?> favoriteClass = f.getFavorite(Class.class);

	System.out.printf("%s %x %s\\n", favoriteString, favoriteInteger
										, favoriteClass.getName());
}
```

- 클라이언트는 저장하거나 얻어올 떄 Class 객체를 알려주면 된다.
- Favorites 인스턴스는 타입안전하다.
String을 요청했는데 Integer 반환하는 일 절대 없다.
- 모든 키의 타입이 제각각이라, 여러 가지 타입의 원소를 담을 수 있다.

### 타입 안전 이종 컨테이너 패턴 - 구현부

```java
public class Favorites{
    private Map<Class<?>, Object> favorites = new HashMap<>();
    public  <T> void putFavorite(Class<T> type, T instance){
        favorites.put(Objects.requireNonNull(type), instance);
    }
    public  <T> T getFavorite(Class<T> type){
        return type.cast(favorites.get(type));
    }
}
```

1. favorites 의 타입이 Map<Class<?>, Object> 비한정적 와일드카드 타입 → 맵 안에 아무것 못넣는다? → 아니다!!

    **와일드카드 타입이 중첩되었다.**

    - 맵이 아니라 **키가 와일드카드 타입**이다.
    - 모든 키가 서로 다른 매개변수화 타입일 수 있다.
    → 첫번째는 Class<String>, 두번째는 Class<Integer>
2. favorites 맵의 값 타입은 단순히 Object이다.
    - 모든 값이 키로 명시한 타입임을 보증하지 않는다.
3. putFavorites 구현은 주어진 Class객체와 즐겨찾기 인스턴스를 favorites에 추가해 관계를 짓는다.
    - 키와 값 사이의 '타입링크' 정보는 버려져 그 값이 그 키 타입의 인스턴스라는 정보가 사라진다. → getFavorites에서 살릴 수 있다.
4. getFavorites
    - 주어진 Class 객체에 해당하는 값을 favorites 맵에서 꺼낸다.
    - 이 객체가 바로 반환행 할 객체가 맞지만, 잘못된 컴파일타임 타입을 가지고 있다.
    → favorites 맵의 값 타입인 Object이나, 이를 T로 바꿔 반환해야 한다.
    - Class의 cast 메서드를 사용해 이 객체 참조를 Class 객체가 가리키는 타입으로 동적변환한다.
    - cast 메서드: 형변환 연산자의 동적 버전으로 타입이 일치하면 인수반환, 아니면 ClassCastException을 던진다.
    - 위 예시 코드는 favorites 맵 안의 값이 해당 키의 타입과 항상 일치함 → cast는 왜사용하나?
        - cast 의 반환타입 = Class 객체의 타입 매개변수
        - T로 비검사 형변환하는 손실 없이 Favorites를 타입안전하게 만드는 비결이다.

    ## Favorites 클래스에서 알아둬야 할 두가지 제약

    ### 1. 악의적인 클라이언트가 Class 객체를 (제네릭이 아닌) 로 타입으로 넘기면 Favorites 인스턴스의 타입 안전성이 쉽게 깨진다.

    ```java
    f.putFavorite((Class)Integer.class, "Integer의 인스턴스가 아닙니다");
    int favoriteInteger = f.getFavorite(Integer.class);
    ```

    - 컴파일타임에 비검사 경고
    - putFavorites호출은 문제 없고, getFavorite를 호출할 때 ClassCastException이 발생한다.

    ### 🔍**해결책 : 동적 형변환으로 런타임 타입 안정성 확보**

    - putFavorite 메서드에서 인수로 주어진 instance타입이 type으로 명시한 타입과 같은지 확인하면 된다.

        ```java
        public <T> void putFavorite(Class<T> type, T instance){
            favorites.put(Object.requireNonNull(type), type.cast(instance));
        }
        ```

    ### 2. 실체화 불가 타입에는 사용할 수 없다.

    - 즐겨 찾는 String이나 String[]은 저장할 수 있고, List<String>은 저장할 수 없다.
    → List<String>용 Class객체를 얻을 수 없기 때문이다.
    - List<String>.Class → 오류
    - List<String>과 List<Integer>는 List.class라는 같은 Class 객체를 공유하기 때문에 
    List<String>.Class 와 List<Integer>.Class 허용하면 난리남

    ## 한정적 타입 토큰

    Favorites가 사용하는 타입 토큰은 비한정적이다. → 어떤 Class든 객체로 받아들인다.

    메서드들이 허용하는 타입을 제한하고 싶을 때 한정적 타입 토큰을 사용한다.

    한정적 타입 토큰이란 단순히 한정적 타입 매개변수나 한정적 와일드카드를 사용하여 표현 가능한 타입을 제한하는 타입 토큰이다.

    ### 한정적 타입토큰을 활용한 애너테이션API

    ```java
    public <T exnteds Annotation> T getAnnotation(Class<T> annotationType);
    ```

    - AnnotatedElement 인터페이스에 선언된 메서드로, 대상 요소에 달려있는 애너테이션을 런타임에 읽어오는 기능을 한다.
    - annotationType 인수는 애너테이션의 타입을 뜻하는 한정적 타입 토큰이다.
    - 이 메서드는 토큰으로 명시한 타입의 애너테이션이 대상 요소에 달려 있다면 그 애네테이션을 반환하고, 없다면 null을 반환한다.
    - 즉, 애너테이션된 요소는 그 키가 애너테이션 타입인, 타입 안전 이종 컨테이너이다.

    ## Class<?> 타입의 객체가 있고, 이를 한정적 타입 토큰을 받는 메서드에 넘기기 위한 방법

    1. 객체를 Class<? extends Annotation>으로 형변환 한다.

        → 비검사이므로 컴파일 경고 발생

    2. **asSubclass 메서드로 호출된 인스턴스 자신의 Class객체를 인수가 명시한 클래스로 형변환한다.**
        - 형변환된다는 것은 이 **클래스가 인수로** **명시한 클래스의 하위 클래스**라는 뜻이다.
        - 형변환에 성공하면 인수로 받은 클래스 객체를 반환하고, 실패하면 ClassCastException을 던진다.

        ```java
        static Annotation getAnnotation(AnnotatedElement element,
        																String annotationTypeName) { 
        	Class<?> annotationType = null; // 비한정적 타입 토큰
        	try {
        		annotationType = Class.forName(annotationTypeName);
        	} catch (Exception ex) {
        		throw new IllegalArgumentException(ex);
        	} 
        	return element.getAnnotation(
        		annotationType.asSubclass(Annotation.class)); 
        		//한정적 타입 토큰을 안전하게 형변환한다.
        }
        ```

        ## ✏️핵심정리

        - 컬렉션 API로 대표되는 일반적인 제네릭 형태에서는 한 컨테이너가 다룰 수 있는 타입 매개변수의 수가 고정되어 있다.
        - 컨테이너 자체가 아닌 키를 타입 매개변수로 바꾸면 매개변수 수의 제약이 없는 타입안전이종컨테이너를 만들 수 있다.
        - 타입 안전 이종 컨테이너는 Class를 키로 쓰며, 이런식으로 쓰이는 Class객체를 타입 토큰이라 한다.
        - 또한, 직접 구현한 키 타입도 쓸 수 있다.
        - 데이터베이스의 행(컨테이너)을 표현한 DatabaseRow 타입에는 제네릭 타입인 Column<T>를 키로 사용할 수 있다.