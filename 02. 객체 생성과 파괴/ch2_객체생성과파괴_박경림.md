# 2️⃣ 객체 생성과 파괴

태그: EffectiveJava

- 객체를 만들어야 할 때와 만들지 말아야 할 때
- 올바른 객체 생성 방법
- 불필요한 생성을 피하는 방법
- 제때 파괴됨을 보장하고 파괴 전에 수행해야 할 정리 작업을 관리하는 요령

# 아이템 1. 생성자 대신 static factory method를 고려하라

클래스의 인스턴스를 얻는 방법

1. public 생성자
    
    기존의 전통적인 수단 
    
2. static factory method 
    
    (디자인 패턴의 factory method와는 다름)
    
    클래스의 인스턴스를 반환 
    
    ```java
    // 기본 타입인 boolean 값을 받아 박싱 클래스인 Boolean 객체 참조로 변환해주는 코드
    public static Boolean valueOf(boolean b) {
    	return b ? Boolean.TRUE : Boolean.FALSE;
    }
    ```
    

→ 기존의 전통적인 수단인 public 생성자 말고, static factory method를 사용하자.

## 장점 1. 이름을 가질 수 있다.

### 생성자

매개변수와 생성자 자체만으로는 반환될 객체의 특성을 제대로 알 수 없다.

BigInteger(int, int, Random)

시그니처(return type, 매개변수 데이터 타입과 개수)에 제약이 있다.

### static factory method

이름을 잘 지으면 반환될 객체의 특성을 쉽게 묘사할 수 있다.

`BigInteger.probablePrime` (값이 소수인 BigInteger를 반환하는 함수) 

시그니처가 같은 생성자가 여러 개 필요할 때, 각각의 차이를 잘 드러내는 이름을 지어준 다음 사용한다.

## 장점 2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.

반드시 새로운 객체를 만들 필요가 없다.

불변 클래스(immutable class; 아이템 17)는 미리 만들어 둔 인스턴스 또는 새로 생성한 인스턴스를 캐싱하여 재활용하는 식으로 불필요한 객체 생성을 피한다. 

예: `Boolean.valueOf(boolean)` 

생성 비용이 큰, 또는 같은 객체가 자주 요청되는 상황이라면 성능 향상 

Flyweight Pattern: 인스턴스를 가능한 공유해서 사용함으로써 메모리를 절약하는 패턴 

instance-controlled class: 언제 어느 인스턴스를 살아 있게 할지를 철저하게 통제하는 클래스

- singleton(아이템 3), 인스턴스화 불가(noninstantiable; 아이템 4)로 만들 수 있다.
- 불변 클래스에서 인스턴스가 단 하나뿐임을 보장할 수 있다.

## 장점 3. 리턴 타입의 하위 타입 객체를 반환할 수도 있다.

리턴할 객체의 클래스를 자유롭게 선택할 수 있게 하는 유연성 제공 

리턴 타입은 인터페이스로 지정하고 인터페이스의 구현체는 API로 노출시키지 않지만 그 구현체의 인스턴스를 만들어 줄 수 있다.

예: `java.util.Collections` 의 45개 클래스를 공개하지 않기 때문에 API를 줄이고, 개념적인 무게, 즉 프로그래머가 API를 사용하기 위해 익혀야 하는 개념의 수와 난이도를 낮췄다. 

자바 8부터 인터페이스에 public static 메소드 추가할 수 있고, 자바 9부터 private static 메소드를 추가할 수 있다. 

## 장점 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

리턴 타입의 하위 타입인 클래스의 객체를 반환할 수 있다. 

예: `EnumSet` 클래스(아이템 36)은 생성자 없이 static method만 제공하는데 원소의 개수에 따라  `RegularEnumSet` 또는 `JumboEnumSet` 을 반환한다.

## 장점 5. static factory method를 작성하는 시점에는 리턴 객체의 클래스가 존재하지 않아도 된다.

service provider framework를 만드는 근간이 되는 유연함

서비스 프로파이더 프레임워크

- 구현체의 동작을 정의하는 서비스 인터페이스
- 제공자가 구현체를 등록할 때 사용하는 제공자 등록 API
- 클라이언트가 서비스의 인스턴스를 얻을 때 사용하는 서비스 접근 API: 원하는 구현체 조건을 명시하지 않은 경우 기본 구현체를 반환하거나 지원하는 구현체들을 하나씩 돌아가면 반환, ‘유연한 정적 팩터리’의 핵심
- 서비스 프로파이더 인터페이스: 부가적인 요소, 없을 경우 리플렉션을 사용해서 구현체를 만들어준다.

