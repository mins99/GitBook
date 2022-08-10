# 아이템11 - equals를 재정의하려거든 hashCode도 재정의하라

* equals를 재정의한 클래스 모두에서 hashCode도 재정의해야 함
* 그렇지 않으면 hashCode 일반 규약을 어기게 되어 해당 클래스의 인스턴스를 `HashMap`이나 `HashSet` 같은 컬렉션의 원소로 사용할 때 문제를 일으킨다

## Object 명세의 equals와 hashCode에 대한 규약

* equals 비교에 사용되는 정보가 변경되지 않았다면 애플리케이션이 실행되는 동안 그 객체의 `hashCode` 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다. 단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관없다.
* equals(Object)의 결과가 두 객체를 같다고 판단했다면, 두 객체의 `hashCode`는 똑같은 값을 반환해야 한다.
* equals(Object)의 결과가 두 객체를 다르다고 판단했더라도, 두 객체의 `hashCode`가 서로 다른 값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시 테이블의 성능이 좋아진다.

### 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다

* 아이템 10에서 equals는 물리적으로 다른 두 객체를 논리적으로는 같다고 할 수 있다고 했다
* 그런데 Object의 명세에 의하면 hashCode 메서드는 이 둘이 전혀 다르다고 판단하므로 **문제가 된다**

## 좋은 hashCode를 작성하는 요령

1. int 변수인 `result`를 선언한 후 값을 c로 초기화한다.
   * 이 때, c는 해당 객체의 첫번째 핵심 필드를 단계 2.1 방식으로 계산한 해시코드이다.
   * 여기서 핵심 필드는 `equals` 비교에 사용되는 필드를 말한다.
2. 해당 객체의 나머지 핵심 필드인 f 각각에 대해 다음 작업을 수행한다.
   1. 해당 필드의 해시코드 c 를 계산한다.
      * 기본 타입 필드라면, `Type.hashCode(f)`를 수행한다. 여기서 _Type_은 해당 기본타입의 박싱 클래스다.
      * 참조 타입 필드면서, 이 클래스의 `equals` 메소드가 이 필드의 `equals`를 재귀적으로 호출하여 비교한다면, 이 필드의 `hashCode`를 재귀적으로 호출한다.
      * 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다.\
        모든 원소가 핵심 원소라면 `Arrays.hashCode`를 사용한다.
   2. 단계 2.1에서 계산한 해시코드 c로 `result`를 갱신한다.
      * `result` = 31 \* `result` + c;
3. `result`를 반환한다.

### 전형적인 hashCode 메서드

```java
@Override
public int hashCode() {
	int result = Integer.hashCode(areaCode);
	result = 31 * result + Integer.hashCode(prefix);
	result = 31 * result + Integer.hashCode(lineNum);
	return result;
}
```

* `PhoneNumber` 인스턴스의 핵심 필드 3개만을 사용해 간단한 계산을 수행한다
* 비결정적 요소는 전혀 없으므로 동치인 `PhoneNumber` 인스턴스들은 **같은 해시코드**

### Object가 제공하는 hashCode 메서드 - 성능이 살짝 아쉽다

```java
@Override
public int hashCode() {
	return Objects.hash(lineNum, prefix, areaCode);
}
```

* 임의의 개수만큼 객체를 받아 해시코드를 계산해주는 정적 메서드 hash를 제공
* 전형적인 hashCode 메서드와 비슷한 수준의 메서드를 단 한 줄로 작성 가능
* 입력 인수를 담기 위한 배열이 만들어지고, 입력 중 기본 타입이 있다면 박싱과 언박싱도 거쳐야 해서 속도가 느리다

### 해시코드를 지연 초기화하는 hashCode 메서드 - 스레드 안전성까지 고려해야 한다

```java
private int hashCode;	// 초기값 0을 가진다.

@Override
public int hashCode() {
	int result = hashCode;
	if(result == 0) {
		int result = Integer.hashCode(areaCode);
		result = 31 * result + Integer.hashCode(areaCode);
		result = 31 * result + Integer.hashCode(areaCode);
		hashCode = result;
	}
	return result;
}
```

* 클래스가 불변이고 해시코드 계산 비용이 크다면 → 캐싱을 고려하자
  * 다만 해시의 키로 사용될 것 같다면 인스턴스가 만들어질 때 해시코드를 계산해야 함
  * 해시의 키로 사용되지 않는 경우라면 hashCode가 처음 불릴 때 계산하는 **지연 초기화 전략**이 어떨까?
* 필드를 지연 초기화하려면 클래스를 스레드 안전하게 만들도록 신경 써야 함(아이템 83)

## 주의할 점

* 성능을 높인답시고 해시코드를 계산할 때 핵심 필드를 생략해서는 안 됨
  * 속도는 빨라지겠지만 해시테이블의 성능이 떨어질 수 있다
* hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 공개해서는 안 됨
  * 개발자가 이 값에 의지하지 않게 하고 계산 방식을 바꾸지 않도록 해야 함

## 실무에서 권장하는 방법

* 실무에서는 `HashCodeBuilder`나 `@EqualsAndHashCode`를 사용하는 것을 추천

```java
// HashCodeBuilder
public int hashCode() {
    return HashCodeBuilder.reflectionHashCode(this);
}
```

* `@EqualsAndHashCode` 어노테이션은 클래스에 추가해주면 됨
* `equals` 메서드와 `hashCode` 메서드를 생성해주는데, static과 transient가 아닌 모든 필드들이 대상

<pre class="language-java"><code class="lang-java"><strong>@EqualsAndHashCode
</strong>public class Example {
    private transient int transientVal = 10;
    private String name;
    private int id;
}</code></pre>

* 다만 리플렉션을 사용하기 때문에 성능에 이슈가 있는지 고민을 해 볼 필요도 있음

## 핵심 정리

* equals를 재정의할 때는 hashCode도 반드시 재정의해야 한다. 그렇지 않으면 프로그램이 제대로 동작하지 않을 것이다.
* 재정의한 hashCode는 Object의 API 문서에 기술된 일반 규약을 따라야 하며, 서로 다른 인스턴스라면 되도록 해시코드도 서로 다르게 구현해야 한다.

참고 : [https://madplay.github.io/post/always-override-hashcode-when-you-override-equals](https://madplay.github.io/post/always-override-hashcode-when-you-override-equals)

[velog - lychee.log](https://velog.io/@lychee/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C-%EC%9E%90%EB%B0%94-%EC%95%84%EC%9D%B4%ED%85%9C-11.-equals%EB%A5%BC-%EC%9E%AC%EC%A0%95%EC%9D%98%ED%95%98%EB%A0%A4%EA%B1%B0%EB%93%A0-hashCode%EB%8F%84-%EC%9E%AC%EC%A0%95%EC%9D%98%ED%95%98%EB%9D%BC#%EC%A3%BC%EC%9D%98%ED%95%A0-%EC%A0%90)
