---
description: 이 글은 '자바 ORM 표준 JPA 프로그래밍' (김영한 저)' 책 내용을 정리한 글입니다.
---

# Chapter14



* 14장에서 다룰 내용
  * 컬렉션 : 다양한 컬렉션과 특징을 설명한다
  * 컨버터 : 엔티티의 데이터를 변환해서 데이터베이스에 저장한다
  * 리스너 : 엔티티에서 발생한 이벤트를 처리한다
  * 엔티티 그래프 : 엔티티를 조회할 때 연관된 엔티티들을 선택해서 함께 조회한단

### 14.1 컬렉션

* JPA는 Collection, List, Set, Map 컬렉션을 지원
* `@OneToMany`, `@ManyToMany`를 사용해서 일대다나 다대다 엔티티를 매핑할 때 사용
* `@ElementCollection`을 사용해서 값 타입을 하나 이상 보관할 때 사용
* 자바 컬렉션 인터페이스의 특징
  * Collection : 자바가 제공하는 최상위 컬렉션. 하이버네이트는 중복을 허용하고 순서를 보장하지 않는다고 가정
  * Set : 중복을 허용하지 않는 컬렉션. 순서를 보장하지 않는다
  * List : 순서가 있는 컬렉션. 순서를 보장하고 중복을 허용한다
  * Map : Key, Value 구조로 되어 있는 특수한 컬렉션

#### **14.1.1 JPA와 컬렉션**

* 하이버네이트는 엔티티를 영속 상태로 만들 때 컬렉션 필드를 하이버네이트에서 준비한 컬렉션으로 감싸서 사용
* Team 클래스에 대해 members는 영속 상태 이전에는 `java.util.ArrayList` 타입이었지만 영속 상태로 만든 직후 하이버네이트가 제공하는 내장 컬렉션인 `org.hibernate.collection.internal.PersistentBag` 타입을 사용하도록 참조를 변경
* 하이버네이트가 제공하는 내장 컬렉션은 원본 컬렉션을 감싸고 있어서 래퍼 컬렉션으로도 부름

```java
  // Team.java
  @Entity
  public class Team {
    @Id
    private String id;

    @OneToMany
    @JoinColumn
    private Collection<Member> members = new ArrayList<Member>();

    ...
  }
```

* 하이버네이트 내장 컬렉션과 특징

| 컬렉션 인터페이스 | 내장 컬렉션 | 중복 허용 | 순서 보관 |
| :--- | :--- | :--- | :--- |
| Collection, List | PersistenceBag | O | X |
| Set | PersistenceSet | X | X |
| List + @OrderColumn | PersistentList | O | O |

#### **14.1.2 Collection, List**

* Collection, List는 중복을 허용한다고 가정하므로 객체를 추가하는 add\(\) 메소드는 내부에서 어떤 비교도 하지 않고 항상 true를 반환
* 엔티티를 추가할 때 중복된 엔티티가 있는지 비교하지 않고 단순히 저장만 하면 된다. 엔티티를 추가해도 지연 로딩된 컬렉션을 초기화 하지 않는다.

#### **14.1.3 Set**

* Set 인터페이스는 HashSet으로 초기화. 중복을 허용하지 않으므로 add\(\) 할 때 마다 equals\(\)로 같은 객체가 있는지 반환. HashSet은 해시 알고리즘을 사용하므로 hashcode\(\)도 함께 사용해서 비교함
* 엔티티를 추가할 때 중복된 엔티티가 있는지 비교해야 한다. 엔티티를 추가할 때 지연 로딩된 컬렉션을 초기화한다.

#### **14.1.4 List + @OrderColumn**

* List 인터페이스에 `@OrderColumn`을 추가하면 순서가 있는 특수한 컬렉션으로 인식
* 순서가 있는 컬렉션은 데이터베이스의 순서 값도 함께 관리한다
* `@OrderColumn`를 사용하여 위치 값을 보관하면 단점이 많다. 그러므로 매핑하지 않고 직접 순서 값을 관리하거나 `@OrderBy`를 사용하기를 권장

**@OrderColumn의 단점**

* 테이블의 일대다 관계 특성상 위치 값이 지정한 엔티티의 필드가 아닌 다\(N\)쪽의 엔티티 테이블에 매핑될 수 있다. 따로 UPDATE SQL이 추가로 발생할 수 있다
* List를 변경하면 연관된 많은 위치 값을 변경해야 함
* 중간에 값이 없는 경우 해당 index에 null값이 보관되어 컬렉션 순회시 NullPointerException 발생할 수 있다

#### **14.1.5 @OrderBy**

* `@OrderColumn`은 데이터베이스에 순서용 컬럼을 매핑해서 관리했다면, `@OrderBy`는 데이터베이스의 order by 절을 사용해서 컬렉션을 정렬한다
* `@OrderBy`는 순서용 컬럼을 매핑하지 않아도 되며, 모든 컬렉션에 사용할 수 있다.
* `@OrderBy`의 값은 JPQL의 order by절처럼 엔티티의 필드를 대상으로 한다.
* 하이버네이트는 Set에 `@OrderBy`를 적용하면 내부에서 LinkedHashSet을 사용

