# 아이템10 - equals는 일반 규약을 지켜 재정의하라

## 재정의하지 않는 것이 좋은 경우

### 1. 각 인스턴스가 본질적으로 고유할 때

* 값을 표현하는 게 아니라 동작하는 개체를 표현하는 클래스인 경우
  * ex) Thread 클래스는 Object의 equals 메서드로도 충분하다.

### 2. '논리적 동치성(logical equality)'을 검사할 일이 없을 때

* ex) 두 개의 Random 객체가 같은 난수열을 만드는지 확인하는 것은 의미가 없음

### 3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 적용가능할 때

* 대부분의 Set 구현체는 AbstractSet이 구현한 equals를 상속받아 씀(List와 Map도 동일)

### 4. 클래스가 private이거나 package-private이고 equals 메서드가 호출할 일이 없을 때

* 이런 경우에는 실수로라도 호출을 막기 위한다면 equals 메서드를 오버라이딩해서 호출되지 않도록 막는다

```java
@Override
public boolean equals(Object o) {
    throw new ~Exception(); // 메서드 호출 방지
}
```

### 5. 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제 클래스일 때

* ex) 싱글턴, Enum

## 재정의하는 것이 좋은 경우

### 객체의 논리적 동치성을 비교해야 할 때

* Integer, String 같은 값 클래스들의 값이 동일한지 알고 싶을 때

## equals 메서드를 재정의할 때는 일반 규약을 따르자

### 1. 반사성(reflexivity)

* `null`이 아닌 모든 참조 값 x에 대해 `x.equals(x)`는 true다
* 이건 위반하는 것이 더 어렵다

### 2. 대칭성(symmertry)

* `null`이 아닌 모든 참조 값 x, y에 대해 `x.equals(y)`가 true면, `y.equals(x)`도 true다
* ex) 대소문자를 구별하지 않는 문자열 클래스의 equals

```java
public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }
}
```

* 대칭성을 위배하는 equals

```java
@Override
public boolean equals(Object o) {
    if (o instanceof CaseInsensitiveString) {
        return s.equalsIgnoreCase((CaseInsensitiveString) o).s);
    }
    if (o instanceof String) {	// 한 방향으로만 작동한다
        return s.equalsIgnoreCase((String) o);
    }
}

CaseInsensitiveString cis = new CaseInsensitiveString("Media");
String s = "media";

cis.equals(s); // true
s.equals(cis); // false
```

* 대칭성을 준수하는 equals

```java
@Override 
public boolean equals(Object o) {
  return o instanceof CaseInsensitiveString &&
    ((CaseInsensitiveString) o).s.equalsIgnoreCase (s);
}
```

### 3. 추이성(transitivity)

* `null`이 아닌 모든 참조 값 x, y, z에 대해 `x.equals(y)`가 true이고, `y.equals(z)`가 true 이면, `x.equals(z)`도 true다
* 추이성을 위배하는 equals

```java
// ColorPoint는 Point를 상속받음
@Override 
public boolean equals(Object o) {
    if(!(o instanceof Point))
      return false;
      
    if(!(o instanceof ColorPoint))
      return o.equals(this);
      
    return super.equals(o) && ((ColorPoint) o).color == color;
}

ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
Point p2 = new Point(1, 2);
ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);

p1.equals(p2);	// true
p2.equals(p3);	// true
p1.equals(p3);	// false
```

* 구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다
* instanceof 검사 대신 getClass 검사로 바꾸면 된다는 의미일까? -> **리스코프 치환 원칙을 위배하므로 NO!**
  * 리스코프 치환 원칙 : 어떤 타입에 있어 중요한 속성이라면 그 하위 타입에서도 마찬가지로 중요하다. 따라서 그 타입의 모든 메서드가 하위 타입에서도 똑같이 잘 작동해야한다.

#### **방법 1) 상속 대신 컴포지션을 사용(아이템 18)**

1. Point를 상속하는 대신 Point를 ColorPoint의 private 필드로 두기
2. ColorPoint와 같은 위치의 일반 Point를 반환하는 뷰 메서드를 public으로 추가

```java
public class ColorPoint {
	private final Point point;
	private final Color color;

	public ColorPoint(int x, int y, Color color) {
		point = new Point(x, y);
		this.color = Objects.requireNonNull(color);
	}

	// ColorPoint의 Point 뷰를 반환
	public Point asPoint() {
		return point;
	}

	@Override 
	public boolean equals(Object o) {
		if(!(o instanceof ColorPoint))
			return false;
		ColorPoint cp = (ColorPoint) o;
		return cp.point.equals(point) && cp.color.equals(color);
	}
}
```

#### **방법 2) 추상 클래스의 하위 클래스 사용하기**

* 추상 클래스의 하위 클래스에서는 equals 규약을 지키면서도 값을 추가할 수 있다
* ex) 아무런 값을 갖지 않는 추상 클래스 Shape와 radius 필드를 추가한 Circle 클래스, length와 width 필드를 추가한 Rectangle 클래스

