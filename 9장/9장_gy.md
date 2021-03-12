# 9장. 일반적인 프로그래밍 원칙

# 아이템57. 지역변수의 범위를 최소화하라.

지역변수의 유효 범위를 최소로 줄이면 코드 가독성과 유지보수성이 높아지고 오류 가능성은 낮아진다.

## 지역변수를 잘 사용하는 방법

1. 지역변수의 범위를 줄이는 가장 강력한 방법 '가장 처음 쓰일 때 선언하기'
2. 거의 모든 지역변수는 선언과 동시에 초기화 한다.
    - try-catch 문에서는 예외

        : 변수를 초기화하는 표현식에서 검사 예외를 던질 가능성이 있다면 try블록 안에서 초기화 해야한다.

        → 그렇지 않으면, 예외가 블록을 넘어 메서드에까지 전파된다.

    - try 블록 바깥에서도 사용해야 한다면 try블록 앞에서 선언해야 한다.
3. 반복변수의 값을 반복문이 종료된 뒤에도 써야하는 상황이 아니라면 while문보다 for문을 쓰는 편이 낫다.

    → 반복문에서는 반복변수의 범위가 반복문의 몸체, for 키워드와 몸체 사이의 괄호 안으로 제한되기 때문

4. 메서드를 작게 유지하고 한 가지 기능에 집중하는 것이다. → 메서드를 기능별로 쪼갠다.

## for문의 장점

- copy & paste 오류를 줄여준다.
- 변수 유효 범위가 for문 범위와 일치하여 똑같은 이름의 변수를 여러 반복문에서 써도 서로 영향이 없다.
- while문 보다 짧아서 가독성이 좋다.

# 아이템58. 전통적인 for문보다는 for-each 문을 사용하라.

## for-each문의 장점

- 반복자와 인덱스 변수를 사용하지 않아 코드가 깔끔하고 오류가 발생할 일이 없다.
- 하나의 관용구로 컬렉션과 배열을 모두 처리할 수 있어서 어떤 컨테이너를 다루는지 신경쓰지 않아도 된다.
- 컬렉션을 중첩해서 순회해야 한다면 for-each 문의 이점은 더욱 커진다.
- Iterable 인터페이스를 구현한 객체라면 무엇이든 순회할 수 있다.

## for문 사용시 버그

```java
...
List<Card> deck = new ArrayList<>();
for(Iterator<Suit> i = suits.iterator(); i.hasNext();)
	for(Iterator<Rank> j = ranks.iterator(); j.hasNext();)
		deck.add(new Card(i.next(), j.next());
```

→ Card에 Suit하나 당 Rank개가 매핑되야하는데 지금은 안쪽 for문에서 j 만큼 Suit가 불려지고있어서 suits의 값의 next가 더이상 없으면 NoSuchElementException을 던진다.

- 해결방법 : 바깥 반복문에 바깥 원소를 저장

    ```java
    for(Iterator<Suit> i = suits.iterator(); i.hasNext();){
    	Suit suit = i.next();
    	for(Iterator<Rank> j = ranks.iterator(); j.hasNext();)
    		deck.add(new Card(suit, j.next());
    }
    ```

- for-each 사용시 간단하게 해결가능

    ```java
    for(Suit suit : suits)
    	for(Rank rank : ranks)
    		deck.add(new Card(suit,rank));
    ```

## for-each문을 사용할 수 없는 세가지 상황

1. 파괴적인 필터링

    : 컬렉션을 순회하면서 선택된 원소를 제거해야 한다면 반복자의 remove메서드를 호출해야 한다.

    → 자바8부터는 Collection의 removeIf메서드를 사용해 컬렉션을 명시적으로 순회하지 않아도 된다.

2. 변형

    : 리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 한다면 리스트의 반복자나 배열의 인덱스를 사용해야 한다.

3. 병렬 반복

    : 여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야 한다.

# 아이템59. 라이브러리를 익히고 사용하라.

## 표준 라이브러리 사용 이점

1. 라이브러리 코드를 작성한 전문가의 지식과 경험을 활용 할 수 있다.
2. 핵심적인 일과 크게 관련 없는 문제 해결에 시간낭비하지 않아도 된다.
3. 따로 노력없이 성능이 지속적으로 개선된다.
4. 기능이 점점 많아진다.
5. 가독성, 유지보수, 재활용이 쉬운 코드가 된다.

알아두면 큰 도움이 되는 라이브러리

1. 컬렉션 프레임워크
2. 스트림
3. java.util.concurrent의 동시성 기능

# 아이템60. 정확한 답이 필요하다면 float와 double은 피하라.

- float와 double 타입은 특히 금융 관련 계산과 맞지 않는다.

    → 0.1 혹은 10의 음의 거듭 제곱 수 (10^-1, 10^-2 등)을 표현할 수 없기 때문이다.

    ex) 1.03 - 0.42 = 0.610000000000001을 출력한다.

- 금융계산에는 BigDecimal, int 혹은 long을 사용해야 한다.
- BigDecimal의 단점
    1. 기본 타입보다 쓰기 불편하다.
    2. 훨씬 느리다.
- BigDecimal의 대안으로 int 혹은 long 타입을 쓸 수도 있다.
    1. 다룰 수 있는 값의 크기가 제한된다.
    2. 소수점을 직접 관리해야 한다.

## ✏️핵심정리

- 코딩 시 불편함이나 성능 저하를 신경쓰지 않아도 되면 BigDecimal을 사용하라.
- BigDecimal이 제공하는 반올림 모드를 이용하여 완벽히 제어 가능하다.
- 성능이 중요하고 소수점을 직접 추적할 수 있고 숫자가 너무 크지 않다면 int나 long을 사용하라.
- 숫자를 9자리 십진수 표현 int
- 18자리 십진수 표현 long
- 18자리 이상 BigDecimal 사용해야 한다.

