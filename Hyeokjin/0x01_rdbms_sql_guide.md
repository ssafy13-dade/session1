# 1️⃣ 관계형 데이터베이스 개요

## **1.1 관계형 데이터베이스란?**

- **관계형 데이터베이스(RDBMS)** 는 데이터를 행(Row)과 열(Column)로 구성된 **테이블(Table)** 에 저장합니다.
- 여러 테이블 간의 관계를 활용하여 데이터를 효율적으로 관리할 수 있습니다.
- 대표적인 관계형 데이터베이스 시스템:
    - **PostgreSQL**: 오픈 소스, 기능이 풍부함.
    - **MySQL**: 빠르고 확장 가능, 상용 버전 존재.
    - **Oracle**: 대기업용, 강력한 성능과 보안 제공.
    - **SQL Server**: Microsoft 제품, 강력한 데이터 분석 기능 제공.
    

## **1.2 SQL(Structured Query Language)이란?**

- 관계형 데이터베이스와 상호작용하기 위한 **질의 언어(Query Language)** 입니다.
- 주요 기능:
    - **데이터 정의(DDL)**: 테이블 생성 및 스키마 변경 (`CREATE`, `ALTER`, `DROP`)
    - **데이터 조작(DML)**: 데이터 추가, 수정, 삭제 (`INSERT`, `UPDATE`, `DELETE`)
    - **데이터 질의(DQL)**: 데이터 조회 (`SELECT`)
    - **데이터 제어(DCL)**: 권한 관리 (`GRANT`, `REVOKE`)

---

# 2️⃣ SQL 기본 명령어

## **2.1 데이터베이스 생성 및 사용자 관리**

```sql
-- 사용자 생성
CREATE USER myuser WITH PASSWORD 'mypassword';

-- 데이터베이스 생성
CREATE DATABASE mydb WITH OWNER myuser;
```

## **2.2 테이블 생성**

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(128),
    email VARCHAR(128) UNIQUE
);
```

## **2.3 INSERT**

```sql
INSERT INTO users (name, email) 
VALUES ('Alice', 'alice@example.com');

INSERT INTO users (name, email) 
VALUES ('Bob', 'bob@example.com');
```

## **2.4 SELECT**

```sql
SELECT * FROM users;

SELECT name 
FROM users 
WHERE email = 'alice@example.com';
```

## **2.5 UPDATE**

```sql
UPDATE users 
SET name = 'Alicia' 
WHERE email = 'alice@example.com';
```

## **2.6 DELETE**

```sql
DELETE FROM users 
WHERE email = 'bob@example.com';
```

---

# 3️⃣ SQL 고급 기능

## **3.1 ORDER BY**

```sql
SELECT * FROM users 
ORDER BY name ASC;
```

## **3.2 LIKE Operator**

```sql
SELECT * FROM users
WHERE name LIKE '%a%';
```

## **3.3 LIMIT & OFFSET**

- **LIMIT**: 반환할 **최대 행(row) 개수를 제한**
- **OFFSET**: 결과에서 **건너뛸 행(row) 개수를 지정**

```sql
SELECT * FROM users 
ORDER BY name DESC LIMIT 5;

SELECT * FROM users 
ORDER BY name OFFSET 2 LIMIT 3;
```

## **3.4 COUNT**

```sql
SELECT COUNT(*) FROM users;
```

---

# 4️⃣ 데이터 타입

## **4.1 문자열 타입**

- `CHAR(n)`: 고정 길이 문자열.
- `VARCHAR(n)`: 가변 길이 문자열.
- `TEXT`: 긴 텍스트 저장 가능.

## **4.2 숫자 타입**

- `SMALLINT`: -32,768 ~ 32,767
- `INTEGER`: -2,147,483,648 ~ 2,147,483,647
- `BIGINT`: 매우 큰 정수 (최대 10^18)

## **4.3 실수 타입**

- `REAL`: 32비트 부동소수점 (7자리 정밀도)
- `DOUBLE PRECISION`: 64비트 부동소수점 (15자리 정밀도)
- `NUMERIC(p, s)`: 정확한 소수점 연산 가능

## **4.4 날짜 및 시간 타입**

- `DATE`: 날짜 (`YYYY-MM-DD` 형식)
- `TIME`: 시간 (`HH:MM:SS` 형식)
- `TIMESTAMP`: 날짜 + 시간 (`YYYY-MM-DD HH:MM:SS` 형식)
- `NOW()`: 현재 날짜와 시간 반환

---

# 5️⃣ 인덱스와 성능 최적화

## **5.1 기본 인덱스 생성**

```sql
CREATE INDEX idx_users_email ON users(email);
```

## **5.2 B-Tree 인덱스**

- 데이터를 정렬하고 빠르게 검색할 수 있도록 하는 **B-Tree** 구조.

## **5.3 해시(Hash) 인덱스**

- 특정 키 값을 빠르게 찾을 수 있도록 하는 해시 기반 인덱스.

```sql
CREATE INDEX idx_users_name ON users USING HASH (name);
```

---

# 6️⃣ SQL 요약 및 정리

## **6.1 주요 SQL 명령어 요약**

```sql
SELECT * FROM users WHERE email='alice@example.com';
UPDATE users SET name='Alicia' WHERE email='alice@example.com';
INSERT INTO users (name, email) VALUES ('Charlie', 'charlie@example.com');
DELETE FROM users WHERE email='charlie@example.com';
SELECT * FROM users ORDER BY email;
SELECT COUNT(*) FROM users WHERE email LIKE '%@example.com';
```

## **6.2 SQL의 강점**

- 데이터를 **체계적으로 관리**할 수 있음.
- 정렬, 검색, 필터링 기능 제공.
- 관계형 데이터베이스에서 **데이터 무결성 유지**에 중요한 역할 수행.