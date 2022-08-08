# 아이템4 - 인스턴스화를 막으려면 private 생성자를 사용하라

### 정적 메서드와 정적 필드만을 담은 클래스가 필요할 때가 있다

* `java.lang.Math`, `java.util.Arrays` 처럼 기본 타입 값이나 배열 관련 메서드를 모으거나
* `java.util.Collections` 처럼 특정 인터페이스를 구현하는 객체를 생성해주는 정적 메서드(혹은 팩터리)를 모아 놓을 수 있다
* `final` 클래스와 관련한 메서드들을 모아놓을 때도 마찬가지
  * final 클래스를 상속해서 하위 클래스에 넣는 것이 불가능하기 때문

### 인스턴스화를 어떻게 막을까?

### 1. 생성자 없이 만들기 - NO!

* 클래스 내부에는 정적 메서드만 있고 클래스의 이름도 유틸리티 기능을 강조하기 위해 **Utility** 라고 정의

```java
public class DateUtility {
    private static String FULL_DATE_FORMAT = "yyyy-MM-dd HH:mm:ss.SSS";

    // 생성자 없음

    public static String convertDateToString(Date date) {
        return new SimpleDateFormat(FULL_DATE_FORMAT).format(date);
    }
}
```

* 생성자를 명시하지 않았으나 기본 생성자는 자동으로 만들어진다
* 또한 유틸성 클래스로 이름을 지었지만 다른 누군가는 인스턴스화를 할 수 있음

```java
public void someMethod() {
    // 이렇게 사용하길 기대했으나!
    DateUtility.convertDateToString(new Date());
    
    // 누군가는 이렇게 사용할 수도
    DateUtility dateUtility = new DateUtility();
    String formattedToday = dateUtility.convertDateToString(new Date());
}
```

### 2. 추상 클래스로 정의하기 - NO!

```java
abstract class DateUtility {
    // ...생략
}

class SubDateUtility extends DateUtility {
    // ...생략
}

public class PrivateConstructorTest {
    public static void main(String[] args) {
        // abstract 클래스는 인스턴스화 불가
        // DateUtility dateUtility = new DateUtility();

        // Okay!
        SubDateUtility subDateUtility = new SubDateUtility();
    }
}
```

* 해당 방법도 인스턴스화를 막을 수 없다. 클래스를 상속해서 하위 클래스를 인스턴스화할 수 있기 때문

### 3. private 생성자를 추가하기 - OK!

```java
class DateUtility {
    private DateUtility() {
        //클래스 내부에서도 호출이 안되도록 막는다.
        throw new AssertionError();
    }

    // 생략
}

public class PrivateConstructorTest {
    public static void main(String[] args) {
        // DateUtility() has private access in DateUtility
        DateUtility dateUtility = new DateUtility();
    }
}
```

* 명시된 생성자가 없는 경우 기본 생성자를 만들기 때문에 `private` 으로 생성자를 생성하자
* 해당 예제에서는 클래스 내부에서 실수로라도 생성자를 호출하는 것을 막기 위해 생성자 내부에서 Assertion Error를 던지도록 처리
* 다만 생성자가 존재하지만 호출이 불가능하도록 처리했으니 주석을 달아주도록 하자
* 외부에 공개된 생성자가 없는 경우 상속도 불가능



참고 : [https://madplay.github.io/post/enforce-noninstantiability-with-private-constructor](https://madplay.github.io/post/enforce-noninstantiability-with-private-constructor)
