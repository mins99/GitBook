---
description: 이 글은 '모던 자바 인 액션' (라울-게이브리얼 우르마 외 저, 우정은 번역)' 책 내용을 정리한 글입니다.
---

# Chapter02 동작 파라미터화 코드 전달하기

* 소비자의 요구사항은 항상 바뀌며, 이 변화하는 요구사항은 소프트웨어 엔지니어링에서 피할 수 없는 문제임
* 시시각각 변하는 사용자 요구사항에 엔지니어링적인 비용이 가장 최소화되고 새로 추가된 기능을 쉽게 구현할 수 있으며 장기적 관점에서 유지보수가 쉽도록 대응하는 방법이 필요
* **동작 파라미터화** - 어떻게 실행할지 결정하지 않은 코드 블록. 실행의 순서를 나중으로 미룬다

### 2.1 변화하는 요구사항에 대응하기 - 사과농장 예제

#### 2.1.1 첫 번째 시도 : 녹색 사과 필터링

```
public static List<Apple> filterGreenApples(List<Apple> inventory) {
	List<Apple> result = new ArrayList<>();
	for (Apple apple : inventory) {
		if (GREEN.equals(apple.getColor())) {	// 녹색 사과만 선택
			result.add(apple);
		}
	}
	return result;
}
```

* 다양한 색의 사과를 필터링 하게 되면 적절하게 대응하기 어렵다
* **거의 비슷한 코드가 반복 존재한다면 그 코드를 추상화 한다**

#### 2.1.2 두 번째 시도 : 색을 파라미터화

```
public static List<Apple> filterApplesByColor(List<Apple> inventory, Color color) {
	List<Apple> result = new ArrayList<>();
	for (Apple apple : inventory) {
		if (apple.getColor().equals(color)) {
			result.add(apple);
		}
	}
	return result;
}
```