### 14.2 @Converter

* 컨버터를 사용하면 엔티티의 데이터를 변환해서 데이터베이스에 저장할 수 있다
* 자바의 boolean 타입을 사용하고 싶지만 데이터베이스에는 Y or N 값으로 저장하고 싶을 때
* AttributeConverter 인터페이스에 엔티티의 데이터를 데이터베이스 컬럼에 저장할 데이터로 변환하는 메소드, 데이터베이스에서 조회한 컬럼 데이터를 엔티티의 데이터로 변환하는 메소드 두개를 구현해야 한다

```java
  // Member.java
  @Entity
  //@Convert(converter=BooleanToYNConverter.class, attributeName = "vip")
  public class Member {
    @Id
    private String id;
    private String username;

    @Convert(converter=BooleanToYNConverter.class)
    private boolean vip;

    // Getter, Setter

    ...
  }

  // BooleanToYNConverter.class
  @Converter
  public class BooleanToYNConverter implements AttibuteConverter<Boolean, String> {
    @Override
    public String convertToDatabaseColumn(Boolean attribute) {
        return (attribute != null && attribute) ? "Y" : "N";
    }

    @Override
    public Boolean convertToEntityAttribute(String dbData) {
        return "Y".equals(dbData);
    }
  }
```

#### **14.2.1 글로벌 설정**

* BooleanToYNConverter.java에 `@Converter(autuApply = true)` 글로벌 설정으로 옵션을 적용하면 모든 Boolean 타입에 대해 자동으로 컨버터가 적용된다
* `@Convert 속성 정리`

| 속성 | 기능 | 기본값 |
| :--- | :--- | :--- |
| converter | 사용할 컨버터를 지정 |  |
| attributeName | 컨버터를 적용할 필드를 지정 |  |
| disableConversion | 글로벌 컨버터나 상속 받은   컨버터를 사용하지 않음 | false |

### 14.3 리스너

* JPA 리스너 기능을 사용하면 엔티티의 생명주기에 따른 이벤트를 처리할 수 있

#### **14.3.1 이벤트 종류**

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FbusT6m%2FbtqEVWFTh4A%2FRK1EezAQpkUKOopYttxHL1%2Fimg.png)

* PostLoad : 엔티티가 영속성 컨텍스트에 조회된 직후 또는 refresh를 호출한 후\(2차 캐시에 저장되어 있어도 호출\)
* PrePersist : persist\(\) 메소드를 호출해서 엔티티를 영속성 컨텍스트에 관리하기 직전에 호출. 식별자 생성 전략을 사용한 경우 엔티티에 식별자는 아직 존재하지 않음. 새로운 인스턴스를 merge할 때도 수행
* PreUpdate : flush, commit을 호출해서 엔티티를 데이터베이스에 수정하기 직전에 호출.
* PreRemove : remove를 호출해서 엔티티를 영속성 컨텍스트에서 삭제하기 직전에 호출. 삭제 명령어로 영속성 전이가 일어날 때도 호출. orphanRemoval에 대해서는 flush나 commit시에 호출
* PostPersist : flush, commit을 호출해서 엔티티를 데이터베이스에 저장한 직후에 호출. 식별자가 항상 존재. 식별자 생성 전략이 IDENTITY면 식별자를 생성하기 위해 persist를 호출하면서 데이터베이스에 해당 엔티티를 저장하므로 persist를 호출한 직후에 바로 PostPersist가 호출
* PostUpdate : flush, commit을 호출해서 엔티티를 데이터베이스에 수정한 직후에 호출.
* PostRemove : flush, commit을 호출해서 엔티티를 데이터베이스에 삭제한 직후에 호출.

#### **14.3.2 이벤트 적용 위치**

* 엔티티에 직접 적용, 별도의 리스너 등록, 기본 리스너 사용

**엔티티에 직접 적용**

* 메소드마다 어노테이션을 추가하여 이벤트가 발생할 때마다 어노테이션으로 지정한 메소드를 실행

**별도의 리스너 등록**

* `@EntityListeners(DuckListener.class)`와 같이 클래스에 어노테이션을 추가하여 엔티티를 파라미터로 받을 수 있다. 반환 타입은 void로 설정해야 한다.

**기본 리스너 사용**

* 모든 엔티티의 이벤트를 처리하려면 META-INF/orm.xml에 기본 리스너로 등록

```markup
  // orm.xml
  <persistence-unit-metedata>
      <persistence-unit-defaults>
          <entity-listeners>
              <entity-listener class="jpabook.jpashop.domain.test.listener.DefaultListener"/>
          </entity-listeners>
      </persistence-unit-defaults>
  </persistence-unit-metedata>
```

* 기본 리스너 -&gt; 부모 클래스 리스너 -&gt; 리스너 -&gt; 엔티티 의 호출 순서

**더 세밀한 설정**