예: `JDBC(Java Database Connectivity)` 

- Connection
- DrvierManager.registerDriver
- DrvierManager.getConnection
- Driver

## 단점 1. public 또는 protected 생성자 없이 static factory method만 제공하는 클래스는 상속할 수 없다.

상속을 하려면 public이나 protected 생성자가 필요하니 static factory method만 제공하면 하위 클래스를 만들 수 없다.

따라서, Collections 프레임워크의 유틸리티 구현 클래스들(java.util.Collections)은 상속할 수 없다.

상속보다는 컴포지션을 사용(아이템 18)하도록 유도하고 불변 타입(아이템 17)인 경우 제약을 지켜야 한다는 점에서 장점이 될 수도 있다.

## 단점 2. 프로그래머가 static factory method를 찾기 어렵다.

생성자처럼 API 문서에 명확히 드러나지 않으니 사용자는 static factory method 방식 클래스를 인스턴스화 방법을 찾아야 한다. 

# 아이템 2. 생성자에 매개변수가 많다면 빌더를 고려하라

생성자와 static factory 둘 다 매개변수가 많아질 경우 제약이 생긴다. 

### 해결책 1. 점층적 생성자 패턴(telescoping constructor pattern)

필수 매개변수, 필수 매겨변수와 선택 매개변수 1개, 선택 매개변수 2개, …를 받는 생성자 형태로 여러 생성자를 만들어 모든 매개변수를 받는다.

클래스의 인스턴스를 만들 때 원하는 매개변수를 모두 포함한 생성자 중 가장 짧은 것을 골라 호출한다.

**매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 읽기 어렵다.**

### 해결책 2. 자바빈즈 패턴(JavaBeans pattern)

매개변수가 없는 생성자로 객체를 만든 후, 세터(setter)를 호출해 원하는 매개변수의 값을 설정하는 방식이다. 

```java
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```

코드의 가독성은 해결됐지만, 객체 하나를 만들기 위해서 여러 메서드들을 호출해야 하고, 객체가 완전히 생성되기 전에 일관성이 깨진 상태에서 사용될 수도 있다.

또한, 클래스를 불변으로 만들 수 없으며 스레드 안전성을 얻으려면 프로그래머가 추가 작업을 해야 한다.

### 해결책 3. 빌더 패턴(Builder pattern)

필수 매개변수만으로 생성자(or factory method)를 호출해 빌더 객체를 얻는다. 빌더 객체가 제공하는 일종의 세터 메서드들로 원하는 선택 매개변수를 설정한다. 마지막으로 매개변수가 없는 build 메서드를 호출해 필요한 객체를 얻는다.

빌더는 보통 생성할 클래스 안에 정적 멤버 클래스로 만들어둔다.

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

빌더의 생성자나 메소드에서 유효성 확인할 수도 있고, 여러 매개변수를 혼합해서 확인해야 하는 경우 build 메소드에서 호출하는 생성자에서 할 수 있다. 빌더로부터 매개변수를 복사한 후 해당 객체 필드들을 검사해 잘못된 점을 발견하면 IllegalArgumentException을 던질 수도 있다.

빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰면 좋다.

```java
// 코드 2-4 계층적으로 설계된 클래스와 잘 어울리는 빌더 패턴 (19쪽)

// 참고: 여기서 사용한 '시뮬레이트한 셀프 타입(simulated self-type)' 관용구는
// 빌더뿐 아니라 임의의 유동적인 계층구조를 허용한다.

public abstract class Pizza {
    public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
    final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }

        abstract Pizza build();

        // 하위 클래스는 이 메서드를 재정의(overriding)하여
        // "this"를 반환하도록 해야 한다.
        protected abstract T self();
    }
    
    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone(); // 아이템 50 참조
    }
}
```

```java
// 코드 2-5 뉴욕 피자 - 계층적 빌더를 활용한 하위 클래스 (20쪽)
public class NyPizza extends Pizza {
    public enum Size { SMALL, MEDIUM, LARGE }
    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        @Override public NyPizza build() {
            return new NyPizza(this);
        }

        @Override protected Builder self() { return this; }
    }

    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }

    @Override public String toString() {
        return toppings + "로 토핑한 뉴욕 피자";
    }
}
```

