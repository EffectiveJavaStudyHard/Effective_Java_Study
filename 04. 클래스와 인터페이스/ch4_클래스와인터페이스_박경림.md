- 자바에서 추상화의 기본 단위: 클래스, 인터페이스

### 📝 아이템 15. 클래스와 멤버의 접근 권한을 최소화하라 
#### 정보 은닉
모든 내부 구현을 완벽히 숨겨, 구현과 API를 깔끔히 분리한다.
-> 잘 설계된 컴포넌트

**장점**
- 여러 컴포넌트를 병렬로 개발하여, 시스템 개발 속도를 높인다. 
- 시스템 관리 비용을 낮춘다.
- 성능 최적화에 도움을 준다.
- 소프트웨어 재사용성을 높인다.
- 큰 시스템을 제작하는 난이도를 낮춰준다.

#### 자바의 정보 은닉 장치
접근 제한자
기본 원칙: 모든 클래스와 멤버의 접근성을 가능한 한 좁혀야 한다.

**톱레벨 클래스와 인터페이스**
- public: 공개 API가 되므로 하위 호환을 위해 영원히 관리해줘야만 한다.
- package-private: 해당 패키지 안에서만 사용, API가 아닌 내부 구현이 되어 언제든 수정 가능 
	- private static으로 중첩시키면 바깥 클래스 하나에서만 접근 가능

**멤버(필드, 메서드, 중첩 클래스, 중첩 인터페이스)**
- private: 멤버를 선언한 클래스에서만 접근 가능
- package-private: 소속된 패키지 안의 모든 클래스에서 접근 가능
- protected: package-private + 하위 클래스에서도 접근 가능
- public: 모든 곳에서 접근 가능

**접근 제한자 부여**
- 공개 API 설계 후, 그 외의 모든 멤버는 private으로 
- 같은 패키지의 다른 클래스가 접근해야 하는 멤버에 한하여 package-private으로 
- private과 package-private은 해당 클래스의 구현에 해당하므로 공개 API에 영향을 주지 않는다.
- protected로 바꾸는 순간 그 멤버에 접근할 수 있는 대상 범위가 엄청나게 넓어진다. 따라서 protected 멤버의 수는 적을수록 좋다.

**멤버 접근성을 좁히지 못하게 방해하는 제약**
- 상위 클래스의 메서드를 재정의할 때 그 접근 수준을 상위 클래스에서보다 좁게 설정할 수 없다.
- 리스코프 치환 원칙(상위 클래스의 인스턴스는 하위 클래스의 인스턴스로 대체해 사용할 수 있어야 한다)에 의해 필요한다. 어길 시 컴파일 오류 발생

#### public 클래스의 인스턴스 필드는 되도록 public이 아니어야 한다(아이템 16)
- final이 아닌 인스턴스 필드를 public으로 선언하면 불변식으로 보장할 수 없게 된다. 
- 상수라면 public static final 필드로 공개해도 괜찮다. 

#### public static final 배열 필드를 두거나 이 필드를 반환하는 접근자 메서드를 제공해서는 안 된다
- 클라이언트에서 배열의 내용을 수정할 수 있게 된다.
```
public static final Thing[] VALUES = { ... }; // 보안 허점 존재 
```
**해결책**

1. 배열을 private으로 만들고 public 불변 리스트를 추가
```
private static final Thing[] PRIVATE_VALUES = { ... };
public static final List<Thing> VALUES =
	Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
```
2. 배열을 private으로 만들고 그 복사본을 반환하는 public 메서드를 추가 (방어적 복사)
```
private static final Thing[] PRIVATE_VALUES = { ... };
public static final Thing[] values() {
	return PRIVATE_VALUES.clone();
}
```
> ✨ 핵심 정리
- 꼭 필요한 것만 골라 최소한의 public API를 설계하자
- public 클래스는 상수용 public static fianl 필드(참조하는 객체는 불변) 외에는 어떠한 public 필드도 가져서는 안 된다.


