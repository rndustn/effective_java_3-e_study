# 3장. 모든 객체의 공통 메서드

Object는 객체를 만들 수 있는 구체 클래스이지만, 기본적으로 상속해서 사용하도록 설계되었다.

Object에서 final이 아닌 메서드(equals,hashCode,toString,clone,finalize)는 모두 재정의(overriding)을 염두에 두고 설계된 것이라 재정의 시 지켜야 하는 일반 규약이 명확히 정의되어 있다.

그래서 **Object를 상속하는 클래스, 즉 모든 클래스는 이 메서드들을 일반 규약에 맞게 재정의해야한다.**

## 아이템 10. equals는 일반 규약을 지켜 재정의하라

equals는 재정의하기 쉬워 보이지만, 잘못정의하면 끔직한 결과를 초래한다. Object equals가 왠만하면 비교를 정확하게수행해주기 때문에 꼭 필요한 경우가 아니면 재정의하지 말아야 하며,

재정의할 경우 다섯가지 규약을 확실히 지켜가며 비교해야 한다.

### equals를 재정의하지 않아도 되는 케이스

- 각 인스턴스가 본질적으로 고유하다. : 동작하는 개체를 표현하는 클래스 ex)thread
- 인스턴스의 논리적 동치성을 검사할 일이 없다.
- 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다. :

      ex) AbstractSet의 하위클래스 set, AbstractList의 하위클래스 list, AbstractMap의 하위클래스 Map

- 클래스가 private이거나 pakage-private이고 equals를 호출할 일이 없다.

그렇다면 equals를 재정의하고 싶을 때는?

값이 같은 지 알고 싶을 때.

### equals를 재정의할 때 반드시 지켜야 하는 규약

- **반사성(reflexivity)** : null이 아닌 모든 참조값 x에 대해, x.equals(x)는 true
- **대칭성(symmetry)** : null이 아닌 모든 참조값 x,y에 대해 x.equals(y) 가 true이면 y.equals(x)도 true다.
- **추이성(transitivity)**: null이 아닌 모든 참조 값 x,y,z에 대해 x.equals(y) 가 true고, y.equals(z)도 true면 x.equals(z) 도 true다
- **일관성(consistency)**: null이 아닌 모든 참조값 x,y에 대해, x.equals(y)를 반복해서 호출해도 항상 true거나 항상 false로 값이 같다.
- **null-아님**: null이 아닌 모든 참조값 x에 대해, x.equals(null)은 false다.

equals 규약을 어기면 그 객체를 사용하는 다른 객체들이 어떻게 반응할 지 알 수 없다.

### 양질의 equals 메서드 구현 방법

1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다. 자기 자신이면 true를 반환한다. 

* float과 double은 특수한 부동소수값등을 다뤄야 하기때문에 ==이 아닌 Float.compare(float,float)와 Double.compare(double,double)로 비교한다.
* 때론 null이 정상 값으로 취급하는 참조 타입 필드도 있다. 이런 필드는 정적 메서드인 Object.equals(Object,Object)로 비교해 NullPointException 발생을 예방하자.

 2.  instanceof 연산자로 입력이 올바른 타입인지 확인한다. 그렇지 않다면 false를 반환한다

 3. 입력을 올바른 타입으로 형변환한다.

 4. 입력 객체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는 지 하나씩 검사한다.

 5. equals를 재정의할 땐 hashcode도 반드시 재정의하자.

 6. Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자.

## 아이템 11. equals를 재정의하려거든 hashcode도 재정의하라

equals를 재정의한 클래스 모두에서 hash코드도 재정의해야 한다.

### Object 명세 규약

- equals 비교에 사용되는 정보가 변경되지 않는다면, 애플리케이션이 실행되는 동안 그 객체의 hashcode 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다.

      단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관겂다.

- equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.

      : 핵심 ! 즉, 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다. 

- equals(Object)가 두 객체를 다르다고 판다했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

### 해시코드를 작성하는 간단한 요령

