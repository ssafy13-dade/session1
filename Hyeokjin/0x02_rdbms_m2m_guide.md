# 1️⃣ 관계형 데이터베이스 개요

## **1.1 관계형 데이터베이스 설계**

- 관계형 데이터베이스(RDBMS)는 데이터를 테이블의 행과 열로 구성하여 저장합니다.
- 데이터베이스 설계의 목표:
    - **중복 최소화**
    - **데이터 무결성 보장**
- 설계 과정:
    - 개념적 모델(ERD - 엔터티 관계 다이어그램)
    - 논리적 모델
    - 물리적 모델

## **1.2 데이터베이스 설계 핵심 개념**

- **기본 키(Primary Key)**: 테이블에서 각 행을 유일하게 식별하는 값.
- **논리 키(Logical Key)**: 외부 애플리케이션이 조회하는 키.
- **외래 키(Foreign Key)**: 다른 테이블의 기본 키를 참조하는 키.

---

# 2️⃣ 데이터베이스 정규화

## **2.1 정규화(Normalization) 개념**

- **제1 정규형(1NF)**: 중복 컬럼 제거, 데이터 원자성 유지.
- **제2 정규형(2NF)**: 부분 함수 종속 제거 (모든 속성이 기본 키 전체에 의존해야 함).
- **제3 정규형(3NF)**: 이행적 함수 종속 제거 (속성들이 기본 키에만 의존해야 함).
- **BCNF(Boyce-Codd Normal Form)**: 후보 키가 아닌 속성이 결정자가 되지 않도록 개선.

---

# 3️⃣ 다대다(Many-to-Many) 관계

## **3.1 다대다 관계란?**

- 하나의 엔터티가 여러 개의 다른 엔터티와 관계를 가질 수 있는 경우.
- 예제: `학생(Student)`은 여러 개의 `강의(Course)`를 수강할 수 있으며, `강의`도 여러 `학생`을 가질 수 있음.

## **3.2 다대다 관계 구현**

- **연결 테이블(Junction Table, Bridge Table)** 사용하여 해결.

### **예제 스키마**

```sql
CREATE TABLE student (
  id SERIAL PRIMARY KEY,
  name VARCHAR(128),
  email VARCHAR(128) UNIQUE
);

CREATE TABLE course (
  id SERIAL PRIMARY KEY,
  title VARCHAR(128) UNIQUE
);

CREATE TABLE member (
  student_id INTEGER REFERENCES student(id) ON DELETE CASCADE,
  course_id  INTEGER REFERENCES course(id) ON DELETE CASCADE,
  role INTEGER,
  PRIMARY KEY (student_id, course_id)
);
```

## **3.3 다대다 관계 데이터 조회**

### **데이터 삽입**

```sql
INSERT INTO student (name, email) VALUES ('Jane', 'jane@example.com');
INSERT INTO course (title) VALUES ('Python');
INSERT INTO member (student_id, course_id, role) VALUES (1, 1, 1);
```

### **JOIN을 이용한 데이터 조회**

```sql
SELECT student.name, course.title, member.role
FROM student
JOIN member ON member.student_id = student.id
JOIN course ON member.course_id = course.id;
```

---

# 4️⃣ JOIN을 활용한 데이터 조회

## **4.1 INNER JOIN**

```sql
SELECT album.title, artist.name
FROM album
JOIN artist ON album.artist_id = artist.id;
```

## **4.2 CROSS JOIN**

**= Cartesian Product**

```sql
SELECT track.title, genre.name
FROM track CROSS JOIN genre;
```

## **4.3 다중 JOIN**

```sql
SELECT track.title, artist.name, album.title, genre.name
FROM track
JOIN genre ON track.genre_id = genre.id
JOIN album ON track.album_id = album.id
JOIN artist ON album.artist_id = artist.id;
```

---

# 5️⃣ ON DELETE CASCADE 및 참조 무결성

## **5.1 ON DELETE CASCADE**

- 참조된 레코드가 삭제되면 연결된 레코드도 함께 삭제됩니다.

```sql
DELETE FROM genre 
WHERE name='Metal';
```

## **5.2 외래 키와 삭제 옵션**

- **RESTRICT**: 부모 키 삭제를 제한.
- **CASCADE**: 부모 키 삭제 시 자식 테이블의 연관 데이터도 삭제.
- **SET NULL**: 부모 키 삭제 시 자식 테이블의 외래 키 값을 NULL로 설정.

---

# 6️⃣ 고급 SQL 기능

## **6.1 JSONB**

- `JSONB`는 **PostgreSQL에서 제공하는 JSON 데이터 타입** 중 하나로, **이진(binary) 형태로 저장되는 JSON**이다.
- 일반 `JSON` 타입과 달리, `JSONB`는 내부적으로 **인덱싱이 가능**하고, **검색 속도가 빠르며**, **중복된 키를 제거**해준다.
- 즉, 단순한 JSON 데이터보다 훨씬 강력한 기능을 제공한다.

### JSONB 연산자 정리

| 연산자 | 설명 | 예제 |
| --- | --- | --- |
| `->` | JSON 필드에서 키를 선택 (JSON 형식 유지) | data -> ‘name’ |
| `->>` | JSON 필드에서 키를 선택 (문자열로 변환) | date ->> ‘name’ |
| `#>` | 중첩된 JSON 배열 및 객체에 접근 (JSON 형식 유지) | data #> '{address, city}’ |
| `#>>` | 중첩된 JSON 배열 및 객체에 접근 (문자열로 변환) | data #>> '{address, city}’ |
| `@>` | JSONB 값이 특정 JSON을 포함하는지 검사 | data @> '{"age": 30}’ |
| `<@` | 특정 JSON이 JSONB 값에 포함되는지 검사 | '{"age": 30}' <@ data |
| `?` | JSONB 키가 존재하는지 검사 | data ? 'age’ |
| `?&` | 모든 키가 존재하는지 검사 | data ?& array['age', 'name'] |

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    data JSONB
);