### 📝 아이템 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라 
#### **퇴보한 클래스**
```
class Point {
	public double x;
    public double y;
}
```
#### private을 바꾸고 public 접근자(getter) 추가
```
class Point {
	private double x;
    private double y;
    
    public Point(double x, double y) {
    	this.x = x;
        this.y = y;
    }
    
    public double getX() { return x; }
    public double getY() { return y; }
    
    public void setX(double x) { this.x = x; }
    public void setY(double y) { this.y = y; }
}
```
#### package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출해도 문제가 없다
- 클래스 선언이나 사용하는 클라이언트 코드에서나 접근자 방식보다 훨씬 깔끔 

### 📝 아이템 17. 변경 가능성을 최소화하라 
**불변 클래스**
- 인스턴스의 내부 값을 수정할 수 없는 클래스
- String, 박싱 클래스, BigInteger, BigDecimal
- 가변 클래스보다 설계하고 구현하고 사용하기 쉬우며, 오류가 생길 여지도 적고 훨씬 안전하다. 
#### 불변 클래스의 다섯 가지 규칙
1. 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다.
2. 클래스를 확장할 수 없도록 한다.
3. 모든 필드를 final로 선언한다.
4. 모든 필드를 private으로 선언한다.
5. 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다. 

#### 장점
- **불변 객체는 단순하다.**
불변 객체는 생성된 시점의 상태를 파괴될 때까지 그대로 간직한다.
- **불변 객체는 근본적으로 스레드 안전하여 따로 동기화할 필요 없다.**
불변 객체는 안심하고 공유할 수 있다. 
- **불변 객체는 자유롭게 공유할 수 있음은 물론, 불변 객체끼리는 내부 데이터를 공유할 수 있다.**

#### 단점
- **값이 다르면 반드시 독립된 객체로 만들어야 한다.**
가변 동반 클래스를 제공하여 문제점 해결(String과 StringBuilder)

#### 클래스 생성
- 클래스는 꼭 필요한 경우가 아니라면 불변이어야 한다.
- 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이자.
- 특별한 이유가 없다면 모든 필드는 private final이어야 한다.
- 생성자는 불변식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야 한다. 

### 📝 아이템 18. 상속보다는 컴포지션을 사용하라
- 상속: 클래스가 다른 클래스를 확장하는 구현 상속만 해당(인터페이스 상속은 무관)
#### 메서드 호출과 달리 상속은 캡슐화를 깨뜨린다.
- 상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 생길 수 있다. 
- 예: HashSet을 상속한 클래스, addAll이 add를 호출해서 count 값이 의도대로 나오지 않는다.
```
// 코드 18-1 잘못된 예 - 상속을 잘못 사용했다! (114쪽)
public class InstrumentedHashSet<E> extends HashSet<E> {
    // 추가된 원소의 수
    private int addCount = 0;

    public InstrumentedHashSet() {
    }

    public InstrumentedHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
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
        InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
        s.addAll(List.of("틱", "탁탁", "펑"));
        System.out.println(s.getAddCount());
    }
}
```
#### 컴포지션
- Set 인터페이스를 구현했고, Set의 인스턴스를 인수로 받는 생성자를 제공
- 임의의 Set에 계측 기능을 덧씌워 새로운 Set으로 만드는 것 
- 래퍼 클래스: 다른 Set 인스턴스를 감싸고 있다.
- 콜백 프레임워크와는 어울리지 않는다. 
```
// 코드 18-2 래퍼 클래스 - 상속 대신 컴포지션을 사용했다. (117-118쪽)
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

#### 전달 클래스
```
// 코드 18-3 재사용할 수 있는 전달 클래스 (118쪽)
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

> ✨ 핵심 정리
- 상속은 상위 클래스와 하위 클래스가 순수한 is-a 관계일 때만 써야 한다.
- 상위 클래스와 하위 클래스의 패키기자 다르고, 상위 클래스가 확장을 고려해 설계되지 않았다면 여전히 문제를 일으킬 수 있다.
- 상속의 취약점을 피하려면 상속 대신 컴포지션과 전달을 사용하자.

