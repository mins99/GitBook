---
description: 이 글은 '자바 ORM 표준 JPA 프로그래밍' (김영한 저)' 책 내용을 정리한 글입니다.
---

# Chapter02



### 2.3 라이브러리와 프로젝트 구조

#### **2.3.1 메이븐과 사용 라이브러리 관리**

* `<dependencies>` : groupId + artifactId + version 을 적어주면 라이브러리(jar 파일)을 메이븐 공식 저장소에서 내려받아 라이브러리에 추가해준다
* JPA 구현체로 하이버네이트를 사용하기 위한 핵심 라이브러리
  * JPA, 하이버네이트(hibernate) : JPA 표준과 하이버네이트를 포함하는 라이브러리.&#x20;
    * `hibernate-core` : 하이버네이트 라이브러리
    * `hibernate-entitymanager` : 하이버네이트가 JPA 구현체로 동작하도록 JPA 표준을 구현한 라이브러리
    * `hibernate-jpa-2.1-api` : JPA 2.1 표준 API를 모아둔 라이브러리
  * H2 데이터베이스

```java
// pom.xml

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>jpa-study</groupId>
    <artifactId>ch02</artifactId>
    <version>1.0.0</version>
    <dependencies>
        <!-- JPA 하이버네이트 -->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-entitymanager</artifactId>
            <version>5.3.10.Final</version>
        </dependency>
        <!-- H2 데이터베이스 -->
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <version>1.4.199</version>
        </dependency>
    </dependencies>
</project>
```

### 2.4 객체 매핑 시작

* 회원 클래스에 사용한 매핑 어노테이션
  * `@Entity` : 테이블과 매핑한다고 JPA에게 알려준다.
  * `@Table` : 엔티티 클래스에 매핑할 테이블 정보를 알려준다. `@name` 속성을 사용하여 엔티티를 테이블에 매핑한다. 생략시 클래스 이름을 테이블 이름으로 매핑한다.
  * `@Id` : 엔티티 클래스의 필드를 테이블의 기본 키(PK)에 매핑시 사용. 해당 어노테이션이 사용된 필드를 식별자 필드 라고 함.
  * `@Column` : `@name`속성을 사용하여 엔티티의 필드를 컬럼에 매핑함.
  * 매핑 정보가 없는 필드 : 매핑 어노테이션이 없는 경우 필드명을 컬럼명으로 매핑.

```java
// Member.java

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity
@Table(name="MEMBER")
public class Member {

    @Id
    @Column(name = "ID")
    private String id;

    @Column(name = "NAME")
    private String username;

    private Integer age;

    // Getter, Setter ...
}
```

### 2.5 persistence.xml 설정

* JPA는 resources/META-INF 하위의 persistence.xml 파일에서 별도의 설정 없이 필요한 설정 정보를 관리한다.
* 영속성 유닛(persistence-unit) : JPA는 일반적으로 연결할 데이터베이스 당 하나의 고유한 이름의 영속성 유닛을 등록한다.
* JPA 표준 속성 : javax.persistence 로 시작하는 속성. 특정 구현체에 종속되지 않음.
* 하이버네이트 속성 : hibernate 로 시작하는 속성. 하이버네이트 전용 속성으로 하이버네이트에서만 사용 가능

```java
// persistence.xml

<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2"
             xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
    <persistence-unit name="jpaStudy">
        <properties>
            <!-- 필수 속성 -->
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>

            <!-- 옵션 -->
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.use_sql_comments" value="true"/>
            <!--<property name="hibernate.hbm2ddl.auto" value="create" />-->
        </properties>
    </persistence-unit>
</persistence>
```

#### **2.5.1 데이터베이스 방언**

* JPA는 특정 데이터베이스에 종속적이지 않은 기술로 다른 데이터베이스로 쉽게 교체 가능
* 방언(Dialect) : 데이터 타입, 다른 함수명, 페이징 등 특정 데이터베이스만의 고유한 기능.
* 하이버네이트를 포함한 대부분의 JPA 구현체들은 이런 문제를 해결하기 위해 다양한 데이터베이스 방언 클래스를 제공

### 2.6 애플리케이션 개발

