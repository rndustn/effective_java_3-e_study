# 3장. 모든 객체의 공통 메서드

[3장. 모든 객체의 공통 메서드]() 

- [ ]  final이 아닌 Object메서드(equals, hashCode, toString, clone, finalize)들을 언제. 어떻게 재정의(overriding)해야하는지를 다룬다.

# 아이템10. equals는 일반 규약을 지켜 재정의하라

## equals를 재정의 하면 안되는 경우

1. **각 인스턴스가 본질적으로 고유한 경우**

    값을 표현하는 게 아니라 동작하는 개체를 표현하는 클래스
    ex) Thread → Object의 equals 메서드는 이러한 클래스에 딱 맞게 구현되어있다.

2. **인스턴스의 '논리적 동치성(logical equality)'을 검사할 일이 없는 경우**

    java.util.regex.Pattern 클래스는 equals를 재정의하여 두 인스턴스가 같은 정규표현식을 나타내는지 검사한다. → 이러한 검사가 필요없다면 Object의 equals만으로 해결 가능하다.

3. **상위 클래스에서 재정의한 equals가 하위클래스에도 딱 들어맞는 경우**

    Set 구현체는 AbstractSet, List 구현체는 AbstractList, Map 구현체는 AbstractMap으로부터 상속받아 그대로 사용한다.

4. **클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없는 경우**

    인스턴스끼리 비교할 일이 없는 경우에 equals 호출시 new AssertionError()를 던진다.

    ```java
    @Override public boolean equals(Object o){
    	throw new AssertionError(); //호출금지
    }
    ```

5. **값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는(ex. Singleton, String), 단 1개만 만들도록 통제하는 정적펙터리메서드( static factory method )를 가진 경우**

    인스턴스가 단 1개만 존재하므로, equals를 오버라이딩할 필요가 없다.
    Enum 클래스도 마찬가지로 논리적 동치성과 객체 식별성이 동일해진다. → Object 의 equals가 논리적 동치성까지 확인해준다고 볼 수 있다.

## equals를 재정의 해야하는 경우

- **객체 식별성**(두 객체가 물리적으로 같은가 == **메모리 주소가 같은가**)이 아니라 **논리적 동치성**(**객체 내 값이 같은가**)을 확인해야 하는데, **상위 클래스의 equals가 논리적 동치성을 비교하도록 오버라이딩 되지 않은 경우**
주로 Integer, String처럼 값을 표현하는 Wrapper Class가 해당된다.
Wrapper Class를 equals()로 사용시 논리적 동치성을 비교하도록 equals를 재정의하면 값을 비교하면서, Map의 Key와 Set의 원소로 사용할 수 있게 된다.

### Object 명세에서 말하는 동치관계란?

- 동치관계: 집합을 서로 같은 원소들로 이뤄진 부분집합으로 나누는 연산
ex) 파이를 여러 조각으로 자른것
- 동치류(동치클래스: equivalence class): 이와 같은 부분집합
ex) 잘린 파이 한 조각
- equals 메서드가 쓸모 있으려면 모든 원소가 같은 동치류에 속한 어떤 원소와도 서로 교환이 가능해야한다.
- 동치클래스 예시
    - X = {a, b, c, a, b, c}
    - A = {a, a}
    - B = {b, b}
    - C = {c, c}
    - **A, B, C가 동치클래스이다.**

## equals 메서드 재정의 시 지켜야 할 다섯가지 규약

1. **반사성(reflexivity): null이 아닌 모든 참조 값 x에 대해 x.equals(x)는 true 이다.**
즉, 객체는 자기 자신과 같다는 뜻이다.
반사성을 어긴 클래스의 인스턴스를 Collection에 넣고 **contains** 메서드를 호출하면 false를 반환한다.
2. **대칭성(symmetry): null이 아닌 모든 참조 값 x,y에 대해, x.equals(y)가 true이면 y.equals(x)도 true이다.**
두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다.

    ```java
    //CaseInsensitiveString 클래스의 equals() 에서는 대소문자를 구별하지 않고 문자열 비교 연산
    public final class CaseInsensitiveString(){
        private final String s;
     
        public CaseInsensitiveString(String s){
            this.s = Objects.requireNonNull(s);
        }
        @Override public boolean equals(Object o) {
            if (o instance of CaseInsensitiveString)
                return s.equalsIgnoreCase(
                    ((CaseInsensitiveString) o).s);
            
            if(o instanceof String) //한 방향으로만 작동한다는 문제점
                return s.equalsIgnoreCase((String) o);
            
            return false;
        }    
        ... // 나머지 
    }

    CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
    String s = "polish";

    cis.equals(s); // true
    s.equals(cis); // false 
    //String의 equals()는 CaseInsensitiveString 클래스의 존재에 대해 알지 못하고 
    //대소문자를 구분하기 때문에 false 

    List<CaseInsensitiveString> list = new ArrayList<>();
    list.add(cis);
    list.contains(s); //를 호출하면 false를 반환하거나
    //JDK 버전에 따라 true를 반환 또는 런타임 예외를 던질 수 있다.

    //해결방안-> 두 클래스의 equals 메서드를 연동하겠다는 생각을 버리면 된다.
    //즉, CaseInsensitiveString과 String의 equals는 서로 아예 다른 로직으로서, 
    //같은 연산으로 보지 않겠다 라는 것이다. 
    //-> equals 메서드를 통해 다른 클래스의 인스턴스와 비교 불가
    @Override public boolean equals(Object o) {
            return o instanceof CaseInsensitiveString 
    				&& ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);      
    	}
    ```

3. **추이성(tansitivity): null이 아닌 모든 참조 값 x,y,z에 대해, x.equals(y)가 true이고 y.equals(z)면 x.equals(z)도 true다.**

1) 대칭성 위배 

