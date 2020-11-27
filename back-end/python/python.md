---
description: 이 글은 '인프런 - 파이썬 무료 강의 (기본편) - 6시간 뒤면 나도 개발자' 강의 내용을 정리한 글입니다.
---

# Python 기본

### 변수

* 변수 선언시에는 변수명 = "값" 형식으로
* 숫자형이나 boolean 인 경우에는 print 할 때 str\(변수명\) 형식으로 감싸주기
  * * 가 아닌 , 를 쓸때는 안써도 됨

```text
animal = "고양이"
name = "터치"
age = 7
hobby = "고기"
isAdult = True

print("우리집 " + animal + " 이름은 " + name + "에요")
hobby = "낮잠"
print(name, "는 ", age, "개월이고, ", hobby, "를 좋아해요")
print("터치는 가출청소냥일까요? " + str(isAdult))

sentence = """
여러줄 입력할땐 \\n이 아니라 큰따옴표 세개
"""
```

### 주석

* \#를 쓰거나
* ''' 로 감싸거나 → 들여쓰기 수준이 같아야 한다
* ctrl + / 를 쓰거나

### 연산자

* 곱하기는 \*\* 나누기는 //
* abs, pow, max, min, round 등은 따로 선언 없이 사용 가능
* from math import \* 선언 → floor, ceil, sqrt 등
* 랜덤함수 random
  * from random import \* 로 선언
  * random\(\) : 0.0 ~ 1.0 미만의 값 생성
  * int\(random\(\)\) 시 정수로 임의의 값 생성
* randrange\(1, 46\) → 1~46 미만의 임의의 값 생성
* randint\(1, 45\) → 1~45 이하의 임의의 값 생성

### 문자열

* jumin\[n:m\] : n번째 값부터 m직전까지
* jumin\[:n\] : 처음부터 n직전까지
* jumin\[-n:\] : 끝을 기준으로 n부터 끝까지
* lower\(\) : 소문자화
* upper\(\) : 대문자화
* isupper\(\) : 문자 하나 대문자화
* len\(변수명\) : 문자열 길이
* 변수명.count\(x\) : 문자열에 특정 문자열이 몇개 있는지 반환
* replace\(old, new\) : old를 new로 변경
* index, find : 문자\(열\)을 찾아 위치값을 반환. index는 값이 없을때 오류가 나고 find는 -1를 반환

```text
# 방법1
# %s 를 사용하는 경우 모든 변수에 대입 가능
print("나는 %d살입니다." % 20)
print("나는 %s를 좋아해요." % "파이썬")
print("나는 %c로 시작해요." % "A")
print("나는 %s색과 %s색을 좋아해요." % ("파랑", "빨강"))

# 방법2
print("나는 {}살입니다.".format(20))
print("나는 {}색과 {}색을 좋아해요.".format("파랑", "빨강"))
print("나는 {0}색과 {1}색을 좋아해요.".format("파랑", "빨강"))

# 방법3
print("나는 {age}살이며, {color}색을 좋아해요.".format(age = 20, color="파랑"))

# 방법4
age = 20
color = "파랑"
print(f"나는 {age}살이며, {color}색을 좋아해요.")
```

* 탈출문자
  * \n : 줄바꿈
  * \" : 쌍따옴표
  * \\ : 문장 내에서 하나의 \
  * \r : 커서를 맨 앞으로 이동
    * print\("Red Apple\rPine"\)
  * \b : 백스페이스
  * \t : 탭

