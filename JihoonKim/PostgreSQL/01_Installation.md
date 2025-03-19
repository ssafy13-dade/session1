# 설치 및 기본 세팅

- Linux/Ubuntu 환경에서 postgresql-common (17.4 버전) 설치

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

#### 주의사항

- 프롬프트가 `#`으로 표시되는 슈퍼 유저에서는 꼭 필요한 명령어만 실행
  - 데이터베이스 생성, 새로운 유저 생성, 유저 권한 변경 등
- 프롬프트가 `=>`로 표시되는 일반 유저에서 DB 관리 진행