```jsx
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;
        Point p = (Point) o;
        return p.x == x && p.y == y;
    }
}
public class ColorPoint extends Point {
    private final Color color;
    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;
        return super.equals(o) && ((ColorPoint) o).color == color;
    }
}
Point p = new Point(1, 2);
ColorPoint cp = new ColorPoint(1, 2, Color.RED);

p.equals(cp) == true //Point의 equals()는 색상을 무시하고,
cp.equals(p) == false //ColorPoint의 equals()는 입력 매개변수의 클래스 종류가 다르다며 매번 false를 반환한다.
```

Point 비교할 때 색상을 무시하도록 한다면? → 대칭성은 지켜도 추이성이 깨진다.

```jsx
@Override
    public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;
        
        //o가 일반 Point면 색상을 무시하고 비교
        if (!(o instanceof ColorPoint))
            return o.equals(this);
        
        //o가 ColorPoint면 색상까지 비교
        return super.equals(o) && ((ColorPoint) o).color == color;
    }
ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
Point p2 = new Point(1, 2);
ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);

p1.equals(p2)는 true, p2.equals(p3)는 true, p1.equals(p3)는 false를 반환한다.
```

구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.

→ equals 안의 instanceof 검사를 getClass 검사로 바꾸면 규약도 지키고 값도 추가하면서 구체 클래스를 상속할 수 있다는 뜻으로 들릴 수 있지만, **아니다.**

```jsx
public boolean equals(Object o) {
    if (o == null || o.getClass() != getClass()) {
         return false; 
    } 
    Point p = (Point) o;
    return p.x == x && p.y == y;
}
```

위의 equals는 같은 구현 클래스의 객체와 비교할 때만 true를 반환한다.

→ [리스코프 치환 원칙(SOLID) : 자식 클래스에서 부모 클래스의 모든 메소드는 정상 작동 되야한다.]을 위반한다.

- 리스코프 치환 원칙 위배 
ex) 주어진 점이 (반지름이 1인) 단위 원안에 있는지 여부 판별 메서드

    ```jsx
    class Point {
      
      private final int x;
      private final int y;

      private static final Set<Point> unitCircle = Set.of(new Point(0, -1),
       new Point(0, 1),
       new Point(-1, 0),
       new Point(1, 0)
      );

      public static boolean onUnitCircle(Point p) {
        return unitCircle.contains(p);
      }

      @Override
      public boolean equals(Object o) {
        if(o == null || o.getClass() != this.getClass()) {
          return false;
        }
        Point p = (Point) o;
        return this.x == p.x && this.y = p.y;
      }
    }

    public class CounterPoint extends Point{
    	private static final AtomicInteger counter = new AtomicInteger();
    	public CounterPoint(int x, int y){
    		super(x,y);
    		counter.incrementAndGet();
    	}
    	public static int numberCreated(){return counter.get();}
    }

    ColorPoint cp = new ColorPoint(1, 0, Color.RED);
    System.out.println(Point.onUnitCircle(cp)); //false

    //Point.onUnitCircle 메서드의 contains 메서드에서 Set내의 element들에 대한 
    //equals 비교가 일어나게 된다.
    //하지만 equals 메서드 첫번째 if문에서 걸리게 된다.
    //o.getClass() -> ColorPoint.class, this.getClass()-> Point.class
    //위 조건식 때문에 ColorPoint 클래스는 Point클래스로서 equals메서드 내에서 활용되지 못했기 때문에
    //위 코드는 리스코프 치환원칙에 위배되는 코드로 전락해 버렸다.
    //o.getClass() != this.getClass() 대신 !(o instanceof Point) 사용하면 원칙위반 안함.
    ```

**상속대신 컴포지션(Composition)을 사용하라.**

Point를 상속하는 대신 Point를 ColorPoint의 private필드로 두고, ColorPoint와 같은 위치의 일반 Point를 반환하는 View메서드를 public으로 추가하는 방식

```jsx
public ColorPoint {
  private Point point;
  private Color color;

  public ColorPoint(int x, int y, Color color) {
    this.point = new Point(x, y);
    this.color = Objects.requireNonNull(color);
  }

  public Point asPoint() {
    return this.point;
  }

  @Override
  public boolean equals(Object o) {
    if(!(o instanceof ColorPoint)) {
      return false;
    }
    ColorPoint cp = (ColorPoint) o;
    return this.point.equals(cp) && this.color.equals(cp.color);
  }
}
```

**4. 일관성(consistency): null이 아닌 모든 참조 값 x,y에 대해 x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.**

equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안된다.
ex) java.net.URL 의 equals는 주어진 URL과 매핑된 호스트의 IP주소를 이용해 비교하는데 호스트 이름을 IP주소로 바꾸려면 네트워크를 통해야 하는데, 그 결과가 항상 같다고 할 수 없다.

→ 항시 메모리에 존재하는 객체만을 사용한 결정적 계산만 수행해야 한다. 

**5. null-아님: null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false다.**

instanceof 연산자는 입력이 null이면 타입확인 단계에서 false를 반환해서 따로 null검사 로직이 필요없다.  → if(o == null) return false; 불필요

## **equals메서드 구현 절차**

1. **== 연산자를 사용해 입력이 자기자신의 참조인지 확인한다.**
    - 성능 최적화
    - equals가 복잡할 때 같은 참조를 가진 객체에 대한 비교를 하지 않기 위함
2. **instanceof 연산자로 입력이 올바른 타입인지 확인한다.**
    - equals가 정의된 클래스인 것이 일반적이지만, 그 클래스가 구현한 특정 인터페이스의 equals 일 수 있다. 같은 인터페이스를 구현한 서로다른 클래스끼리도 비교하는 경우가 있다.
    ex) Set, List, Map, Map.Entry 등의 컬렉션 인터페이스
    - 묵시적 null체크 용도로도 사용
3. **입력이 올바른 타입으로 형변환한다.**
    - Object 타입의 파라미터를 비교하고자 하는 타입으로 형변환한다.
