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