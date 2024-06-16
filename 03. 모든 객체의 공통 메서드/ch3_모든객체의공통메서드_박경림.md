- Object의 final이 아닌 메서드(equals, hashCode, toString, clone, finalize)와 Comparable.compareTo를 언제 어떻게 재정의해야 하는지

## 📝 아이템 10. equals는 일반 규약을 지켜 재정의하라
equals: 재정의하지 않는 것이 최선이고, 그냥 두면 그 클래스의 인스턴스는 오직 자기 자신과만 같다.

#### 재정의하지 않아도 되는 상황
- 각 인스턴스가 본질적으로 고유하다. (예: Thread)
- 인스턴스의 논리적 동치성(logical equality)을 검사할 일이 없다.
- 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.
- 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.
  ```
  @Override public boolean equals(Object o) {
      throw new AssertionError(); // 실수로라도 호출 방지
  }
  ```
#### equals 재정의가 필요한 상황
객체 식별성(object identity; 물리적으로 같은지)이 아니라 논리적 동치성을 확인해야 될 때
- 주로 값 클래스들이 해당(Integer, String 등)
- 객체가 같은지가 아니라 값이 같은지를 알고 싶을 때 equals 재정의
- 값 클라스라고 해도 인스턴스가 둘 이상 만들어지지 않음이 보장되면 equals 재정의가 필요 없음(enum), 논리적 동치성 == 객체 식별성

### equals 재정의 시 따라야 할 일반 규약
Object 명세에 적힌 규약, 동치관계를 만족시키기 위한 다섯가지 요건 
#### 반사성(reflexivity)
null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true이다.
- 객체는 자기 자신과 같다.
#### 대칭성(symmetry)
null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)가 true면 y.equals(x)도 true다.
- 두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다.
#### 추이성(transitivity)
null이 아닌 모든 참조 값 x, y, z에 대해, x.equals(y)가 true이고 y.equals(z)도 true면 x.equals(z)도 true다.
- 구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.
- 상속 대신 컴포지션을 사용하는 방법이 있다. 
#### 일관성(consistency)
null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
- 두 객체가 같다면 앞으로도 영원히 같아야 한다. 
#### null-아님
null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false다.

### equals 메서드 구현 방법
1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
	자기 자신이면 true를 반환
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
	올바르지 않으면 false를 반환 
    인터페이스를 구현한 클래스라면 equals에서 해당 인터페이스를 사용(Set, List, Map 등 해당)
3. 입력을 올바른 타입으로 형변환한다.
4. 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다. 
	모든 필드가 일치하면 true, 하나라도 다르면 false를 반환 
    기본 타입 필드는 == 연산자, float과 double은 각각 Float.compare, Double.compare, 참조 타입 필드는 equals 메서드로 비교 
    다를 가능성이 더 크거나 비교하는 비용이 싼 필드를 먼저 비교하여 성능 향상 
    
-> 대칭적인가? 추이성이 있는가? 일관적인가? 확인

### 주의사항
- equals를 재정의할 땐 hashCode도 반드시 재정의하자. 
- 필드들의 동치성만 검사해도 equals 규약을 어렵지 않게 지킬 수 있다.
- Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자.
```
	public boolean equals(MyClass o) { // 잘못된 예 - 다중정의, 재정의x
    }
```

> ✨ 핵심 정리
	- 꼭 필요한 경우가 아니면 equals 재정의 x
    - 대부분 Object의 equals가 원하는 비교를 수행해 줌
    - 재정의해야 할 때는 클래스의 핵심 필드 모두 빠짐없이, 다섯 가지 규약을 확실히 지켜가며 비교하기
    
## 📝 아이템 11. equals를 재정의하려거든 hashCode도 재정의하라 
equals를 재정의한 클래스에서 hashCode를 재정의하지 않으면 HashMap이나 HashSet의 원소로 사용할 때 문제가 발생한다.
#### Object 명세에서 발췌한 규약
- equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode도 서로 같은 값을 반환해야 한다.
- equals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.
### 좋은 hashCode를 작성하는 간단한 요령
1. int 변수 result를 선언한 후 값 c로 초기화한다.
2. 해당 객체의 나머지 핵심 필드 f 각각에 대해 다음 작업을 수행한다.
	a. 해당 필드의 해시코드 c를 계산한다.
    	ⅰ. 기본 타입 필드라면, Type.hashCode(f)를 수행, Type(박싱 클래스)
        ⅱ. 참조 타입 필드면서 이 클래스의 equals 메서드가 이 플드의 equals를 재귀적으로 호출해 비교한다면, 이 필드의 hashCode를 재귀적으로 호출한다. 계산이 더 복잡해질 것 같으면, 이 필드의 표준형을 만들어 그 표준형의 hashCode를 호출한다. 필드의 값이 null이면 0을 사용한다.
        ⅲ. 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다. 이상의 규칙을 재귀적으로 적용해 각 핵심 원소의 해시코드를 계산한 다음, 2.b 방식으로 갱신한다. 배열에 핵심 원소가 하나도 없다면 단순히 상수(0)를 사용한다. 모든 원소가 핵심 원소라면 Arrays.hashCode를 사용한다.
	b. 2.a 단계에서 계산한 해시코드 c로 result를 갱신한다.
    	result = 31 * result + c;