```java
// 코드 2-6 칼초네 피자 - 계층적 빌더를 활용한 하위 클래스 (20~21쪽)
public class Calzone extends Pizza {
    private final boolean sauceInside;

    public static class Builder extends Pizza.Builder<Builder> {
        private boolean sauceInside = false; // 기본값

        public Builder sauceInside() {
            sauceInside = true;
            return this;
        }

        @Override public Calzone build() {
            return new Calzone(this);
        }

        @Override protected Builder self() { return this; }
    }

    private Calzone(Builder builder) {
        super(builder);
        sauceInside = builder.sauceInside;
    }

    @Override public String toString() {
        return String.format("%s로 토핑한 칼초네 피자 (소스는 %s에)",
                toppings, sauceInside ? "안" : "바깥");
    }
}
```

```java
// 계층적 빌더 사용 (21쪽)
NyPizza pizza = new NyPizza.Builder(SMALL)
		.addTopping(SAUSAGE).addTopping(ONION).build();
Calzone calzone = new Calzone.Builder()
                .addTopping(HAM).sauceInside().build();
```

생성자 대신 빌더를 이용하면 가변인수 매개변수를 여러 개 사용할 수 있다. 아니면 메서드를 여러 번 호출하고 각 호출 때 넘겨진 매개변수들을 하나의 필드로 모을 수도 있다. (addTopping) 

빌더 패턴은 유연해서 빌더 하나로 여러 객체를 순회하면서 만들 수 있고, 매번 생성하는 객체에 시리얼 번호를 증가하는 식으로 객체에 조금씩 변화를 줄 수도 있다.

객체를 만들기 전에 먼저 빌더를 만들어야 하기 때문에 성능에 민감한 상황에서는 그 점이 문제가 될 수도 있다. 또한 코드가 장황해서 매개변수가 4개 이상 될 때 사용하는 것이 좋다.

# 아이템 3. private 생성자나 enum 타입으로 싱글톤임을 보증하라

싱글톤이 인터페이스를 구현한 게 아니라면 mock으로 교체하는 것이 어렵기 때문에 싱글톤을 사용하는 클라이언트 코드를 테스트 하는 것은 어렵다. 

싱글톤을 만드는 두 가지 방법 모두 생성자를 private으로 만들고, public static 멤버를 사용해서 유일한 인스턴스를 제공한다.

## final 필드

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() {
    }
}
```

**장점**

static factory method를 사용하는 방법에 비해 더 명확하고 간단하다.

## static factory method

```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() {
    }
    public static Elvis getInstance() {
        return INSTANCE;
    }
}
```

**장점**

API를 바꾸지 않고도 싱글톤이 아니게 변경할 수 있다. (다른 인스턴스를 반환) 

필요하다면 Generic 싱글톤 팩토리(아이템 30)로 만들 수도 있다.

static 팩토리 메소드를 Supplier<Elvis>에 대한 메소드 레퍼런스로 사용할 수도 있다.

## 직렬화(Serialization)

두 방법 모두 직렬화된 인스턴스를 역직렬화할 때마다 새로운 인스턴스를 만들어낸다. 문제를 해결하기 위해 모든 인스턴스 필드에 transient를 추가하고 readResolve 메소드를 다음과 같이 구현한다.

```java
    // 싱글톤임을 보장해주는 메소드
    private Object readResolve() {
		    // 진짜 Elvis를 반환하고, 가짜 Elvis는 가비지 컬렉터에 맡긴다.
        return INSTANCE;
    }
