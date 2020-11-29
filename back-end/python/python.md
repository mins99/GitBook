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

### 자료구조

* 리스트

  ```text
  subway = ["시우민", "백현", "카이"]

  # index : 특정 값 인덱스 찾기
  print(subway.index("시우민"))

  # append : 리스트 맨 끝에 값 추가
  subway.append("디오")
  print(subway)

  # insert : 리스트 중간에 값 추가
  subway.insert(1, "수호")
  print(subway)

  # pop : 리스트 맨 끝에 있는 값 삭제
  subway.pop()
  print(subway)

  # count : 리스트 특정 값의 갯수
  subway.append("시우민")
  print(subway.count("시우민"))

  num_list = [5,2,3,4,1]

  # sort : 리스트를 정렬
  num_list.sort()
  print(num_list)

  # reverse : 리스트를 역순으로 정렬
  num_list.reverse()
  print(num_list)

  # clear : 리스트를 초기화
  num_list.clear()
  print(num_list)

  # 다양한 자료형 함께 사용 가능
  mix_list = ["시우민", 20, True]
  print(mix_list)

  # extend : 리스트를 합침
  subway.extend(mix_list)
  print(subway)
  ```

* 사전

  ```text
  cabinet = {99 : "시우민", 4 : "백현"}
  print(cabinet)
  print(cabinet[99])      # 값이 없는 경우 오류 발생
  print(cabinet.get(99))  # 값이 없는 경우 None 으로 출력
  print(cabinet.get(3, "사용가능"))   # 값이 없는 경우 출력할 기본값을 설정 가능

  print(3 in cabinet)     # key 존재 유무를 확인 가능

  cabinet2 = {"EXO-M" : "시우민", "EXO-K" : "백현"}   # String 타입 key도 가능
  print(cabinet2)

  # key가 없는 경우 값 추가, 있는 경우 update
  cabinet2["EXO-CBX"] = "시우민"
  print(cabinet2)

  # del : key에 해당하는 값 삭제
  del cabinet2["EXO-CBX"]
  print(cabinet2)

  # keys : key만 출력, values : value만 출력, items : key-value 쌍으로 출력
  print(cabinet2.keys())
  print(cabinet2.values())
  print(cabinet2.items())

  # clear : 모든 값 삭제
  cabinet2.clear()
  print(cabinet2)
  ```

* 튜플
  * 변경되지 않는 목록. 리스트보다 속도가 빠름
  * 변수명 = \(변수1, 변수2, 변수3, ...\)
  * add로 추가 불가
* 세트

  * 중복이 안되며, 순서가 없음

  ```text
  exo = {"시우민", "수호", "카이", "디오"}
  exocbx = set(["시우민", "백현"])

  # 교집합
  print(exo & exocbx)
  print(exo.intersection(exocbx))

  # 합집합
  print(exo | exocbx)
  print(exo.union(exocbx))

  # 차집합
  print(exo - exocbx)
  print(exo.difference(exocbx))

  # 값 추가
  exo.add("세훈")
  print(exo)

  # 값 삭제
  exo.remove("세훈")
  print(exo)
  ```

* 자료구조의 변경

  ```text
  menu = {"커피", "우유", "쥬스"}
  print(menu, type(menu))     # menu의 type은 set

  menu = list(menu)       # set -> list
  print(menu, type(menu))

  menu = tuple(menu)       # list -> tuple
  print(menu, type(menu))

  menu = set(menu)       # tuple -> set
  print(menu, type(menu))
  ```

### 제어문

* if ~ elif ~ else

  ```text
  weather = input("오늘 날씨는 어때요? ")

  if weather == "비" or weather == "눈":
      print("우산을 챙기세요")
  elif weather == "미세먼지" : 
      print("마스크를 챙기세요")
  else : 
      print("그래도 마스크를 챙기세요")

  temp = int(input("기온은 어때요? "))

  if temp >= 30 :
      print("너무 더워요. 반팔 고고")
  elif temp >= 10 and temp < 30 :
      print("적당해요")
  elif 0 <= temp < 10 :
      print("겉옷을 챙겨요")
  else : 
      print("나가지 마세요")
  ```

* for 변수명 in 리스트

  ```text
  for waiting_no in range(1,6) :
      print("대기번호 : {0}".format(waiting_no))

  starbucks = ["시우민", "백현", "카이"]
  for customer in starbucks :
      print("{0}, 커피가 준비되었습니다.".format(customer))
  ```

* while 조건

  ```text
  customer = "토르"
  index = 5
  while index >= 1 :
      print("{0}, 커피가 준비 되었습니다. {1}번 남았어요.".format(customer, index))
      index -= 1
      if index == 0 :
          print("커피가 끝났습니다")

  # customer = "아이언맨"
  # while True :
  #     print("{0}, 커피가 준비 되었습니다.".format(customer))      # True 인 동안 무한루프

  customer = "토르"
  person = "Unknown"
  while person != customer : 
      print("{0}, 커피가 준비 되었습니다.".format(customer))
      person = input("이름이 어떻게 되세요? ")
  ```

* continue, break

  ```text
  absent = [2, 5]
  no_book = [7]
  for student in range(1, 10) :
      if student in absent :
          continue
      elif student in no_book :
          print("오늘 수업 여기까지. {0}은 교무실로".format(student))
          break;
      print("{0}, 책을 읽어봐".format(student))
  ```

* 한 줄 for문

  ```text
  students = [1,2,3,4,5]
  print(students)
  students = [i+100 for i in students]
  print(students)

  students = ["시우민", "백현", "카이", "디오"]
  print(students)
  students = [len(i) for i in students]
  print(students)

  students = ["Xiumin", "BackHyun", "Kai", "do"]
  print(students)
  students = [i.upper() for i in students]
  print(students)
  ```