* 농부가 무게로 필터링 하는 조건도 추가를 원하는 경우 filterApplesByWeight와 같이 개선 가능
  * 목록 검색, 사과에 필터링 조건 적용 부분이 대부분 중복됨
  * 소프트웨어 공학의 DRY(Don't repeat yourself, 같은 것을 반복하지 말 것) 원칙을 어기는 것

#### 2.1.3 세 번째 시도 : 가능한 모든 속성으로 필터링

```
public static List<Apple> filterApples(List<Apple> inventory, Color color, int weight, boolean flag) {
	List<Apple> result = new ArrayList<>();
	for (Apple apple : inventory) {
		if ((flag && apple.getColor().equals(color)) ||
			(!flag && apple.getWeight() > weight)) {
			result.add(apple);
		}
	}
	return result;
}

// 정말 마음에 들지 않는 코드다
List<Apple> greenApples = filterApples(inventory, GREEN, 0, true);
List<Apple> heavyApples = filterApples(inventory, null, 150, false);
```

* true, false의 의미를 알 수 없고, 요구사항에 대해 유연한 대응이 불가능함

### 2.2 동작 파라미터화

* **프레디케이트** : 참 또는 거짓을 반환하는 함수

![](<../../.gitbook/assets/image (1).png>)

* 선택 조건을 결정하는 인터페이스를 정의하고, 조건에 따라 filter 메서드가 다르게 동작할 수 있도록 함
  * 전략 디자인 패턴 - 알고리즘 전략을 캡슐화하는 알고리즘 패밀리를 정의해 둔 다음, 런타임에 알고리즘을 선택하는 기법
* 동작 파라미터화를 통해 메서드가 다양한 동작(전략)을 받아서 내부적으로 다양한 동작을 수행할 수 있다

#### 2.2.1 네 번째 시도 : 추상적 조건으로 필터링

```
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
	List<Apple> result = new ArrayList<>();
    for(Apple apple : inventory){
        if(p.test(apple)){			// 프레디케이트 객체로 사과 검사 조건 캡슐화
            result.add(apple);
        }
    }
    return result;
}
```

**코드/동작 전달하기**

* 필요한대로 다양한 ApplePredicate를 만들어서 filterApples 메서드로 전달할 수 있게 되었다
  * filterApples 메서드의 동작을 ApplePredicate 객체로서 파라미터화 함

**한 개의 파라미터, 다양한 동작**

* 동작 파라미터화의 강점 - 컬렉션 탐색 로직과 각 항목에 적용할 동작을 분리가능

### 2.3 복잡한 과정 간소화&#x20;

* filterApples 메서드의 단점 : ApplePredicate를 구현하는 여러 클래스를 정의한 다음 인스턴스화 해야 함. 번거롭고 시간 낭비이다
* 클래스의 선언과 인스턴스화를 동시에 수행할 수 있는 **익명 클래스**를 활용해보자

#### 2.3.2 다섯 번째 시도 : 익명 클래스 사용

```
List<Apple> redApples = filterApples(inventory, new ApplePredicate() {
    public boolean test(Apple a) {
        return RED.equals(a.getColor());
    }
});
```

* 선택 기준을 가리키는 boolean test(Apple a)를 구현해야 한다는 점은 변하지 않음

#### 2.3.3 여섯 번째 시도 : 람다 표현식 사용

```
List<Apple> result = filterApples(inventory, (Apple apple) -> RED.equals(apple.getColor()));
```

* 간결해지면서 문제를 잘 설명하는 코드가 되었다!

#### 2.3.4 일곱 번째 시도 : 리스트 형식으로 추상화

* filter 메서드를 정의 하여 다양한 필터링 조건에 대응 -> 유연성과 간결함 두 마리 토끼를 모두 잡음

```
public static <T> List<T> filter(List<T> list, Predicate<T> p) {
    List<T> result = new ArrayList<>();
    for(T e : list) {
        if(p.test(e)) {
            result.add(e);
        }
    }
    return result;
}

List<Apple> redApples = filter(inventory, (Apple apple) -> RED.equals(apple.getColor()));
List<Integer> evenNumbers = filter(numbers, (Integer i) -> i%2 == 0);
```

### 2.4 실전 예제

* 동작 파라미터화 패턴은 동작을 한 조각의 코드로 캡슐화한 다음 메서드로 전달해서 메서드의 동작을 파라미터화 함
* 자바 API의 많은 메서드를 다양한 동작으로 파라미터화할 수 있다 - Comparator, Runnable, GUI

#### 2.4.1 Comparator로 정렬하기

* 개발자에게는 변화하는 요구사항에 쉽게 대응할 수 있는 다양한 정렬 동작을 수행할 수 있는 코드가 절실
* 자바 8의 List에는 sort 메서드가 포함되어 있으며, java.util.Comparator 객체를 이용해서 sort의 동작을 파라미터화 할 수 있다

```
// Comparator
inventory.sort(new Comparator<Apple>() {  
	public int compare(Apple a1, Apple a2) {  
		return a1.getWeight().compareTo(a2.getWeight());  
	}  
});

// 람다
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
```

#### 2.4.2 Runnable로 코드 블록 실행하기

* 자바 스레드를 이용하면 병렬로 코드 블록 실행 가능
* 여러 스레드가 각자 다른 코드를 실행할 수 있다
* 자바에서는 Runnable 인터페이스를 이용해서 실행할 코드 블록을 지정할 수 있다

```
Thread t = new Thread(new Runnable() {
	public void run() {
		System.out.println("Hello World");
	}
});

// 람다 표현식 이용
Thread t = new Thread(() -> System.out.println("Hello World"));
```

#### 2.4.3 GUI 이벤트 처리하기

* ExecutorService 인터페이스는 태스크를 스레드 풀로 보내고 결과를 Future로 저장할 수 있다는 점이 스레드와 Runnable을 이용하는 방식과는 다르다
  * Callable 인터페이스를 이용해 결과를 반환하는 태스크를 만든다(Runnable의 업그레이드 버전)

```
ExecutorService executorService = Executors.newCachedThreadPool(); 
Future<String> threadName = executorService.submit(new Callable<String>() { 
	@Override  
	public  String  call()  throws  Exception { 
		return Thread.currentThread().getName(); 
	} 
});

// 람다 표현식 이용
Future<String> threadName = executorService.submit(() -> Thread.currentThread().getName());
```