```

## enum

원소가 하나인 enum 타입을 선언하여 싱글톤을 만드는 방법

직렬화 문제도 코딩으로 해결할 필요도 없고, 리플렉션으로 호출되는 문제도 고민할 필요없다.

```java
public enum Elvis {
    INSTANCE;
}
```

만들려는 싱글톤이 enum 외의 클래스를 상속해야 한다면 사용할 수 없는 방법이다. (인터페이스는 구현할 수 있다.)

대부분 상황에서는 원소가 하나뿐인 열거 타입이 싱글톤을 만드는 것이 가장 좋은 방법이다.

# 아이템 4. 인스턴스화를 막으려거든 private 생성자를 사용하라

static 메서드와 static 필드만을 담은 클래스를 만든 경우 아무 생성자를 만들지 않으면 컴파일러가 기본적으로 기본 생성자를 만들어준다. 사용자는 이 생성자가 자동으로 만들어졌는지 구분할 수 없다.

추상 클래스로 만들어도 하위 클래스를 만들어 인스턴스화할 수도 있다.

→ 명시적으로 private 생성자를 추가하여 클래스의 인스턴스화를 막을 수 있다.

```java
public class UtilityClass {
    // 기본 생성자가 만들어지는 것을 막는다(인스턴스화 방지용).
    private UtilityClass() {
        throw new AssertionError();
    }
}
```

AssertionError는 꼭 필요하지는 않지만, 클래스 내에서 실수로라도 호출되지 않게 해준다. 

생성자를 존재하는데 호출할 수 없다는 것은 직관적이지 않다. 따라서 코드처럼 적절한 주석을 달아두는 것이 좋다.

생성자를 private으로 선언하여 상속도 막을 수 있다. 

# 아이템 5. 리소스를 직접 명시하지 말고 의존성 주입을 사용하라

맞춤법 검사기는 사전에 의존한다.

## 부적절한 구현

```java
// 부적절한 static 유틸리티 사용 예 - 유연하지 않고 테스트 할 수 없다.
public class SpellChecker {

    private static final Lexicon dictionary = new KoreanDicationry();

    private SpellChecker() {
        // Noninstantiable
    }

    public static boolean isValid(String word) {
        throw new UnsupportedOperationException();
    }

    public static List<String> suggestions(String typo) {
        throw new UnsupportedOperationException();
    }
}

interface Lexicon {}

class KoreanDicationry implements Lexicon {}
```

```java
// 부적절한 싱글톤 사용 예 - 유연하지 않고 테스트 할 수 없다.
public class SpellChecker {

    private final Lexicon dictionary = new KoreanDicationry();

    private SpellChecker() {
    }

    public static final SpellChecker INSTANCE = new SpellChecker() {
    };

    public boolean isValid(String word) {
        throw new UnsupportedOperationException();
    }

    public List<String> suggestions(String typo) {
        throw new UnsupportedOperationException();
    }

}
```

두 방법 모두 사전을 하나만 사용한다고 가정하면 괜찮은 구현일 수도 있다. 실제로는 용도별 다양한 사전이 필요할 수가 있다.

사용하는 리소스에 따라 동작이 달라지는 클래스에는 위와 같은 방법이 부적절하다.

## 적절한 구현

```java
public class SpellChecker {

    private final Lexicon dictionary;

    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public boolean isValid(String word) {
        throw new UnsupportedOperationException();
    }
    
    public List<String> suggestions(String typo) {
        throw new UnsupportedOperationException();
    }

}