4. **입력 객체와 자기 자신의 대응되는 '핵심'필드들이 모두 일치하는지 하나씩 검사한다.**
    - 필드가 하나라도 다르면 false반환
    - 2단계에서 인터페이스 사용했다면 필드정보를 가져오는 메서드가 인터페이스에 정의되어있어야하고, 구현체 클래스에서는 메서드를 재정의해야 한다.
5. float와 double을 제외한 기본 타입 필드는 == 연산자로 비교
6. 참조타입 필드는 각각의 equals 메서드로 비교
7. float와 double 필드는 각각 정적메서드인 Float.compare(float, float) 와  Double.compare(double, double)로 비교
    - Float.NaN, -0.0f, 특수한 부동소수 값 등을 다뤄야 하기 때문
    - equals 메서드를 사용할 수는 있지만, float → Float, double → Double 오토박싱을 수반할 수 있어 성능상 좋지 않다.
8. 배열의 모든 원소가 핵심필드라면 Arrays.equals 사용
9. null도 정상값으로 취급하는 참조타입 필드의 경우 정적메서드는 Objects.equals(Object, Object)로 비교해 NullPointerException발생 예방
10. equals의 성능을 높이기 위해서는 다를 가능성이 더 크거나 비교하는 비용이 싼 필드를 먼저 비교한다.

## **equals 구현시 주의사항**

1. 대칭성, 추이성, 일관성 확인
2. equals를 재정의할 땐 hashCode도 반드시 재정의 하자
3. 너무 복잡하게 해결하러 들지 말자.
4. equals의 매개변수는 Object 외의 타입으로 선언하지 말자(컴파일 에러발생)
5. 구글에서 만든 @AutoValue을 이용해서 equals와 hashcode를 자동으로 재정의→ 테스트 코드 작성하지 않아도 된다.

# 아이템11. equals를 재정의하려거든 hashCode도 재정의하라

**equals()** : 두 객체의 내용이 같은지, 논리적 동등성(equality)를 비교 

**hashCode()** : 두 객체가 같은 객체인지, 동일성(identity)를 비교

**Reference 동치** : Heap에 있는 한 객체를 서로 다른 reference가 참조하는 경우

두 reference에 대해 hashCode() 메서드를 호출하면 같은 값을 반환합니다.

**객체 동치** : Heap에 두 개의 객체가 들어있고 두 reference가 각 객체를 참조하지만, 그 두 객체가 동치인 것으로 간주할 수 있는 경우

## equals를 재정의한 클래스 모두에서 hashCode도 재정의해야 한다.

같은 필드를 가진 두 개의 객체를 같다고 판단하기 위해 equals를 재정의(Override)하여 사용하는데 이 때, equals만 재정의하고 hashCode를 재정의 하지 않으면 각 객체는 다른 hashcode를 가지고 있게 되어 클래스의 인스턴스를 HashMap이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제가 발생한다.

### **Object 명세에서 발췌한 규약**

1. equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 hashCode메서드는 항상 같은 값을 반환해야한다. 단, 애플리케이션을 다시 실행하면 값이 달라져도 된다.
2. equals(Object)가 두객체가 같다고 판단하면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.
3. equals(Object)가 두객체가 다르다고 판단해도 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단, 성능이 좋으려면 달라야 한다.

**hashCode 재정의를 잘못했을 때 크게 문제가 되는 조항은 두번째다. 즉, 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.**

equals는 참조하고 있는 값이 같을 경우 물리적으로 다른 두 객체를 논리적으로 같다고 할 수 있지만, Object의 기본 hashCode 메서드는 두 객체가 전혀 다르다고 판단하여 서로 다른 값을 반환한다.

```jsx
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5309), "제니");

m.get(new PhoneNumber(707, 867, 5309)); //"제니"가 아닌 null 반환
```

- PhoneNumber 클래스는 hashCode를 재정의하지 않았기 때문에 논리적 동치인 두 객체가 서로 다른 해시코드를 반환하여 **두 번째 규약을 지키지 못한 것이다.**
    - new를 통해서 서로 다른 객체를 key로 사용하여 hashCode를 뽑아내어 두 객체는 서로 다른 해시코드가 반환된다.
    - 즉, get 메소드는 엉뚱한 해시 버킷을 찾아가서 객체를 찾으려고 했던 것이다. **하지만, 두 객체가 동일한 버킷에 존재했더라도 결과값은 null이었을 것이다.**
    - 그 이유는, **HashMap은 해시코드가 다른 엔트리끼리는 동치성 비교를 시도조차 하지 않도록 최적화되어 있기 때문이다.**
    - 이를 해결하기 위해 PhoneNumber에 적절한 hashCode 메서드만 작성해주면 해결된다.

        ```jsx
        @Overrdie
        public int hashcode() {
          return 42;
        }
        ```

        하지만, 절대 사용해서는 안된다. → 해시코드를 고정값으로 지정하면 모든 객체가 해시테이블의 버킷 하나에 담겨 마치 linked List처럼 동작해서 평균 수행속도 O(1) → O(n) 감소한다.

### **좋은 hashCode 작성법**

1. **int 변수 result를 선언한 후 값 c로 초기화한다.**

    이때 c는 해당 객체의 첫번째 핵심 필드를 단계 2.1 방식으로 계산한 해시코드다.