### 📝 아이템 19. 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라
- 상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지 문서로 남겨야 한다. 
- 상속용 클래스를 시험하는 유일한 방법은 직접 하위 클래스를 만들어보는 것이다. 
- 상속용 클래스의 생성자는 재정의 가능 메서드를 호출해서는 안 된다.
	- private, final, static 메서드는 재정의가 불가능하니 생성자에서 호출해도 된다.
- clone과 readObject 모두 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안 된다. 
- 상속용으로 설계하지 않은 클래스는 상속을 금지한다.
    - final 클래스
    - 모든 생성자는 private이나 package-private으로 선언하고 public 정적 factory를 만들어주는 방법
> ✨ 핵심 정리
	- 상속용 클래스는 문서를 꼭 남긴다.
    - 상속을 금지하려면 final 클래스를 선언하거나 생성자를 private이나 package-private으로 선언해 외부에서 접근하지 못하도록 한다. 
    
### 📝 아이템 20. 추상 클래스보다는 인터페이스를 우선하라
- 자바가 제공하는 다중 구현 메커니즘: 인터페이스, 추상 클래스
- 자바 8부터는 인터페이스도 default method 제공
- 자바는 단일 상속만 지원, 추상 클래스 방식은 새로운 타입을 정의하는 데 제약이 있음
- 인터페이스는 다른 어떤 클래스를 상속했든 같은 타입으로 취급 
#### 장점
**기존 클래스에도 쉽게 새로운 인터페이스를 구현해넣을 수 있다** 
- 기존 클래스 위에 새로운 추상 클래스를 끼워넣기는 어렵다. 

**인터페이스는 믹스인(mixin) 정의에 안성맞춤이다**
- 믹스인: 클래스가 구현할 수 있는 타입, 원래 주된 타입 외에도 특정 선택적 행위를 제공 

**인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다 **
```
public interface SingerSongwriter extends Singer, Songwriter {
	AudioClip strum();
    void actSensitive();
}
```
**인터페이스는 기능을 향상시키는 안전하고 강력한 수단이 된다**

#### 인터페이스와 추상 골격 구현(sekeletal implementaion) 클래스를 함께 제공
- 인터페이스: 타입 정의, 필요하면 디폴트 메서드 몇 개도 함께 제공
- 골격 구현 클래스: 나머지 메서드들까지 구현 
- 템플릿 메서드 패턴

### 📝 아이템 21. 인터페이스는 구현하는 쪽을 생각해 설계하라
**디폴트 메서드는 기존 구현체에 런타임 오류를 일으킬 수 있다**
- 디폴트 메서드는 인터페이스로부터 메서드를 제거하거나 기존 메서드의 시그니처를 수정하는 용도가 아니다.

### 📝 아이템 22. 인터페이스는 타입을 정의하는 용도로만 사용하라
- 인터페이스: 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할
- 상수 인터페이스: static final 필드로만 가득 찬 인터페이스 
```
// 코드 22-1 상수 인터페이스 안티패턴 - 사용금지! (139쪽)
public interface PhysicalConstants {
    // 아보가드로 수 (1/몰)
    static final double AVOGADROS_NUMBER   = 6.022_140_857e23;

    // 볼츠만 상수 (J/K)
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;

    // 전자 질량 (kg)
    static final double ELECTRON_MASS      = 9.109_383_56e-31;
}
```
- 클래스 내부에서 사용하는 상수는 외부 인터페이스가 아니라 내부 구현에 해당한다.
- 상수를 공개할 목적이라면 그 클래스나 인터페이스 자체에 추가해야 한다. (예: Integer와 Double의 MIN_VALUE와 MAX_VALUE)
- 아니면 인스턴스화할 수 없는 유틸리티 클래스(아이템 4)
```
// 코드 22-2 상수 유틸리티 클래스 (140쪽)
public class PhysicalConstants {
  private PhysicalConstants() { }  // 인스턴스화 방지

  // 아보가드로 수 (1/몰)
  public static final double AVOGADROS_NUMBER = 6.022_140_857e23;

  // 볼츠만 상수 (J/K)
  public static final double BOLTZMANN_CONST  = 1.380_648_52e-23;

  // 전자 질량 (kg)
  public static final double ELECTRON_MASS    = 9.109_383_56e-31;
}
```
> ✨ 핵심 정리
인터페이스는 상수 공개용 수단으로 사용하면 안 된다. 타입을 정의하는 용도로만 사용하자.