1. int 변수 result를 선언한 후 값 c로 초기화 한다. 이 때 c는 해당 객체의 첫번째 핵심 필드를 단계 2.a 방식으로 계산한 해시코드다.

      * 핵심 필드 : equals 비교에 사용되는 필드

 2. 해당 객체의 나머지 핵심필드 f 각각에 대해 다음 작업을 수행한다.

   a. 해당 필드의 해시코드 c를 계산한다. 

      i. 기본 타입 필드라면, Type.hashCode(f)를 수행한다. 여기서 Type은 해당 기본타입의 박싱클래스다.

      ii. 참조 타입 필드면서 이 클래스의 equals 메서드가 이 필드의 equals를 재귀적으로 호출해 비교한다면, 이  

          필드의 hashCode를 재귀적으로 호출한다. 계산이 더 복잡해질 것 같으면, 이 필드의 표준형을 만들어 그 

          표준형의 hashCode를 호출한다. 필드의 값이 null이면 0을 사용한다. (다른 상수도 괜찮지만 정통적으로)

     iii. 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다. 이상의 규칙을 재귀적으로 적용해 각 핵심 원소

         의 해시코드를 계산한 다음, 단계 2.b 방식으로 갱신한다. 배열에 핵심 원소가 하나도 없다면 단순히 상수(0      

         을 추천)을 사용한다. 모든 원소가 핵심원소라면 arrays.hashcode를 사용한다.

   b. 단계 2.a에서 계산한 해시코드 c 로 result를 갱신한다. 코드로는 다음과 같다.

        result = 31 * result + c;

3. result를 반환한다.

*** 파생필드는 해시코드 계산에서 제외해도 된다. 또한, equals 비교에 사용되지 않는 필드는 반드시 제외한다.**

Objects.hash : 아쉽게도 속도가 느리다. 성능에 민감하지 않은 상황에서만 사용하자.

  

## 아이템 12. toString을 항상 재정의하라

toString은 간결하고 읽기 쉽다고 볼 수 있지만, 유익한 정보를 담고 있지는 않다.

toString을 잘 이용한 클래스는 사용하기 좋고, 디버깅하기쉽다.

toString은, 정적 유틸리티 클래스나 대부분의 열거타입은 자바가 이미 완벽하게 제공하니 재정의하지 않아도 되지만, 하위 클래스들이 공유해야 할 문자열 표현이 있는 추상클래스라면 재정의해줘야 한다.

대다수의 컬렉션 구현체는 추상컬렉션 클래스들의 toString 메서드를 상속해서 쓰기때문이다.

**재정의 시 주의해야할 점**

- toString은 그 객체가 가진 주요 정보를 모두 반환하는게 좋다.
- toString을 구현할 때면 반환값의 포맷을 문서화할지 정해야 한다.  포맷을 한번 명시하면 그 포맷에 얽매이게 될 수도 있지만, 포맷이든 아니든 의도는 명확하게 밝혀야 한다.
- toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하자.

## 아이템13. clone 재정의는 주의해서 진행하라

**Clonable인터페이스가 하는일은?**

이 인터페이스는 놀랍게도 Object의 protected메서드인 clone의 동작 방식을 결정한다.

Clonable을 구현한 클래스의 인스턴스에서 clone을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환하며, 그렇지 않은 클래스의 인스턴스에서 호출하면 CloneNotSupportedException을 던진다.

Clonable은 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스지만, 아쉽게도 의도한 목적을 제대로 이루지 못했다.

가장 큰 문제는 clone메서드가 선언된 곳이고 Clonable이 아닌 Object이고, 그 마저도 protected라는데 있다.

그래서 Cloneable을 구현하는 것만으로는 외부 객체에서 clone 메서드를 호출 할 수 없다.

실무에서 Cloneable을 구현한 클래스는 clone 메서드를 public으로 제공하며, 사용자는 당연히 복제가 제대로 이뤄지리라 기대한다.

이 기대를 만족시키려면 그 클래스와 모든 상위 클래스는 복잡하고, 강제할 수 없고, 허술하게 기술된 프로토콜을 지켜야만 하는데, 그 결과로 깨지기 쉽고 모순적인 메커니즘이 탄생한다. 생성자를 호출하지 않ㄷ고도 객체를 생성할 수 있게 되는 것이다.

clone 메서드의 일반 규약을 허술하다. Object 명세에서 가져온 다음 설명을 보자

