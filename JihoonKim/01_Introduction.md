# Introduction

### SQL (Structured Query Language)

- Database에 명령어를 입력하기 위한 언어
- CRUD
  - Create & Insert data
  - Read & Select some data
  - Update data
  - Delete data

### 기본 용어

- Database: 한개 또는 여러개의 테이블을 가진 구조
- Relation (Table): Record와 Field를 가지는 구조
- Record (Tuple, Row): 필드들에 대한 단일 집합이며, 일반적으로 객체라고 부름
- Field (Attribute, Column): 객체들이 가지는 하나의 속성

<img src="./Images/01_01.png" width="600px">

### 주요 Database 시스템

- PostgreSQL: 완전 오픈 소스인 DB 시스템
- Oracle: 상업용 DB 시스템이며, 규모가 크고 기업에서 주로 이용
- MySQL: 상업적 오픈 소스인 DB 시스템이며, 빠르고 확장성이 좋음
- SqlServer: 마이크로소프트에서 개발한 관계형 DB 시스템

### PostgreSQL 시작하기 (Linux/Ubuntu)

- PostgreSQL 설치

  ```bash
  sudo apt install -y postgresql-common
  sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh

  sudo apt install -y postgresql
  ```

- postgres 계정 (superuser) 설정
  
  ```bash
  # 설정 파일에서 로그인 방식 수정
  sudo vi /etc/postgresql/17/main/pg_hba.conf
  
  # "local  all  postgres  peer"을 "local  all  postgres  md5"로 변경
  # "local  all  all  peer"을 "local  all  all  md5"로 변경

  # postgresql 재시작
  sudo systemctl restart postgresql

  # user를 postgres로 변경 후 psql 실행
  # 이 단계에서 설정한 적 없는 비밀번호를 요청 시 postgres의 md5를 trust로 임시 변경하고,
  # postgres의 비밀번호를 설정한 후 다시 md5로 변경
  sudo -i -u postgres
  psql

  # postgres의 비밀번호 설정
  # ALTER USER postgres PASSWORD 'New Password';

  # exit로 psql 종료 및 기본 유저로 변경 후 psql 실행 및 비밀번호 입력
  psql -U postgres
  ```

- 유저 생성 및 초기 데이터베이스 생성

  ```sql
  -- 'pg4e' 유저를 생성하고, 비밀번호를 '1234'로 설정
  CREATE USER pg4e WITH PASSWORD '1234';
 
  -- 'people' 데이터베이스를 생성하고, 'pg4e' 유저의 소유로 설정
  CREATE DATABASE people WITH OWNER 'pg4e'

  -- 세션 종료
  \q
  ```

- 생성한 유저 및 데이터베이스로 로그인

  ```bash
  # people 데이터베이스에 pg4e 유저로 로그인 및 사전 설정한 비밀번호 입력
  psql people pg4e
  ```

- 주의사항
  - 프롬프트가 `#`으로 표시되는 슈퍼 유저에서는 꼭 필요한 명령어만 실행
  - 프롬프트가 `=>`로 표시되는 일반 유저에서 DB 관리 진행