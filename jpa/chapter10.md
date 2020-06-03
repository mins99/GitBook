---
description: 이 글은 '자바 ORM 표준 JPA 프로그래밍' (김영한 저)' 책 내용을 정리한 글입니다.
---

# Chapter10



* 이 장에서 다루는 내용
  * 객체지향 쿼리 소개
  * JPQL
  * Criteria
  * QueryDSL
  * 네이티브 SQL
  * 객체지향 쿼리 심화
* JPA는 복잡한 검색 조건을 사용해서 엔티티 객체를 조회할 수 있는 다양한 쿼리 기술을 지원
* JPQL은 가장 중요한 객체지향 쿼리 언어. Criteria, QueryDSL은 JPQL을 편리하게 사용하도록 도와주는 기술

### 10.1 객체지향 쿼리 소개

* em.find\(\) 메소드를 사용하면 식별자로 엔티티 하나를 조회할 수 있고 조회한 엔티티에 객체 그래프 탐색을 사용하면 연관된 엔티티들을 찾을 수 있다
* 단순한 검색 방법\(em.find\(\), a.getB\(\)\)만으로 애플리케이션을 개발하기는 어렵다
* JPQL은 엔티티 객체를 대상으로 하는 객체지향 쿼리이다
  * 테이블이 아닌 객체를 대상으로 검색
  * SQL을 추상화해서 특정 데이터베이스 SQL에 의존하지 않음
* JPA는 JPQL을 분석한 다음 적절한 SQL을 만들어 데이터베이스를 조회한 후 엔티티 객체를 생성해서 반환한다
* JPA가 공식 지원하는 검색 기능
  * JPQL\(Java Persistence Query Language\)
  * Criteria 쿼리 : JPQL을 편하게 작성하도록 도와주는 API, 빌더 클래스 모음
  * 네이티브 SQL : JPA에서 JPQL 대신 직접 SQL을 사용할 수 있다
* 그 외 기능
  * QueryDSL : Criteria 처럼 JPQL을 편하게 작성하도록 도와주는 빌더 클래스 모음. 비표준 오픈소스 프레임워크
  * JDBC, Mybatis 등의 매퍼 프레임워크 : 필요하면 JDBC를 직접 사용할 수 있다

#### **10.1.1 JPQL 소개**

* JPQL은 엔티티 객체를 조회하는 객체지향 쿼리이다
* 문법은 SQL과 비슷하고 ANSI 표준 SQL이 제공하는 기능을 유사하게 지원
* SQL을 추상화해서 특정 데이터베이스에 의존하지 않음
* 엔티티 직접 조회, 묵시적 조인, 다형성 지원으로 SQL 보다 코드가 간결하다

  ```java
    // java 코드
    String jpql = "select m from Member as m where m.username = 'kim'";
    List<Member> resultList = em.createQuery(jpql, Member.class).getResultList();

    // 실행한 JPQL
    select m from Member as m where m.username = 'kim'

    // 실제 실행된 SQL
    select member.id as id, member.age as age, member.team_id as team, member.name as name
    from Member member where member.name='kim'
  ```

#### **10.1.2 Criteria 쿼리 소개**

* Criteria의 장점은 문자가 아닌 query.select\(m\).where\(...\) 처럼 프로그래밍 코드로 JPQL을 작성할 수 있다는 점
* 문자가 아닌 코드로 JPQL을 작성하므로 컴파일 시점에 오류를 발견할 수 있다
* IDE를 사용하면 코드 자동완성을 지원한다
* 동적 쿼리를 작성하기 편하다

  ```java
    // Criteria 사용 준비
    CriteriaBuilder cb = em.getCriteriaBuilder();
    CriteriaQuery<Member> query = cb.createQuery(Member.class);

    // 루트 클래스(조회를 시작할 클래스)
    Root<Member> m = query.from(Member.class);

    // 쿼리 생성
    CriteriaQuery<Member> cq = query.select(m).where(cb.equal(m.get("usermane"), "kim"));
    List<Member> resultList = em.createQuery(cq).getResultList();
  ```

* m.get\("username"\)을 보면 필드 명을 문자로 작성했다. 코드로 작성하고 싶으면 메타 모델을 사용하면 된다.

#### **10.1.4 네이티브 SQL 소개**

* SQL을 직접 사용할 수 있는 기능을 JPA가 제공
* 특정 데이터베이스에 의존하는 기능을 사용해야 할 때\(오라클 connect by 등\) 사용
* SQL은 지원하지만 JPQL이 지원하지 않는 기능 사용시 네이티브 SQL 사용
* 특정 데이터베이스에 의존하는 SQL을 작성해야 함. 데이터베이스 변경시 네이티브 SQL도 수정해야 함

  ```java
    // 네이티브 SQL
    String sql = "SELECT ID, AGE, TEAM_ID, NAME FROM MEMBER WHERE NAME = 'kim'";
    List<Member> resultList = em.createNativeQuery(sql, Member.class).getResultList();
  ```

#### **10.1.5 JDBC 직접 사용, 마이바티스 같은 SQL 매퍼 프레임워크 사용**

* JDBC 커넥션에 직접 접근하고 싶으면 JPA 구현체가 제공하는 방법을 사용해야 한다
* JPA EntityManager에서 하이버네이트 Session을 구한 후 doWork\(\) 호출

  ```java
    // 하이버네이트 JDBC 획득
    Session session = entityManager.unwrap(Session.class);
    session.doWork(new Work() {
        @Override
        public void execute(Connection connection) throws SQLExcetion {
            // work...
        }
    });
  ```

* JDBC나 마이바티스를 JPA와 함께 사용하면 영속성 컨텍스트를 적절한 시점에 강제로 플러시해야 함
* JPA 우회하는 SQL에 대해서는 JPA가 전혀 인식하지 못함. 영속성 컨텍스트와 데이터베이스를 불일치 상태로 만들어 데이터 무결성을 훼손할 수 있다
* SQL 실행 직전 영속성 컨텍스트를 수동으로 플러시해서 동기화 하면 된다
* 스프링 프레임워크를 사용하면 JPA와 마이바티스를 손쉽게 통합할 수 있다

### 10.2 JPQL

* 엔티티를 쿼리하는 다양한 방법들 중 어떤 방법을 사용하든 JPQL에서 모든 것이 시작됨
* JPQL의 특징
  * 객체지향 쿼리 언어이다. 엔티티 객체를 대상으로 쿼리한다
  * SQL을 추상화해서 특정 데이터베이스의 SQL에 의존하지 않는다
  * 결국 SQL로 변환된다
* 예제로 사용할 도메인 모델

  ![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FxUW6x%2FbtqEBmYnhE0%2FniMkzIIW17zrBxeGKd39aK%2Fimg.png)

#### **10.2.1 기본 문법과 쿼리 API**

* JPQL도 SELECT, UPDATE, DELETE 문을 사용할 수 있다. INSERT는 em.persist\(\)를 사용한다

**SELECT 문**

```java
  SELECT m FROM Member AS m where m.username = 'Hello'
```

* 대소문자 구분
  * 엔티티와 속성은 대소문자를 구분하며, SELECT, FROM, AS 같은 JPQL 키워드는 대소문자를 구분하지 않는다
* 엔티티 이름
  * JPQL에서 사용한 Member는 클래스명이 아니라 엔티티명이다. 엔티티명은 `@Entity(name = "XXX")`로 지정. 기본값인 클래스명을 엔티티명으로 사용하는 것을 추천.
* 별칭은 필수
  * JPQL은 별칭을 필수로 사용해야 한다. 별칭 없이 작성시 오류가 발생한다

**TypeQuery, Query**

* 작성한 JPQL을 실행하려면 쿼리 객체를 만들어야 함. 
* 반환할 타입을 명확하게 지정할 수 있으면 TypeQuery, 반환 타입을 명확하게 지정할 수 없으면 Query 객체를 사용
* Query 객체는 SELECT 절의 조회 대상이 예제처럼 둘 이상이면 Object\[\]를 반환하고 하나이면 Object를 반환한다.

  ```java
  TypedQuery<Member> query = em.createQuery("SELECT m FROM Member m", Member.class);
  Query<Member> query2 = em.createQuery("SELECT m.username, m.age FROM Member m");

  List<Member> resultList = query.getResultList();
  List resultList2 = query.getResultList();

  for(Member member :  resultList) {
    System.out.println("member = " + member);
  }
  for(Object o : resultList2) {
    Object[] result = (Object[]) o;    // 결과가 둘 이상이면 Object[] 반환
    System.out.println("username = " + result[0]);
    System.out.println("age = " + result[1]);
  }
  ```