2. **해당 객체의 나머지 핵심 필드 f각각에 대해 다음 작업을 수행한다.**
    1. 해당 필드의 해시코드 c를 계산한다.
        1. 기본 타입 필드라면, Type.hashCode(f)를 수행한다. 여기서 Type은 해당 기본타입의 박싱 클래스다.
        2. 참조타입 필드면서 이클래스의 equals 메서드가 이 필드의 equals를 재귀적으로 호출해 비교한다면, 이 필드의 hashCode를 재귀적으로 호출한다.
        필드의 값이 null이면 0을 사용한다.
        3. 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다.
        배열에 핵심원소가 없다면 단순히 상수(0추천)을 사용하고, 모든 원소가 핵심원소면 Arrays.hashCode를 사용한다.

    2. 단계 2.1에서 계산한 해시코드 c로 result를 갱신한다.
    result = 31 * result  + c;
    31을 곱하는 이유는 클래스에 비슷한 필드가 여러 개일 때 그냥 더하면 같은 값이 나올 확률이 높다. 그래서 31을 곱해서 해시 효과를 높여준다.
3. result를 반환한다.

```jsx
@Override
public int hashCode() {
    int c = 31;
    //1. int변수 result를 선언한 후 첫번째 핵심 필드에 대한 hashcode로 초기화 한다.
    int result = Integer.hashCode(firstNumber);

    //2.1.1 기본타입 필드라면 Type.hashCode()를 실행한다
    //Type은 기본타입의 Boxing 클래스이다.
    result = c * result + Integer.hashCode(secondNumber);

    //2.1.2 참조타입이라면 참조타입에 대한 hashcode 함수를 호출 한다.
    //값이 null이면 0을 더해 준다.
    result = c * result + address == null ? 0 : address.hashCode();

    //2.1.3 필드가 배열이라면 핵심 원소를 각각 필드처럼 다룬다.
    for (String elem : arr) {
      result = c * result + elem == null ? 0 : elem.hashCode();
    }
    //배열의 모든 원소가 핵심필드이면 Arrays.hashCode를 이용한다.
    result = c * result + Arrays.hashCode(arr);

    //2.2 result = 31 * result + c 형태로 초기화 하여 
    //3. result를 리턴한다.
    return result;
}
```

**주의할 점**

- 이 메서드가 동치인 인스턴스에 대해 똑같은 해시코드를 반환하는지 단위테스트 작성(AutoValue로 생성했다면 스킵)
- 파생필드는 해시코드 계산에서 제외
- equals 비교에 사용되지 않은 필드는 반드시 제외해야 한다. 그렇지 않으면 두번째 규약을 어기게 될 위험이 있다.

**PhoneNumber 인스턴스의 핵심 필드 3개를 사용한 전형적인 hashCode 메서드**

```jsx
@Override public int hashCode() { 
	int result = Short.hashCode(areaCode); 
	result = 31 * result + Short.hashCode(prefix); 
	result = 31 * result + Short.hashCode(lineNum); 
	return result; 
}
```

위 계산에 비결정적(undeterministic)요소는 전혀 없으므로 동치인 객체들은 같은 해시코드를 가질 것이 확실하다.

해시충돌이 더 적은 방법은 구아바의 com.google.common.hash.Hashing 참고

**Object 클래스는 임의의 개수만큼 객체를 받아 해시코드를 계산해주는 정적 메서드인 hash를 제공한다.** 

```jsx
@Override
public int hashCode() {
    return Objects.hash(lineNum,prefix,areaCode);
}
```

한줄로 작성할 수 있지만, 입력인수를 담을 배열도 만들고 입력이 기본타입이면 박싱/언박싱을 거쳐야해서 성능이 느리다.

**클래스가 불변이고 해시코드를 계산하는 비용이 크면, 매번 새로 계산하기보다 캐싱하는 방식을 고려해야 한다.**

- 이 타입의 객체가 주로 **해시의 키**로 사용될 것 같다면 인스턴스가 만들어질 때 해시코드를 계산해둬야한다.

**해시의 키로 사용되지 않는 경우이고 해쉬계산에 많은 시간이 걸린다면 hashCode가 처음 불릴 때 계산하는 지연초기화하면 좋다.**

- 필드를 지연초기화하려면 그 클래스를 스레드 안전하게 만들어야 한다. 동시에 여러 스레드가 hashCode를 호출하면 여러 스레드가 동시에 계산하여 문제가 발생할 수 있다.

```jsx
private int hashCode;    //자동으로 0으로 초기화
@Override
public int hashCode() {
  int result = hashCode;
  if(result == 0) {    //스레드 안정성 고려
    result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
      result = 31 * result + Short.hashCode(lineNum);
    hashCode = result;
  }
  return result;
}
```

**주의할 점**

1. **성능을 높인답시고 해시코드를 계산할 때 핵심 필드를 생략해서는 안 된다.**
속도는 향상되도 해시 품질이 나빠져 해시테이블의 성능을 떨어트릴 수 있다.
2. **hashCode가 반환하는 값의 생성규칙을 API 사용자에게 자세히 공표하지 말자. 그래야 클라이언트가 이 값에 의지하지 않게 되고, 추후에 계산 방식을 바꿀 수 있다.**

# 아이템12. toString을 항상 재정의하라.

1. java.lang.Object 클래스가 제공하는 toString메서드은 클래스_이름@16진수로_표시한_해시코드를 반환한다.
2. toString 규약은 "모든 하위 클래스에서 이 메서드를 재정의하라"고 한다.
3. toString을 잘 구현한 클래스는 사용하기 좋고, **디버깅과 로깅이 용이**하다.
4. Collection과 같이 인스턴스를 포함하는 경우에 유용하게 사용된다.
5. toString 메서드가 **자동호출**되는 경우
    - println, printf
    - 문자열 연결 연산자(+)
    - assert 구문에 넘길 때
    - 디버거가 객체를 출력할 때
