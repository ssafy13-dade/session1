# 학습 목표
1. 일반적인 `psql` 명령어.
2. CLI prompt를 통해 데이터 베이스 생성.
3. Common SQL commands
    e.g. INSERT INTO, WHERE, and ORDER BY

## 1. psql command
- \c + DB Name + Connection User
```SQL
-- db1 데이터베이스 인스턴스에 LWH 유저로 접속
\c db1 LWH
```
- \list: 전체 DB instancce 목록
- \dt: 접속한 DB instance의 table 목록 (relation = table)
- \ds: Sequence 목록
- \df: Function 목록
- \dv: View 목록
- \du: User 목록
- \d+: 기본 정보 + 스토리지 정보, 테이블 크기, 설명 등
- \dd: postgreSQL은 테이블, 컬럼, 뷰 등에 COMMENT를 달 수 있는데 이런 COMMENT를 출력해 줌.
- \q: 세션 종료

### 1-1. SERIAL과 SEQUENCE
- SERIAL
    - 자동 생성 + 단반향 연결
    - 특정 테이블에 속하게 되며, 다른 테이블과 `공유 불가`.
```SQL
-- SERIAL로 생성
CREATE TABLE users (
    id SERIAL PRIMARY KEY
);

-- SEQIEMCE를 OWNED BY 하는 것으로 이해할 수 있다.
-- OWNED BY란 시퀀스를 특정 컬럼 전용으로 하겠다는 것.
-- 아래는 시퀀스 user_id_seq를 users.id에 종속시킴.
ALTER SEQUENCE uers_id_seq OWNED BY users.id
```
- SEQUENCE
    - 테이블간 `공유`가 가능하다.
```SQL
CREATE SEQUENCE shared_seq;

CREATE TABLE table_a (
    id INTEGER DEFAULT nextval('shared_seq')
);

CREATE TABLE table_b (
    id INTEGER DEFAULT nextval('shared_seq')
);
```

## 2. INSERT INTO
- Table에 row를 `추가`한다.
```SQL
-- FORMAT
INSERT INTO table name (column1, column2, ...) VALUES (value1, value2, ...);
INSERT INTO table name (column1, column2, ...) VALUES (value1, value2, ...);

-- Sample
INSERT INTO table name (name, email, ...) VALUES ('Chunk', 'csev@umich.edu', ...);
INSERT INTO table name (name, email, ...) VALUES ('Somesh', 'somesh@umich.edu', ...);
```

## 3. DELETE
- Table에서 row를 `제거`한다.
- `WHERE`문을 사용하여 조건에 해당하는 행만 삭제할 수 있다.
- DELETE FROM은 `모든 행`이 삭제될 수 있다.
    - DELETE FROM 자체가 `반복`을 암묵저긍로 포함하고 있기 때문이다.
    -> 따라서 항상 WHERE문을 명확히 작성해야 한다.
```SQL
-- FORMAT
DELETE FROM table name WHERE column = value condition;

-- Caution!
-- 이 경우 WHERE문이 없기 때문에 모든 데이터가 삭제될 수 있다.
DELETE FROM table name;

-- Sample
DELETE FROM users WHERE email = 'ted@umich.edu';
```

## 4. UPDATE
- 특정 열의 값을 변경할 수 있다.
- DELETE와 마찬가지로 `WHERE`문을 명시하지 않으면 모든 행이 업데이트 되기에 주의해야 한다.
```SQL
-- FORMAT
UPDATE table name SET column = new_value WHERE column = condition;

-- Sample
UPDATE users SET name = 'Charles' WHERE email = 'csev@umich.edu';

-- CAUTION!
-- WHERE문이 없기 때문에 모든 값이 변경된다.
UPDATE users SET name = 'Charles'
```

## 5. SELECT
- `*`을 사용하면 모든 열을 조회할 수 있다.
    - 강의에서는 마치 for loop 처럼 동작한다는 듯하다.
- `Index`를 사용하면 속도를 올릴 수 있다고 하시는데 지금은 신경쓰지 말라신다.
```SQL
-- FORMAT
SELECT * | column name FROM table name;

-- Sample
SELECT * FROM users WHERE email = 'csev@umich.edu';
```

## 6. ORDER BY
- 데이터를 특정 열을 기준으로 `정렬`하여 조회한다.
    - ASC: 오름차순
    - DESC: 내림차순
```SQL
-- FORMATS
SELECT * FROM table name ORDER BY column;
```

## 7. LIKE
- 특정 `패턴`을 포함하는 데이터를 찾을 수 있다.
- 와일드 카드
    - %: 어떤 문자든 포함할 수 있다.
- %가 문자의 `앞과 뒤` 모두에 포함된다면 `전체 테이블`을 스캔하기에 속도가 느려진다.
    - B-Tree idx를 사용하는 WHERE보다 느려질 수 있다. (%가 앞에 붙으면 앞부분이 불명확하여 B-Tree를 사용할 수 없기 때문).
    -> 따라서 최적화를 고민한다면 %를 앞에 사용하지 않거나, WHERE 등 다른 방법을 생각해야 한다.
```SQL
-- FORMAT
SELECT * | columns FROM table name WHERE column LIKE 'pattern (e.g. %e%)';

-- Sample
SELECT * FROM users WHERE name LIKE '%e%';
```

## 8. LIMIT & OFFSET
- `LIMIT` 혹은 `OFFSET`을 사용하면 첫 'n' 개의 행, 혹은 몇 개의 행을 생략한 후 첫 'n' 개의 행을 조회할 수 있다.
- LIMIT과 OFFSET은 WHERE문과 ORDER BY문 이후에 수행된다.
    - 정확한 순서는 `WHERE(필터링) -> ORDER BY(정렬) -> LIMIT & OFFSET(페이징)` 이다.
    - 조회할 때 row으 수를 결정하기에 `최적화`에 도움이 된다.
- OFFSET의 시작은 `0` 번째임을 주의해야 한다.
    - 즉, 파이썬의 idx와 같다.
```SQL
-- 두 개의 행 조회
SELECT * FROM users ORDER BY email DESC LIMIT 2;

-- 첫 두 행을 제외한 뒤, 두 개의 행 조회
SELECT * FROM users ORDER BY email OFFSET 1 LIMIT 2;
```

## 9. COUNT
- `행의 개수`를 count 한다.
    - count(*): NULL을 포함.
    - count(column name): NULL을 미포함.
```SQL
-- Sample
SELECT COUNT(*) FROM users;
SELECT COUNT(*) FROM users WHERE email = 'csev@umich.edu';
```

## 10. Summary
```SQL
-- INSERT INTO
INSERT INTO users (name, eamil) VALUES ('Ted', 'ted@umich.edu');

-- DELETE FROM
DELETE FROM users WHERE email = 'ted@umich.edu';

-- UPDATE SET
UPDATE users SET name = 'Charles' WHERE email = 'csev@umich.edu';

-- SELECT FROM
SELECT * FROM users WHERE eamil = 'csev@umich.edu';

-- ORDER BY
SELECT * FROM users ORDER BY eamil;

-- LIKE
SELECT * FROM users WHERE name LIKE '%e%';

-- OFFSET & LIMIT
SELECT * FROM users ORDER BY email OFFSET 1 LIMIT 2;

-- COUNT
SELECT COUNT(*) FROM users WHERE email = 'csev@umich.edu';
```