**결과 조회**

* query.getResultList\(\) : 결과를 예제로 반환. 결과가 없으면 빈 컬렉션 반환
* query.getSingleResult\(\) : 결과가 정확히 하나일 때 사용. 결과가 없으면 NoResultException, 1개보다 많으면 NonUniqueResultException 예외 발생.

#### **10.2.2 파라미터 바인딩**

* JPQL은 위치 기준 파라미터, 이름 기준 파라미터 바인딩을 지원
* 파라미터 바인딩이 아닌 직접 문자를 더해 만들어 넣으면 SQL 인젝션 공격을 당할 수 있으며, 성능 이슈도 있으므로 선택이 아닌 필수로 사용해야함

**이름 기준 파라미터**

* 파라미터를 이름으로 구분하는 방법. 앞에 :를 사용한다

  ```java
  String usernameParam = "User1";

  TypedQuery<Member> query = em.createQuery("SELECT m FROM Member m where m.username = :username", Member.class);
  query.setParameter("username", usernameParam);

  List<Member> resultList = query.getResultList();
  ```

**위치 기준 파라미터**

* ? 다음에 위치 값을 주면 된다. 위치 값은 1부터 시작

```java
  String usernameParam = "User1";

  List<Member> members = em.createQuery("SELECT m FROM Member m where m.username = ?1", Member.class)
                            .setParameter(1, usernameParam)
                            .getResultList();
```

* 위치 기준 파라미터 방식보다는 이름 기준 파라미터 바인딩 방식을 사용하는 것이 더 명확

#### **10.2.3 프로젝션**

* SELECT 절에 조회할 대상을 지정하는 것을 프로젝션이라 함
* \[SELECT {프로젝션 대상} FROM\]으로 대상 선택
* 프로젝션 대상은 엔티티, 임베디드 타입, 스칼라\(숫자, 문자 등 기본 데이터\) 타입

**엔티티 프로젝션**

```java
  SELECT m FROM Member m    // 회원
  SELECT m.team FROM Member m   // 팀
```

* 엔티티를 프로젝션 대상으로 사용하여 원하는 객체를 바로 조회한다
* 컬럼을 하나하나 나열해서 조회해야 하는 SQL과는 차이가 있다
* 조회한 엔티티는 영속성 컨텍스트에서 관리된다

**임베디드 타입 프로젝션**

```java
  String query = "SELECT o.address FROM Order o";
  List<Address> addresses = em.createQuery(query, Address.class)
                                .getResultList();
```

* JPQL에서 임베디드 타입은 엔티티와 거의 비슷하게 사용된다
* 임베디드 타입은 조회의 시작점이 될 수 없다\(Address가 아닌 Order로 조회해야 함\)
* 임베디드 타입은 값 타입이므로 영속성 컨텍스트에서 관리되지 않는다

**스칼라 타입 프로젝션**

```java
  List<String> username = em.createQuery("SELECT DISTINCT username FROM Member m", String.class)
                            .getResultList();

  Double orderAmountAvg = em.createQuery("SELECT AVG(o.orderAmount) FROM Order o", Double.class)
                            .getSingleResult();
```

* 스칼라 타입 : 숫자, 문자, 날짜와 같은 기본 데이터 타입들
* 통계 쿼리도 주로 스칼라 타입으로 조회

**여러 값 조회**

```java
  List<Object[]> resultList = em.createQuery("SELECT o.member, o.product, o.orderAmount FROM Order o");
                                .getResultList();

  for(Object[] row : resultList) {
      Member member = (Member) row[0];  // 엔티티
      Product product = (Product) row[1];   // 엔티티
      int orderAmount = (Integer) row[2];   // 스칼라
  }
```

* 엔티티를 대상으로 하지 않고 꼭 필요한 데이터들만 선택해서 조회해야 할 때도 있다
* 프로젝션에 여러 값을 선택하면 Query를 사용해야 한다
* 제네릭의 Object\[\] 사용시 간결하게 개발 가능
* 조회한 엔티티는 영속성 컨텍스트에서 관리된다

**NEW 명령어**

```java
  TypedQuery<UserDTO> query = em.createQuery("SELECT new jpabook.jpql.UserDTO(m.username, m.age) FROM Member m", UserDAO.class);

  List<UserDTO> resultList = query.getResultList();
```

* 실제 애플리케이션 개발 시 Object\[\]를 직접 사용하지 않고 UserDTO 처럼 의미 있는 객체로 변환해서 사용
* SELECT 다음 new 명령어를 사용하여 클래스 생성자에 JPQL 조회 결과를 넘겨주고, TypeQuery를 사용
* new 명령어 사용시 1\) 패키지 명을 포함한 전체 클래스 명을 입력해야 하고 2\) 순서와 타입이 일치하는 생성자가 필요하다

#### **10.2.4 페이징 API**

```java
  TypedQuery<Member> query = em.createQuery("SELECT m FROM Member m ORDER BY m.username DESC", Member.class);
                                .setFirstResult(10);
                                .setMaxResults(20);
                                .getResultList();
```

* 페이징 처리용 SQL을 작성하는 일은 지루하고 반복적이며, 데이터베이스마다 페이징을 처리하는 SQL문법이 다르다는 문제가 있다
* JPA는 페이징을 두 API로 추상화 했다
  * setFirstResult\(int startPosition\) : 조회 시작 위치\(0부터 시작\)
  * setMaxResults\(int maxResult\) : 조회할 데이터 수
* JPQL이 데이터베이스 방언에 따라 다르게 처리를 하여 SQL을 변환한다
* 페이징 SQL을 더 최적화하고 싶다면 네이티브 SQL을 직접 사용해야 함

#### **10.2.5 집합과 정렬**

* 집합은 집합함수와 함께 통계 정보를 구할 때 사용

**집합 함수**

| 함수 | 설명 |
| :--- | :--- |
| COUNT | 결과 수를 구한다. 반환 타입: Long |
| MAX, MIN | 최대, 최소 값을 구한다. 문자, 숫자, 날짜 등에 사용한다. |
| AVG | 평균값을 구한다. 숫자타입만 사용할 수 있다. 반환 타입: Double |
| SUM | 합을 구한다. 숫자타입만 사용할 수 있다. 반환 타입: 정수합 Long, 소수합: Double,   Biginteger합: Biginteger, BigDecimal합: BigDecimal |

**집합 함수 사용 시 참고사항**

* NULL 값은 무시하므로 통계에 잡히지 않는다\(DISTINCT가 정의되어 있어도 무시\)
* 값이 없는데 SUM, AVG, MAX, MIN 함수를 사용하면 NULL 값이 된다. COUNT는 0이 된다
* DISTINCT를 집합 함수 안에 사용해서 중복된 값을 제거하고 나서 집합을 구할 수 있다

  
   `select COUNT(DISTINCT m.age) from Member m`

* DISTINCT를 COUNT에서 사용할 때 임베디드 타입은 지원하지 않는다

**GROUP BY, HAVING**

* **GROUP BY**는 통계 데이터를 구할 때 특정 그룹끼리 묶어준다
  * groupby\_절 ::= GROUP BY {단일값 경로 \| 별칭}+
* **HAVING**은 GROUP BY와 함께 사용하는데 GROUP BY로 그룹화한 통계 데이터를 기준으로 필터링한다
  * having\_절 ::= HAVING 조건식
* 이런 쿼리들을 리포팅 쿼리나 통계 쿼리라고 한다
* 전체 데이터를 기준으로 처리하므로 실시간으로 사용하기에는 부담이 많음

```sql
  select t.name, count(m.age), sum(m.age), avg(m.age), max(m.age), min(m.age)
  from Member m LEFT JOIN m.team t
  GROUP BY t.name
  HAVING AVG(m.age) >= 10
```

**정렬\(ORDER BY\)**

* 결과를 정렬할 때 사용
* orderby\_절 ::= ORDER BY {상태필드 경로 \| 결과 변수 \[ASC \| DESC\]}+
* 상태필드는 t.name 같이 객체의 상태를 나타내는 필드를 말하며, 결과 변수는 SELECT 절에 나타나는 값을 말한다

```sql
  select t.name, COUNT(m.age) as cnt
  from Member m LEFT JOIN m.team t
  GROUP BY t.name
  ORDER BY cnt
```

