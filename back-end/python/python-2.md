---
description: 이 글은 '인프런 - 파이썬 무료 강의 (기본편) - 6시간 뒤면 나도 개발자' 강의 내용을 정리한 글입니다.
---

# Python 기본2

### 입출력

* print에서 출력 형식을 지정할 수 있다(sep, end...)
* 숫자형이나 boolean 인 경우에는 print 할 때 str(변수명) 형식으로 감싸주기
* ljust, rjust : 해당 크기만큼 공간을 만든 후 정렬
* zfill : 값이 없는 부분에 대해 0으로 채움
*   input으로 받은 값은 무조건 type이 str

    ```
    print("Python", "Java", sep=",", end="?")

    import sys
    print("Python", "Java", file=sys.stdout)
    print("Python", "Java", file=sys.stderr)

    scores = {"수학":0, "영어":50, "코딩":100}
    for subject, score in scores.items() :
        print(subject.ljust(8), str(score).rjust(4), sep=":")

    # 은행 대기순번표 - 001, 002, 003 ..
    for num in range(1,21) :
        print("대기번호 : " + str(num).zfill(3))
    ```
*   출력시 다양한 조건을 줄 수 있다

    ```
    # 10자리 길이로 출력하되 빈 자리는 공백으로 두고 오른쪽 정렬
    print("{0: >10}".format(500))

    # 양수는 +, 음수는 -
    print("{0: >+10}".format(-500))

    # 왼쪽 정렬하고, 빈 자리는 _로 채움
    print("{0:_<10}".format(500))

    # 3자리마다 , 찍기
    print("{0:,}".format(1000000000000))

    # 3자리마다 , 찍고 빈 자리는 ^로 채움
    print("{0:^<30,}".format(1000000000000))

    # 소수점 출력
    print("{0:f}".format(5/3))

    # 소수점 특정 자리수 까지만 표시
    print("{0:.2f}".format(5/3))
    ```
*   파일 입출력 : open(파일명, type, encoding)

    ```
    score_file = open("score.txt", "w", encoding="utf8")  # w : 쓰기, 덮어쓰기
    print("수학 : 0", file=score_file)
    print("영어 : 100", file=score_file)
    score_file.close()

    score_file = open("score.txt", "a", encoding="utf8")    # a : 추가로 쓰기(append)
    score_file.write("과학 : 80")
    score_file.write("\\n코딩 : 100")
    score_file.close()

    score_file = open("score.txt", "r", encoding="utf8")    # r : 읽기
    print(score_file.read())
    score_file.close()

    score_file = open("score.txt", "r", encoding="utf8")
    print(score_file.readline(), end="")       # 한 줄씩 읽기
    while True :
        line = score_file.readline()
        if not line :
            break
        print(line, end="")
    score_file.close()

    score_file = open("score.txt", "r", encoding="utf8")
    lines = score_file.readlines()      # 한 줄씩 읽어서 리스트 형태로 저장
    for line in lines :
        print(line, end="")
    score_file.close()
    ```
*   pickle : 소스코드의 데이터를 파일 데이터로 변환

    ```
    import pickle

    profile_file = open("profile.pickle", "wb")     # w은 write, b는 binary를 의미
    profile = {"이름":"시우민", "나이":30, "취미":["축구", "골프", "코딩"]}
    print(profile)
    pickle.dump(profile, profile_file) # profile의 정보를 profile_file에 저장

    profile_file = open("profile.pickle", "rb")     # r은 read
    profile2 = pickle.load(profile_file)
    print(profile2)
    profile_file.close()
    ```
*   with : 파일 입출력을 좀 더 간단하게 할 수 있도록 함. with를 벗어나면 close가 자동으로 된다는 장점이 있다

    ```
    import pickle
    with open("profile.pickle", "rb") as profile_file :
        print(pickle.load(profile_file))

    with open("study.txt", "w", encoding="utf8") as study_file:
        study_file.write("파이썬을 열심히 공부하고 있어요")

    with open("study.txt", "r", encoding="utf8") as study_file2:
        print(study_file2.read())
    ```

### 클래스

* 클래스 : 자료형과 메서드를 모은 집합. 일종의 템플릿
*   생성자 : def **init** (self, ... ) 로 선언. 매개변수 갯수만큼 선언 필수이며 인스턴스 생성시 self를 제외한 부분만 선언하면 됨

    ```
    class Unit:
        def __init__(self, name, hp, damage):
            self.name = name
            self.hp = hp
            self.damage = damage
            print("{0} 유닛이 생성되었습니다.".format(self.name))
            print("체력 {0}, 공격력 {1}".format(self.hp, self.damage))

    marine1 = Unit("마린", 40, 5)          # 인스턴스 생성
    marine2 = Unit("마린", 40, 5)
    tank = Unit("탱크", 150, 35)
    ```