### 📝 아이템 23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라
**태그 달린 클래스**
```
// 코드 23-1 태그 달린 클래스 - 클래스 계층구조보다 훨씬 나쁘다! (142-143쪽)
class Figure {
    enum Shape { RECTANGLE, CIRCLE };

    // 태그 필드 - 현재 모양을 나타낸다.
    final Shape shape;

    // 다음 필드들은 모양이 사각형(RECTANGLE)일 때만 쓰인다.
    double length;
    double width;

    // 다음 필드는 모양이 원(CIRCLE)일 때만 쓰인다.
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
- 장황하고, 오류 내기 쉽고, 비효율적

**클래스 계층구조**
```
// 코드 23-2 태그 달린 클래스를 클래스 계층구조로 변환 (144쪽)
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
        this.width  = width;
    }
    @Override double area() { return length * width; }
}
```
- 간결, 명확
- final 필드 

### 📝 아이템 24. 멤버 클래스는 되도록 static으로 만들어라
- 중첩 클래스(nested class): 다른 클래스 안에 정의된 클래스 
- 정적 멤버 클래스, (비정적) 멤버 클래스, 익명 클래스, 지역 클래스
- 정적 멤버 클래스 제외 나머지는 inner class에 해당
#### 정적 멤버 클래스
- 메서드 밖에서도 사용해야 하거나 메서드 안에 정의하기엔 너무 길고, 바깥 인스턴스를 참조하지 않을 때
- 다른 클래스 안에 선언되고, 바깥 클래스의 private 멤버에도 접근할 수 있다.
- 멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여서 정적 멤버 클래스로 만들자.
#### 비정적 멤버 클래스
- 메서드 밖에서도 사용해야 하거나 메서드 안에 정의하기엔 너무 길고, 바깥 인스턴스를 참조할 때
- 비정적 멤버 클래스의 인스턴스는 바깥 클래스의 인스턴스와 암묵적으로 연결된다. 
#### 익명 클래스 
- 중첩 클래스가 한 메서드 안에서만 쓰이면서 그 인스턴스를 생성하는 지점이 단 한 곳이고 해당 타입으로 쓰기에 적합한 클래스나 인터페이스가 이미 있다면 
- 멤버와 달리, 쓰이는 시점에 선언과 동시에 인스턴스가 만들어진다. 
#### 지역 클래스
- 멤버 클래스: 이름이 있고 반복해서 사용할 수 있다.
- 익명 클래스: 비정적 문맥에서 사용될 때만 바깥 인스턴스를 참조할 수 있다.

### 📝 아이템 25. 톱레벨 클래스는 한 파일에 하나만 담으라
- 컴파일러가 한 클래스에 대한 정의를 여러 개 만들어 내는 일이 없다.
- 소스 파일을 어떤 순서로 컴파일하든 바이너리 파일이나 프로그램의 동작이 달라지는 일은 일어나지 않는다. 
1. 톱레벨 클래스들을 서로 다른 소스 파일로 분리하거나
2. 정적 멤버 클래스를 사용하는 방법
```
// 코드 25-3 톱레벨 클래스들을 정적 멤버 클래스로 바꿔본 모습 (151-152쪽)
public class Test {
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