```
이 객체의 복사본을 생성해 반환한다. '복사'의 정확한 뜻은 그 객체를 구현한 클래스에 따라 다를 수 있다.
일반적인 의도는 다음과 같다. 어떤 객체 x에 대해 다음 식은 참이다.
x.clone() != x
또한 다음식도 참이다.
x.clone().getClass == x.getClass()
하지만 이상의 요구를 반드시 만족해야 하는 것은 아니다.
한편 다음 식도 일반적으로 참이지만, 역시 필수는 아니다.
x.clone().equals(x)
관례상, 이 메서드가 반환하는 객체는 super.clone을 호출해 얻어야 한다.
이 클래스와(Object를 제외한) 모든 상위 클래스가 이 관례를 따른다면 다음 식은 참이다.
x.clone().getClass() == x.getClass()
관례상, 반환된 객체와 원복 객체는 독립적이어야 한다. 이를 만족하려면 super.clone으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수 도 있다.
```

clone 메서드는 사실상 생성자와 같은 효과를 낸다. 즉, clone은 원복 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다.

Cloneable 아키텍처는 "가변 객체를 참조하는 필드는 final로 선언하라"는 일반 용법과 충돌한다.

(단, 원본과 복제된 객체가 그 가변 객체를 공유해도 안전하다면 괜찮다.) 

그래서 복제할 수 있는 클래스를 만들기 위해 일부 필드에서 final 한정자를 제거해야 할 수도 있다.

clone을 잘 구현할 수 없는 상황에서는

복사 생성자와 복사 팩터리라는 더 나은 객체 복사 방식을 제공할 수 있다.

```java
public Yum(Yum yum){....}; //복사 생성자
```

```java
public static Yum newInstance(Yum yum){...}; //복사 팩터리
```

언어 모순적이고 위험천만한 객체 생성 메커니즘(생성자를 쓰지 않는 방식)을 사용하지 않으며,

엉성하게 문서화된 규약을 기대지 않고, 정상적인 final필드 용법과도 충돌하지 않으며, 불필요한 검사 예외를 던지지 않고, 형변환도 필요치 않다.

여기서 끝이 아니다. 복사 생성자와 복사 팩터리는 해당 클래스가 구현한 '인터페이스'타입의 인스턴스를 인수로 받을 수 있다. 예컨대 관례상 모든 범용 컬렉션 구현체는 Collection이나 Map타입을 받는 생성자를 제공한다.

## 아이템 14. Compareble을 구현할 지 고려하라.

Comparable 인터페이스의 유일무이한 메서드인 CompareTo를 알아보자.

성격은 두가지만 빼면 Object와 equals와 같다.

compareTo는 단순 동치성 비교에 더해 순서까지 비교할 수 있으며, 제네릭하다.

Compareble을 구현했다는 것은 그 클래스의 인스턴스들에는 자연적인 순서가 있음을 의미한다.

알파벳,숫자,연대같이 순서가 명확한 값 클래스를 작성한다면 반드시 Comparable 인터페이스를 구현하라.

**equals메서드와 다른점**

- compareTo메서드는 타입이 다른 객체를 신경쓰지 않아도 된다. 타입이 다른 객체가 주어지면 간단히 ClassCastException을 던져도된다.

**compareTo의 규약**

1. 두 객체 참조의 순서를 바꿔 비교해도 예상한 결과가 나와야 한다.
2. 첫번째가 두번째보다 크고 두번째가 세번째보다 크면, 첫번째는 세번째보다 커야 한다. 
3. 크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같아야 한다. : 필수는 아니지만, 꼭 지키길 권장한다. 간단히 말하면, compareTo 메서드로 수행한 동치성 테스트의 결과가 equals와 같아야 한다는 것이다.

이 세가지의 규약은 compareTo 메서드로 수행하는 동치성 검사도 equals 규약과 똑같이 반사성, 대칭성, 추이성을 충족해야 함을 뜻한다.

Comparable은 타입을 인수로 받는 제네릭 인터페이스이므로 compareTo 메서드의 인수타입은 컴파일 타임에 정해진다. 입력 인수의 타입을 확인하거나 형변환할 필요가 없다는 뜻이다. 인수의 타입이 잘못되면 컴파일 자체가 되지 않는다.

또한 null을 인수로 넣어 호출하면 NullPointerException을 던져야 한다.

compareTo 메서드는 각 필드가 동치인지를 비교하는게 아니라 그 순서를 비교한다.

```java
static Comparator<Object> hashCodeOrder = new Comparator<>(){
     public int compare(Object o1, Object o2){
          return Integer.compare(o1.hashCode(), o2.hashCode());
}
}; //정적 compare 메서드를 활용한 비교자
```

```java
static Comparator<Object> hashCodeOrder =
        Comparator.comparingInt(o -> o.hashCode()); //비교자 생성 메서드를 활용한 비교자
```