6. 실전에서 toString은 그 객체가 가진 주요 정보 모두를 반환하는게 좋다.
하지만 객체의 필드가 많거나 문자열로 표현하기 적합하지 않으면 요약정보를 반환해도 된다.
7. 반환값의 포맷을 문서화할지 정해야 한다.
    - 장점: 포맷을 명시하면 그 객체는 표준적이고 명확하기에 그 값 그대로 입출력에 사용하거나 객체로 저장할 수 있다.
    - 명시한 포맷에 맞는 문자열과 객체를 상호 전환할 수 있는 정적 팩터리나 생성자를 함께 제공하면 좋다.
    ex) BigInteger, BigDecimal과 같은 대부분의 기본 타입 클래스
    - 단점: 포맷에 평생 구속되어 향후 릴리즈에서 정보 추가나 포맷을 개선할 유연성이 없다.
8. 포맷을 명시하든 아니든 의도를 명확히 밝혀야 한다. → 코드 내 주석으로 명시유무 알려주기
9. 포맷 명시 여부와 상관없이 toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하자.

    예를들어 이동전화번호의 국번용, 가입자번호용 등 각 **필드별 접근자(getter)를 제공**해야 한다.
    그렇지 않으면, **일일히 toString의 반환값을 파싱**해야 한다. → 성능저하, 불필요 작업
    게다가 향후 포맷변경시 시스템이 망가질 수 있다.
    → 접근자를 제공하지 않으면 그 포맷이 사실상 준-표준 API나 다름없어진다.

### toString을 재정의하지 않아도 되는 경우

1. 정적 유틸리티 클래스 : 상태를 갖는 클래스가 아니라 인스턴스를 생성할 필요가 없으므로
2. 열거타입 : 자바가 이미 완변한 toString을 제공한다.

### toString을 재정의해야 하는 경우

하위클래스들이 공유해야 할 문자열 표현이 있는 추상클래스의 경우
대다수의 컬렉션 구현체는 추상 컬렉션 클래스(AbstractMap, AbstractSet 등)들의 toString 메소드를 상속해서 사용한다.

아이템10에서 언급한 구글의 AutoValue 프레임워크는 toString도 자동생성해준다.

# 아이템13. Clone 재정의는 주의해서 진행하라.

**Cloneable** : 복제해도 되는 클래스임을 명시하는 용도의 Mixin Interface

Mixin : 믹스인이란 클래스가 본인의 기능 이외에 추가로 구현할 수 있는 자료형으로, 어떤 선택적 기능을 제공한다는 사실을 선언하기 위해 쓰인다.

하지만, clone 메서드가 선언된 곳이 Clonable이 아닌 **Object**이고, **protected**로 선언되어 있기 때문에 **Cloneable을 선언하는 것만으로는 외부 객체에서 clone 메서드를 호출할 수 없다.**

→ reflection을 사용하면 호출 할 수 있지만, 해당 객체가 접근이 허용된 clone 메서드를 제공한다는 보장이 없기에 100% 성공 보장 못한다.

Cloneable 인터페이스는 선언된 메서드가 없는 **빈 인터페이스**이다.

## Colneable 인터페이스의 역할

- **Object의 protected 메서드인 clone의 동작방식을 결정한다.**
- Cloneable을 구현한 클래스의 인스턴스에서 clone을 호출하면 그 객체의 필드들을 하나씩 복사한 객체를 반환한다.
- Cloneable을 상속받지않고 clone을 호출하면 CloneNotSupportedException을 던진다.

실무에서 Cloneable을 구현한 클래스는 clone 메서드를 **public**으로 제공하며, 사용자는 당연히 복제가 제대로 이뤄지리라 기대한다. → 클래스와 모든 상위 클래스는 복잡하고, 강제할 수 없고 허술하게 기술된 프로토콜을 지켜야해서 깨지기 쉽고, 위험하고 모순적인 메커니즘이 탄생한다. → **생성자를 호출하지 않고도 객체를 생성 할 수 있게 된다.**

## Object clone 메서드의 일반규약

- **x.clone() ≠ x 는 참이다. 원본객체와 복사객체는 서로 다른 객체이다.**
이를 만족하려면 super.clone으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도있다.
- **x.clone().getClass() == x.getClass()는 참이다. 하지만 반드시 만족해야 하는 것은 아니다.**
- **x.clone().equal(x) 는 참이지만, 필수는 아니다.**
- **x.clone().getClass() == x.getClass()
clone 메서드가 반환하는 객체가 super.clone을 호출해 얻은 객체라면 참이다.**

기본적으로 Object.clone은 clone대상 클래스에 대해 새로운 객체를 new로 생성한다.

clone메서드가 super.clone이 아닌, 생성자를 호출해 얻은 인스턴스를 반환해도 컴파일러는 불평하지 않는다. 하지만 이 클래스의 하위 클래스에서 super.clone을 호출한다면 잘못된 클래스의 객체가 만들어져 하위 클래스의 clone메서드가 제대로 작동하지 않게 된다.

- 예시

    클래스 B가 클래스 A를 상속할 때, 하위 클래스인 B인 clone()은 B 타입 객체를 반환해야 한다. 그런데 A의 clone()이 자신의 생성자, 즉 new A(...)로 생성한 객체를 반환한다면 B의 clone()도 A타입의 객체를 반환할 수밖에 없다.

    → super.clone을 연쇄적 호출하면 clone이 처음 호출된 상위 클래스의 객체가 만들어진다.

    → clone을 재정의한 클래스가 final이라면 하위클래스가 없으니 무시해도 된다.

## clone 메서드 구현

모든 필드가 기본타입이거나 불변객체를 참조한다면 clone으로 반환된 객체는 원본의 완벽한 복제본이다.
→ 쓸데없는 복사를 지양하기에 불변 클래스는 굳이 clone 메서드를 제공하지 않는게 좋다.

### 1. 가변 상태를 참조하지 않는 클래스용 clone 메서드

```jsx
class PhoneNumber implements Cloneable {
  @Override
  public PhoneNumber clone() {
    try {
      return (PhoneNumber) super.clone();			
    } catch(ClassNotSupportedException e) {
      throw new AssertionError(); //implements Cloneable하면 일어날 수 없는 일이다.		
    }
  }
}
```

