---
description: '''Node.js 교과서(조현영 저)'' 책 내용을 정리한 페이지입니다.'
---

# MongoDB란

### 1. NoSQL vs SQL

* MongoDB : 자바스크립트 문법을 사용하는 DB로, SQL을 사용하지 않는 NoSQL(Not only SQL)

SQL과 NoSQL 비교

| SQL(MySQL)      | NoSQL(MongoDB)    |
| --------------- | ----------------- |
| 규칙에 맞는 데이터 입력   | 자유로운 데이터 입력       |
| 테이블 간 JOIN 지원   | 컬렉션 간 JOIN 미지원    |
| 트랜잭션 지원         | 트랜잭션 미지원          |
| 안정성, 일관성        | 확장성, 가용성          |
| 용어(테이블, 로우, 컬럼) | 용어(컬렉션, 다큐먼트, 필드) |

### 2. MongoDB, Compass 설치하기

* MongoDB : Community ver. for Mac [https://docs.mongodb.com/manual/tutorial/install-mongodb-on-os-x/](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-os-x/)
* Compass : [https://www.mongodb.com/try/download/compass](https://www.mongodb.com/try/download/compass)&#x20;

### 3. 데이터베이스 및 컬렉션 생성하기&#x20;

* 데이터베이스 생성 : use \[데이터베이스명]
* 데이터베이스 목록 확인 : show dbs
* 컬렉션 생성 : db.createCollection('컬렉션명')
* 컬렉션 목록 확인 : show collection

### 4. CRUD 작업하기

#### [https://docs.mongodb.com/manual/crud/](https://docs.mongodb.com/manual/crud/)

####

