# 📚 Introduction to SQL and Relational Databases

---

## 1. Relational Databases and PostgreSQL
[PG4E](https://www.pg4e.com)  

---

## 2. History of Databases
- **Random Access**  
  - 데이터에 랜덤하게 접근할 수 있을 때, 가장 효율적으로 데이터를 배치하는 방법 고민
  - 정렬이 항상 최선의 방법은 아님  

---

## 3. Relational Databases
- **Relational databases**는 데이터를 행(row)과 열(column)로 이루어진 **테이블**(table)에 저장
- 여러 테이블 간의 관계를 통해 데이터를 효율적으로 검색 가능  

**Reference:** [Relational Database](https://en.wikipedia.org/wiki/Relational_database)  

---

## 4. Structured Query Language (SQL)
- **SQL (Structured Query Language)**  
  - 데이터베이스 명령어 사용 언어
  - 미국 정부와 산업계의 협력으로 개발됨
  - **NIST (National Institute of Standards and Technology)** 주도

---

## 5. 주요 SQL 명령어
| Command | Description |
|---------|-------------|
| `CREATE` | 테이블 및 데이터 생성 |
| `INSERT` | 테이블에 데이터 삽입 |
| `SELECT` | 테이블에서 데이터 조회 |
| `UPDATE` | 테이블의 데이터 수정 |
| `DELETE` | 테이블의 데이터 삭제 |

---

## 6. Terminology
- **Database** – 하나 이상의 테이블을 포함  
- **Relation (Table)** – 튜플(tuple)과 속성(attribute)으로 구성된 집합  
- **Tuple (Row)** – 한 개의 객체에 대한 정보 (예: 사용자 정보)  
- **Attribute (Column)** – 하나의 데이터 속성  

---

## 7. 주요 데이터베이스 시스템
- **PostgreSQL** – 오픈 소스, 기능이 풍부함  
- **Oracle** – 대규모 상용 시스템, 엔터프라이즈급  
- **MySQL** – 빠르고 확장성이 뛰어남, 오픈 소스  
- **SQL Server** – 마이크로소프트 제공, 상용  

---

## 8. SQL 아키텍처
- **Database Server**  
- **PostgreSQL**  
- **pgAdmin** (웹 기반 관리 도구)  
- **psql** (명령어 기반 인터페이스)  

---

## 9. PostgreSQL 명령어 예제
### 🔹 사용자 생성 및 데이터베이스 생성
```sql
CREATE USER myuser WITH PASSWORD 'secret';
CREATE DATABASE people WITH OWNER 'myuser';
```

### 🔹 데이터베이스 연결
```bash
$ psql people
```

### 🔹 테이블 생성
```sql
CREATE TABLE users (
    name VARCHAR(128),
    email VARCHAR(128)
);
```

### 🔹 데이터 삽입
```sql
INSERT INTO users (name, email) VALUES ('Chuck', 'csev@umich.edu');
INSERT INTO users (name, email) VALUES ('Somesh', 'somesh@umich.edu');
```

### 🔹 데이터 조회
```sql
SELECT * FROM users;
SELECT * FROM users WHERE email = 'csev@umich.edu';
```

### 🔹 데이터 수정
```sql
UPDATE users SET name = 'Charles' WHERE email = 'csev@umich.edu';
```

### 🔹 데이터 삭제
```sql
DELETE FROM users WHERE email = 'ted@umich.edu';
```

---

## 10. SELECT 문과 다양한 옵션
### 🔹 정렬 (ORDER BY)
```sql
SELECT * FROM users ORDER BY email;
```

### 🔹 와일드카드 검색 (LIKE)
```sql
SELECT * FROM users WHERE name LIKE '%e%';
```

### 🔹 제한 및 오프셋 (LIMIT, OFFSET)
```sql
SELECT * FROM users ORDER BY email DESC LIMIT 2;
SELECT * FROM users ORDER BY email OFFSET 1 LIMIT 2;
```

### 🔹 행 수 카운트
```sql
SELECT COUNT(*) FROM users;
```

---

## 11. 데이터 타입
### 🔸 문자열 타입
- `CHAR(n)` – 고정된 길이 문자열
- `VARCHAR(n)` – 가변 길이 문자열
- `TEXT` – 매우 큰 문자열 저장 가능

### 🔸 숫자 타입
- `SMALLINT` – -32,768 ~ +32,767
- `INTEGER` – 약 -21억 ~ +21억
- `BIGINT` – 매우 큰 정수 저장 가능

### 🔸 실수 타입
- `REAL` – 32비트 실수
- `DOUBLE PRECISION` – 64비트 실수
- `NUMERIC(p, d)` – 정확한 소수점 계산

### 🔸 날짜 및 시간 타입
- `TIMESTAMP` – 날짜 및 시간 (`YYYY-MM-DD HH:MM:SS`)
- `DATE` – 날짜 (`YYYY-MM-DD`)
- `TIME` – 시간 (`HH:MM:SS`)
- `NOW()` – 현재 시간 반환

---

## 12. AUTO_INCREMENT (SERIAL)
- 기본 키(primary key) 자동 생성
```sql
CREATE TABLE users (
    id SERIAL,
    name VARCHAR(128),
    email VARCHAR(128) UNIQUE,
    PRIMARY KEY(id)
);
```

---

## 13. 인덱스 (Indexes)
- 대규모 테이블에서 검색 속도를 향상시키기 위해 사용  
- **B-Tree** – 삽입/삭제/탐색이 빠름  
- **Hash** – 특정 키 값으로 빠른 탐색 가능  

---

## 14. Summary
- SQL을 사용하면 다음과 같은 작업이 가능
  - 데이터 구조 정의
  - CRUD (생성, 읽기, 수정, 삭제) 작업 수행
  - 정렬, 필터링, 집계 등 다양한 쿼리 작성  

---

## 🎯 주요 SQL 예제 요약
```sql
-- SELECT
SELECT * FROM users WHERE email = 'csev@umich.edu';

-- UPDATE
UPDATE users SET name = 'Charles' WHERE email = 'csev@umich.edu';

-- INSERT
INSERT INTO users (name, email) VALUES ('Ted', 'ted@umich.edu');

-- DELETE
DELETE FROM users WHERE email = 'ted@umich.edu';

-- ORDER BY
SELECT * FROM users ORDER BY email;

-- LIKE
SELECT * FROM users WHERE name LIKE '%e%';

-- COUNT
SELECT COUNT(*) FROM users WHERE email = 'csev@umich.edu';

-- LIMIT, OFFSET
SELECT * FROM users ORDER BY email OFFSET 1 LIMIT 2;
```