#### **10.2.6 JPQL 조인**

* JPQL의 조인은 SQL 조인과 기능은 같고 문법만 약간 다르다

**내부 조인**

* 내부 조인은 \(INNER\) JOIN을 사용한다
* JPQL 조인의 가장 큰 특징은 연관 필드를 사용한다는 것이다. 연관 필드는 다른 엔티티와 연관관계를 가지기 위해 사용하는 필드이다\(m.team\)

```sql
  SELECT m
  FROM Member m INNER JOIN m.team t
  where t.name = :teamName

  // FROM Member m JOIN Team t 는 잘못된 JPQL 조인이므로 오류 발생
```

**외부 조인**

* 외부 조인은 기능상 SQL의 외부 조인과 같다

```sql
  SELECT m
  FROM Member m LEFT [OUTER] JOIN m.team t
```

**컬렉션 조인**

* 일대다 관계나 다대다 관계처럼 컬렉션을 사용하는 곳에 조인하는 것을 컬렉션 조인이라 한다
* t.members m은 팀과 팀이 보유한 회원목록을 컬렉션 값 연관 필드로 사용한 것

```sql
  SELECT t, m FROM Team t LEFT JOIN t.members m
```

**세타 조인**

* WHERE 절을 사용해서 세타 조인을 할 수 있다
* 세타 조인은 내부 조인만 지원한다

```sql
  SELECT count(m) from Member m, Team t
  where m.username = t.name
```

**JOIN ON 절\(JPA 2.1\)**

* on 절을 사용하면 조인 대상을 필터링하고 조인할 수 있다

```sql
  SELECT m,t from Member m
  left join m.team t on t.name = 'A'
```

#### **10.2.7 페치 조인**

* 페치 조인\(fetch join\)은 JPQL에서 성능 최적화를 위해 제공하는 기능
* 연관된 엔티티나 컬렉션을 한 번에 같이 조회하는 기능인데 `join fetch` 명령어로 사용할 수 있다
* 페치 조인 ::= \[ LEFT \[OUTER\] \| INNER \] JOIN FETCH 조인경로

**엔티티 페치 조인**

* 회원 엔티티를 조회하면서 연관된 팀 엔티티도 함께 조회하는 JPQL

  ```sql
  select m
  from Member m join fetch m.team
  ```

* 일반적인 JPQL 조인과는 다르게 m.team 다음에 별칭이 없는데 페치 조인은 별칭을 사용할 수 없다\(하이버네이트는 별칭 허용\)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2Fct1RWd%2FbtqEBnXiUGL%2FLGbkYXOtku73xHiXHMkrq0%2Fimg.png)   
 ![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2Fmu9A1%2FbtqECh3kAaB%2FyJkqC4Tnc7okT7hAnmKGV1%2Fimg.png)   
 ![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FEdwar%2FbtqECMaIf5W%2FDo1jEBhyuES2hYqx7kMeZ1%2Fimg.png)   


* 회원과 팀을 지연 로딩으로 설정했어도 페치 조인을 사용하면 팀도 함께 조회하므로 지연 로딩이 일어나지 않고 연관된 팀 엔티티는 프록시가 아닌 실제 엔티티이다

**컬렉션 페치 조인**

* 일대다 관계인 컬렉션을 페치 조인

  ```sql
  select t
  from Team join fetch t.members
  where t.name = '팀A'
  ```

* 팀만 선택했지만 실행 시 팀과 연관된 회원도 함께 조회한 것을 확인
* 일대다 조인은 결과가 증가할 수 있지만 일대일, 다대일 조인은 결과가 증가하지 않음

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FbrIGMD%2FbtqEAL5E4IL%2Fp82yrqHJIsR4LyQ7CicmKK%2Fimg.png) ![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FotAQZ%2FbtqEBONRAG3%2FtyLXzzJfXK08gd4CYKU4dK%2Fimg.png) ![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FmRvmr%2FbtqECiHWvTN%2FuZDSzH7UEvUsvLpg0TKaD1%2Fimg.png)

**페치 조인과 DISTINCT**

* SQL의 DISTINCT는 중복된 결과를 제거하는 명령
* JPQL의 DISTINCT는 SQL의 DISTINCT를 추가하고 애플리케이션에서 한 번 더 중복을 제거
* 직전의 컬렉션 페치 조인의 쿼리에 DISTINCT를 추가해도 각 로우의 데이터가 다르므로 SQL에서는 효과가 없음. 애플리케이션에서는 팀 엔티티인 팀A를 중복으로 보고 하나만 조회하게 됨

  ```sql
  select DISTINCT t
  from Team t join fetch t.members
  where t.name = '팀A'
  ```

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FV8MuW%2FbtqEz0B8CDa%2FDAXKlDsOGu3dGbSL1gQw3K%2Fimg.png)

**페치 조인과 일반 조인의 차이**

* JPQL은 결과를 반환할 때 연관관계 까지 고려하지 않는다
* 지연 로딩 설정하고 일반 조인만 사용시 프록시나 아직 초기화하지 않은 컬렉션 래퍼를 반환한다
* 즉시 로딩으로 설정하면 회원 컬렉션을 즉시 로딩하기 위해 쿼리를 한 번 더 실행
* 페치 조인 사용하면 연관된 엔티티도 함께 조회한다

**페치 조인의 특징과 한계**

* 페치 조인의 특징
  * 페치 조인을 사용하면 SQL 한 번으로 연관된 엔티티들을 함께 조회할 수 있어서 SQL 호출 횟수를 줄여 성능을 최적화 할 수 있다
  * 엔티티에 직접 적용하는 로딩 전략은 애플리케이션 전체에 영향을 미치므로 글로벌 로딩 전략이라 부른다
  * 페치 조인은 글로벌 로딩 전략보다 우선한다
  * 글로벌 로딩 전략은 될 수 있으면 지연 로딩을 사용하고 최적화가 필요하면 페치 조인을 적용하는 것이 효과적
  * 페치 조인을 사용하면 연관된 엔티티를 쿼리 시점에 조회하므로 준영속 상태에서도 객체 그래프 탐색 가능
* 페치 조인의 한계
  * 페치 조인 대상에는 별칭을 줄 수 없다. 하이버네이트에서는 가능하지만 연관된 데이터 수가 달라져 데이터 무결성이 깨질 수 있다
  * 둘 이상의 컬렉션을 페치할 수 없다. 컬렉션 \* 컬렉션의 카테시안 곱이 만들어지므로 주의. 하이버네이트에서는 예외 발생\(PersistenceException\)
  * 컬렉션 페치 조인시 페이징 API\(setFirstResult, setMaxResults\)를 사용할 수 없다
    * 단일 값 연관 필드\(일대일, 다대일\)들은 페치 조인을 사용해도 페이징 API를 사용할 수 있다
    * 하이버네이트에서 컬렉션을 페치 조인 하고 페이징 API를 사용하면 메모리에서 페이징 처리를 한다. 성능 이슈와 메모리 초과 예외가 발생할 수 있어 위험하다
* 페치 조인은 SQL 한 번으로 연관된 여러 엔티티를 조회할 수 있어 성능 최적화에 상당히 유용하다
* 페치 조인은 객체 그래프를 유지할 때 사용하면 효과적이고, 여러 테이블을 조회해야 한다면 필요한 필드들만 조회해서 DTO로 반환하는 것이 더 효과적

#### **10.2.8 경로 표현식**

* 경로 표현식이라는 것은 .\(점\)을 찍어 객체 그래프를 탐색하는 것

**경로 표현식의 용어 정리**

* 상태 필드\(state field\) : 단순히 값을 저장하기 위한 필드\(필드 or 프로퍼티\)
* 연관 필드\(association field\) : 연관관계를 위한 필드, 임베디드 타입 포함\(필드 or 프로퍼티\)
  * 단일 값 연관 필드 : `@ManyToOne`, `@OneToOne`, 대상이 엔티티
  * 컬렉션 값 연관 필드 : `@OneToMany`, `@ManyToMany`, 대상이 컬렉션
* 상태 필드는 단순히 값을 저장하는 필드, 연관 필드는 객체 사이의 연관관계를 맺기 위해 사용하는 필드

```java
  // Member.java
  @Entity
  public class Member {

    @Id @GeneratedValue
    private Long id;

    @Column(name = "name")
    private String usernmae;    // 상태 필드
    private Integer age;    // 상태 필드

    @ManyToOne (..)
    private Team team;      // 연관 필드(단일 값 연관 필드)

    @OneToMany (..)
    private List<Order> orders;     // 연관 필드(컬렉션 값 연관 필드)
```

