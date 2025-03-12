# 1️⃣ 텍스트 데이터 생성 및 성능 테스트

PostgreSQL에서는 **repeat()** 함수로 긴 문자열을 생성하고, **generate_series()**로 다량의 행 생성 가능

```sql
-- 특정 문자열을 반복 생성 (가로 확장)
SELECT repeat('Neon ', 5);
-- 결과: 'Neon Neon Neon Neon Neon'

-- 연속된 숫자 생성 (세로 확장)
SELECT generate_series(1, 5);
-- 결과:
-- 1
-- 2
-- 3
-- 4
-- 5
```

**랜덤 숫자 생성 예제**

```sql
SELECT random(), random(), trunc(random() * 100);
```

- `random()`: 0~1 사이의 난수를 생성
- `trunc(random() * 100)`: 0~100 사이의 정수 생성

---

# 2️⃣ 문자열 관련 함수

## **대소문자 변환**

```sql
SELECT UPPER('hello world'); -- 'HELLO WORLD'
SELECT LOWER('HELLO WORLD'); -- 'hello world'
```

## **부분 문자열 추출**

```sql
SELECT LEFT('PostgreSQL', 4);  -- 'Post'
SELECT RIGHT('PostgreSQL', 4); -- 'SQL'
SELECT SUBSTRING('PostgreSQL' FROM 5 FOR 3); -- 'gre'
```

## **문자열 포함 여부 (위치 검색)**

```sql
SELECT strpos('PostgreSQL', 'SQL'); -- 7 (SQL이 시작하는 위치)
```

## **문자열 대체**

```sql
SELECT REPLACE('Hello World', 'World', 'PostgreSQL');
-- 결과: 'Hello PostgreSQL'
```

---

# 3️⃣ LIKE 및 정규 표현식 활용

## **LIKE 연산자**

```sql
SELECT * FROM textfun WHERE content LIKE '%example%';
-- 'example'이 포함된 문자열 검색
```

## **정규 표현식 활용**

```sql
SELECT * FROM textfun WHERE content ~ 'regex_pattern';
```

- `~` : 정규 표현식 패턴과 일치하는 값 찾기
- `~*` : 대소문자 구분 없이 검색
- `!~` : 정규 표현식과 일치하지 않는 값 찾기

**예제: 이메일 형식 검증**

```sql
SELECT * FROM email_table WHERE email ~ '^[a-z0-9._%+-]+@[a-z0-9.-]+\\.[a-z]{2,}$';
```

---

# 4️⃣ 텍스트 검색 최적화

## **B-Tree 인덱스**

```sql
CREATE INDEX textfun_b ON textfun(content);
```

## **EXPLAIN ANALYZE**

- **SQL 쿼리의 실행 계획을 분석하고 성능을 최적화하는 데 사용**
- 실행 경로를 보여주며, **실제 실행 시간, 사용된 인덱스, 비용(Cost) 등을 출력**
- **쿼리 최적화 및 인덱스 튜닝에 필수적인 도구**

```sql
EXPLAIN ANALYZE SELECT content FROM textfun WHERE content LIKE 'example%';
```

**① `EXPLAIN ANALYZE`**

- 쿼리를 실제 실행하고 **성능 분석 결과를 출력**

**② `SELECT content FROM textfun WHERE content LIKE 'example%';`**

- `textfun` 테이블에서 **`content` 필드가 'example'로 시작하는 값 검색**

## **인덱스를 활용한 검색**

```sql
EXPLAIN ANALYZE
SELECT content FROM textfun WHERE content IN ('example1', 'example2');
```

---

# 5️⃣ 해싱과 인덱스

## **MD5 해시**

> Message-Digest algorithm 5
> 

```sql
SELECT md5('hello');
-- 결과: 5d41402abc4b2a76b9719d911017c592

CREATE INDEX textfun_md5 ON textfun (md5(content));
```

## **UUID**

> Universally Unique Identifier
: 고유한 식별자를 생성하는 128비트 값
> 
- **충돌 가능성이 극히 낮아** **데이터베이스의 기본 키(PK), 트랜잭션 ID, API 키 등**에 활용됨
- PostgreSQL에서 `UUID`를 **데이터 타입으로 지원**하며, 다양한 방법으로 생성 가능

