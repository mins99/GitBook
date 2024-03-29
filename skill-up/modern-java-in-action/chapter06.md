---
description: 이 글은 '모던 자바 인 액션' (라울-게이브리얼 우르마 외 저, 우정은 번역)' 책 내용을 정리한 글입니다.
---

# Chapter06 스트림으로 데이터 수집

###

## Chapter 6 스트림으로 데이터 수집

#### 이장의 내용

* Collectors 클래스로 컬렉션을 만들고 사용하기
* 하나의 값으로 데이터 스트림 리듀스하기
* 특별한 리듀싱 요약 연산
* 데이터 그룹화와 분할
* 자신만의 커스텀 컬렉터 개발

***

* 스트림의 연산은 중간 연산과 최종 연산으로 구성
  * 중간 연산 : 한 스트림을 다른 스트림으로 변환하는 연산. 여러 연산을 연결 가능. 스트림 파이프라인을 구성하며 스트림의 요소를 소비하지 않음
    * filter, map
  * 최종 연산 : 스트림의 요소를 소비해서 최종 결과를 도출(예 : 스트림의 가장 큰 값 반환)
    * count, findFirst, forEach, reduce
* [예제 - 통화별로 트랜잭션을 그룹화하기](https://github.com/MINS99/Modern\_Java\_in\_Action/blob/master/src/Modern\_Java\_in\_Action/chapter6/CollectorsPractice.java)

### 6.1 컬렉터란 무엇인가

* 함수형 프로그래밍에서는 **무엇**을 원하는지 직접 명시할 수 있어서 **어떤 방법**으로 이를 얻을지는 신경 쓸 필요가 없다
  * toList : 각 요소를 List로 만들어라
  * groupingBy : 각 key와 key에 대응하는 요소 List를 값으로 포함하는 Map으로 만들어라
* 명령형 코드에서는 다중 루프와 조건문을 사용해야함 -> 가독성과 유지보수성이 떨어짐

#### 6.1.1 고급 리듀싱 기능을 수행하는 컬렉터

* 함수형 API의 장점 : 높은 수준의 조합성과 재사용성
* collect로 결과를 수집하는 과정이 간단하면서도 유연한 방식으로 정의할 수 있다는 강점
* 스트림에 collect를 호출하면 스트림의 요소에 컬렉터로 파라미터화된 리듀싱 연산이 수행 [![](https://camo.githubusercontent.com/41c403a1f7c50851d615e6c57ac5bedfa3bedbd6ad253b2e1bfc203c09499ce3/68747470733a2f2f696d67312e6461756d63646e2e6e65742f7468756d622f523132383078302f3f73636f64653d6d746973746f72793226666e616d653d6874747073253341253246253246626c6f672e6b616b616f63646e2e6e6574253246646e2532466246764d63692532466274726f6248594b48713425324664445472484f4c306f634a534d694646634545374f6b253246696d672e706e67)](https://camo.githubusercontent.com/41c403a1f7c50851d615e6c57ac5bedfa3bedbd6ad253b2e1bfc203c09499ce3/68747470733a2f2f696d67312e6461756d63646e2e6e65742f7468756d622f523132383078302f3f73636f64653d6d746973746f72793226666e616d653d6874747073253341253246253246626c6f672e6b616b616f63646e2e6e6574253246646e2532466246764d63692532466274726f6248594b48713425324664445472484f4c306f634a534d694646634545374f6b253246696d672e706e67)

#### 6.1.2 미리 정의된 컬렉터

* Collectors 클래스에서 제공하는 팩터리 메서드 - groupingBy 등
* Collectors에서 제공하는 메서드의 기능
  * 스트림 요소를 하나의 값으로 리듀스하고 요약
  * 요소 그룹화
  * 요소 분할(프레디케이트를 그룹화 함수로 사용)

### 6.2 리듀싱과 요약

* 컬렉터로 스트림의 항목을 컬렉션으로 재구성 할 수 있다
  * counting() : 메뉴에서 요리 수를 계산

```
long howManyDishes = menu.stream().collect(Collectors.counting());
long howManyDishes2 = menu.stream().count();
```

#### 6.2.1 스트림값에서 최댓값과 최솟값 검색

* Collectors.maxBy, Collectors.minBy

```
Comparator<Dish> dishCaloriesComparator = Comparator.comparingInt(Dish::getCalories);
Optional<Dish> mostCalorieDish = menu.stream().collect(maxBy(dishCaloriesComparator));
```

* Optional\<Dish> : menu가 비어있는 경우를 대비한 메서드(11장)
* 스트림의 최댓값과 최솟값을 계산하거나 합계나 평균을 반환하는 연산을 **요약 연산**이라 부른다

#### 6.2.2 요약 연산 - 예제

[![](https://camo.githubusercontent.com/51cc102c9c45d72ebc89b47929c7de7f77a3f4ffd3989018b66866274d5d4876/68747470733a2f2f696d67312e6461756d63646e2e6e65742f7468756d622f523132383078302f3f73636f64653d6d746973746f72793226666e616d653d6874747073253341253246253246626c6f672e6b616b616f63646e2e6e6574253246646e2532464352374e372532466274726f645352337568522532463443566d57646e5256564755394f61785175417a7430253246696d672e706e67)](https://camo.githubusercontent.com/51cc102c9c45d72ebc89b47929c7de7f77a3f4ffd3989018b66866274d5d4876/68747470733a2f2f696d67312e6461756d63646e2e6e65742f7468756d622f523132383078302f3f73636f64653d6d746973746f72793226666e616d653d6874747073253341253246253246626c6f672e6b616b616f63646e2e6e6574253246646e2532464352374e372532466274726f645352337568522532463443566d57646e5256564755394f61785175417a7430253246696d672e706e67)

* summingInt, summingLong, summingDouble, averagingInt, averagingLong, averagingDouble 등

#### 6.2.3 문자열 연결 - 예제

* joining : 내부적으로 StringBuilder를 이용해서 문자열을 하나로 만든다
  * 대상 클래스가 toString 메서드를 포함하고 있는 경우 map으로 이름을 추출 하는 과정 생략 가능
  * 요소 사이의 구분 문자열을 넣을 수 있도록 오버로드된 joining 팩터리 메서드도 존재

#### 6.2.4 범용 리듀싱 요약 연산

* collect 대신 reducing 팩토리 메서드로도 정의 가능

```
// 메뉴의 모든 칼로리 합계 계산
int totalCalories = menu.stream().collect(reducing(0, Dish::getCalories, (i, j) -> i + j));
```

* reducing 메서드의 인수
  * 첫 번째 : 리듀싱 연산의 시작값 또는 인수가 없는 경우 반환값
  * 두 번째 : 변환 함수
  * 세 번째 : 같은 종류의 두 항목을 하나의 값으로 더하는 BinaryOperator

```
// 메뉴에서 가장 칼로리가 높은 요리를 찾음
Optional<Dish> mostCaloriesDish = menu.stream().collect(reducing(d1, d2) -> d1.getCalories() > d2.getCalories() ? d1 : d2));
```

* 한 개의 인수를 갖는 reducing 컬렉터는 시작값이 없으므로 빈 스트림이 넘겨졌을 때 시작값이 설정되지 않음 -> Optional\<Dish> 사용

#### collect와 reduce

* collect 메서드는 도출하려는 결과를 누적하는 컨테이너를 바꾸도록 설계된 메서드
* reduce 메서드는 두 값을 하나로 도출하는 불변형 연산
* 가변 컨테이너 관련 작업이면서 병렬성 확보 -> collect로 리듀싱 연산을 구현하는 것이 바람직

#### 컬렉션 프레임워크 유연성 : 같은 연산도 다양한 방식으로 수행할 수 있다

* 람다표현식 대신 Integer 클래스의 sum 메서드 참조를 이용하기
* 스트림을 IntStream으로 매핑한 다음 sum 메서드를 호출하는 방법으로도 얻을 수 있다

### 6.3. 그룹화

* 팩토리메서드 `Collectors.groupingBy` 를 이용해서 쉽게 메뉴를 그룹화 할 수 있다

```
Map<Dish.Type, List<Dish>> dishesByType = menu.stream().collect(groupingBy(Dish::getType));
```

* type을 기준으로 스트림이 그룹화 -> 분류 함수
* 더 복잡한 분류 기준이 필요하면 메서드 참조 대신 **람다 표현식**으로 구현

```
Map<CaloricLevel, List<Dish>> dishesByCaloricLevel 
        = menu.stream().collect(groupingBy(dish -> {
        		if(dish.getCalories() <= 400) return CaloricLevel.DIET;
        		else if(dish.getCalories() <= 700) return CaloricLevel.NORMAL;
        		else return CaloricLevel.FAT;
        }));
```

#### 6.3.1 그룹화된 요소 조작

* 그룹화 된 요소를 조작하는 연산 사용가능
  * 예) 500 칼로리가 넘는 요리만 필터링 -> filter 사용
  * 그러나 조건에 만족하지 않는 그룹은 사라짐 -> filtering 사용(자바9)
* 그룹화 된 요소를 맵핑 함수를 이용해 요소 변환 가능
  * 예) 그룹의 각 요리를 관련 이름 목록으로 변환 -> mapping 사용

#### 6.3.2 다수준 그룹화

* 두 인수를 받는 팩토리 메서드 Collectors.groupingBy를 이용해서 항목을 다수준으로 그룹화 가능

#### 6.3.3 서브그룹으로 데이터 수집

* groupingBy의 두번째 인수의 컬렉터 형식에는 제한이 없음
  * 예) counting 컬렉터를 전달해서 요리의 수를 종류별로 계산
* 분류 함수 한 개의 인수를 갖는 `groupingBy(f)`는 사실 `groupingBy(f, toList())`의 축약형
  * groupingBy는 스트림의 첫 번째 요소를 찾은 이후에 그룹화 맵에 값을 추가하므로 Optional 래퍼를 사용할 필요 없음
* 컬렉터 결과를 다른 형식에 적용하기
  * Collectors.collectingAndThen
    * 적용할 컬렉터와 변환 함수를 인수로 받아 다른 컬렉터를 반환.
    * 리뉴싱 컬렉터는 Optional.empty()를 반환하지 않으므로 값을 반환함
* groupingBy와 함께 사용하는 다른 컬렉터 예제
  * mapping
  * toSet : 집합으로 스트림의 요소 누적(중복 값 저장X)
  * toCollection : 원하는 방식으로 결과 제어 가능(HashSet::new 추가)

### 6.4 분할

* 분할 : 분할 함수라 불리는 프레디케이트를 분류 함수로 사용하는 특수한 그룹화 기능
* key 형식은 Boolean으로 그룹화 맵은 최대 참 아니면 거짓의 값을 갖는 두 개의 그룹으로 분류됨
  * 예) 채식 요리와 채식이 아닌 요리

#### 6.4.1 분할의 장점

* 분할 함수가 반환하는 참, 거짓 두 가지 요소의 스트림 리스트를 모두 유지하고 있음
* 컬렉터를 두 번째 인수로 전달 가능
* 내부적으로 partitioningBy는 특수한 맵과 두 개의 필드로 구현되어 있음

#### 6.4.2 숫자를 소수와 비소수로 분할하기

* [예제 - 정수 n을 인수로 받아서 2에서 n까지의 자연수를 소수와 비소수로 나누는 프로그램](https://github.com/MINS99/Modern\_Java\_in\_Action/blob/master/src/Modern\_Java\_in\_Action/chapter6/Prime.java)
  1. 주어진 수가 소수인지 아닌지 판단하는 프레디케이트를 구현한 뒤(isPrime)
  2. isPrime 메서드를 프레디케이트로 이용하고 partitioningBy 컬렉터로 리듀스 해서 분류
* p.223\~224 표6-1 Collectors 클래스의 정적 팩토리 메서드 참고하기

### 6.5 Collector 인터페이스

* Collector 인터페이스는 리듀싱 연산을 어떻게 구현할지 제공하는 메서드 집합으로 구성

```
 public interface Collector<T, A, R> {
    Supplier<A> supplier();
    BiConsumer<A, T> accumulator();
    Function<A, R> finisher();
    BinaryOperator<A> combiner();
    Set<Characteristics> characteristics();
 }
```

* T : 수집될 스트림 항목의 제네릭 형식
* A : 누적자, 수집 과정에서 중간 결과를 누적하는 객체의 형식
* R : 수집 연산 결과 객체의 형식(대부분 컬렉션 형식)
* Stream\<T>의 모든 요소를 List\<T>로 수집하는 ToListCollector\<T>
  * `public class ToListCollector<T> implements Collector<T, List<T>, List<T>`

#### 1) supplier - 새로운 결과 컨테이너 만들기

* 빈 결과로 이루어진 Supplier를 반환. 수집 과정에서 빈 누적자 인스턴스를 만드는 파라미터가 없는 함수.

```
 public Supplier<List<T>> supplier() {
    return () -> new ArrayList<T>();
 }
```

* 생성자 참조를 전달하는 방법도 있다

```
 public Supplier<List<T>> supplier() {
    return ArrayList::new;
 }
```

#### 2) accumulator - 결과 컨테이너에 요소 추가하기

* 리듀싱 연산을 수행하는 함수를 반환

```
 public BiConsumer<List<T>, T> accumulator() {
    return (list, item) -> list.add(item);
 }
```

* 메서드 참조 사용

```
 public BiConsumer<List<T>, T> accumulator() {
    return List::add;
 }
```

#### 3) finisher - 최종 변환값을 결과 컨테이너로 적용

* 스트림 탐색을 끝내고 누적자 객체를 최종 결과로 반환하면서 누적 과정을 끝낼 때 호출할 함수를 반환
* 누적자 객체가 이미 최종 결과인 상황에는 항등 함수를 반환

```
 public Function<List<T>, List<T>> finisher() {
    return Function.identity();
 }
```

#### 4) combiner - 두 결과 컨테이너 병합

* 스트림의 서로 다른 서브파트를 병렬로 처리할 때 누적자가 결과를 어떻게 처리할지 정의
* toList의 combiner는 두번째 서브파트에서 수집한 항목 리스트를 첫번째 서브파트 결과 리스트의 뒤에 추가하는 방법으로 구현

```
 public BinaryOperator<List<T>> combiner() {
    return (list1, list2) -> { list1.addAll(list2); return list1; };
 }
```

#### 5) characteristics

* 컬렉터의 연산을 정의하는 Characteristics 형식의 불변 집합을 반환
* 스트림을 병렬로 리듀스할 것인지, 한다면 어떤 최적화를 선택해야 할지 힌트를 제공
  * UNORDERED
  * CONCURRENT
  * IDENTITY\_FINISH

### 6.7 마치며

* collect는 스트림의 요소를 요약 결과로 누적하는 다양한 방법(컬렉터라 불리는)을 인수로 갖는 최종 연산이다
* 스트림의 요소를 하나의 값으로 리듀스하고 요약하는 컬렉터뿐 아니라 최솟값, 최댓값, 평균값을 계산하는 컬렉터 등이 미리 정의되어 있다
* 미리 정의된 컬렉터인 groupingBy로 스트림의 요소를 그룹화하거나, partitioningBy로 스트림의 요소를 분할할 수 있다
* 컬렉터는 다수준의 그룹화, 분할, 리듀싱 연산에 적합하게 설계되어 있다
* Collector 인터페이스에 정의된 메서드를 구현해서 커스텀 컬렉터를 개발할 수 있다