# 아이템61. 박싱된 기본 타입보다는 기본 타입을 사용하라.

- 기본타입 : int, double, boolean, ..
- 참조타입: String, List,..
- 박싱된 기본타입: Integer, Double, Boolean,..

## 기본 타입과 박싱된 기본 타입의 차이점

1. 기본 타입은 값만 가지고 있으나, 박싱된 기본 타입은 값에 더해 식별성이란 속성을 갖는다.

    → 박싱된 기본 타입의 두 인스턴스는 같은 값을 가지고 있어도 서로 다르다고 식별된다.

2. 기본 타입의 값은 언제나 유효하나, 박싱된 기본 타입은 유효하지 않은 값, 즉 null을 가질 수 있다.
3. 기본 타입이 박싱된 기본 타입보다 시간과 메모리 사용면에서 더 효율적이다.

## 박싱타입 사용시 주의할 점

1. 같은 객체를 비교하는게 아니라면 박싱된 기본 타입에 == 연산자를 사용하면 오류가 일어난다.
2. 기본 타입과 박싱된 기본타입을 혼용한 연산에서는 박싱된 기본 타입의 박싱이 자동으로 풀린다.
3. null참조를 언박싱하면 NPE이 발생한다.
4. 기본 타입을 박싱하는 작업은 필요 없는 객체를 생성하는 부작용을 나을 수 있다.

## 박싱된 기본타입 사용에 적절한 경우

1. 컬렉션의 원소, 키, 값으로 쓴다.
    - 컬렉션은 기본타입을 담을 수 없으므로 박싱된 기본타입을 써야한다.
2. 매개변수화 타입이나 매개변수화 메서드의 타입 매개변수로는 박싱된 기본타입을 써야한다.
3. 리플렉션을 통해 메서드를 호출할 때도 박싱된 기본 타입을 사용해야 한다.

# 아이템62. 다른 타입이 적절하다면 문자열 사용을 피하라.

## 문자열을 쓰지말아야 하는 이유

1. 문자열은 다른 값 타입을 대신하기에 적합하지 않다.
    - 문자열로 받은 데이터가 수치형이면 int, float, BigInteger 등 적당한 수치타입으로 변환해야 한다.
    - 예/아니오 질문의 답이라면 적절한 열거타입이나 boolean으로 변환해야 한다.
    - 적절한 값 타입이 있다면 그것을 사용하고, 없다면 새로 작성해야 한다.
2. 문자열은 열거 타입을 대신하기에 적합하지 않다.
    - 상수를 열거할 때는 문자열보다 열거 타입이 월등히 낫다.
3. 문자열은 혼합 타입을 대신하기에 적합하지 않다.

    ```java
    String compoundKey = className + "#" + i.next();
    ```

    - 각 요소를 개별로 접근하려면 문자열을 파싱해야해서 느리고,오류 가능성도 커진다.
    - 적절한 equals, toString, compareTo 메서드를 제공할 수 없다.
4. 문자열은 권한을 표현하기에 적합하지 않다.
    - 문자열을 사용해 권한을 구분한 예

        ```java
        public class ThreadLocal {
        	private ThreadLocal() {} // 객체 생성 불가
        	//현 스레드의 값을 키로 구분해 저장한다.
        	public static void set(String key, Object value);
        	//(키가 가리키는) 현 스레드의 값을 반환한다.
        	public static Object get(String key);
        }
        ```

        - 문제점: 스레드 구분용 문자열 키가 전역 이름 공간에 공유 된다.
        - 각 클라이언트가 고유한 키를 제공해야하는데 같은 키 제공하면 같은 값을 공유하게 된다.
        - 보안 취약
    - 별도의 타입인 Key 클래스로 권한을 구분한 예

        ```java
        public class ThreadLocal {
        	private ThreadLocal() {} // 객체 생성 불가
        	public static class Key { // (권한)
        		key(){}
        	}
        	//위조 불가능한 고유 키를 생성한다.
        	public static Key getKey(){
        		return new Key();
        	}
        	public static void set(Key key, Object value);
        	public static Object get(Key key);
        ```

        - set/ get 메서드는 이제 정적 메서드일 이유가 없으니, Key 클래스의 인스턴스 메서드로 바꿔준다.

            → key는 더이상 스레드 지역변수를 구분하기 위한 키가 아니라, 그 자체가 스레드 지역변수가 된다.

    - 리팩터링

        ```java
        public final class ThreadLocal<T> {
        	public ThreadLocal();
        	public void set(T value);
        	public T get();
        }
        ```

        - 위 코드에서 톱클래스 ThreadLocal 부분은 필요없으니 지우고 중첩클래스 Key이름을 ThreadLocal로 변경해준다.
        - get으로 얻은 Object를 실제 타입으로 형변환해야해서 타입안전하지 않으니 타입안전을 위해 ThreadLocal을 매개변수화 타입으로 선언한다.

# 아이템63. 문자열 연결은 느리니 주의하라.

문자열 연결 연산자 (+)는 여러 문자열을 하나로 합려주는 편리한 수단이다.

문자열 연결 연산자로 문자열 n개를 잇는 시간은 n^2에 비례한다.

문자열은 불변이라서 두 문자열을 연결할 경우 양쪽의 내용을 모두 복사해야하므로 성능저하는 피할 수 없다.

**성능을 위해서는 String 대신 StringBuilder를 사용하자.**

```java
Public String statement(){
	StringBuilder b = new StringBuilder(numItems() * LINE_WIDTH);
	for (int i = 0; i < numItems(); i++)
		b.append(lineForItem(i));
	return b.toString();
}
```