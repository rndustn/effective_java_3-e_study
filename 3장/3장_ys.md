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