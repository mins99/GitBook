# 아이템3 - private 생성자나 Enum 타입으로 싱글턴임을 보증하라

## 싱글턴(singleton)

* 인스턴스를 하나만 생성할 수 있는 클래스
  * 예) 함수(아이템 24)와 같은 무상태 객체, 설계상 유일해야 하는 시스템 컴포넌트
* 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워질 수 있다
  * 싱글턴 인스턴스를 mock 구현으로 대체할 수 없기 때문

## 싱글턴을 만드는 방식

### 1. public static final 필드 방식

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis () {...}
	
    public void leaveTheBuilding() {...}
}
```

* private 생성자는 `Elvis.INSTANCE`를 초기화할 때 딱 한 번만 호출
* public, protected 생성자가 없으므로 Elvis 클래스가 초기화될 때 만들어진 인스턴스가 전체 시스템에서 하나뿐임이 보장
* 다만, 리플렉션 API의 `AccessibleObject.setAccessible` 을 통해 private 생성자를 호출 가능

#### **장점**

1. 해당 클래스가 싱글턴임이 API에 명확히 드러난다.
2. 간결하다

### 2. 정적 팩터리 메서드 방식

```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() {}
  	
    public Elvis getInstance() {
      	return INSTANCE;
    }
}
```

* `Elvis.getInstance()`는 항상 같은 객체의 참조를 반환하므로 하나뿐임이 보장(리플렉션 예외는 동일하게 적용)

#### **장점**

1. API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다
   * 호출하는 스레드별로 다른 인스턴스를 넘겨주도록 수정 가능
2. 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다(아이템 30)
3. 정적 팩터리의 메서드 참조를 공급자(supplier)로 사용할 수 있다
   * 예) `Elvis::getInstance`를 `Supplier<Elvis>` 로 사용

### 3. 열거 타입 방식

```java
public enum Elvis {
    INSTANCE;
  
    public void leaveTheBuilding() {...}
}
```

* 싱글턴을 만드는 가장 좋은 방법

#### **장점**

1. 간결하고, 추가 구현 없이 직렬화할 수 있다
2. 복잡한 직렬화 상황이나 리플렉션 공격에서도 추가 인스턴스가 생기지 않는다

#### **단점**

1. 만들려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 사용할 수 없다