* javax.persistence.ExcludeDefaultListeners : 기본 리스너 무시
* javax.persistence.ExcludeSuperclassListeners : 상위 클래스 이벤트 리스너 무시
* 이벤트를 잘 활용하면 대부분의 엔티티에 공통으로 적용하는 등록 일자, 수정 일자 처리와 해당 엔티티를 누가 등록하고 수정했는지에 대한 기록을 리스너 하나로 처리할 수 있다

### 14.4 엔티티 그래프

* 엔티티 조회시 연관된 엔티티들을 함께 조회하려면 글로벌 fetch 옵션을 FetchType.EAGER로 설정하거나 JPQL에서 페치 조인을 사용
* 글로벌 fetch 옵션은 애플리케이션 전체에 영향을 주고 변경할 수 없는 단점이 있다
* 페치 조인을 사용하면 같은 JPQL을 중복해서 작성하는 경우가 많다
* 엔티티 그래프 기능을 사용하면 엔티티를 조회하는 시점에 함께 조회할 연관된 엔티티를 선택할 수 있다
* JPQL은 데이터를 조회하는 기능만 수행, 연관된 엔티티를 함께 조회하는 기능은 엔티티 그래프를 사용
* 정적으로 정의하는 Named 엔티티 그래프와 동적으로 정의하는 엔티티 그래프

#### **14.4.1 Named 엔티티 그래프**

* Named 엔티티 그래프는 `@NamedEntityGraph`로 정의
  * name : 엔티티 그래프의 이름을 정의
  * attributeNodes : 함께 조회할 속성 선택. `@NamedAttributeNode`를 사용하고 그 값으로 함께 조회할 속성을 선택. 
* 둘 이상 정의하려면 `@NamedEntityGraphs`를 사용

#### **14.4.2 em.find\(\)에서 엔티티 그래프 사용**

* Named 엔티티 그래프를 사용하려면 em.getEntityGraph\(\) 사용
* JPA 힌트 기능을 사용해서 힌트의 키로 javax.persistence.fetchgraph를 사용하고 힌트의 값으로 찾아온 엔티티 그래프를 사용

```java
  EntityGraph graph = em.getEntityGraph("Order.withMember");

  Map hints = new HashMap();
  hints.put("javax.persistence.fetchgraph", graph);

  Order order = em.find(Order.class, orderId, hints);
```

#### **14.4.3 subgraph**

* Order -&gt; OrderItem -&gt; Item 과 같은 연관구조에서는 subgraph 사용

```java
  @NamedEntityGraph(name = "Order.withAll", attributeNodes = {
                @NamedAttributeNode("member"),
                @NamedAttributeNode(value = "orderItems", subgraph = "orderItems")
                },
                subgraphs = @NamedSubgraph(name = "orderItems", attributeNodes = {
                    @NamedAttributeNode("item")
                })
  )

  @Entity
  @Table(name = "ORDERS")
  public class Order {
    ...
  }

  @Entity
  @Table(name = "ORDER_ITEM")
  public class OrderItem {
    ...
  }
```

```java
  Map hints = new HashMap();
  hints.put("javax.persistence.fetchgraph", em.getEntityGraph("Order.withAll"));

  Order order = em.find(Order.class, orderId, hints);
```

#### **14.4.4 JPQL에서 엔티티 그래프 사용**

* JPQL에서 엔티티 그래프를 사용하는 방법은 em.find\(\)와 동일하게 힌트만 추가한다

```java
  List<Order> resultList 
            = em.createQuery("select o from Order o where o.id = :orderId", Order.class)
                .setParameter("orderId", orderId)
                .setHint("javax.persistence.fetchgraph", em.getEntityGraph("Order.withAll"))
                .getResultList();
```

* JPQL에서 엔티티 그래프를 사용할 떄는 항상 SQL 외부 조인을 사용. 내부 조인을 사용하려면 내부 조인을 명시해주면 된다

#### **14.4.5 동적 엔티티 그래프**

* 엔티티 그래프를 동적으로 구성하려면 createEntityGraph\(\) 메소드 사용

```java
  EntityGraph<Order> graph = em.createEntityGraph(Order.class);
  graph.addAttributeNodes("member");

  Subgraph<OrderItem> orderItems = graph.addSubgraph("orderItems");
  orderItems.addAttributeNodes("item");

  Map hints = new HashMap();
  hints.put("javax.persistence.fetchgraph", graph);

  Order order = em.find(Order.class, orderId, hints);
```

#### **14.4.6 엔티티 그래프 정리**

* ROOT에서 시작
  * 엔티티 그래프는 항상 조회하는 엔티티의 ROOT에서 시작.
* 이미 로딩된 엔티티
  * 영속성 컨텍스트에 해당 엔티티가 이미 로딩되어 있으면 엔티티 그래프가 적용되지 않음
  * 조회하는 경우 처음 조회한 인스턴스가 반환
* fetchgraph, loadgraph
  * fetchgraph 속성은 엔티티 그래프에 선택한 속성만 함께 조회
  * loadgraph 속성은 엔티티 그래프에 선택한 속성뿐만 아니라 글로벌 fetch 모드가 FetchType.EAGER로 설정된 연관관계도 포함해서 함께 조

