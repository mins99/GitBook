---
description: 이 글은 '처음 배우는 스프링 부트2' (김영재 저)' 책 내용을 정리한 글입니다.
---

# 스프링 부트 배치

## 1. Spring Boot BATCH?

* batch : 프로그램의 흐름에 따라 순차적으로 자료를 정리한다. 배치 처리란 일괄 처리를 의미함. 회원 중 장기 미접속자에 대해 일괄적으로 휴면 처리를 하는 등의 처리가 필요할 때 사용
* Spring boot에서의 배치 : 스프링 배치의 설정 요소들을 간편화시켜 스프링 배치를 빠르게 설정하는 데 도움을 줌
* 장점
  * 대용량 데이터 처리에 최적화 되어 고성능 발휘
  * 효과적인 로깅, 통계 처리, 트랜잭션 관리 등 재사용 가능한 기능 지원
  * 수동으로 처리하지 않도록 자동화
  * 예외사항과 비정상 동작에 대한 방어 가능
  * 스프링 부트 배치를 이해하면 비즈니스 로직에만 집중 가능
* 주의사항
  * 단순화하여 복잡한 구조와 로직은 피해야 함
  * 데이터 직접 접근이 잦으므로 데이터 무결성을 유지하는 유효성 검사 필요
  * 배치 처리시 시스템 I/O 사용을 최소화 해야 함. 잦은 I/O로 DB 커넥션과 네트워크 비용이 커져 성능에 영향을 줄 수 있다
  * 배치 처리시 같은 서비스에 사용되는 웹, API, 기타 프로젝트들이 서로 영향을 줄 수 있으므로 주의해야 함
  * 스프링 부트 배치는 스케줄러를 제공하지 않음 → 스케줄링 기능은 다른 기능을 사용해야 함

## 2. 스프링 부트 배치 이해하기

* 배치 처리 절차\(4.0.0.RELEASE\)
  * 읽기\(read\) : 데이터 저장소에서 특정 데이터 레코드를 읽음
  * 처리\(processing\) : 원하는 방식으로 데이터를 가공/처리
  * 쓰기\(write\) : 수정된 데이터를 다시 저장소에 저장

![Job&#xC774;&#xB77C;&#xB294; &#xD558;&#xB098;&#xC758; &#xD070; &#xC77C;&#xAC10;\(Job\)&#xC5D0; &#xC5EC;&#xB7EC; &#xB2E8;&#xACC4;\(Step\)&#xB97C; &#xB450;&#xACE0;, &#xAC01; &#xB2E8;&#xACC4;&#xB97C; &#xBC30;&#xCE58;&#xC758; &#xAE30;&#xBCF8; &#xD750;&#xB984;&#xB300;&#xB85C; &#xAD6C;&#xD604;](../../.gitbook/assets/image.png)

### 1. Job

* 배치 처리 과정을 하나의 단위로 표현한 객체. 전체 배치 처리에 있어 최상단 계층에 존재. 하나의 Job 객체가 여러개의 Step 인스턴스를 포함
* JobBuilderFactory의 get\(\) 메서드로 JobBuilder를 생성 및 이용.
* 새로운 JobBuilder 생성마다 JobBuilderFactory가 생성될 때 주입받은 JobRepository를 사용하도록 하여 모든 JobBuilder가 동일한 리포지토리를 사용하도록 함

```java
@Autowired
private JobBuilderFactory jobBuilderFactory;

@Bean
public Job simpleJob() {
    return jobBuilderFactory.get("simpleJob").start(simpleStep()).build();
}
```

* "simpleJob" 이라는 이름의 JobBuilder 생성 → \(simpleStep이라는 간단한 Step 인스턴스를 생성하여 반환하는 메서드를 매개변수로 사용\) → JobBuilder의 start 메서드로 SimpleJobBuilder를 리턴받고 → SimpleJobBuilder의 build 메서드를 호출하여 "simpleJob" 이라는 이름의 Job이 생성되어 반환 됨

### 1.1 JobInstance

* 배치에서 Job이 실행될 때 하나의 Job 실행 단위. 어제와 오늘 실행한 각각의 Job을 JobInstance들이라고 할 수 있다.

### 1.2 JobExecution

* JobInstance에 대한 한 번의 실행을 나타내는 객체. 어제 실패하고 오늘 성공하면 어제와 오늘의 JobExecution이 동일하지만, 오늘 성공하고 내일 성공하면 오늘과 내일의 JobExecution은 다른 것. Job 실행에 대한 정보\(JobInstance, 배치 실행 상태, 시작 시간, 끝난 시간, 실패 메시지 등의 정보\)를 담고 있다

### 1.3 JobParameters

* Job이 실행될 때 필요한 파라미터들을 Map 타입으로 저장하는 객체. JobInstance를 구분하는 기준이 되기도 함

### 2. Step

* 실질적인 배치 처리를 정의하고 제어하는 데 필요한 모든 정보가 들어 있는 도메인 객체. Job을 처리하는 실질적인 단위로 모든 Job에는 하나 이상의 Step이 있어야 함.

### 2.1 StepExecution

* Step의 실행 정보를 담는 객체로 Step이 실행될 때마다 StepExecution이 생성됨

### 3.1 ItemReader

* Step의 대상이 되는 배치 데이터를 읽어오는 인터페이스. File, XML, DB등 여러 타입의 데이터를 읽어올 수 있음

### 3.2 ItemProcessor

* ItemReader로 읽어온 배치 데이터를 변환하는 역할.
* ItemWriter는 저장만 수행, ItemProcessor는 로직 처리만 수행하여 비즈니스 로직을 명확하게 분리
* 배치 데이터와 쓰여질 데이터의 타입이 다를 경우에 대응이 가능함

### 3.3 ItemWriter

* 배치 데이터를 DB나 File로 저장