3. result를 반환한다. 

## 📝 아이템 12. toString을 항상 재정의하라 
toString: 간결하면서 사람이 읽기 쉬운 형태의 유익한 정보를 반환해야 함 
- 모든 구체 클래스에서 Object의 toString을 재정의하자

## 📝 아이템 13. clone 재정의는 주의해서 진행하라
Cloneable 인터페이스: Object의 protected 메서드인 clone의 동작 방식을 결정 
Clonealbe을 구현한 클래스의 인스턴스에서 clone을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환하며, 그렇지 않은 클래스의 인스턴스에서 호출하면 CloneNotSupportedException을 던짐 

## 📝 아이템 14. Comparable을 구현할지 고려하라
compareTo
- Comparable 인터페이스의 메서드
- 단순 동치성 비교, 순서 비교, 배열 정렬 가능 
- 자바 플랫폼 라이브러리의 모든 값 클래스와 열거 타입은 Comparable 구현 
```
pulic interface Comparable<T> {
	int compareTo(T t);
}
```
- 타입이 다른 객체가 주어지면 ClassCastException 던짐 
- compareTo 규약을 지키지 못하면 비교를 활용하는 클래스와 어울리지 못한다. (TreeSet, TreeMap, Collections, Arrays)
#### compareTo 규약
1. 두 객체 참조의 순서를 바꿔 비교해도 예상한 결과가 나와야 한다. 
2. 첫 번째가 두 번째보다 크고 두 번째가 세 번째보다 크면, 첫 번째는 세 번째보다 커야 한다. 
3. 크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같아야 한다. compareTo 결과가 equals랑 같아야 한다. 
#### compareTo 작성
- 자바 7부터는 compare 함수 이용
```
public int compareTo(PhoneNumber pn) {
	int result = Short.compare(areaCode, pn.areaCode); // 가장 중요한 필드 
    if (result == 0) {
    	result = Short.compare(prefix, pn.prefix);
        if (result == 0) {
        	result = Short.comprea(lineNum, pn.lineNum);
        }
    }
}
```
- 비교자 생성 메서드를 활용하면 성능 향상할 수 있다.
```
private static final Comparator<PhoneNumber> COMPARATOR =
	comparingInt((PhoneNumber pn) -> pn.areaCode)
    	.thenComparingInt(pn -> pn.prefix)
        .thenComparingInt(pn -> pn.lineNum);
public int compareTo(PhoneNumber pn) {
	return COMPARATOR.compare(this, pn);
}
```
- 해시코드 값의 차를 기준으로 하는 비교자는 추이성을 위배한다. (정수 오버플로, 부동소수점 오류)
```
static Comparator<Object> hashCodeOrder = new Comparator<>() {
	public int compare(Object o1, Object o2) {
    	return o1.hashCode() - o2.hashCode();
    }
};
```
- 정적 compare 메서드를 활용한 비교자
```
static Comparator<Object> hashCodeOrder = new Comparator<>() {
	public int compare(Object o1, Object o2) {
    	return Integer.compare(o1.hashCode(), o2.hashCode());
    }
}
```
- 비교자 생성 메서드를 활용한 비교자
```
static Comparator<Object> hashCodeOrder =
	Comparator.comparingInt(o -> o.hashCode());
```
> ✨ 핵심 정리
	- 순서를 고려해야 하는 값 클래스에서는 꼭 Comparable 인터페이스를 구현하여, 인스턴스들을 쉽게 정렬하고, 검색, 비교 기능을 제공하는 컬렉션과 어우러지도록 해야 한다.
    - compareTo 메서드를 구현할 때는 정적 compare 메서드나 Comparator 인터페이스가 제공하는 비교자 생성 메서드를 사용하자.
