---
description: 이 글은 '처음 배우는 스프링 부트2' (김영재 저)' 책 내용을 정리한 글입니다.
---

# 스프링 부트 시작하기

### 1. 환경 프로퍼티 파일 설정하기

* 스프링 부트 프로퍼티 파일은 설정 관련 및 기타 정적인 값을 **키값 형식**으로 관리
* 기존에는 application.properties 파일을 사용했지만 표현의 한계로 YAML 파일을 많이 사용함
* 프로퍼티 설정값의 깊이에 따라 들여쓰기를 해서 계층 구조를 쉽게 파악 가능

  ```text
  // application.yml

  server:
    port: 80
  ```

#### YAML 파일 매핑하기

* @Value
  * yaml 파일에어 설정한 키값을 @Value의 프로퍼티값으로 주면 해당 값이 필드값에 할당. 주로 단일 필드값 가져오는데 사용
* @ConfigurationProperties
  * 루트 접두사\(root prefix\)를 활용하여 원하는 객체를 바인딩. @Value 보다 더 객체 지향적으로 프로퍼티 매핑
  * 사용시 해당 클래스에 @Component 선언을 해야 의존성 주입 가능
  * 기본 컬렉션 타입, POJO 타입 매핑을 제공

```text
  // application.yml

  property:
    test:
        name: property depth test
  fruit:
    list:
        - name: banana
          color: yellow      
```

```text
  @Value("${property.test.name}")
  private String propertyTestName;

  @Component
  @ConfigurationProperties("fruit")
  public class FruitProperty {
    private List<Map> list;
  }
```

### 2. 자동 환경 설정 이해하기

* 스프링 부트 자동 설정은 Web, H2, JDBC를 비롯한 100여개의 자동 설정을 제공
* 새로 추가되는 라이브러리\(JAR\)는 스프링 부트 자동 설정 의존성에 따라 설정이 자동 적용
* @SpringBootApplication 또는 @EnableAutoConfiguration + @Configuration 사용

#### 자동 환경 설정 어노테이션

* @SpringBootApplication = @SpringBootConfiguration + @EnableAutoConfiguration + @ComponentScan
  * @SpringBootConfiguration : 스프링 부트의 설정을 나타내는 어노테이션.
  * @EnableAutoConfiguration
    * 자동 설정의 핵심 어노테이션. 클래스 경로에 지정된 내용을 기반으로 설정 자동화 수행
  * @ComponentScan : 특정 패키지 경로를 기반으로 @Configuration에서 사용할 @Component 설정 클래스를 찾음. basePackage 프로퍼티값에 경로 미설정시 해당 어노테이션이 위치한 패키지가 루트 경로로 설정

#### @EnableAutoConfiguration 살펴보기

* @EnableAutoConfiguration 어노테이션 내부의 @Import\(AutoConfigurationImportSelector.class\)
  1. getCandidateConfigurations\(\) 메서드로 모든 후보 빈을 불러옴\(META-INF/spring.factories에 설정이 미리 정의되어 있음\)
  2. getExclusions\(\), removeDuplications\(\) 메서드로 제외할 설정과 중복된 설정을 제외
  3. 그 후 프로젝트에서 사용하는 빈만 임포트할 자동 설정 대상으로 선택

#### H2 Console 자동 설정 적용하기

* spring.h2.console.enabled의 기본값이 false이므로 true로만 변경해주면 사용 가능
* h2는 메모리 데이터베이스로 보통 테스트용으로 사용됨. 런타임 의존성으로 dependency 추가

```text
  // application.yml
  // h2 메모리DB 사용하기 위한 설정
  datasource:
    url: jdbc:h2:mem:testdb

  spring:
    h2:
        console:
            enabled: true
```

```text
  // build.gradle

  dependencies {
    ...
    runtime('com.h2database:h2')
    ...
  }
```