#### **2.6.1 엔티티 매니저 설정**

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2\&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FdPhijz%2FbtqDmV9xbSD%2FKkcGOFh1dfKb06uh0KncTK%2Fimg.png)

* 엔티티 매니저 팩토리 생성(EntityManagerFactory)
  * persistence.xml의 설정 정보 중 영속성 유닛명을 찾아서 엔티티 매니저 팩토리를 생성
  * 엔티티 매니저 팩토리 생성시 JPA를 동작시키기 위한 기반 객체를 만들고, 구현체에 따라 DB 커넥션 풀도 생성하므로 생성비용이 크다.
  * 애플리케이션 전체에서 한 번만 생성하고 공유해서 사용해야 함.
* 엔티티 매니저 생성(EntityManager)
  * 엔티티 매니저 팩토리에서 엔티티 매니저를 생성
  * 엔티티 매니저를 사용하여 엔티티를 데이터베이스에 등록/수정/삭제/조회 가능
  * 내부에 데이터소스(데이터베이스 커넥션)을 유지하면서 데이터베이스와 통신
  * DB 커넥션과 관계가 있으므로 스레드간 공유, 재사용하면 안 된다.
* 종료
  * 사용이 끝난 엔티티 매니저, 엔티티 매니저 팩토리는 종료해야 한다.

#### **2.6.2 트랜잭션 관리**

* JPA는 트랜잭션 없이 데이터 변경시 예외가 발생하므로 항상 트랜잭션 안에서 데이터를 변경해야 함
* 트랜잭션을 시작하려면 엔티티 매니저에서 트랜잭션 API를 받아와야 한다.
* 트랜잭션 API를 사용해서 비즈니스 로직이 정상 동작하면 트랜잭션을 커밋하고, 예외가 발생하면 트랜잭션을 롤백한다.

```java
// JpaMain.java

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityTransaction;
import javax.persistence.Persistence;
import java.util.List;

public class JpaMain {

    public static void main(String[] args) {

        //엔티티 매니저 팩토리 생성
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpaStudy");
        EntityManager em = emf.createEntityManager(); //엔티티 매니저 생성

        EntityTransaction tx = em.getTransaction(); //트랜잭션 기능 획득

        try {

            tx.begin(); //트랜잭션 시작
            logic(em);  //비즈니스 로직
            tx.commit();//트랜잭션 커밋

        } catch (Exception e) {
            e.printStackTrace();
            tx.rollback(); //트랜잭션 롤백
        } finally {
            em.close(); //엔티티 매니저 종료
        }

        emf.close(); //엔티티 매니저 팩토리 종료
    }
}
```

#### **2.6.3 비즈니스 로직**

* 비즈니스 로직의 등록, 수정, 삭제, 조회 작업은 엔티티 매니저를 통해 수행됨
* 등록 : 엔티티 매니저의 `persist()` 메소드에 저장할 엔티티를 넘겨준다.
* 수정 : set 메소드를 통해 엔티티의 값을 변경한다.
* 삭제 : 엔티티 매니저의 `remove()` 메소드에 삭제하려는 엔티티를 넘겨준다.
* 조회 : `find()` 메소드에 조회할 엔티티 타입과 `@Id`로 매핑한 식별자 값을 넘겨준다.

```java
// JpaMain.java

public class JpaMain {

    public static void main(String[] args) { ... }

    public static void logic(EntityManager em) {

        String id = "id1";
        Member member = new Member();
        member.setId(id);
        member.setUsername("MS");

        //등록
        em.persist(member);

        //수정
        member.setAge(20);

        //한 건 조회
        Member findMember = em.find(Member.class, id);
        System.out.println("findMember=" + findMember.getUsername() + ", age=" + findMember.getAge());

        //목록 조회
        List<Member> members = em.createQuery("select m from Member m", Member.class).getResultList();  // JPQL
        System.out.println("members.size=" + members.size());

        //삭제
        em.remove(member);

    }
}
```

#### **2.6.4 JPQL**

* JPA는 엔티티 객체를 중심으로 개발하므로 테이블이 아닌 엔티티 객체를 대상으로 검색해야 함
* JPA는 JPQL이라는 엔티티 객체를 대상으로 쿼리하는 객체지향 쿼리 언어를 제공
* select, from, where, group by, having, join 등 사용 가능
