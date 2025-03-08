# SQL Architecture

- 클라이언트에서 SQL 명령어로 서버에 요청하면, DB에서 데이터를 찾아 응답하는 방식으로 진행됨

### 기초 PostgreSQL 명령어

- `\l`: 데이터베이스 목록 조회
- `\q`: 세션 종료
- `\d`: 현재 DB의 모든 객체(테이블, 뷰, 시퀀스 등) 조회
  - `\dt`: 현재 DB의 모든 테이블 조회
  - `\d+ [Object]`: 현재 DB의 모든 객체의 상세 정보 조회. Object 이름을 명시하면 해당 Object의 상세 정보를 조회
    - 상세 정보: 각 field의 이름, 타입, 제약 조건 및 객체의 크기, 저장된 데이터량 등
- `CREATE`: DB나 테이블을 생성
  
  ```sql
  CREATE DATABASE people    -- people DB 생성

  CREATE TABLE users (      -- users 테이블 생성
    name VARCHAR (128),     -- name 필드는 최대 128자의 문자열을 저장
    email VARCHAR (128)     -- email 필드는 최대 128자의 문자열을 저장
  );
  ```

- `INSERT`: 테이블에 새로운 Record를 추가

  ```sql
  -- users 테이블에 name이 Chuck, email이 csev@umich.edu인 Record 추가
  INSERT INTO users (name, email) VALUES ('Chuck', 'csev@umich.edu');
  ```

- `DELETE`: Record를 삭제

  ```sql
  -- (주의!) users 테이블의 모든 Record 삭제
  DELETE FROM users;

  -- users 테이블의 email이 ted@umich.edu인 Record 삭제
  DELETE FROM users WHERE email='ted@umich.edu';
  ```

- `UPDATE`: Record의 값을 변경

  ```sql
  -- (주의!) users 테이블의 모든 Record의 name을 Charles로 변경
  UPDATE users SET name='Charles'

  -- users 테이블의 email이 csev@umich.edu인 Record의 name을 Charles로 변경
  UPDATE users SET name='Charles' WHERE email='csev@umich.edu'
  ```

- `SELECT`: 테이블에서 특정 Record의 특정 Field를 조회

  ```sql
  -- users 테이블의 모든 Record의 모든 Field를 조회
  SELECT * FROM users;

  -- users 테이블 중 email이 csev@umich.edu인 Record의 모든 Field를 조회
  SELECT * FROM users WHERE email = 'csev@umich.edu';

  -- users 테이블의 모든 Record의 name Field를 조회
  SELECT name FROM users;
  ```

- `ORDER BY`: `SELECT`문과 함께 쓰이며, `SELECT`문의 결과를 특정 Field를 기준으로 정렬

  ```sql
  -- users 테이블의 모든 Record를 email을 기준으로 오름차순 정렬
  SELECT * FROM users ORDER BY email;
  ```

- `LIKE`: `WHERE`문과 함께 쓰이며, 특정 Field의 값이 와일드카드와 일치하는지 확인

  ```sql
  -- users 테이블 중 name에 e가 들어가는 Record의 모든 Field를 조회
  SELECT * FROM users WHERE name LIKE '%e%';
  ```

  - SQL에서 사용하는 와일드카드

    |와일드카드|의미|예시|
    |:-:|:-:|:-:|
    |_ (언더스코어)|1개 문자와 일치|`J_n` => `Jon`, `Jan`|
    |%|0개 이상 문자와 일치|`J%` => `J`, `Jon`, `Jack`|
  
  - `LIKE`가 없는 `WHERE`문은 인덱스를 활용해 빠르게 목표 Record를 탐색
  - `LIKE`를 통해 특정 와일드카드와 일치하는지 확인할 때 인덱스가 없으면 전체 테이블을 탐색해야 함

- `LIMIT`: `SELECT`문과 함께 쓰이며, 조회 결과의 개수를 제한

  ```sql
  -- users 테이블의 위에서 2개 Record를 조회
  SELECT * FROM users LIMIT 2;
  ```

- `OFFSET`: `SELECT`문과 함께 쓰이며, 특정 차례의 Record부터 조회
  - 0번부터 시작

  ```sql
  -- 1번 Record (위에서 2번째)부터 조회
  SELECT * FROM users OFFSET 1;
  ```

- `COUNT`: `SELECT`문과 함께 사용되는 함수이며, Record의 개수를 반환

  ```sql
  -- users 테이블의 모든 Record 개수를 반환
  SELECT COUNT(*) FROM users;

  -- users 테이블의 email이 csev@umich.edu인 Record의 개수를 반환
  SELECT COUNT(*) FROM users WHERE email='csev@umich.edu';
  ```