super.clone()의 반환타입은 Object이지만 자바가 **공변 반환 타이핑**을 지원하여 캐스팅이 가능하고 권장하는 방식이다.
→재정의한 메서드의 반환 타입은 상위 클래스의 메서드가 반환하는 타입의 하위타입일 수 있다.

try-catch 부분으로 감싼것은 super.clone() 메서드에서 ClassNotSupportedException이라는 
검사예외(checked exception)를 리턴하기 때문에 처리 해주었다.
하지만 PhoneNumber가 Cloneable을 구현하기 때문에 절대 실패하지 않기에 RuntimeException으로 처리하거나, 아무것도 설정하지 않아야 한다.

### 2. 가변 상태를 참조하는 클래스용 clone 메서드

```jsx
public class Stack implements Cloneable{
  private Object[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;
  public Stack() {
    this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
  }
	public void push(Object e) {
		ensureCapacity();// 원소를 위한 공간확보용
		elements[size++] = e;
	}
	public Object pop() {
		if(size == 0) throw new EmptyStackException();
		Object result = elements[--size];
		elements[size] = null; //다 쓴 참조 해제
		return result;
	}
	//원소를 위한 공간을 적어도 하나 이상 확보
	private void ensureCapacity() {
		if(elements.length == size)
		elements = Arrays.copyOf(elements, 2*size +1);
	}  
}
```

clone 메서드가 단순히 super.clone의 결과를 그대로 반환하면 new Stack()을 통해 새로은 객체가 생성되어 Stack 인스턴스의 size 필드는 올바른 값을 갖지만, elements필드는 원본 Stack 인스턴스와 똑같은 배열을 참조한다.

→ 원본이나 복제본 중 하나를 수정하면 다른 하나도 수정되어 불변식을 해친다.

**clone 메서드는 사실상 생성자와 같은 효과를 낸다. 즉, clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다.**

- 스택 내부정보를 복사하는 가장 쉬운 방법 :  **elements 배열의 clone을 재귀적으로 호출한다.**

```jsx
@Override
  public Stack clone() {
    try {
      Stack result = (Stack) super.clone();
			result.elements = elements.clone();
			return result;
    } catch(CloneNotSupportedException e) {
    }
  }
```

배열의 clone()은 런타임 타입과 컴파일타임 타입 모두가 원본 배열과 똑같은 배열을 반환하므로 elements.clone의 결과는 형변환이 필요 없다.

**배열은 clone 기능을 제대로 사용하는 유일한 예이다.**

하지만, elements 필드가 final이라면 객체 생성 이후 초기화가 불가능하기 때문에 array.clone()을 통한 초기화가 불가능하다.

**→ Cloneable아키텍처는 '가변 객체를 참조하는 필드는 final로 선언하라'는 일반 용법과 충돌한다.**

그래서 복제할 수 있는 클래스를 만들기 위해 일부 필드에서 final 한정자를 제거해야 할 수도 있다.

- **clone 메서드의 재귀적 호출만으로 부족한 경우**
ex) HashTable → 내부는 버킷들의 배열이고, 각 버킷은 key,value쌍을 담는 연결리스트의 첫번째 Entry를 참조한다.

```jsx
public class HashTable implements Cloneable  {
  private Entry[] buckets = ...;
  private static class Entry {
    final Object key;
    Object value;
    Entry next;

    Entry(Object key, Object value, Entry next) {
      this.key = key;
      this.value = value;
      this.next = next;
    }
  }

  @Override
  public HashTable clone() {
    try {
      HashTable result = (HashTable) super.clone();
      result.buckets = buckets.clone();
      return result;
    } catch(CloneNotSupportedException e) {
      throw new Assertion();
    }
  }
}
```

Stack처럼 단순히 버킷배열의 clone을 재귀적 호출하면 복제본은 자신만의 버킷 배열을 갖지만, 이 배열은 원본과 같은 연결리스트를 참조하여 불변성이 깨진다.

**→ 해결방법: 각 버킷을 구성하는 연결 리스트를 복사해야 한다.**