INSERT INTO users (data) 
VALUES ('{"name": "Ham", "age": 30}');

SELECT data->>'name' 
FROM users
WHERE data->>'age' = '30';
```

### JSONB 인덱싱

- `JSONB` 필드에서 검색 성능을 높이려면 **GIN(Generalized Inverted Index) 인덱스**를 사용하는 것이 좋음.

```sql
CREATE INDEX idx_users_data ON users USING GIN (data);
```

- 위 인덱스를 생성하면 `@>`, `?`, `?|`, `?&` 등의 연산자의 속도가 개선됨.

### **JSONB 활용 예제**

**1) 특정 키의 값 조회**

```sql
SELECT data->>'name' FROM users;
```

- `data` JSONB 필드에서 `"name"` 값을 문자열로 반환.

**2) 특정 JSON 값이 포함된 데이터 조회**

```sql
SELECT * FROM users WHERE data @> '{"age": 30}';
```

- `data` 필드에서 `"age": 30`을 포함하는 레코드 검색.

**3) 특정 키가 존재하는지 확인**

```sql
SELECT * FROM users WHERE data ? 'age';
```

- `data` 필드에 `"age"` 키가 존재하는 레코드 검색.

**4) 여러 개의 키 중 하나라도 존재하는 레코드 검색**

```sql
SELECT * FROM users WHERE data ?| array['age', 'name'];
```

- `"age"` 또는 `"name"` 키가 존재하는 레코드 검색.

**5) 중첩된 JSON 데이터에서 값 검색**

```sql
SELECT data#>>'{address, city}' FROM users;
```

- `data` 안에 있는 `"address"` 객체의 `"city"` 값을 문자열로 반환.

## **6.2 공통 테이블 표현식 (CTE)**

- **CTE**는 SQL 쿼리 내에서 **임시적인 결과 집합을 저장하고 재사용할 수 있도록 하는 기능**이다.
- `WITH` 키워드를 사용하여 **하위 쿼리를 정의한 뒤**, 해당 결과를 마치 테이블처럼 사용할 수 있다.
- 서브쿼리보다 **가독성이 좋고**, **복잡한 SQL을 더 쉽게 관리**할 수 있도록 돕는다.

```sql
WITH recent_users AS (
    SELECT * FROM users WHERE created_at > now() - interval '7 days'
)

SELECT * FROM recent_users;
```

**CTE vs 서브쿼리**

| 비교 항목 | CTE | 서브쿼리 |
| --- | --- | --- |
| **가독성** | 가독성이 뛰어남 | 복잡한 경우 이해하기 어려움 |
| **재사용성** | 여러 번 사용할 수 있음 | 한 번만 사용 가능 |
| **퍼포먼스** | 최적화될 가능성이 높음 | 실행마다 다시 계산됨 |
| **재귀 지원** | 가능 (`WITH RECURSIVE`) | 불가능 |

## **6.3 윈도우 함수**

```sql
SELECT name, department, salary,
       RANK() OVER (PARTITION BY department ORDER BY salary DESC) as rank
FROM employees;
```

| 함수 | 설명 |
| --- | --- |
| **RANK()** | 동일한 값이 있으면 같은 순위를 부여하고, 다음 순위는 건너뜀 |
| **DENSE_RANK()** | 동일한 값이 있어도 다음 순위를 건너뛰지 않음 |
| **ROW_NUMBER()** | 동일한 값이 있어도 고유한 순위를 부여 |
| **SUM()** | 윈도우 내 누적합 계산 |
| **AVG()** | 윈도우 내 평균 계산 |
| **COUNT()** | 윈도우 내 행 개수 계산 |
| **MIN()** | 윈도우 내 최소값 계산 |
| **MAX()** | 윈도우 내 최대값 계산 |
| **LAG()** | 이전 행의 값을 가져옴 |
| **LEAD()** | 다음 행의 값을 가져옴 |
| **FIRST_VALUE()** | 윈도우에서 첫 번째 값 반환 |
| **LAST_VALUE()** | 윈도우에서 마지막 값 반환 |
| **NTILE(n)** | 윈도우를 `n`개 그룹으로 나누고 그룹 번호 부여 |

## **6.4 트랜잭션 관리**

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

| 명령어 | 설명 |
| --- | --- |
| **BEGIN** | 트랜잭션 시작 |
| **COMMIT** | 트랜잭션을 정상적으로 종료하고 변경 사항을 반영 |
| **ROLLBACK** | 트랜잭션을 취소하고 변경 사항을 되돌림 |
| **SAVEPOINT** | 트랜잭션 내 특정 지점을 저장하여 부분 롤백 가능 |
| **RELEASE SAVEPOINT** | 저장한 지점을 제거 |

---

# 7️⃣ 정리

- **관계형 데이터베이스**는 데이터를 구조적으로 관리하여 무결성을 보장
- **정규화**를 통해 중복을 제거하고 효율성을 향상
- **다대다 관계**는 **연결 테이블**을 사용하여 구현
- **JOIN 연산**을 통해 여러 테이블을 효과적으로 조회
- **고급 SQL 기능(JSONB, CTE, 윈도우 함수, 트랜잭션)** 을 활용하면 강력한 데이터 처리가 가능