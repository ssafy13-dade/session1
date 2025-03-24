
# Single Table SQL

### Introduction

#### SQL (Structured Query Language)

- Database에 명령어를 입력하기 위한 언어
- CRUD
  - Create & Insert data
  - Read & Select some data
  - Update data
  - Delete data

#### 기본 용어

- Database: 한개 또는 여러개의 테이블을 가진 구조
- Relation (Table): Record와 Field를 가지는 구조
- Record (Tuple, Row): 필드들에 대한 단일 집합이며, 일반적으로 객체라고 부름
- Field (Attribute, Column): 객체들이 가지는 하나의 속성

<img src="./Images/01_01.png" width="600px">

#### 주요 Database 시스템

- PostgreSQL: 완전 오픈 소스인 DB 시스템
- Oracle: 상업용 DB 시스템이며, 규모가 크고 기업에서 주로 이용
- MySQL: 상업적 오픈 소스인 DB 시스템이며, 빠르고 확장성이 좋음
- SqlServer: 마이크로소프트에서 개발한 관계형 DB 시스템

### SQL Architecture

- 클라이언트에서 SQL 명령어로 서버에 요청하면, DB에서 데이터를 찾아 응답하는 방식으로 진행됨

#### 기초 SQL 문법

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

#### PostgreSQL의 명령어

- `\l`: 데이터베이스 목록 조회
- `\q`: 세션 종료
- `\d`: 현재 DB의 모든 객체(테이블, 뷰, 시퀀스 등) 조회
  - `\dt`: 현재 DB의 모든 테이블 조회
  - `\d+ [Object]`: 현재 DB의 모든 객체의 상세 정보 조회. Object 이름을 명시하면 해당 Object의 상세 정보를 조회
    - 상세 정보: 각 field의 이름, 타입, 제약 조건 및 객체의 크기, 저장된 데이터량 등

### Data Types

#### String Types (문자열)

- `CHAR (N)`: 길이가 N으로 고정된 문자열을 저장할 수 있는 필드
  - 해시 값처럼 고정된 길이를 가지는 문자열을 저장할 때 사용
  - 입력 값이 N보다 작은 경우 부족한 부분을 공백으로 채움
  - 입력 값이 N보다 큰 경우 오류 발생
- `VARCHAR (N)`: 길이가 N 이하인 문자열을 저장할 수 있는 필드
- 참고사항
  - PostgreSQL에서는 N의 제한이 없어 N을 생략하는 경우 Text Fields와 같은 역할을 수행
  - 다른 DBMS에서는 N의 제한이 있는 경우가 많음

#### Text (텍스트)

- `TEXT`: 여러 길이의 문자열을 저장할 수 있는 필드
  - `WHILE`문, 인덱싱, 정렬 등에서 사용하면 전체 테이블을 스캔하기 때문에 성능이 감소
    - `LIKE`문은 원래 전체 테이블을 스캔하기 때문에 관계 없음
- 참고사항
  - 사용하는 Character Set (UTF-8, latin1 등)에 따라 문자를 정렬하는 방식이 달라질 수 있음

#### Binary Type (이진수)

- `BYTEA`: 이진수 데이터를 저장할 수 있는 필드
- 이미지, 파일, 암호화된 데이터 등을 저장할 때 사용할 수 있지만, 잘 사용되지는 않음

#### Integer Numbers (정수)

- `SMALLINT`: 16비트 정수형(-2<sup>15</sup> ~ 2<sup>15</sup>, ±3만 정도)까지 저장할 수 있는 필드
- `INTEGER`: 32비트 정수형(-2<sup>31</sup> ~ 2<sup>31</sup>, ±20억 정도)까지 저장할 수 있는 필드
- `BIGINT`: 64비트 정수형(-2<sup>65</sup> ~ 2<sup>65</sup>)까지 저장할 수 있는 필드

#### Floating Point Numbers (부동 소수점)

- `REAL`: 32비트 부동소수점을 저장할 수 있는 필드
- `DOUBLE PRECISION`: 64비트 부동소수점을 저장할 수 있는 필드
- `NUMERIC (precision, scale)`: 정확한 계산을 위한 소수점을 저장할 수 있는 필드
  - Precision: 소수점을 포함한 전체 자릿수
  - Scale: 소수점 이하 자릿수
  - Precision과 Scale을 명시하지 않는 경우 시스템 메모리가 허용하는 한도까지 저장할 수 있음
- 컴퓨터에서 소수점 연산을 하는 경우 비트 제한에 따라 대략적인 값만 계산할 수 있음

#### Dates (날짜 및 시간)

- `TIMESTAMP`: `YYYY-MM-DD HH:MM:SS`를 저장할 수 있음
- `DATE`: `YYYY-MM-DD`를 저장할 수 있음
- `TIME`: `HH:MM:SS`를 저장할 수 있음
- `NOW`: PostgreSQL의 내장 함수이며, 현재 시간을 반환

### Index

- 데이터베이스에서 빠르게 데이터를 스캔하기 위한 기술
  - 엄청나게 많은 양의 데이터를 모두 탐색해 원하는 데이터를 찾는 것은 비효율적
  - 빠르게 원하는 데이터에 접근할 수 있는 방법을 제시

#### B-Tree

- 이진 탐색 트리의 확장형이며 자식 노드가 2개보다 많을 수 있는 자료 구조
  - 전체 데이터의 범위를 나눠 인덱스를 지정해두고, 하위 집합에 대해서도 같은 방법으로 인덱스를 설정
  - 조회하려는 데이터가 범위에 속하는지 판단하면서 빠르게 탐색할 수 있음
- 자료를 정렬된 상태로 보관하고 빠른 시간 내에 조회, 삽입, 삭제할 수 있어 DB에서 자주 이용됨

#### 해시

- 임의의 길이의 데이터를 고정된 길이의 데이터로 매핑해 빠르게 검색을 수행할 수 있는 자료 구조
- 데이터의 크기를 획기적으로 줄이지만, 정렬이 보장되지 않고 접두사 탐색이 불가능함
  - 정렬이 필요 없고 고유한 값을 찾아야 하는 Primary Key에 주로 이용됨