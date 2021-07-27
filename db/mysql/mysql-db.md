---
description: 'linux cmd에서 DB 계정 생성, 스키마 생성, 권한 추가 방법'
---

# MySQL DB계정 생성 및 스키마 생성과 권한 추가하기

mysql 테이블에 대해 CRUD를 하기 위해서는 1\) 계정을 생성하고, 2\) 스키마\(database\)를 생성한 후 3\) 계정에 대해 스키마에 대한 권한을 추가해야 합니다. 이와 관련된 내용을 정리했습니다.

### 1. 계정 생성

1\) root 계정으로 접속 후 use mysql; 명령으로 mysql 스키마에 접속합니다.

```text
mysql -u root -p

MariaDB [(none)]> use mysql;
Database changed
```

2\) create user ~ 로 계정을 생성합니다. 계정 정보 입력시 host로 표시한 부분의 입력에 따라 외부에서 접근 가능 여부가 달라집니다. 서버가 아닌 외부 DB툴에서 해당 계정으로 DB에 접속하려면 %를 입력해주면 됩니다.

```text
MariaDB [mysql]> create user 'migration'@'%' identified by '#########';
                             ----------   -                -----------
                              계정이름    host               password
```

3\) user가 잘 생성되었는지 확인하려면 user 테이블을 조회하면 됩니다.

```text
MariaDB [mysql]> select user, host from user;
+-----------+-----------+
| user      | host      |
+-----------+-----------+
| migration | %         |
| root      | 127.0.0.1 |
| root      | ::1       |
| root      | localhost |
+-----------+-----------+
```

### 2. 스키마 생성

1\) root 계정으로 접속합니다.

```text
mysql -u root -p
```

2\) create database ~ 로 스키마를 생성합니다. default character set ~ 부분은 해당 스키마의 캐릭터셋을 설정하는 부분으로 없으면 기본 mysql의 설정을 따라갑니다. 저의 경우 기본 utf8 설정에 이모티콘과 같은 특수문자까지 저장할 수 있도록 utf8mb4 를 선택했습니다.

```text
MariaDB [(none)]> create database migration default character set utf8mb4; 
--> 데이터베이스 스키마 생성 & 기본 캐릭터셋 입력
Query OK, 1 row affected
```

3\) 스키마가 생성되었는지 확인하려면 show databases; 명령을 사용합니다.

```text
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| eztalk_hist        |
| information_schema |
| jmocha             |
| migration          |
| mysql              |
| performance_schema |
+--------------------+
```

### 3. 권한 추가하기

1\) root 계정으로 접속합니다.

```text
mysql -u root -p
```

2\) grant \[권한목록\] privileges on \[스키마명\].\[테이블명\] to \[계정명\]@\[host\]; 명령으로 권한을 추가합니다.

권한목록의 경우 select, update, delete, insert 권한을 하나씩 주거나 여러개 줄 수 있으며, 전체를 지정하려면 all 을 사용하면 됩니다.

스키마명과 테이블명은 전체를 사용시 \*를 사용하며, 하나에 대해 적용할 경우 해당하는 스키마 또는 테이블명을 적으면 됩니다.

```text
MariaDB [mysql]> grant all privileges on migration.* to 'migration'@'%';    
--> migration 계정에 migration 스키마 권한 부여
Query OK, 0 rows affected (0.000 sec)

MariaDB [mysql]> grant all privileges on *.* to 'migration'@'%';    
--> migration 계정에 대해 모든 스키마 권한 부여
Query OK, 0 rows affected (0.000 sec)
```