**경로 표현식과 특징**

* 상태 필드 경로 : 경로 탐색의 끝. 더는 탐색할 수 없다
* 단일 값 연관 경로 : 묵시적으로 내부 조인. 계속 탐색 가능
* 컬렉션 값 연관 경로 : 묵시적으로 내부 조인. 더는 탐색할 수 없다. 단 FROM 절에서 조인을 통해 별칭을 얻어 계속 탐색 가능
* 명시적 조인 : JOIN을 직접 적어주는 것
* 묵시적 조인 : 경로 표현식에 의해 묵시적으로 조인이 일어나는 것. 내부 조인\(INNER JOIN\)만 가능
* t.members 처럼 컬렉션까지는 경로 탐색이 가능하지만 그 이후의 컬렉션에서 경로 탐색을 하는 것은 허락하지 않음. `join t.members m` 같이 조인을 사용해서 새로운 별칭을 얻어야 경로 탐색이 가능
* 참고로 컬렉션에서 `size` 사용시 count 함수로 변환된다

**경로 탐색을 사용한 묵시적 조인 시 주의사항**

* 경로 탐색을 사용하면 묵시적 조인이 발생해서 SQL에서 내부 조인이 일어날 수 있다.
* 컬렉션은 경로 탐색의 끝이다. 경로 탐색을 하려면 명시적으로 조인해서 별칭을 얻어야 한다
* 묵시적 조인으로 인해 SQL의 FROM 절에 영향을 준다
* 묵시적 조인은 조인이 일어나는 상황을 파악하기 어렵다. 단순하고 성능에 이슈가 없으면 문제가 안 되지만 성능이 중요하면 명시적 조인을 사용하자

#### **10.2.9 서브 쿼리**

* JPQL의 서브 쿼리에는 몇 가지 제약이 있다. WHERE, HAVING 절에서만 사용할 수 있고 SELECT, FROM 절에서는 사용할 수 없다

**서브 쿼리 함수**

* EXISTS
  * \[NOT\] EXISTS \(subquery\)
  * 서브쿼리에 결과가 존재하면 참. NOT은 반대
* {ALL \| ANY \| SOME}
  * {ALL \| ANY \| SOME} \(subquery\)
  * 비교 연산자와 같이 사용\(=, &gt;, &gt;=, &lt;, &lt;=, &lt;&gt;\)
* IN
  * \[NOT\] IN \(subquery\)
  * 서브쿼리의 결과 중 하나라도 같은 것이 있으면 참. 서브쿼리가 아닌 곳에서도 사용한다

#### **10.2.10 조건식**

**타입 표현**

| 종류 | 예제 |
| :--- | :--- |
| 문자 | ‘HELLO’, ‘She’’s’ |
| 숫자 | 10L\(Long\), 10D\(Double\), 10F\(Float\) |
| Boolean | TRUE, FALSE |
| ENUM | jpabook.MemberType.Admin \(패키지명 포함\) |
| 엔티티 타입 | TYPE\(m\) = Member \(상속 관계에서 사용\) |

**연산자 우선 순위**

1. 경로 탐색 연산\(.\)
2. 수학 연산 : +, -, \*, /
3. 비교 연산 : =, &gt;, &lt;, &lt;&gt;, between, like, in, is null, is empty, member, exists...
4. 논리 연산 : NOT, AND, OR

**논리 연산과 비교식**

* 논리 연산 : AND, OR, NOT
* 비교식 : =, &gt;, &lt;, &gt;=, &lt;=, &lt;&gt;

**Between, IN, Like, NULL 비교**

* Between 
  * X \[NOT\] BETWEEN A AND B
  * X는 A~B 사이의 값이면 참\(A,B 값 포함\)
* IN
  * X \[NOT\] IN
  * X와 같은 값이 예제에 하나라도 있으면 참. subquery 사용 가능
* Like
  * 문자표현식 \[NOT\] LIKE 패턴값 \[ESCAPE\]
  * 문자표현식과 패턴값을 비교

```sql
  select m from Member m 
  where m.username like '%원%'

  select m from Member m 
  where m.username like '__3'

  select m from Member m 
  where m.username like '회원\%' ESCAPE '\'
```

* NULL 비교식
  * { 단일값 경로 \| 입력 파라미터 } IS \[NOT\] NULL
  * NULL 인지 비교. NULL은 =으로 비교하면 안 되고 꼭 IS NULL을 사용해야 함

```sql
  where m.username is null
  where null = null // 거짓
  where 1=1 // 참
```

**컬렉션 식**

* 컬렉션에만 사용하는 특별한 기능. 컬렉션 식 이외의 다른 식은 사용할 수 없다
* 빈 컬렉션 비교 식
  * {컬렉션 값 연관 경로} IS \[NOT\] EMPTY
  * 컬렉션에 값이 비었으면 참

```sql
  select m from Member m
  where m.orders is not empty
```

* 컬렉션의 멤버 식
  * {엔티티나 값} \[NOT\] MEMBER \[OF\] {컬렉션 값 연관 경로}
  * 엔티티나 값이 컬렉션에 포함되어 있으면 참

**스칼라 식**

* 숫자, 문자, 날짜, case, 엔티티 타입\(엔티티의 타입 정보\) 같은 가장 기본적인 타입들
* 수학 식 : +, -\(단항 연산자\), \*, /, +, -\(사칙연산\)
* 문자함수 : CONCAT, SUBSTRING, TRIM, LOWER, UPPER, LENGTH, LOCATE
* 수학함수 : ABS, SQRT, MOD, SIZE, INDEX
* 날짜함수 : CURRENT\_DATE, CURRENT\_TIME, CURRENT\_TIMESTAMP
  * 하이버네이트는 날짜 타입에서 년\(YEAR\), 월\(MONTH\), 일\(DAY\), 시간\(HOUR\), 분\(MINUTE\), 초\(SECOND\) 값을 구하는 기능을 지원
  * 데이터베이스들은 각자의 방식으로 더 많은 날짜 함수를 지원. 오라클 방언 사용시 to\_date, to\_char 함수 지원

**CASE 식**

* 특정 조건에 따라 분기할 때 CASE 식을 사용
* 기본 CASE

  ```text
  CASE
        { WHEN <조건식> THEN <스칼라식> } +
        ELSE <스칼라식>
  END
  ```

* 심플 CASE : 조건식을 사용할 수 없지만, 문법이 단순하다. 자바의 switch case 문과 비슷

  ```text
  CASE <조건대상>
        { WHEN <스칼라식1> THEN <스칼라식2> } +
        ELSE <스칼라식>
  END
  ```

* COALESCE
  * COALESCE\(&lt;스칼라식&gt; {, &lt;스칼라식&gt;}+\)
  * 스칼라식을 차례대로 조회해서 null이 아니면 반환한다
* NULLIF
  * NULLIF\(&lt;스칼라식&gt;, &lt;스칼라식&gt;\)
  * 두 값이 같으면 null을 반환하고 다르면 첫 번째 값을 반환한다. 보통 집합 함수와 함께 사용

#### **10.2.11 다형성 쿼리**

* JPQL로 부모 엔티티를 조회하면 자식 엔티티도 함께 조회

**TYPE**

* 엔티티의 상속 구조에서 조회 대상을 특정 자식 타입으로 한정할 때 사용

  ```sql
  //JPQL
  select i from Item i
  where type(i) IN (Book, Movie)

  //SQL
  SELECT i FROM Item i
  WHERE i.DTYPE in ('B', 'M')
  ```

**TREAT\(JPA 2.1\)**

* 자바의 타입 캐스팅과 비슷하다
* 상속 구조에서 부모 타입을 특정 자식 타입으로 다룰 때 사용
* JPA 표준은 FROM, WHERE 절에서 사용 가능, 하이버네이트는 SELECT 절에도 가능

  ```sql
  //JPQL
  select i.* from Item i where treat(i as Book).author = 'kim'

  //SQL
  select i.* from Item i
  where
    i.DTYPE = 'B'
    and i.author='kim'
  ```

#### **10.2.12 사용자 정의 함수 호출\(JPA 2.1\)**

* function\_invocation::= FUNCTION\(function\_name {, function\_arg}\*\)
* 하이버네이트 구현체를 사용하면 방언 클래스를 상속해서 구현하고 사용할 데이터베이스 함수를 미리 등록해야 한다