class Lexicon {}
```

인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식이다. 

의존성 주입은 생성자, static factory(아이템 1), 빌더(아이템 2)에도 적용할 수 있다. 유연성과 테스트 용이성을 개선해주지만, 의존성이 수천 개나 되는 큰 프로젝트에서는 오히려 코드가 장황해진다. 그 점은 스프링 같은 프레임워크를 사용해서 해결할 수 있다.

클래스가 내부적으로 하나 이상의 리소스에 의존하는 경우 싱글톤과 static 유틸리티 클래스는 사용하지 않는 것이 좋다. 필요한 리소스를 생성자(static factory, builder)에 넘겨서 의존성 주입을 하자.

의존성 주입 → 클래스의 유연성, 재사용성, 테스트 용이성 개선 

# 아이템 6. 불필요한 객체 생성을 피하라

똑같은 기능의 객체를 매번 생성하기보다는 객체 하나를 재사용하는 것이 대부분 적절한다. 불변 객체(아이템 17)는 항상 재사용할 수 있다.

## 문자열 객체 생성

자바의 String을 new로 생성하면 항상 새로운 객체를 만든다.

```java
String s = new String("bikini"); // 따라 하지 말 것!
String s = "bikini"; // 올바른 방법 
```

같은 가상 머신에 동일한 문자열 리터럴이 존재한다면 그 리터럴을 재사용한다.

## static 팩토리 메소드

Boolean(String) (자바9에서는 deprecated됨) 대신 Boolean.valueOf(String) 같은 static 팩토리 메소드를 사용하여 불필요한 객체 생성을 피한다.

## 생성 비용이 비싼 객체

만드는 데 메모리나 시간이 오래 걸리는 비싼 객체를 반복적으로 만들어야 한다면 캐싱해두고 재사용하는 것을 고려한다.

## 어댑터

어댑터는 실제 작업은 뒷단 객체에 위임하고, 자신은 제2의 인터페이스 역할을 하는 객체다. 어댑터는 뒷단 객체만 관리하면 되므로 뒷단 객체 하나당 어댑터 하나씩만 만들어지면 충분하다.

## 오토박싱

오토박싱은 기본 타입과 그에 대응하는 박싱된 기본 타입의 구분을 흐리게 해주지만, 그 경계를 완전 없애는 것은 아니다.

```java
public class AutoBoxingExample {
    public static void main(String[] args) {
        Long sum = 0l;
        for (long i = 0 ; i <= Integer.MAX_VALUE ; i++) {
            sum += i;
        }
    }
}
```

sum 변수를 Long에서 long으로만 바꿔주면 6.3초에서 0.59초로 약 10배 이상 빨라진다. (Long 인스턴스가 약 2^31개가 만들어지기 때문에)

박싱 타입보다는 기본 타입을 사용하고, 의도치 않은 오토박싱을 조심하자. 

# 아이템 7. 다 쓴 객체 참조를 해제하라

예: 자바에서 스택에서 꺼내진 객체들을 가비지 컬렉터가 회수하지 않는다. 이럴 경우 메모리 누수가 일어난다. 

**해법**

해당 참조를 다 썼을 때 null 처리(참조 해제)하면 된다.

- 모든 경우는 아니고, 자기 메모리를 직접 관리하는 클래스(stack)라면 원소를 다 사용한 즉시 그 원소가 참조한 객체들을 다 null 처리해준다.

캐시도 시간이 지날수록 엔트리의 가치를 떨어뜨리는 방식을 사용하여 메모리 누수를 방지한다.

리스너 혹은 콜백을 약한 참조로 저장하여 조치해주지 않을 경우 gc가 수거할 수 있도록 한다.

# 아이템 8. finalizer와 cleaner 사용을 피하라

자바의 소멸자, 예측할 수 없고,  위험하고, 느리고, 일반적으로 불필요하다.

try-with-resources와 try-finally를 사용해 해결한다. (아이템 9)

종료해야 할 자원을 담고 있는 객체의 클래스는 AutoCloseable 구현해 해결한다.

**적절한 쓰임새**

1. 자원의 소유자가 close 메서드를 호출하지 않는 것에 대비한 안전망 역할
2. 네이티브 피어와 연결된 객체에서 gc 역할

# 아이템 9. try-finally보다는 try-with-resouces를 사용하라

자바 라이브러리에는 InputStream, OutputStream, java.sql.Connection 등 close 메서드를 호출해 직접 닫아줘야 하는 자원이 많다.

전통적으로 try-finally를 사용해 자원을 제대로 닫아줬다.

```java
    static void copy(String src, String dst) throws IOException {
        InputStream in = new FileInputStream(src);
        try {
            OutputStream out = new FileOutputStream(dst);
            try {
                byte[] buf = new byte[BUFFER_SIZE];
                int n;
                while ((n = in.read(buf)) >= 0)
                    out.write(buf, 0, n);
            } finally {
                out.close();
            }
        } finally {
            in.close();
        }
    }
```

자원이 둘 이상이면 try-finally 방식은 너무 지저분하다! 

기기에 물리적인 문제가 생긴다면 try 안에 코드도 예외를 던지고, finally 안에 close도 예외를 던질 것이다. 스택 추적 내역에 첫 번째 예외에 관한 정보를 남기지 않게 된다. (예외가 덮힘)

→ try-with-resources를 사용해 이 문제들을 해결한다.

try-with-resources 사용을 위해 해당 자원이 AutoCloseable 인터페이스를 구현해야 한다.

```java
    // 코드 9-3 try-with-resources - 자원을 회수하는 최선책! (48쪽)
    static String firstLineOfFile(String path) throws IOException {
        try (BufferedReader br = new BufferedReader(
                new FileReader(path))) {
            return br.readLine();
        }
    }
        
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
