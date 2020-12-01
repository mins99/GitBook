---
description: 이 글은 '인프런 - 파이썬 무료 강의 (기본편) - 6시간 뒤면 나도 개발자' 강의 내용을 정리한 글입니다.
---

# Python 기본2

### 입출력

* print에서 출력 형식을 지정할 수 있다\(sep, end...\)
* 숫자형이나 boolean 인 경우에는 print 할 때 str\(변수명\) 형식으로 감싸주기
* ljust, rjust : 해당 크기만큼 공간을 만든 후 정렬
* zfill : 값이 없는 부분에 대해 0으로 채움
* input으로 받은 값은 무조건 type이 str

  ```text
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

* 출력시 다양한 조건을 줄 수 있다

  ```text
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

* 파일 입출력 : open\(파일명, type, encoding\)

  ```text
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

* pickle : 소스코드의 데이터를 파일 데이터로 변환

  ```text
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

* with : 파일 입출력을 좀 더 간단하게 할 수 있도록 함. with를 벗어나면 close가 자동으로 된다는 장점이 있다

  ```text
  import pickle
  with open("profile.pickle", "rb") as profile_file :
      print(pickle.load(profile_file))

  with open("study.txt", "w", encoding="utf8") as study_file:
      study_file.write("파이썬을 열심히 공부하고 있어요")

  with open("study.txt", "r", encoding="utf8") as study_file2:
      print(study_file2.read())
  ```