```java
// persistence.xml
  <property name="hibernate.dialect" value="hello.MyH2Dialect">

// MyH2Dialect.java
  public class MyH2Dialect extends H2Dialect {
    public MyH2Dialect() {
      registerFunction("group_concat", new StandardSQLFunction("group_concat", StandardBasicTypes.STRING));
    }
  }
```

* 하이버네이트 구현체를 사용하면 축약해서 사용 가능

  `select group_concat(i.name) from Item i`

#### **10.2.13 기타 정리**

* enum은 = 비교 연산만 지원
* 임베디드 타입은 비교를 지원하지 않음

**EMPTY STRING**

* JPA 표준은 ''을 길이 0인 Empty String으로 정했지만 데이터베이스에 따라 ''을 NULL로 사용하는 데이터베이스도 있으므로 확인

**NULL 정의**

* 조건을 만족하는 데이터가 하나도 없으면 NULL
* NULL은 알 수 없는 값. NULL과의 모든 수학적 계산 결과는 NULL이 된다
* NULL == NULL은 알 수 없는 값
* NULL is NULL은 참

#### **10.2.14 엔티티 직접 사용**

**기본 키 값**

* 객체 인스턴스는 참조 값으로 식별하고 테이블 로우는 기본 키 값으로 식별한다
* JPQL에서 엔티티 객체를 직접 사용하면 SQL에서는 해당 엔티티의 기본 키 값을 사용

```sql
  // JPQL
  select count(m.id) from Member m      // 엔티티의 아이디를 사용
  select count(m) from Member m     // 엔티티를 직접 사용

  // SQL
  select count(m.id) as cnt
  from Member m                 // 엔티티를 직접 사용해도 SQL 변환시 엔티티의 기본 키 사용
```

* 엔티티를 파라미터로 직접 받거나, 식별자 값을 직접 사용해도 결과가 같다

```java
  // JPQL
  select m from Member m where m = :member
  select m from Member m where m.id = :memberId

  // SQL
  select m.* from Member m where m.id = ?
```

**외래 키 값**

* 팀 엔티티를 파라미터로 사용하면 외래 키와 매핑되어 있는 m.team의 외래 키인 team\_id를 사용
* 외래 키 식별자를 직접 사용하여도 Member가 team\_id를 가지고 있으므로 Member와 Team 간 묵시적 조인이 일어나지 않음

```java
  // JPQL
  select m from Member m where m.team = :team
  select m from Member m where m.team.id = :teamId

  // SQL
  select m.* from Member m where m.team_id = ?
```

#### **10.2.15 Named 쿼리 : 정적 쿼리**

* 동적 쿼리 : em.createQuery\(\) 처럼 JPQL을 문자로 완성해서 직접 넘기는 것. 런타임에 특정 조건에 따라 JPQL을 동적으로 구성할 수 있다
* 정적 쿼리 : 미리 정의한 쿼리에 이름을 부여해서 필요할 때 사용하며 Named 쿼리라고 한다. 한 번 정의하면 변경할 수 없다
* Named 쿼리
  * 애플리케이션 로딩 시점에 JPQL 문법을 체크하고 미리 파싱해둔다
  * 오류를 빨리 확인할 수 있고, 사용 시점에 파싱된 결과를 재사용하므로 성능상 이점
  * 변하지 않는 정적 SQL이 생성되므로 데이터베이스 조회 성능 최적화에 도움
  * `@NamedQuery` 또는 XML 문서에 작성할 수 있다

**Named 쿼리를 어노테이션에 정의**

```java
  // Member.java
  @Entity
  @NamedQuery(
    name = "Member.findByUsername",
    query = "select m from Member m where m.username = :username")
  public class Member {
    ...
  }

  // Member2.java
  @Entity
  @NamedQueries({
     @NamedQuery(
          name = "Member.findByUsername",
          query = "select m from Member m where m.username = :username"),
     @NamedQuery(
          name = "Member.count",
          query = "select count(m) from Member m")
  })
  public class Member2 {
      ...
  }

  // JpaMain.java
  List<Member> resultList = em.createNamedQuery("Member.findByUsername", Member.class)
            .setParameter("username", "회원1")
            .getResultList();
```

```java
  @Target({TYPE})
  public @interface NamedQuery {
    String name();  // Named 쿼리 이름(필수)
    String query();     // JPQL 정의(필수)
    LockModeType lockMode() default NONE;   // 쿼리 실행 시 락을 건다
    QueryHint[] hints() default {};         // JPA 구현체에 쿼리 힌트를 줄 수 있다. 2차 캐시 다룰 때 사용
```

**Named 쿼리를 XML에 정의**

* JPA에서 어노테이션으로 작성할 수 있는 것은 XML로도 작성할 수 있다
* xml 파일로 작성 후 persistence.xml 파일에 코드 추가

```markup
  // META-INF/ormMember.xml
  <?xml version="1.0" encoding="UTF-8"?>
  <entity-mappings xmlns="http://java.sun.com/xml/ns/persistence/orm" version="2.1">
      <named-query name="Member.findByUsername">
          <query><CDATA[
              select m
              from Member m
              where m.username = :username
          ]</query>
      </named-query>

      <named-query name="Member.count">
          <query>select count(m) from Member m</query>
      </named-query>
  </entity-mappings>

  // persistence.xml
  <persistence-unit name="jpabook">
   <mapping-file>META-INF/ormMember.xml</mapping-file>
   ...
```

**환경에 따른 설정**

* XML과 어노테이션에 같은 설정이 있는 경우 XML이 우선권을 가진다



### 10.4 QueryDSL

* JPA Criteria는 문자가 아닌 코드로 JPQL 작성
  * 문법 오류를 컴파일 단계에서 잡을 수 있다
  * IDE 자동완성 기능의 도움을 받을 수 있다
  * 너무 복잡하고 어렵다. JPQL이 어떻게 생성될지 파악하기 쉽지 않다
* QueryDSL은 JPQL 빌더 역할을 하며 Criteria 대체 가능
  * 쉽고 간결하며 모양도 쿼리와 비슷하게 개발 가능
  * 오픈소스 프로젝트이며 JPA, JDBC, Hibernate Search 등 다양하게 지원
  * 데이터 조회에 기능이 특화되어 있다

#### **10.4.1 QueryDSL 설정**

**필요 라이브러리\(pom.xml\)**

* querydsl-jpa : QueryDSL JPA 라이브러리
* querydsl-apt : 쿼리 타입\(Q\)을 생성할 때 필요한 라이브러리

**환경설정**

* QueryDSL을 사용하려면 엔티티를 기반으로 쿼리 타입이라는 쿼리용 클래스를 생성해야 함
* 쿼리 타입 생성용 플러그인을 pom.xml에 추가
  * 콘솔에서 `mvn compile`을 입력하면 outputDirectory에 지정한 target/generated-sources 위치에 QMember.java처럼 Q로 시작하는 쿼리 타입들이 생성

#### **10.4.2 시작 - 예제**

* QueryDSL 사용법 
  1. JPAQuery 객체를 생성. 이 때 엔티티 매니저를 생성자에 넘겨준다
  2. 사용할 쿼리 타입\(Q\) 생성. 생성자에는 별칭을 준다\(JPQL에서 사용\)

**기본 Q 생성**

* 쿼리 타입\(Q\)은 사용하기 편리하도록 기본 인스턴스를 보관하고 있다
* 같은 엔티티를 조인하거나 같은 엔티티를 서브쿼리에 사용하면 같은 별칭이 사용되므로 이때는 별칭을 직접 지정해서 사용

  ```java
    QMember qMember = new QMember("m");     // 직접 지정
    QMember qMember2 = QMember.member;      // 기본 인스턴스 사용
  ```

* import static 기본 인스턴스를 사용시 코드를 더 간결하게 작성할 수 있다 - 예제

#### **10.4.3 검색 조건 쿼리 - 예제**

* where 절에는 and, or을 사용할 수 있다. 여러 검색 조건을 사용해도 된다\(and 연산\)
* 쿼리 타입의 필드는 필요한 대부분의 메소드를 명시적으로 제공한다
* where에서 사용되는 메소드 : between, contains, startsWith

#### **10.4.4 결과 조회**