### 4. 일관성(consistency)

* `null`이 아닌 모든 참조 값 x, y에 대해 `x.equals(y)`를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다
* 가변 객체는 비교 시점에 따라 서로 다를 수 있으나 불변 객체는 한번 다르면 끝까지 달라야함
* 클래스가 불변이든 가변이든 **equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안 됨**
* ex) `java.net.URL`의 equals - 주어진 URL과 매핑된 호스트의 IP 주소를 이용하여 비교하는데, 이는 시간이 바뀌면서 바뀔 수 있으므로 사용하면 안 된다

### 5. null 이 아니다

* `null`이 아닌 모든 참조 값 x에 대해서 `x.equals(null)`은 false다
* 명시적 검사 보다는 묵시적 검사를 하자

```java
// 명시적 null 검사 - 사실상 필요 없다
@Override 
public boolean equals(Object o) {
	if(o == null) {
		return false;
	}
}

// 묵시적 null 검사 - 이쪽이 낫다
@Override 
public boolean equals(Obejct o) {
	// instanceof 자체가 타입과 무관하게 null이면 false 반환
	if(!(o instanceof MyType))
		return false;
	MyType mt = (MyType) o;
}
```

## 양질의 equals 메서드 구현 단계

### 1단계. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다

* 자기 자신이면 true를 반환하며, 이 단계는 성능 최적화용이다

### 2단계. instanceof 연산자로 입력이 올바른 타입인지 확인한다

* 올바른 타입 : equals가 정의된 클래스 또는 그 클래스가 구현한 특정 인터페이스
  * ex) List, Map, Map.Entry 등의 컬렉션 인터페이스

### 3단계. 입력을 올바른 타입으로 형변환 한다

* 2단계에서 instatnceof 검사를 했기 때문에 해당 단계는 100% 성공함

### 4단계. 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다

* 2단계에서 인터페이스를 사용했다면 입력의 필드값을 가져올 때도 인터페이스 메서드를 사용해야함

## equals 구현시 주의사항

### 1. 기본타입과 참조타입에 따른 비교 방법을 적용하자

* **float, double를 제외한 기본타입**은 == 연산자로 비교
* **참조타입**은 equals로 비교
* **float, double**의 경우 정적 메서드인 `Float.compare(float, float)`, `Double.compare(double, double)`로 비교 (부동소수 값)
  * Float.equals(float)나 Double.equals(double) 은 오토박싱을 수반할 수 있어 성능상 좋지 않음
* **배열 필드**는 원소 각각을 앞서의 지침대로 비교
  * 배열의 모든 원소가 핵심 필드라면 Arrays.equals 메서드들 중 하나를 사용

### 2. null을 정상 값으로 취급하지 않도록 방지하자

* null을 정상 값으로 취급하는 참조 타입 필드가 있음
* `Object.equals(object, object)` 로 비교하여 NullPointException 발생을 예방할 것

### 3. 필드의 표준형을 저장하자

* 비교하기 복잡한 필드는 필드의 표준형(canonical form)을 저장한후 비교하자
  * ex) 앞의 CaseInsensitiveString와 String
* 불변 클래스에 제격이며, 가변 객체인 경우 값이 바뀔 때 마다 표준형을 최신 상태로 갱신해줘야 함

### 4. 필드 비교 순서가 equals 성능을 좌우한다

* 다를 가능성이 더 크거나 비교하는 비용이 싼 필드를 먼저 비교하자
* 동기화용 락(lock) 필드와 같은 객체의 논리적 상태와 관련 없는 필드는 비교하지 말자
* 핵심필드 / 파생필드 구분하여 더 빠른 필드를 비교하자

### 전형적인 equals 메서드의 예

```java
public final class phoneNumber {
    private final short areaCode, prefix, lineNum;

    @Override
    public boolean equals(Object o) {
        if(o == this) {
            return true;
        }
        
        if(!(o instanceof PhoneNumber)) {
            return false;
        }

        PhoneNumber pn = (PhoneNumber)o;
        return pn.lineNum == lineNum && pn.prefix == prefix
                        && pn.areaCode == areaCode;
    }
}
```

### **5. equals를 재정의할 땐 hashCode도 반드시 재정의하자**

### **6. 너무 복잡하게 해결하려 들지 말자**

### **7. Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자**

* 입력 타입은 반드시 Object여야 한다

## 핵심 정리

* 꼭 필요한 경우가 아니라면 equals를 재정의하지 말자
* 많은 경우에 Object의 equals가 원하는 비교를 정확히 수행한다
* 재정의해야 할 때는 그 클래스의 핵심 필드 모두를 빠짐없이, 다섯 가지 규약(반사성, 대칭성, 추이성, 일관성, null 아님)을 확실히 지켜가며 비교해야 한다
