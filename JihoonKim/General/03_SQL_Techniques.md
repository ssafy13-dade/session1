# SQL Techniques

### 테이블 변환

- `ALTER`: 테이블에 필드를 추가하거나, 삭제하거나, 데이터 타입 또는 제약 조건을 변경
  - 데이터 타입을 변경하는 경우, 기존의 데이터가 새로운 데이터 타입과 호환되지 않으면 변경이 거부됨
  - 제약 조건을 변경하는 경우, 기존의 데이터가 새로운 제약 조건을 만족하지 않으면 변경이 거부됨

  ```sql
  -- fav 테이블에서 oops 필드를 삭제
  ALTER TABLE fav DROP COLUMN oops;

  -- post 테이블의 content 필드의 데이터 타입을 TEXT로 변경
  ALTER TABLE post ALTER COLUMN content TYPE TEXT;

  -- fav 테이블에 INTEGER 데이터 타입을 가지는 howmuch 필드를 추가
  ALTER TABLE fav ADD COLUMN howmuch INTEGER;
  ```

### 날짜 및 시간

- `TIMESTAMPZ`: PostgreSQL의 날짜 및 시간 데이터 타입
  - UTC 기준으로 데이터를 변환해 저장하고, 조회할 때는 클라이언트의 시간대에 맞춰 변환
  - `NOW()`의 결과도 `TIMESTAMPZ` 타입

  ```sql
  -- 테이블 생성 시 TIMESTAMPZ로 데이터 타입 설정
  CREATE TABLE fav (
    id SERIAL PRIMARY KEY,
    created_at TIMESTAMPZ NOT NULL DEFAULT NOW()
  )

  -- 사용 가능한 시간대를 조회
  SELECT * FROM pg_timezone_names;

  ```

- `INTERVAL`: 날짜 및 시간 간격에 대한 데이터 타입
  - `-`, `AGE()` 등으로 날짜 및 시간 간격에 대해 연산하면 `INTERVAL`형식으로 반환
  - `INTERVAL` 키워드 뒤에 계산할 간격을 `INTERVAL` 형식으로 넣어 시간을 계산할 수 있음

  ```sql
  -- 두 날짜 및 시간 사이의 간격을 계산 / 결과: '1 day 2:00:00'
  SELECT AGE(TIMESTAMP '2025-03-18 12:00:00', TIMESTAMP '2025-03-17 10:00:00');

  -- 현재 날짜에서 3일 이후를 계산
  SELECT NOW() + INTERVAL '3 days';
  ```

- `DATE_TRUNC()`: `TIMESTAMP` 형식의 값을 특정 단위로 절단하는 함수
  - 특정 단위의 시작점을 반환
  - 단위: `year`, `month`, `hour`, `quarter`, `century` 등

  ```sql
  -- 연 단위로 절단 / 결과: '2025-01-01 00:00:00'
  SELECT DATE_TRUNC('year', TIMESTAMP '2025-03-18 15:45:10');

  -- 분 단위로 절단 / 결과: '2025-03-18 15:45:00'
  SELECT DATE_TRUNC('year', TIMESTAMP '2025-03-18 15:45:10');
  ```

### 캐스팅(Casting)

- `CAST()`: 데이터 타입을 변환하는 함수
  - 변환할 데이터 타입이 호환되지 않는 경우 오류가 발생함
  - PostgreSQL에서는 `::`로도 사용할 수 있음

  ```sql
  -- 문자열 '123'을 숫자 형식 123으로 변환
  SELECT CAST('123' AS INTEGER);

  -- TIMESTAMPZ 형식을 DATE 형식으로 변환
  SELECT NOW()::DATE;
  ```

### 조회 결과 압축

- `DISTINCT`: 레코드의 중복 값 중 하나만 표시
- `DISTINCT ON`: 레코드의 특정 필드가 동일한 중복 레코드 중 하나만 표시
  - 기본적으로 `ORDER BY`와 함께 사용해 어떤 레코드를 남길 지 선택
    - `ORDER BY`를 사용하지 않으면 실행 결과가 달라질 수도 있음
  
  ```sql
  -- users 테이블에서 중복된 레코드는 한 개만 조회
  SELECT DISTINCT * FROM users;

  -- users 테이블에서 중복된 이름은 한 개만 조회
  SELECT DISTINCT name FROM users;

  -- users 테이블을 name 오름차순, age 내림차순으로 정렬한 후 name 필드가 중복된 레코드는 한 개만 조회
  SELECT DISTINCT ON (name) name, age, email FROM users ORDER BY name, age DESC;
  ```

- `GROUP BY`: 조회 결과를 특정 필드를 기준으로 그룹화해 제공
  - 집계 함수인 `COUNT()`, `MAX()`, `AVG()` 등과 함께 사용되는 편