* 쿼리 작성이 끝나고 결과 조회 메소드를 호출하면 실제 데이터베이스를 조회
* uniqueResult\(\), list\(\) 사용. 파라미터로 프로젝션 대상을 넘겨준다
* 결과 조회 메소드
  * uniqueResult\(\) : 조회 결과가 한 건일 때 사용. 결과가 없으면 null을 반환, 하나 이상이면 NonUniqueResultException 예외 발생
  * singleResult\(\) : uniqueResult\(\)와 같지만 결과가 하나 이상이면 처음 데이터를 반환
  * list\(\) : 결과가 하나 이상일 때 사용. 결과가 없으면 빈 컬렉션을 반환

#### **10.4.5 페이징과 정렬**

* 정렬은 orderBy에 asc, desc를 사용
* 페이징은 offset과 limit를 적절히 조합해서 사용. restrict\(\)에 QueryModifiers를 파라미터로 사용해도 됨
* listResults\(\) : 전체 데이터 조회를 위한 count 쿼리를 한 번 더 실행하여 실제 페이징 처리시 검색된 전체 데이터 수를 알려준다. 그리고 반환하는 SearchResults 객체로 전체 데이터 수를 조회할 수 있다

```java
  QItem item = QItem.item;

  query.from(item)
    .where(item.price.gt(20000))
    .orderBy(item.price.desc(), item.stockQuantity.asc())
    .offset(10).limit(20)
    .list(item);

  QueryModifiers queryModifiers = new QueryModifiers(20L, 10L);     // limit, offset
  List<Item> list = query.from(item)
                        .restrict(queryModifiers)
                        .list(item);
```

#### **10.4.6 그룹**

* groupBy를 사용하고 having을 사용한다

#### **10.4.7 조인**

* innerJoin, leftJoin, rightJoin, fullJoin을 사용할 수 있다
* JPQL의 on, 성능 최적화를 위한 fetch 조인도 사용할 수 있다
* 첫 번째 파라미터에 조인 대상을 지정하고, 두 번째 파라미터에 별칭으로 사용할 쿼리 타입을 지정

  ```sql
    join(order.member, member)

    leftJoin(order.orderItems, orderItem)
    .on(orderItem.count.gt(2))

    innerJoin(order.member, member).fetch()

    query.from(order, member)   // from 절에 여러 조인을 사용하는 세타 조인
  ```

#### **10.4.8 서브 쿼리**

* 서브 쿼리는 JPASubQuery를 생성해서 사용
* 결과가 하나이면 unique\(\), 여러 건이면 list\(\) 사용

  ```sql
    QItem item = QItem.item;
    QItem itemSub = new QItem("itemSub");

    query.from(item)
        .where(item.in(new JPASubQuery().from(itemSub)
                    .where(item.name.eq(itemSub.name))
                    .list(itemSub)
        ))
        .list(item);
  ```

#### **10.4.9 프로젝션과 결과 반환**

* select 절에 조회 대상을 지정하는 것을 프로젝션이라 한다

**프로젝션 대상이 하나**

* 프로젝션 대상이 하나이면 해당 타입으로 반환한다

**여러 컬럼 반환과 튜플**

* 프로젝션 대상으로 여러 필드를 선택하면 QueryDSL은 기본으로 Tuple이라는 Map과 비슷한 내부 타입을 사용
* 조회 결과는 tuple.get\(\) 메소드에 조회한 쿼리 타입을 지정

**빈 생성**

* 쿼리 결과를 엔티티가 아닌 특정 객체로 받고 싶으면 빈 생성\(Bean population\) 기능 사용
* 프로퍼티 접근, 필드 직접 접근, 생성자 사용

  ```java
    QItem item = QItem.item;

    // 프로퍼티 접근
    List<ItemDTO> result = query.from(item)
                                .list(Projections.bean(ItemDTO.class, item.name.as("username"), item.price)); 

    // 필드 직접 접근
    List<ItemDTO> result = query.from(item)
                                .list(Projections.fields(ItemDTO.class, item.name.as("username"), item.price));

    // 생성자 사용
    List<ItemDTO> result = query.from(item)
                                .list(Projections.constructor(ItemDTO.class, item.name, item.price));
  ```

**DISTINCT**

* query.distinct\(\).from\(item\)...

#### **10.4.10 수정, 삭제 배치 쿼리**

* JPQL 배치 쿼리와 같이 영속성 컨텍스트를 무시하고 데이터베이스를 직접 쿼리한다는 점에 유의
* 수정 배치 쿼리는 JPAUpdateClause를, 삭제 배치 쿼리는 JPADeleteClause를 사용

  ```java
  QItem item = QItem.item;

  JPAUpdateClause updateClause = new JPAUpdateClause(em, item);
  long count = updateClause.where(item.name.eq("JPA책"))
                        .set(item.price, item.price.add(100))
                        .execute();

  JPADeleteClause deleteClause = new JPADeleteClause(em, item);
  long count = deleteClause.where(item.name.eq("JPA책"))
                           .execute();
  ```

#### **10.4.11 동적 쿼리**

* BooleanBuilder를 사용하면 특정 조건에 따른 동적 쿼리를 편리하게 생성할 수 있다

```java
  QItem item = QItem.item;

  BooleanBuilder builder = new BooleanBuilder();

  if(StringUtils.hasText("개발자")) {
    builder.and(item.name.contains("개발자"));
    builder.and(item.price.gt(10000));
  }
  List<Item> result = query.from(item)
                            .where(builder)
                            .list(item)
```

#### **10.4.12 메소드 위임 - 예제**

* 메소드 위임 기능을 사용하면 쿼리 타입에 검색 조건을 직접 정의할 수 있다
* static\(정적\) 메소드를 만들고 `@QueryDelegate` 어노테이션 속성으로 기능을 적용할 엔티티를 지정한다. 정적 메소드의 첫 번째 파라미터에는 대상 엔티티의 쿼리 타입을 지정, 나머지는 필요한 파라미터를 정의
* String, Date 같은 자바 기본 내장 타입에도 메소드 위임 기능 사용 가능

```java
  public class ItemExpression {

    @QueryDelegate(Item.class)
    public static BooleanExpression isExpensive(QItem item, Integer price) {
        return item.price.gt(price);
    }

    @QueryDelegate(String.class)    // ext/java/lang에 QString 생성
    public static BooleanExpression isHelloStart(StringPath stringPath) {
        return stringPath.startsWith("Hello");
    }
  }
```

### 10.5 네이티브 SQL

* JPQL은 표준 SQL이 지원하는 대부분의 문법과 SQL 함수들을 지원하지만 특정 데이터베이스에 종속적인 기능은 지원하지 않는다
  * 특정 데이터베이스만 지원하는 함수, 문법, SQL 쿼리 힌트
  * 인라인 뷰\(from 절 서브쿼리\), UNION, INTERSECT
  * 스토어드 프로시저
* 특정 데이터베이스에 종속적인 기능을 지원하는 방법
  * 특정 데이터베이스만 사용하는 함수
    * JPQL에서 네이티브 SQL 함수 호출
    * 하이버네이트는 데이터베이스 방언에 각 데이터베이스에 종속적인 함수들을 정의. 직접 호출할 함수를 정의할 수도 있다
  * 특정 데이터베이스만 지원하는 SQL 쿼리 힌트
    * 하이버네이트를 포함한 JPA 구현체들이 지원
  * 인라인뷰, UNION, INTERSECT
    * 하이버네이트는 지원하지 않음. 일부 JPA 구현체들이 지원
  * 스토어드 프로시저
    * JPQL에서 호출 가능
  * 특정 데이터베이스만 지원하는 문법
    * 오라클의 CONNECT BY 같은 SQL 문법은 지원하지 않음. 이때 네이티브 SQL을 사용
* 다양한 이유로 JPQL을 사용할 수 없을 때 SQL을 직접 사용할 수 있는 네이티브 SQL을 제공.
* 네이티브 SQL을 사용하면 엔티티를 조회할 수 있고 JPA가 지원하는 영속성 컨텍스트의 기능을 그대로 사용할 수 있다
* JDBC API를 직접 사용하면 단순히 데이터의 나열을 조회한다

#### **10.5.1 네이티브 SQL 사용**

* 네이티브 쿼리 API의 종류
  * 엔티티 조회
  * 값 조회
  * 결과 매핑 사용

**엔티티 조회**

* em.createNativeQuery\(SQL, 결과 클래스\)
* JPQL 사용과 비슷하지만 실제 데이터베이스 SQL을 사용한다는 것과 위치기반 파라미터만 지원한다는 차이가 있다
* 네이티브 SQL로 SQL만 직접 사용할 뿐이지 나머지는 JPQL을 사용할 때와 같다
* 하이버네이트는 네이티브 SQL에 이름 기반 파라미터를 사용할 수 있다