*   멤버변수 : 생성된 인스턴스에 대해 각각의 변수에 접근 가능.

    * 외부에서 해당 인스턴스에 대해 추가로 변수를 만들어 값을 넣을 수 있으며, 이 경우에는 해당 인스턴스에서만 사용이 가능함

    ```
    wraith1 = Unit("레이스", 80, 5)
    print("유닛 이름 : {0}, 공격력 : {1}".format(wraith1.name, wraith1.damage))

    wraith2 = Unit("레이스", 80, 5)
    wraith2.clocking = True            # wraith1에는 clocking이 없다

    if wraith2.clocking == True:
        print("{0}는 현재 클로킹 상태입니다.".format(wraith2.name))
    ```
*   상속 : A라는 클래스의 생성자, 메소드가 B 클래스의 생성자, 메소드에 포함되는 범위라면 상속을 통해 구현 가능

    * class 클래스명(상속받을 클래스명)

    ```
    class Unit:
        def __init__(self, name, hp):
            self.name = name
            self.hp = hp 

    class AttackUnit(Unit):        # Unit 클래스를 상속받음
        def __init__(self, name, hp, damage):
            Unit.__init__(self, name, hp)
            self.damage = damage

    firebat1 = AttackUnit("파이어뱃", 50, 16)
    firebat1.attack("5시")

    firebat1.damaged(25)
    firebat1.damaged(25)
    ```
*   다중상속 : 상속 받는 부모 클래스가 여러개일 때 사용

    ```
    class Flyable:
        def __init__(self, flying_speed):
            self.flying_speed = flying_speed

        def fly(self, name, location):
            print("{0} : {1} 방향으로 날아갑니다. [속도 {2}]".format(name, location, self.flying_speed))

    class FlyableAttackUnit(AttackUnit, Flyable):
        def __init__(self, name, hp, damage, flying_speed):
            AttackUnit.__init__(self, name, hp, damage)
            Flyable.__init__(self, flying_speed)

    valkyrie = FlyableAttackUnit("발키리", 200, 6, 5)
    valkyrie.fly(valkyrie.name, "3시")
    ```
*   메소드 오버라이딩 : 자식 클래스에서 선언한 메소드를 부모 클래스에서 사용하고 싶을 때 사용

    * 지상유닛, 공중유닛의 경우 움직일때 각각 move, fly를 해야함 → 이름은 다르지만 로직은 동일

    ```
    class Unit:
        def __init__(self, name, hp, speed):
            self.name = name
            self.hp = hp 
            self.speed = speed
        
        def move(self, location):
            print("[지상 유닛 이동]")
            print("{0} : {1} 방향으로 이동합니다. [속도 {2}".format(self.name, location, self.speed))

    class FlyableAttackUnit(AttackUnit, Flyable):
        def __init__(self, name, hp, damage, flying_speed):
            AttackUnit.__init__(self, name, hp, 0, damage)        # 지상 speed 0
            Flyable.__init__(self, flying_speed)
        
        def move(self, location):
            print("[공중 유닛 이동]")
            self.fly(self.name, location)

    vulture = AttackUnit("벌쳐", 80, 10, 20)
    battlecruiser = FlyableAttackUnit("배틀크루저", 500, 25, 3)

    vulture.move("11시")
    #battlecruiser.fly(battlecruiser.name, "9시")
    battlecruiser.move("9시")
    ```
*   pass : 메소드 내에서 사용시 아무런 선언이 없어도 그냥 넘어감

    ```
    # 건물
    class BuildingUnit(Unit):
        def __init__(self, name, hp, location):
            pass

    supply_depot = BuildingUnit("서플라이 디폿", 500, "7시")

    def game_start():
        print("[알림] 새로운 게임을 시작합니다.")

    def game_over():
        pass
    ```
*   super : 상속 받은 클래스에서 선언시 super를 통해 선언 가능

    * super 사용시에는 super().**init**(변수1, 변수2 ...) 형식으로 self를 제외한다
    * 다중 상속시 하나의 부모에 대해서만 init이 적용됨

    ```
    # 건물
    class BuildingUnit(Unit):
        def __init__(self, name, hp, location):
            #Unit.__init__(self, name, hp, 0)
            super().__init__(name, hp, 0)
            self.location = location
    ```
* isinstance : 어떤 주어진 인스턴스가 특정 클래스/데이터 타입인지 체크

### 예외처리

* 에러상황에서의 처리(계산시 0으로 나눈다거나 문자로 나누는 등)
*   try: \~ except:

    ```
    try:
        print("나누기 전용 계산기")
        nums = []
        nums.append(int(input("첫 번째 숫자를 입력하세요 : ")))
        nums.append(int(input("두 번째 숫자를 입력하세요 : ")))
        # nums.append(int(nums[0] / nums[1]))
        print("{0} / {1} = {2}".format(nums[0], nums[1], nums[2]))
    except ValueError:
        print("에러! 잘못된 값을 입력하였습니다.")
    except ZeroDivisionError as err:
        print(err)
    except Exception as err:
        print("알 수 없는 에러가 발생하였습니다.")
        print(err)
    ```