- `HAVING`: `GROUP BY`절에서 특정 조건을 지정할 때 사용
  - `WHERE`절은 반드시 `GROUP BY`절 앞에 와야 함
    - `WHERE`절이 `GROUP BY`절 앞에 오면서 레코드를 필터링 한 후 `GROUP BY`절에 넘겨주기 때문
  - 집계 함수 결과를 필터링하는 등 그룹화 이후 결과를 필터링하려면 `HAVING`절을 사용

### 서브쿼리

- 쿼리문 안에 쿼리문을 넣는 것으로, 테이블이나 특정 값을 쿼리문으로 대체해 사용할 수 있음
  - 서브쿼리가 실행된 후 그 결과를 바탕으로 메인쿼리가 실행됨

  ```sql
  -- 계정 아이디가 email이 'ed@umich.edu'인 계정의 아이디와 같은 레코드의 content를 조회
  SELECT content FROM comment
  WHERE account_id = (
    SELECT id FROM account
    WHERE email='ed@umich.edu'
  );
  ```

- 서브쿼리는 반드시 괄호로 감싸야 함
- 서브쿼리를 반복 실행되거나, 인덱스를 활용하지 못하는 경우 비효율적임

### 동시성(Concurrency)

- 데이터베이스는 여러 곳에서 SQL 명령어를 받아 순서대로 작업을 처리함
- 여러 명령어가 동시에 들어오는 경우 트랜잭션으로 명령어들을 묶어 처리하고, 그 동안 데이터 영역을 잠가 둠
  - 잠겨 있는 동안은 다른 명령어로 데이터에 접근할 수 없고, 앞선 명령어가 끝날때까지 기다림
    - 잠금 수준에 따라 다르지만, 일반적으로 변경 및 삭제가 불가능

### 트랜잭션(Transaction)

- 데이터베이스에서 하나의 논리적인 작업 단위
- 여러 개의 SQL 연산을 하나의 단위로 처리하는 것
- ACID 속성
  - Atomicity: 모두 성공하거나 모두 실패해야 함. 중간에 오류가 발생하면 전체 작업이 롤백됨
  - Consistency: 데이터 무결성이 보장됨. 트랜잭션 전후 데이터가 유효해야 함
  - Isolation: 동시에 실행되는 트랜잭션이 서로 영향을 주지 않음
  - Durability: 트랜잭션이 완료되면 변경 사항이 영구적으로 반영됨
- 사용 방법

  ```sql
  -- 트랜잭션 시작
  START TRANSACTION;

  -- 1번 계좌에서 2번 계좌로 입금하는 상황(동시에 업데이트되어야 함)
  UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
  UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;

  -- 트랜잭션을 종료하고 내용을 반영
  COMMIT;
  ```

  - `ROLLBACK`으로 트랜잭션을 취소할 수 있으며, 예외 발생시 또는 디버깅 중에 트랜잭션을 반영하지 않기 위해 사용

### 저장된 프로시저

- SQL문과 제어 흐름을 하나의 블록으로 저장해 재사용할 수 있도록 한 객체
- `CREATE PROCEDURE`로 생성하고, `CALL`로 사용
- 명시적으로 호출해야 사용할 수 있음

  ```sql
  -- acc_id와 amount를 받아 계좌의 balance를 update
  CREATE PROCEDURE AddBalance(IN acc_id INT, IN amount DECIMAL(10, 2))
  LANGUAGE plpgsql
  AS $$   -- $$는 PostgreSQL에서 함수나 프로시저의 본문을 정의할 때 사용하는 구분자
  BEGIN
    UPDATE accounts
    SET balance = balance + amount
    WHERE account_id = acc_id;
  END;
  $$;

  CALL AddBalance(1, 500.00);
  ```

### 트리거

- 특정 상황이 발생했을 때 자동으로 호출되는 함수
- `CREATE FUNCTION`으로 함수를 생성하고, `CREATE TRIGGER`로 트리거를 생성하면서 함수를 연결
  - 트리거 함수도 저장된 프로시저의 일종이지만, `CREATE FUNCTION`으로 생성해야하고, 자동 호출된다는 차이가 있음
  - 트리거를 선언할 때 트리거 함수를 호출하려면 `EXECUTE FUNCTION` 사용
    - PostgreSQL 11 이전 버전에서는 `EXECUTE PROCEDURE`를 사용했었음

  ```sql
  -- updated_at을 현재 시간으로 업데이트하는 트리거 함수
  CREATE OR REPLACE FUNCTION trigger_set_timestamp()
  RETURNS TRIGGER AS $$
  BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
  END;
  $$ LANGUAGE plpgsql;

  -- fav 테이블에 변경이 발생했을 때 해당 레코드에 트리거 함수 적용
  CREATE TRIGGER set_timestamp
  BEFORE UPDATE ON fav
  FOR EACH ROW
  EXECUTE FUNCTION trigger_set_timestamp();
  ```