**값 조회**

* em.createNativeQuery\(SQL\)
* JPA는 조회한 값들을 Object\[\]에 담아서 반환한다. 영속성 컨텍스트가 관리하지 않는다

**결과 매핑 사용**

* em.createNativeQuery\(SQL, String resultSetMapping\)
* 엔티티와 스칼라 값을 함께 조회하는 것처럼 매핑이 복잡해지면 `@SqlResultSetMapping`을 정의해서 결과 매핑을 사용해야 함
* 쿼리 결과에서 id, age, name, team\_id는 Member 엔티티와 매핑, order\_count는 단순히 값으로 매핑

```java
  // SQL
  String sql = "SELECT M.ID, AGE, NAME, TEAM_ID, I.ORDER_COUNT "
            + "FROM MEMBER M LEFT JOIN "
            + "(SELECT IM.ID, COUNT(*) AS ORDER_COUNT "
            + "FROM ORDERS O, MEMBER IM WHERE O.MEMBER_ID = IM.ID) I "
            + "ON M.ID = I.ID";

  // 매핑 정보          
  @Entity
  @SqlResultSetMapping(name = "memberWithOrderCount",
    entities = {@EntityResult(entityClass = Member.class)},
    columns = {@ColumnResult(name = "ORDER_COUNT")}
  )
  public class Member { ... }
```

* `@FieldResult`를 사용해서 컬럼명과 필드명을 직접 매핑 가능. 엔티티 필드에 정의한 `@Column`보다 앞선다. `@FieldResult`를 한 번이라도 사용하면 전체 필드를 매핑해야 한다
* 엔티티 조회시 컬럼명이 중복될 때도 `@FieldResult`를 사용

```java
  // SQL
  String sql = "SELECT o.id AS order_id, o.quantity AS order_quantity, o.item AS order_item, i.name AS item_name "
                + "FROM Order o, Item i WHERE (order_quantity > 25) AND (order_item = i.id)"

  // 매핑 정보
  @SqlResultSetMapping(name = "OrderResults",
    entities = {
        @EntityResult(entityClass=com.acme.Order.class, fields = {
            @FieldResult(name="id", column="order_id"),
            @FieldResult(name="quantity", column="order_quantity"),
            @FieldResult(name="item", column="order_item")})},
    columns = {
        @ColumnResult(name="item_name")})
```

**결과 매핑 어노테이션**

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FLs6et%2FbtqECv775bg%2FcRTXS3FNz15oOUGh5OtL21%2Fimg.png)

#### **10.5.2 Named 네이티브 SQL**

* JPQL처럼 네이티브 SQL도 Named 네이티브 SQL을 사용해서 정적 SQL을 작성할 수 있다
* JPQL Named 쿼리와 같은 createNamedQuery 메소드를 사용한다\(TypeQuery 사용 가능\)

  ```java
  @Entity
  @NamedNativeQuery(
        name = "Member.memberSQL",
        query = "SELECT ID, AGE, NAME, TEAM_ID "
            + "FROM MEMBER WHERE AGE > ?",
        resultClass = Member.class
  )
  public class Member { ... }

  // 엔티티 조회
  TypedQuery<Member> nativeQuery = em.createNamedQuery("Member.memberSQL", Member.class)
                                    .setParameter(1, 20);
  ```

* Named 네이티브 SQL에서 결과 매핑 사용이 가능하다

  ```java
  @Entity
  @SqlResultSetMapping(name = "memberWithOrderCount",
        entities = {@EntityResult(entityClass = Member.class)},
        columns = {@ColumnResult(name = "ORDER_COUNT")}
  )
  @NamedNativeQuery(
    name = "Member.memberWithOrderCount",
    query = "SELECT M.ID, AGE, NAME, TEAM_ID, I.ORDER_COUNT "
            + "FROM MEMBER M LEFT JOIN "
            + "(SELECT IM.ID, COUNT(*) AS ORDER_COUNT "
            + "FROM ORDERS O, MEMBER IM WHERE O.MEMBER_ID = IM.ID) I "
            + "ON M.ID = I.ID",
    resultSetMapping = "memberWithOrderCount"
  )
  public class Member { ... }

  // 엔티티 조회
  List<Object[]> resultList = em.createNamedQuery("Member.memberWithOrderCount")
                                .getResultList();
  ```

**@NamedNativeQuery**

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FoZQTi%2FbtqEACVkMRC%2FfuUEr3TOlxuhBbZfodkfa1%2Fimg.png)

* 여러 Named 네이티브 쿼리를 선언하는 방법

  ```text
    @NamedNativeQueries({
        @NamedNativeQuery(...),
        @NamedNativeQuery(...)
    })
  ```

#### **10.5.3 네이티브 SQL XML에 정의**

* XML에 정의할 때는 순서대로 를 먼저 정의하고 을 정의해야 한다
* 네이티브 SQL의 쿼리들은 대체로 복잡하고 라인수가 많아 XML을 사용하는 것이 편리하다

  ```markup
  <entity-mappings ...>
    <named-native-query name="Member.memberWithOrderCountXml"
        result-set-mapping="memberWithOrderCountResultMap">
        <query> ... </query>
    </named-native-query>

    <sql-result-set-mapping name="memberWithOrderCountResultMap">
        <entity-result entity-class="jpabook.domain.Member"/>
        <column-result name="ORDER_COUNT"/>
    </sql-result-set-mapping>
  </entity-mappings>
  ```

#### **10.5.4 네이티브 SQL 정리**

* 네이티브 SQL도 JPQL을 사용할 때와 마찬가지로 Query, TypedQuery를 반환한다.
* 페이징 처리와 같은 JPQL API를 그대로 사용할 수 있다
* 네이티브 SQL은 JPQL이 자동 생성하는 SQL을 수동으로 직접 정의하는 것. JPA 제공 기능 대부분을 그대로 사용 가능
* 네이티브 SQL은 관리하기 쉽지 않고 자주 사용시 특정 데이터베이스에 종속적인 쿼리가 많아져 이식성이 떨어진다
* 표준 JPQL 사용 -&gt; 하이버네이트 같은 JPA 구현체가 제공하는 기능 사용 -&gt; 네이티브 SQL 순서로 사용 -&gt; MyBatis...

#### **10.5.5 스토어드 프로시저\(JPA 2.1\)**

**스토어드 프로시저 사용**

* em.createStoredProcedureQuery\(\) 메소드에 사용할 스토어드 프로시저 이름을 입력
* registerSotredProcedureParameter\(\) 메소드를 사용해서 프로시저에서 사용할 파라미터를 순서, 타입, 파라미터 모드 순으로 정의
* ParameterMode는 IN, INOUT, OUT, REF\_CURSOR가 있다
* 파라미터에 순서 대신 이름 사용도 가능

```java
  // 순서 기반 파라미터
  StoredProcedureQuery spq = em.createStoredProcedureQuery("proc_multiply");
  spq.registerStoredProcedureParameter(1, Integer.class, ParameterMode.IN);
  spq.registerStoredProcedureParameter(2, Integer.class, ParameterMode.OUT);

  spq.setParameter(1, 100);
  spq.execute();

  Integer out = (Integer)spq.getOutputParameterValue(2);
  System.out.println("out = " + out);   // 결과 = 200

  // 이름 기반 파라미터
  StoredProcedureQuery spq = em.createStoredProcedureQuery("proc_multiply");
  spq.registerStoredProcedureParameter("inParam", Integer.class, ParameterMode.IN);
  spq.registerStoredProcedureParameter("outParam", Integer.class, ParameterMode.OUT);

  spq.setParameter("inParam", 100);
  spq.execute();

  Integer out = (Integer)spq.getOutputParameterValue("outParam");
  System.out.println("out = " + out);   // 결과 = 200
```

**Named 스토어드 프로시저 사용**

* 스토어드 프로시저 쿼리에 이름을 부여해서 사용하는 것을 Named 스토어드 프로시저라 한다
* `@NamedStoredProcedureQuery`로 정의하고 name 속성으로 이름을 부여
* procedureName 속성에 실제 호출할 프로시저 이름을 적어준다
* `@StoredProcedureParameter`를 사용해서 파라미터 정보를 정의
* 둘 이상의 Named 스토어드 프로시저 사용시 `@NamedStoredProcedureQueries` 사용
* XML에서 정의하는 방식도 가능

