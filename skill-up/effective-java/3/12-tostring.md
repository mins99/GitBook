# 아이템12 - toString을 항상 재정의하라

* 기본 Object에서 제공하는 toString : PhoneNumber@adbbd
* toString의 일반 규약에 따른 toString : phoneNumber : 123-456-7890 → 모든 하위 클래스에서 이 메서드를 재정의하라

### toString 재정의가 필요한 경우

* 콘솔로 객체를 확인할 때
  * `System.out.println()`
  * 객체에 문자열을 연결(`SomeObject + ""`)
  * assert 구문 사용 시
  * 등등...
* 디버깅 할 때
* 로깅할 때

### toString 재정의 하는 좋은 방법

* 객체가 가진 주요 정보 모두를 반환하도록 재정의한다
* 주석을 통해 toString이 반환하는 포맷을 명시하든 아니든 의도를 명확하게 밝힌다
  * 포맷 명시시 포맷에 얽매이게 된다는 단점에 주의
* 포맷 명시 여부와 상관없이 toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공한다

### toString 재정의가 필요 없는 경우

* 정적 유틸리티 클래스
* 대부분의 열거 타입 → 자바가 이미 제공
* 수퍼 클래스에서 이미 적절하게 재정의한 경우
  * ex) BigInteger, AbstractMap\<K,V>, AbstractCollection

### toString을 재정의할 때 순환 참조를 조심하자

* toString 사용시 혹은 Lombok에 있는 `@ToString` 사용시 `StackOverflowError`가 일어날 수 있음
* 아래 코드처럼 A에서는 b를 출력하려고 하고, B에서는 a를 출력하려고 해서 무한 호출로 `StackOverflowError`가 일어남

```java
class A {
    private B b;

    @Override
    public String toString() {
        return "A : [" + "b='" + b + "]";
    }
}

class B {
    private A a;

    @Override
    public String toString() {
        return "B : [" + "a='" + a + "]";
    }
}
```

* toString 사용시 : 서로가 서로를 참조하는 필드를 제외
* lombok 사용시 : lombok을 사용하지 않거나 `@ToString(exclude = "b")`를 통해 참조하는 필드를 제외

### 핵심 정리

* 모든 구체 클래스에서 Object의 toString을 재정의하자
* 상위 클래스에서 이미 알맞게 재정의한 경우는 예외
* toString을 재정의한 클래스는 사용하기도 즐겁고 그 클래스를 사용한 시스템을 디버깅하기 쉽게 해준다
* toString은 해당 객체에 관한 명확하고 유용한 정보를 읽기 좋은 형태로 반환해야 한다

참고 : https://madplay.github.io/post/methods-common-to-all-objects&#x20;

[Meet-Coder-Study](https://github.com/Meet-Coder-Study/book-effective-java/blob/main/3%EC%9E%A5/12\_toString%EC%9D%84\_%ED%95%AD%EC%83%81\_%EC%9E%AC%EC%A0%95%EC%9D%98%ED%95%98%EB%9D%BC\_%EC%9D%B4%ED%98%B8%EB%B9%88.md)