```sql
CREATE TABLE cr3 (
    id SERIAL,
    url TEXT,
    url_md5 UUID UNIQUE,
    content TEXT
);

UPDATE cr3 SET url_md5 = md5(url)::UUID;
```

**`url_md5 UUID UNIQUE`**

- `url`의 **MD5 해시값을 `UUID` 형식으로 변환하여 저장**.
- `UNIQUE` 제약 조건을 설정하여 **중복된 URL이 저장되지 않도록 보호**.

---

# 6️⃣ UTF-8 및 문자 인코딩

## **문자셋 확인**

```sql
SHOW SERVER_ENCODING; -- UTF8
```

## **ASCII 값 변환**

```sql
SELECT ascii('A'), chr(65); -- 65, 'A'
```

---

# 7️⃣ 정규 표현식 활용 (예제)

## **이메일 검색**

```sql
SELECT email FROM users WHERE email ~ '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-z]{2,}$';
```

## **문장에서 해시태그 추출**

```sql
SELECT regexp_matches(tweet, '#([A-Za-z0-9_]+)', 'g') FROM tweets;
```

---

# 8️⃣ 인덱스를 활용한 성능 최적화

## **B-Tree 인덱스 vs 해시 인덱스**

| 인덱스 유형 | 설명 | 장점 | 단점 |
| --- | --- | --- | --- |
| **B-Tree 인덱스** | 기본 인덱스 방식, **범위 검색 및 정렬 지원** | 범위 검색(`BETWEEN`, `<`, `>`, `LIKE 'abc%'`) 가능 | 단순한 `=` 비교에서는 성능이 해시 인덱스보다 느릴 수 있음 |
| **해시 인덱스** | **단순한 `=` 비교에 최적화된 인덱스** | `=` 연산의 속도가 빠름 | **범위 검색 및 정렬 불가**, PostgreSQL 10 이상에서만 `WAL`(로그) 지원 |

## **EXPLAIN ANALYZE를 활용한 성능 비교**

```sql
-- B-Tree 인덱스 검색
EXPLAIN ANALYZE SELECT content FROM textfun WHERE content = 'example';

-- 해시 인덱스 검색 (PostgreSQL 10 이상 권장)
EXPLAIN ANALYZE SELECT content FROM textfun WHERE content = 'example'::text
```

```
Planning Time: 0.115 ms
Execution Time: 0.230 ms
```

```
Planning Time: 0.103 ms
Execution Time: 0.190 ms
```

**→** 비교 연산에서 해시 속도가 더 빠를 수 있음

---

# 9️⃣ 해싱을 활용한 대규모 데이터 처리

## **SHA-256**

```sql
SELECT sha256('hello');
```

## MD5 + UUID

```sql
CREATE TABLE cr4 (
    id SERIAL,
    url TEXT,
    url_md5 UUID UNIQUE,
    content TEXT
);

UPDATE cr4 SET url_md5 = md5(url)::UUID;
```

| 알고리즘 | 출력 길이 | 속도 | 보안 강도 | 충돌 가능성 |
| --- | --- | --- | --- | --- |
| **MD5** | 128비트(32자리) | 빠름 | 낮음(충돌 가능) | 높음(보안 취약) |
| **SHA-256** | 256비트(64자리) | 느림 | 높음(암호화 강도 좋음) | 매우 낮음 |
| **UUID** | 128비트(36자리) | 중간 | 중간 | 낮음(거의 없음) |

---

# 🔟 PostgreSQL의 정규 표현식 활용

## **문자열 특정 패턴 검색**

```sql
SELECT * FROM users WHERE email ~* '^[a-z]+@[a-z]+\\.[a-z]+$';
```

## **특정 문자 포함 여부**

```sql
SELECT * FROM comments WHERE comment_text ~ 'error|fail|bug';
```

## **문장에서 특정 패턴 추출**

```sql
SELECT regexp_matches(comment_text, '#([A-Za-z0-9_]+)', 'g') FROM comments;
```