*   raise : 실제 오류는 아니지만 의도적으로 오류 발생

    ```
    try:
        print("한 자리 나누기 전용 계산기")
        num1 = int(input("첫 번째 숫자를 입력하세요 : "))
        num2 = int(input("두 번째 숫자를 입력하세요 : "))
        if num1 >= 10 or num2 >= 10:
            raise ValueError
        print("{0} / {1} = {2}".format(num1, num2, int(num1/num2)))
    except ValueError:
        print("에러! 잘못된 값을 입력하였습니다. 한 자리 숫자만 입력하세요.")
    ```
*   사용자 정의 예외처리 : 직접 클래스를 만들어 예외를 만들 수 있다. 선언시 Exception 상속받아 사용

    ```
    class BigNumberError(Exception):
        def __init__(self, msg):
            self.msg = msg

        def __str__(self):
            return self.msg

    try:
        print("한 자리 나누기 전용 계산기")
        num1 = int(input("첫 번째 숫자를 입력하세요 : "))
        num2 = int(input("두 번째 숫자를 입력하세요 : "))
        if num1 >= 10 or num2 >= 10:
            raise BigNumberError("입력값 : {0}, {1}".format(num1, num2))
    except BigNumberError as err:
        print("에러가 발생하였습니다. 한 자리 숫자만 입력하세요.")
        print(err)
    ```
*   finally : try문이 정상 종료 되거나 exception 발생과 상관없이 무조건 실행하는 부분

    ```
    finally:
        print("계산기를 이용해주셔서 감사합니다.")
    ```

### 모듈과 패키지

* import \[파일명] as \[별명] : 다른 파일의 함수를 사용. 별명.함수명 형식
*   from \[파일명] import \* : 별칭 없이 해당 파일의 함수명만 사용하여 쓸 수 있음

    * import 이후의 함수를 제한두는 방식도 가능하고, 함수명에 별칭을 주는 방식도 가능하다

    ```
    # import theater_module
    # theater_module.price(3)
    # theater_module.price_morning(4)
    # theater_module.price_soldier(5)

    # import theater_module as mv
    # mv.price(3)
    # mv.price_morning(4)
    # mv.price_soldier(5)

    # from theater_module import price, price_morning
    # price(3)
    # price_morning(4)
    # price_soldier(5)

    from theater_module import price_soldier as price
    price(5)    # price_soldier가 실행됨
    ```
* **all** : **init**.py를 생성후 **all** = \["패키지명"] 추가시 해당 패키지의 사용 가능 범위가 전체로 변경되어 from 패키지 import \* 형식으로 사용 가능해진다
* /Library/Frameworks/Python.framework/Versions/3.9/lib/python3.9/random.py 과 같이 lib 아래에 패키지들의 집합이 있음

### 내장 함수

* input : 사용자 입력을 받는 함수
* dir : 어떤 객체를 넘겨줬을 때 그 객체가 어떤 변수와 함수를 가지고 있는지 표시
* [built-in Functions](https://docs.python.org/ko/3/library/functions.html) 사이트 에서 내장 함수 목록을 볼 수 있다

```
print(dir())
import random
print(dir())
lst = [1,2,3]
print(dir(lst))

결과
['__annotations__', '__builtins__', '__cached__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__spec__']
['__annotations__', '__builtins__', '__cached__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__spec__', 'random']
['__add__', '__class__', '__class_getitem__', '__contains__', '__delattr__', '__delitem__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__gt__', '__hash__', '__iadd__', '__imul__', '__init__', '__init_subclass__', '__iter__', '__le__', '__len__', '__lt__', '__mul__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__reversed__', '__rmul__', '__setattr__', '__setitem__', '__sizeof__', '__str__', '__subclasshook__', 'append', 'clear', 'copy', 'count', 'extend', 'index', 'insert', 'pop', 'remove', 'reverse', 'sort']
```

### 외장 함수

* [Python Module Index](https://docs.python.org/ko/3/py-modindex.html) 사이트에서 외장 함수 목록을 조회 가능
* glob : 경로 내의 폴더 / 파일 목록 조회 (윈도우 dir)
* os : 운영체제에서 제공하는 기본 기능
* time, datetime : 시간 관련 함수
* timedelta : 두 날짜 사이의 간격

```
import glob
print(glob.glob("*.py")) # 확장자가 py인 모든 파일

import os
print(os.getcwd())  # 현재 디렉토리

folder = "sample_dir"

if os.path.exists(folder):
    print("이미 존재하는 폴더입니다.")
    os.rmdir(folder)
    print(folder, "폴더를 삭제하였습니다.")
else:
    os.makedirs(folder)     # 폴더 생성
    print(folder, "폴더를 생성하였습니다.")

print(os.listdir())

import time 
print(time.localtime())
print(time.strftime("%Y-%m-%d %H:%M:%S"))

import datetime
print("오늘 날짜는", datetime.date.today())

today = datetime.date.today()       # 오늘 날짜 저장
td = datetime.timedelta(days=100)
print("우리가 만난지 100일은", today + td)
```