```java
  @NamedStoredProcedureQuery(
        name = "multiply",
        procedureName = "proc_multiply",
        parameters = {
            @StoredProcedureParameter(name = "inParam", mode = ParameterMode.IN, type = Integer.class),
            @StoredProcedureParameter(name = "outParam", mode = ParameterMode.OUT, type = Integer.class)
        }
  }
  @Entity
  public class Member { ... }
```

### 10.6 객체지향 쿼리 심화

* 벌크 연산 : 여러 데이터를 수정
* JPQL과 영속성 컨텍스트
* JPQL과 플러시 모드

#### **10.6.1 벌크 연산**

* 엔티티를 수정하려면 영속성 컨텍스트의 변경 감지 기능이나 병합을 사용하고, 삭제하려면 em.remove\(\) 사용
* 수백개 이상의 엔티티를 하나씩 처리하기에는 시간이 오래 걸린다
* 벌크 연산을 사용하면 여러건을 한 번에 수정하거나 삭제가 가능하다
* executeUpdate\(\) 메소드로 벌크 연산의 영향을 받은 엔티티 건수를 반환
* JPA 표준은 아니지만 하이버네이트는 INSERT 벌크 연산도 지원

```java
  //UPDATE
  String qlString = "update Product p set p.price = p.price * 1.1 where p.stockAmount < :stockAmount";

  int resultCount = em.createQuery(qlString).setParameter("stockAmount", 10).executeUpdate();

  //DELETE
  String qlString = "delete from Product p where p.price < :price";

  int resultCount = em.createQuery(qlString).setParameter("price", 100).executeUpdate();

  //INSERT
  String qlString = "insert into ProductTemp(id, name, price, stockAmount) "
                    + "select p.id, p.name, p.price, p.stockAmount from Product p "
                    + "where p.price < :price";

  int resultCount = em.createQuery(qlString).setParameter("price", 100).executeUpdate();
```

**벌크 연산의 주의점 - 예제**

* 벌크 연산 사용시 영속성 컨텍스트를 무시하고 데이터베이스에 직접 쿼리한다는 점에 주의해야 한다 ![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2Fb4deTW%2FbtqEz0hRLgM%2FhU6t2HN93XT8OOUIJ1NzB0%2Fimg.png) ![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FYAeSi%2FbtqEA2eZqtM%2F5omPNErS6TuKYfciCzktIk%2Fimg.png)
* 문제를 해결하는 방법
  * em.refresh\(\) 사용하여 다시 조회하면 원하는 결과로 조회 가능하다
  * 벌크 연산을 먼저 실행하면 변경된 데이터로 조회가 된다. JPA와 JDBC를 함께 사용할 때도 유용하다
  * 벌크 연산 수행 후 영속성 컨텍스트를 초기화
* 벌크 연산은 영속성 컨텍스트와 2차 캐시를 무시하고 데이터베이스에 직접 실행한다. 따라서 영속성 컨텍스트와 데이터베이스 간에 데이터 차이가 발생할 수 있으므로 주의해서 사용해야 한다

#### **10.6.2 영속성 컨텍스트와 JPQL**

**쿼리 후 영속 상태인 것과 아닌 것**

* JPQL의 조회 대상은 엔티티, 임베디드 타입, 값 타입 같이 다양한 종류가 있다
* JPQL로 엔티티를 조회하면 영속성 컨텍스트에서 관리되지만 엔티티가 아니면 영속성 컨텍스트에서 관리되지 않는다
* 영속성 컨텍스트가 관리하지 않는 임베디드 타입은 조회해서 값을 변경해도 변경 감지에 의한 수정이 발생하지 않는다
* 조회한 엔티티만 영속성 컨텍스트가 관리한다

  ```sql
    select m from Member m  // 엔티티 조회(관리O)
    select o.address from Order o   // 임베디드 타입 조회(관리X)
    select m.id, m.username from Member m     // 단순 필드 조회(관리X)
  ```

**JPQL로 조회한 엔티티와 영속성 컨텍스트**

* 영속성 컨텍스트에 이미 있는 엔티티를 다시 조회하면 JPQL로 데이터베이스에서 조회한 결과를 버리고 영속성 컨텍스트에 있던 엔티티를 반환한다\(식별자 값을 사용해서 비교\)
* 왜 새로 조회한 엔티티를 버리고 기존 엔티티를 반환하는 것일까?
  * 새로운 엔티티를 영속성 컨텍스트에 하나 더 추가하기
    * 영속성 컨텍스트는 기본 키 값을 기준으로 엔티티를 관리하므로 동일 엔티티를 추가할 수 없다
  * 기존 엔티티를 새로 검색한 엔티티로 대체하기
    * 영속성 컨텍스트에 수정중인 데이터가 사라질 수 있으므로 위험하다
  * 기존 엔티티를 그대로 두고 새로 검색한 엔티티를 버린다
    * 영속성 컨텍스트는 엔티티의 동일성을 보장하므로

**find\(\) vs JPQL**

* em.find\(\) 메소드는 엔티티를 영속성 컨텍스트에서 먼저 찾고 없으면 데이터베이스에서 찾는다. 엔티티가 영속성 컨텍스트에 있으면 메모리에서 바로 찾으므로 성능상 이점이 있다\(1차 캐시\)
* JPQL은 항상 데이터베이스에 SQL을 실행해서 결과를 조회한다. 그 후 영속성 컨텍스트에 이미 조회한 같은 엔티티가 있으면 영속성 컨텍스트에 있는 기존 엔티티를 반환한다

#### **10.6.3 JPQL과 플러시 모드**

* 플러시는 영속성 컨텍스트의 변경 내역을 데이터베이스에 동기화하는 것이다
* JPA는 플러시가 일어날 떄 영속성 컨텍스트에 등록, 수정, 삭제한 엔티티를 찾아서 INSERT, UPDATE, DELETE SQL을 만들어 데이터베이스에 반영한다
* 플러시를 호출하려면 em.flush\(\)를 직접 호출하거나 플러시 모드에 따라 커밋 직전이나 쿼리 실행 직전에 자동으로 플러시가 호출된다\(em.setFlushMode\(FlushModeType.AUTO\)\)
* 플러시 모드는 FlushModeType.AUTO가 기본값이다. JPA는 트랜잭션 커밋 직전이나 쿼리 실행 직전에 자동으로 플러시를 호출

**쿼리와 플러시 모드**

* JPQL은 영속성 컨텍스트에 있는 데이터를 고려하지 않고 데이터베이스에서 데이터를 조회한다
* JPQL 실행 전 영속성 컨텍스트의 내용을 데이터베이스에 반영해야 한다. FlushModeType.COMMIT으로 설정시 쿼리시 플러시를 하지 않으므로 의도하지 않은 결과가 발생할 수 있다

**플러시 모드와 최적화**

* FlushModeType.COMMIT은 데이터 무결성에 심각한 피해를 줄 수 있지만 플러시가 자주 일어나는 상황에서 플러시 횟수를 줄여 성능을 최적화 할 수 있다는 장점이 있다
* JPA를 통하지 않고 JDBC로 쿼리를 직접 실행하면 JPA는 JDBC가 실행한 쿼리를 인식할 방법이 없다. 이때는 JDBC로 쿼리를 실행하기 직전 em.flush\(\)를 호출해서 영속성 컨텍스트의 내용을 데이터베이스에 동기화하는 것이 안전하다

### 10.7 정리

1. JPQL은 SQL을 추상화해서 특정 데이터베이스 기술에 의존하지 않는다
2. Criteria나 QueryDSL은 JPQL을 만들어주는 빌더 역할을 할 뿐이므로 핵심은 JPQL을 잘 알아야 한다
3. Criteria나 QueryDSL을 사용하면 동적으로 변하는 쿼리를 편리하게 작성할 수 있다
4. Criteria는 JPA가 공식 지원하는 기능이지만 직관적이지 않고 사용하기에 불편하다. 반면에 QueryDSL은 JPA가 공식 지원하는 기능은 아니지만 직관적이고 편리하다
5. JPA도 네이티브 SQL을 제공하므로 직접 SQL을 사용할 수 있다. 하지만 특정 데이터베이스에 종속적인 SQL을 사용하면 다른 데이터베이스로 변경하기 쉽지 않다. 따라서 최대한 JPQL을 사용하고 그래도 방법이 없을 때 네이티브 SQL을 사용하자
6. JPQL은 대량에 데이터를 수정하거나 삭제하는 벌크 연산을 지원한다