```jsx
...	
	//엔트리가 가르키는 연결 리스트를 재귀적으로 복사
	Entry deepCopy() {
		return new Entry(key, value, next == null ? null : next.deepCopy());
	}
}
@Override
protected HashTable clone() {

    try {
        HashTable result = (HashTable) super.clone();
        result.buckets = new Entry[buckets.length];
        for (int i = 0; i < buckets.length; i++) {
            if (buckets[i] != null)
                result.buckets[i] = buckets[i].deepCopy();
        }
        return result;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

private Entry클래스 내부에 deepCopy 메서드 추가하여 연결리스트에 대한 next를 복제할 때 재귀적으로 clone을 호출한다.

재귀 호출 때문에 연결리스트의 size만큼 스택프레임을 소비하여, 리스트가 길면 stackoverflow에러를 발생시킬 위험이 있다.

**→ 재귀 호출 대신 반복자를 사용하여 순회하는 방법**

```jsx
Entry deepCopy() {
    Entry result = new Entry(key, value, next);
    for (Entry p = result; p.next != null; p = p.next)
        p.next = new Entry(p.next.key, p.next.value, p.next.next);
    return result;
}
```

## clone 재정의시 주의사항

1. **재정의 될 수 있는 메서드를 호출하지 않아야 한다.**
만약 clone이 하위 클래스에서 재정의한 메서드를 호출하면, 하위 클래스는 복제과정에서 자신의 상태를 교정할 기회를 잃어 원본과 복제본의 상태가 달라질 수 있다. 
2. **재정의 된 clone메서드는 throws절을 없애야 한다.**
Object의 clone은 CloneNotSupportedException을 던지지만 public인 clone메서드(재정의 된)는 throws절을 없애던가, RuntimeException으로 throws 해야한다. → 안그러면 호출할때마다 예외처리해야 됨
3. 재정의한 메소드에서는 throws을 지우고 try-catch로 적절히 수정해야한다. 또한 **접근 제한자를 public 으로 반환 타입은 클래스 자신으로 변경한다.**
4. **상속용 클래스는 Cloneable을 구현해서는 안 된다.**
상속해서 쓰기 위한 클래스 설계방식 두가지
    - Object 방식을 모방해 protected로 clone메서드를 구현해 CloneNotSupportedException도 던질 수 있다고 선언하면 Object를 바로 상속할 때처럼 Cloneable 구현 여부를 하위클래스에서 선택한다.
    - clone을 동작하지 않게 구현해놓고 하위 클래스에서 재정의하지 못하게 한다.

        ```jsx
        @Override
        protected final Object clone() throws CloneNotSupportedException {
          throw new CloneNotSupportedException();
        }
        ```

5. **스레드 안정성을 고려하면 clone 메서드 역시 적절히 동기화 해줘야 한다.**

    Object의 clone메서드는 동기화를 신경쓰지 않는다. 그러므로 super.clone호출 외에 다른 작업이없더라도 clone을 재정의하여 동기화 한다.

    ```jsx
    @Override
    public synchronized Object clone() {
      try {
        Object result = super.clone();
      } catch(CloneNotSupportedException e) {
      }
    }
    ```

6. **Cloneable을 구현하는 모든 클래스는 clone을 재정의해야 한다.**
7. 재정의시 가장 먼저 super.clone을 호출한 후 필요한 필드를 전부 적절히 수정한다.
→ 모든 가변객체를 복사하고 복제본이 가진 객체 참조 모두가 복사된 객체를 가리키게 한다.

## **복사 생성자와 복사팩터리**

Cloneable을 기존에 구현하지 않은 상황이라면 이 방법으로 복제하자.

복사 생성자란 단순히 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자이다.

```jsx
public Yum(Yum yum) {}
```

복사 팩터리는 복사 생성자를 모방한 정적 팩터리이다.

```jsx
public static Yum newInstance(Yum yum) {}
```

- 언어 모순적이고 위험한 객체 생성 메커니즘을 사용하지 않는다. 
(생성자를 쓰지 않는 방식 super.clone())
- clone 규약에 기대지 않는다.
- 정상적인 final필드 용법과도 충돌하지 않는다.
- 불필요한 check exception 처리가 필요없다.
- 형변환도 필요없다.
- 복사 생성자와 복사 팩터리는 해당 클래스가 구현한 인터페이스 타입의 인스턴스를 인수로 받을 수 있다. (변환 생성자, 변환 팩터리)
→ 원본 구현 타입에 얽매이지 않고 복제본 타입을 직접 선택할 수 있다.

    ```jsx
    Set set = new HashSet<>(); 
    Set treeSet = new TreeSet(set);// 변환 생성자
    ```

## 정리

Cloneable의 이슈로 보면 복사생성자와 복사팩터리를 구현하는 것이 좋다.

final클래스라면 Clonable을 구현해도 위험이 크지 않지만, 성능 검토 후 드물게 사용해야 한다.

단, 배열의 경우 clone메서드 방식이기 때문에 가장 합당한 예외이다.

# 아이템14. Comparable을 구현할지 고려하라.

Comparable의 메서드는 compareTo가 유일하다.

## compareTo의 특징

- Object의 메서드가 아니다.
- 성격은 두가지만 빼면 Object의 equals와 같다.
    1. 단순 동치성 비교에 더해 순서까지 비교할 수 있으며, 제네릭하다.
    2. Comparable을 구현했다는 것은 그 클래스의 인스턴스들에는 **자연적인 순서(natural order)**가 있음을 뜻한다.
    따라서 Comparable을 구현한 배열은 아래와 같이 쉽게 정렬 할 수 있다.

        ```jsx
        Array.sort(a)
        ```

사실상 자바 플랫폼 라이브러리의 모든 **값 클래스와 열거타입은 Compareable을 구현했다.**

알파벳, 숫자, 연대 같이 **순서가 명확한 값 클래스**를 작성한다면, 반드시 **Comparable** 인터페이스를 구현하자

## CompareTo 메서드의 일반 규약

이 객체와 주어진 객체의 **순서**를 비교한다. 
이 객체가 주어진 객체보다 **작으면 음의 정수를, 같으면 0을, 크면 양의 정수**를 반환한다.
이 객체와 비교할 수 없는 타입의 객체가 주어지면 **ClassCastException**을 던진다.
(모든 객체에 대해 전역 동치관계를 비교하는 equals 와 달리 **타입이 다른 객체는 신경쓰지 않아도 된다.**)

1. **대칭성** : sgn(x.compareTo(y)) == -sgn(y.compareTo(x))
2. **추이성** : x.compareTo(y) > 0 && y.compareTo(z) > 0 이면 x.compareTo(z) >0 이다.
3. **반사성** : x.compareTo(y) == 0 이면 sgn(x.compareTo(z)) == sgn(y.compareTo(z)) 다.
4. **equals** : 필수는 아니지만 x.compareTo(y) == 0 이면 x.equals (y) 만족하는 것이 좋다. 
이를 지키지 않으려면 "이 클래스의 순서는 equals메서드와 일관되지 않는다."라고 명시하자.
- 1~3 까지 equals 규약과 동일하며 주의사항도 같다. → 기존 클래스를 확장한 구체 클래스에서 새로운 값 컴포넌트를 추가했다면 compareTo 규약을 지킬 방법이 없다. → 우회법으로 구현
- 4. 규약을 어긴 클래스를 정렬된 컬렉션에 넣으면 해당 컬렉션이 구현한 인터페이스(Collection, Set, Map)에 정의된 동작과 엇박자를 낼 것이다.
→ 인터페이스(Collection, Set, Map)는 equals 메서드 규약을 따른다 되어있다.
→ 정렬된 컬렉션들은 동치성 비교시 equals대신 compareTo 사용한다.

## compareTo 메서드 작성 요령

1. Comparable은 타입을 인수로 받는 제네릭 인터페이스다.
    - **compareTo 메서드의 인수 타입은 컴파일타임에 정해진다.**
    - 입력 인수의 타입을 확인하거나 형변환 할 필요가 없다.
    - 타입이 잘못됐다면 컴파일 자체가 안된다.
    - null을 인수로 넣어 호출하면 NullPointerException을 던져야 한다.
2. compareTo 메서드는 필드의 **동치가 아니라 순서를 비교**한다.
    - 객체 참조 필드를 비교하려면 compareTo 메서드를 재귀호출 한다.
    - Comparable을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면 비교자(Comparator)를 대신 사용한다.
        - 객체 참조 필드가 하나뿐인 비교자

            ```jsx
            import java.util.Objects;

            public class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
                private final String s;

                public CaseInsensitiveString(String s) {
                    this.s = Objects.requireNonNull(s);
                }

                @Override
                public int compareTo(CaseInsensitiveString cis) {
                    return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
                }
            }
            ```

3. compareTo 메서드에서 관계연산자 **<, >**를 사용하는 이전 방식이 아닌 **박싱된 기본 타입클래스들에 자바7 부터 추가 된 정적 메서드인 compare를 이용**한다.
4. 클래스에 핵심 필드가 여러개라면 가장 핵심적인 필드부터 비교하여 불필요 연산을 줄인다.
    - 기본 타입 필드가 여럿일 때의 비교자

        ```jsx
        public int compareTo(PhoneNumber pn) {
          int result = Short.compare(this.areaCode, pn.areaCode);//가장 주요필드
          if(result == 0) {
            result = Short.compare(this.prefix, pn.prefix);//두 번째로 중요한 필드
            if(result == 0) {
              result = Short.compare(this.lineNum, pn.lineNum);//세 번째로 중요한 필드
            }
          }
        }
        ```

5. 자바8 부터 Comparator 인터페이스가 일련의 **비교자 생성 메서드**와 팀을 꾸려 메서드 연쇄 방식으로 비교자를 생성한다. → 간결하지만 성능 저하
    - 비교자 생성 메서드를 활용한 비교자

        ```jsx
        private static final Comparator<PhoneNumber> COMPARATOR 
        	= comparingInt((PhoneNumber pn) -> pn.areaCode) //comparingInt라는 static 메서드를 import하여 사용하고,
        		.thenComparingInt(pn -> pn.prefix)  //2번째 조건 부터 thenComparingInt를 사용하여 비교자를 추가 할 수 있다.
        		.thenComparingInt(pn -> pn.lineNum);

        public int compareTo(PhoneNumber pn){
        	return COMPARATOR.compare(this, pn);
        }

        ```

        - Comparator의 보조 생성 메서드

            comparingLong, thenComparingLong  comparingDouble, thenComparingDouble,
            comparingInt, thenComparingInt

        - 객체 참조용 비교자 생성 메서드
            1. comparing이라는 **정적 메서드** 2개가 다중정의되어 있다.
                - 첫 번째는 **키 추출자를 받아서 그 키의 자연적 순서를 사용**한다.
                - 두 번째는 키 추출자 하나와 추출된 키를 비교할 비교자까지 총 2개의 인수를 받는다.

                → 앞의 예시에서는 comparingInt는 객체 참조를 int타입키에 매핑하는 키 추줄자(람다함수)가 PhoneNumber에서 추출한 키(지역코드)를 기준으로 전화번호의 순서를 정하는 Comparator<PhoneNumber>를 반환한다.

            2. thenComparing이라는 **인스턴스 메서드**가 3개 다중정의되어 있다.
                - 첫 번째는 비교자 하나만 인수로 받아 그 비교자로 부차순서를 정한다.(comparing 1차, then~2,3차)
                - 두 번째는 키 추출자를 인수로 받아 그 키의 자연적 순서로 보조 순서를 정한다.
                - 세 번째는 키 추출자 하나와 추출된 키를 비교할 비교자까지 2개의 인수를 받는다.

                → 앞의 예시에서는 Comparator의 인스턴스 메서드인 thenComparingInt메서드는 int 키 추출자 함수를 입력받아 다시 비교자를 반환한다.

주의 : 해시코드 값의 차를 기준으로 하는 비교자는 추이성을 위배하기 때문에 사용하지 않는다.
→ 정수 오버플로를 일으키거나 IEEE 754 부동소수점 계산 방식에 따른 오류를 낼 수 있다.

```jsx
static Comparator<Object> hashCodeOrder = new Comparator<Object>() {
    @Override
    public int compare(Object o1, Object o2) {
        return o1.hashCode() - o2.hashCode();
    }
}
```

아래 둘 중 하나의 방법을 사용해야 한다.

1. **정적 compare메서드를 활용한 비교자**

    ```jsx
    static Comparator<Object> hashCodeOrder = new Comparator<Object>() {
        @Override
        public int compare(Object o1, Object o2) {
            return Integer.compare(o1.hashCode(), o2.hashCode());
        }
    }
    ```

2. **비교자 생성 메서드를 활용한 비교자 ( java8 )**

    ```jsx
    static Comparator<Object> hashCodeOrder = Comparator.comparingInt(o -> o.hashCode());
    ```

## 정리

- **순서를 고려해야 하는 값 클래스**라면 꼭 Comparable 인터페이스를 구현하여 **Collection**과 어우러지도록 해야한다.
- CompareTo 메서드에서 필드 값 비교시 **<, > 연산자는 사용하지 않도록 한다.**
- 박싱된 기본 타입 클래스가 제공하는 **정적 compare메서드**나 Comparator 인터페이스가 제공하는 **비교자 생성 메서드**를 ****사용하자.