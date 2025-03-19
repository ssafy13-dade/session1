# 학습 목표
- 테이블을 만들 때 사용하는 다양한 `데이터 타입`.

## 문자형
### 1. VARCHAR(n)
- `가변` 길이 문자열을 저장한다.
    - 메모리상 유연
    - 최소 공간을 사용할 수 있다.
- 최대 `n` 길이까지 저장이 가능하다.gi
    - 일반적으로 64 글자 이상인 문자열에 사용한다. (교수님 기준).
```SQL
-- Sample
CREATE TABLE users (
    name VARCHAR(128) -- 최대 길이 128 문자열 저장
);
```

### 2. CHAR(n)
- `고정` 길이 문자열 저장.
- 길이가 n보다 작은 경우 빈 공간을 `채운다`.
    - 일정한 길이의 데이터 (e.g. GUID, 해시, etc)에 적합하다.
```SQL
CREATE TABLE tokens (
    token CHAR(64) -- 고정된 64 길이의 문자열 저장
)
```

### 3. TEXT
- 길이 제한이 `없다`.
    - VARCHAR 처럼 최대 길이도 지정하지 않는다.
- 인덱스의 생성 및 정렬이 불리하다.
    - WHERE, ORDER BY 등을 사용하기 부적합하다.
- 블로그 게시글, 댓글, 웹페이지 데이터 저장 등에 적합하다.
```SQL
CREATE TABLE blog_posts (
    content TEXT -- 길이 제한 없는 문자열 저장.
)
```

### Summary
- VARCHAR: 짧고 검색이 필요한 문자열
- CHAR: 고정된 길이 문자열
- TEXT: 길이가 길고, 검색보다 `저장`이 중요한 문자열

## 정수형
- SMALLINT: (-32768, +32768)
    e.g. 나이, 개수 등 작은 값
- INTEGER: (-2 Billion, +2 Billion)
    e.g. 일반적인 정수
- BIGNT: 셋 중 가장 큰 값을 갖는다.
    e.g. 2억을 넘는 큰 값
```SQL
CREATE TABLE table name (
    sample_column1 SMALLINT,
    sample_column2 INTEGER,
    sample_column3 BIGINT
);
```

## 실수형 데이터
- REAL: 4바이트, 소수점 이하 7자리
    e.g. 간단한 연산
- DOUBLE PRECISION: 8바이트, 소수점 이하 15자리
    e.g. 과학적 계산, 고정밀도 연산
- NUMERIC(p, s): 길이 `p`의 숫자 중 `s`개 가 소수점 자리.
    e.g. 금융 데이터, 정밀 계산
```SQL
CREATE TABLE table name (
    sample_column1 REAL,
    sample_column2 DOUBLE PRECISION,
    sample_column3 NUMERIC(5, 2) -- 5 자리 중 2 자리가 소수점
);
```

## 날짜 및 시간
1. DATE: 연, 월, 일
    - `YYYY-MM-DD` 형식
```SQL
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    birth_date DATE -- 생년월일
);

INSERT INTO employees (name, birth_date) VALUES ('Alice', '1995-06-25');

```
2. TIME: 시, 분, 초
    - `HH:MI:SS` 형식
```SQL
CREATE TABLE working_hours (
    employee_id SERIAL PRIMARY KEY,
    start_time TIME, -- 출근 시간
    end_time TIME -- 최근 시간
);
```
3. TIMESTAMP: 날짜 + 시간
    - `YYYY-MM-DD HH:MI:SS` 형식
- 로컬(local) 기준 시간 즉, 어느 지역인지 알 수 없음.
- 변환 시 타임존 지정.
```SQL
-- 테이블 생성
CREATE TABLE events (
    event_id SERIAL PRIMARY KEY,
    event_time TIMESTAMP -- 타임존 정보 X
); 

-- 값 삽입
INSERT INTO events (event_time) VALUES ('2025-03-18 14:30:00');
```
4. TIMESTAMP WITH TIME ZONE
    - TIMESTAMP와 동일하지만 `타임존(Time Zone)` 정보를 추가 저장.
    - `타임존`이란 지역별 시간을 나타내기 위한 표기이다. 
        e.g. 2025-03-18 14:30:15 .123456`+09` 해당 부분이 `한국 기준 타임존`.
    - `AT TIME ZONE` 함수
        - 타임존 변환
```SQL
-- 테이블 생성
CREATE TABLE flights (
    flight_id SERIAL PRIMARY KEY,
    departure TIMESTAMP WITH TIME ZONE
);

-- 값 삽입
-- 타임존(+09)값이 붙음.
INSERT INTO flights (departure) VALUES ('2025-03-18 14:30:00+09');

-- 타임존 변환 (UTC)
SELECT NOW() AT TIME ZONE 'UTC';
-- 타임존 변환 (한국)
SELECT NOW() AT TIME ZONE 'Asia/Seoul';
```

## 날짜 및 시간 함수
1. NOW()
    - 현재의 TIMESTAMP WITH TIME ZONE 값을 반환.
```SQL
-- 현재 시간 + 1일 후 데이터 조회
SELECT * FROM events WHERE event_time > NOW() + INTERVAL '1 day';
```
2. CURRENT_DATE
    - 오늘 날짜 반환.
    - DATE type 임.
```SQL
SELECT CURRENT_DATE;
```
3. CURRENT_TIME
    - 현재 시간 반환
    - TIME type 임.
```SQL
SELECT CURRENT_TIME;
```
4. AGE()
    - 날짜 차이를 계산
```SQL
SELECT AGE('2025-03-18', '2000-01-01');
```

## 기타 데이터 타입
### BYREA (Binary data)
- 이미지, 파일, 암화된 데이터 등을 저장.
- 일반 텍스트가 아닌 `바이너리 데이터`를 저장할 경우.
```SQL
CREATE TABLE files (
    id SERIAL PRIMARY KEY,
    data BYTEA
);
```

### 자동 증가 필드 (Auto-Increment)
- `자동 증가`하는 정수 값을 저장한다.
- SERIAL: INTEGER와 동일.
- BIGSERIAL: BIGINT와 동일.
```SQL
-- 테이블 생성
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

-- 값 삽입
-- 두 개의 row가 삽입된다.
-- -> 자동 증가 (SERIAL)이 적용되어 있기에
-- -> id 컬럼은 Alice = 1, 그리고 Bob = 2가 된다.
INSERT INTO users (name) VALUES ('Alice');
INSERT INTO users (name) VALUES ('Bob');
```