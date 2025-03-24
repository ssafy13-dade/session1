# 1️⃣ 테이블 생성 및 데이터 조작

## **1.1 테이블 생성**

```sql
CREATE TABLE account (
    id SERIAL PRIMARY KEY,
    email VARCHAR(128) UNIQUE,
    created_at DATE NOT NULL DEFAULT NOW(),
    updated_at DATE NOT NULL DEFAULT NOW()
);

CREATE TABLE post (
    id SERIAL PRIMARY KEY,
    title VARCHAR(128) UNIQUE NOT NULL,
    content TEXT,
    account_id INTEGER REFERENCES account(id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE comment (
    id SERIAL PRIMARY KEY,
    content TEXT NOT NULL,
    account_id INTEGER REFERENCES account(id) ON DELETE CASCADE,
    post_id INTEGER REFERENCES post(id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE fav (
    id SERIAL PRIMARY KEY,
    post_id INTEGER REFERENCES post(id) ON DELETE CASCADE,
    account_id INTEGER REFERENCES account(id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    UNIQUE (post_id, account_id)
);
```

## **1.2 테이블 수정 (ALTER TABLE)**

- **컬럼 삭제:**

```sql
ALTER TABLE fav 
DROP COLUMN oops;
```

- **컬럼 타입 변경:**

```sql
ALTER TABLE post 
ALTER COLUMN content TYPE TEXT;
```

- **새로운 컬럼 추가:**

```sql
ALTER TABLE fav 
ADD COLUMN howmuch INTEGER;
```

---

# 2️⃣ 시간 및 날짜 관련 기능

## **2.1 기본 날짜 및 시간 타입**

- `DATE`: 날짜만 (`YYYY-MM-DD` 형식)
- `TIME`: 시간만 (`HH:MM:SS` 형식)
- `TIMESTAMP`: 날짜 + 시간 (`YYYY-MM-DD HH:MM:SS` 형식)
- `TIMESTAMPTZ`: 날짜 + 시간 + 타임존 정보

```sql
SELECT NOW();
SELECT NOW() AT TIME ZONE 'UTC';
SELECT NOW() AT TIME ZONE 'Asia/Seoul';
```

## **2.2 날짜 연산 및 변환**

```sql
SELECT NOW(), NOW() - INTERVAL '2 days', (NOW() - INTERVAL '2 days')::DATE;
SELECT DATE_TRUNC('day', NOW());
```

---

# 3️⃣ 성능 최적화 및 데이터 조회

## **3.1 테이블 스캔과 성능 최적화**

- 날짜 필드에서 특정 날짜만 조회할 때:

```sql
SELECT id, content, created_at 
FROM comment 
WHERE created_at::DATE = NOW()::DATE;
```

→ `::`  : 타입 캐스팅 연산자

## **3.2 DISTINCT 및 GROUP BY**

- **중복 제거(DISTINCT)**

```sql
SELECT DISTINCT model 
FROM racing;
```

- **그룹화 및 집계(GROUP BY)**

```sql
SELECT make, model
FROM racing 
GROUP BY model;
```

## **3.3 HAVING 절 활용**

```sql
SELECT COUNT(abbrev), abbrev 
FROM pg_timezone_names 
GROUP BY abbrev 
HAVING COUNT(abbrev) > 10;
```

---

# 4️⃣ 서브쿼리 활용

- **특정 사용자(email)가 작성한 댓글 조회:**

```sql
SELECT content 
FROM comment 
WHERE account_id = (
	SELECT id 
	FROM account 
	WHERE email='ed@umich.edu'
);
```

---

# 5️⃣ 동시성 처리 및 트랜잭션

## **5.1 트랜잭션 처리**

```sql
BEGIN;
UPDATE tracks SET count=count+1 WHERE id = 42;
COMMIT;
```

## **5.2 충돌 처리 (ON CONFLICT)**

- `INSERT` 시 **기본 키(PK) 또는 유니크(UNIQUE) 제약 조건 위반**이 발생하면, 충돌을 처리하는 방법을 지정할 수 있음.
- **PostgreSQL 전용 기능**으로, `UPSERT`(INSERT + UPDATE)를 구현하는데 사용됨.
- **중복 데이터가 들어올 경우, 새 데이터를 무조건 덮어쓰는 것이 아니라 적절히 처리하는 데 유용함**.

| 옵션 | 설명 |
| --- | --- |
| **DO NOTHING** | 충돌이 발생하면 아무 작업도 하지 않음 |
| **DO UPDATE** | 충돌이 발생하면 기존 데이터를 업데이트 |
| **EXCLUDED** | 새롭게 삽입하려던 값을 참조할 때 사용 |

```sql
INSERT INTO fav (post_id, account_id, howmuch) 
VALUES (1, 1, 1)
ON CONFLICT (post_id, account_id) 
DO UPDATE SET howmuch = fav.howmuch + 1 
RETURNING *;
```

**① `INSERT INTO fav (post_id, account_id, howmuch) VALUES (1, 1, 1);`**

- `fav` 테이블에 `(post_id=1, account_id=1, howmuch=1)` 값을 삽입하려고 함.
- 하지만 `post_id`, `account_id`가 **UNIQUE 제약 조건**을 갖고 있는 경우, 이미 해당 값이 존재하면 **충돌(Conflict)이 발생**.

**② `ON CONFLICT (post_id, account_id)`**

- **충돌 기준**을 `post_id, account_id`로 설정 → **이미 존재하는 값이라면 INSERT 대신 UPDATE 실행**.

**③ `DO UPDATE SET howmuch = fav.howmuch + 1`**

- 기존 데이터가 존재하면, `howmuch` 값을 **기존 값에서 +1 증가**시키도록 업데이트.

**④ `RETURNING *;`**

- **변경된 행을 반환**하여, 업데이트된 `howmuch` 값을 확인할 수 있음.

```sql
INSERT INTO fav (post_id, account_id, howmuch)
VALUES (1, 1, 1)
ON CONFLICT (post_id, account_id)
DO NOTHING;
```

- 기존에 **동일한 데이터가 있으면 아무 작업도 하지 않음**.
- 새로운 값이 추가되지 않으며, 오류도 발생하지 않음.

```sql
INSERT INTO fav (post_id, account_id, howmuch)
VALUES (1, 1, 5)
ON CONFLICT (post_id, account_id)
DO UPDATE SET howmuch = EXCLUDED.howmuch;
```

- `EXCLUDED.howmuch`: **새롭게 INSERT하려던 값(5)**을 의미.
- 기존 값은 무시하고, `howmuch`를 **5로 덮어쓰기**.

---

# 6️⃣ 트리거 및 저장 프로시저

## **6.1 트리거**

> 특정 **이벤트(INSERT, UPDATE, DELETE)가 발생할 때 자동으로 실행되는 데이터베이스 개체**
> 
- **자동화된 데이터 관리**를 위해 사용되며, **데이터 변경을 감지하고 특정 작업을 수행**하는 데 유용

| 트리거 시점 | 설명 |
| --- | --- |
| **BEFORE** | 데이터 변경 전에 트리거 실행 |
| **AFTER** | 데이터 변경 후에 트리거 실행 |
| **INSTEAD OF** | 원래 실행될 작업을 대체(주로 VIEW에서 사용) |

| 변수 | 설명 |
| --- | --- |
| `NEW` | 새롭게 삽입되거나 수정된 행 |
| `OLD` | 기존 행 (UPDATE 또는 DELETE 시 사용) |

```sql
CREATE OR REPLACE FUNCTION trigger_set_timestamp() RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER set_timestamp
BEFORE UPDATE ON fav
FOR EACH ROW
EXECUTE PROCEDURE trigger_set_timestamp();
```

### **활용 사례**

✅ **자동 필드 업데이트** → `created_at`, `updated_at` 자동 설정

✅ **데이터 검증** → 특정 필드 값이 변경될 때 조건을 검사

✅ **로깅(Audit Logging)** → 데이터 변경 내역을 로그 테이블에 기록

✅ **자동 계산** → 특정 열을 변경할 때 관련된 데이터를 자동으로 업데이트

✅ **무결성 유지** → 데이터 변경이 발생할 때 추가적인 작업 수행

## **6.2 저장 프로시저**

> 데이터베이스에서 **미리 저장된 SQL 코드 블록**을 의미하며, **함수(Function) 또는 트리거(Trigger)로 동작**
> 

```sql
CREATE FUNCTION update_howmuch() RETURNS TRIGGER AS $$
BEGIN
    NEW.howmuch = NEW.howmuch + 1;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER increase_howmuch
BEFORE UPDATE ON fav
FOR EACH ROW
EXECUTE FUNCTION update_howmuch();
```

| 개념 | 설명 |
| --- | --- |
| **NEW** | 트리거에서 **업데이트된 데이터 행을 참조**할 때 사용 |
| **BEFORE UPDATE** | **업데이트 전에 트리거 실행** |
| **FOR EACH ROW** | **각 행(Row)마다 트리거 실행** |

---

# 7️⃣ CSV 파일 처리 및 정규화

- CSV(Comma-Separated Values) 파일은 **데이터를 쉼표(,)로 구분하여 저장하는 텍스트 형식의 파일**.
- PostgreSQL에서 CSV 파일을 **테이블로 불러오거나, 내보내기** 할 수 있음.

```sql

CREATE TABLE xy (
    id SERIAL PRIMARY KEY,
    x_value INTEGER,
    y_value INTEGER
);
```

- `xy` 테이블을 생성하여 **(x, y) 좌표 데이터를 저장**.

```sql
COPY xy(x_value, y_value)
FROM '/path/to/xy.csv'
DELIMITER ','
CSV HEADER;
```

- `xy.csv` 파일을 `xy` 테이블에 로드.
- `DELIMITER ','`: 데이터 구분자는 `,`(쉼표).
- `CSV HEADER`: 첫 번째 행을 **컬럼명(헤더)으로 인식**.

### 정규화 과정

**① 기존 `xy` 테이블 확인**

```sql
SELECT * FROM xy;
```

| id | x_value | y_value |
| --- | --- | --- |
| 1 | 10 | 100 |
| 2 | 20 | 200 |
| 3 | 30 | 100 |
| 4 | 40 | 300 |

🚨 **`y_value` 값이 중복(100이 두 번 등장)** → 중복 제거 필요

**② `y_value`를 별도 테이블(`y`)로 분리**

```sql
CREATE TABLE y (
    id SERIAL PRIMARY KEY,
    y_value INTEGER UNIQUE
);
```

- `y_value` 값을 저장하는 **새로운 테이블 생성**.
- `y_value` 컬럼을 `UNIQUE` 제약 조건으로 설정 → **중복 제거 가능**.

**③ `xy` 테이블을 `y` 테이블과 연결**

```sql
CREATE TABLE xy_new (
    id SERIAL PRIMARY KEY,
    x_value INTEGER,
    y_id INTEGER REFERENCES y(id) ON DELETE CASCADE
);
```

- `xy_new` 테이블에서 `y_value`를 **참조키(FK, Foreign Key)**로 대체.

**④ `y` 테이블에 데이터 삽입**

```sql
INSERT INTO y (y_value)
SELECT DISTINCT y_value FROM xy;
```

- `xy` 테이블에서 **중복 없이(`DISTINCT`) `y_value` 값을 `y` 테이블에 삽입**.

**⑤ `xy_new`에 정규화된 데이터 삽입**

```sql
INSERT INTO xy_new (x_value, y_id)
SELECT x_value, y.id
FROM xy
JOIN y ON xy.y_value = y.y_value;
```

- 기존 `xy` 테이블에서 **정규화된 `y` 테이블의 ID를 참조**하여 `xy_new`에 삽입.

**`y` 테이블**

| id | y_value |
| --- | --- |
| 1 | 100 |
| 2 | 200 |
| 3 | 300 |

**`xy_new` 테이블**

| id | x_value | y_id |
| --- | --- | --- |
| 1 | 10 | 1 |
| 2 | 20 | 2 |
| 3 | 30 | 1 |
| 4 | 40 | 3 |

---

# 8️⃣ 고급 SQL 테크닉

## **8.1 테이블 락 (LOCK)**

> 동시에 실행되는 여러 트랜잭션이 같은 테이블을 조작할 때, 데이터 무결성을 보장하기 위해 테이블을 잠그는 기능
> 
- 테이블이 락이 걸리면, 특정 작업이 완료될 때까지 **다른 트랜잭션이 해당 테이블에 대한 읽기 또는 쓰기 작업을 제한**함.

```sql
LOCK TABLE table_name IN lock_mode;
```

| 락 모드 | 설명 | SELECT 허용 | UPDATE, INSERT, DELETE 허용 |
| --- | --- | --- | --- |
| ACCESS SHARE | **읽기 전용** 락 (SELECT 시 자동 적용) | ✅ 가능 | ✅ 가능 |
| ROW SHARE | **SELECT ... FOR UPDATE** 시 적용 | ✅ 가능 | ✅ 가능 |
| ROW EXCLUSIVE | **UPDATE, DELETE, INSERT 시 자동 적용** | ✅ 가능 | ✅ 가능 |
| SHARE | 다른 트랜잭션이 읽기 가능하지만, 변경 금지 | ✅ 가능 | ❌ 불가능 |
| SHARE ROW EXCLUSIVE | 다른 트랜잭션이 공유 잠금 불가 | ✅ 가능 | ❌ 불가능 |
| EXCLUSIVE | 테이블 변경 허용, 다른 트랜잭션은 접근 제한 | ❌ 불가능 | ❌ 불가능 |
| **ACCESS EXCLUSIVE** | **가장 강한 락, 다른 트랜잭션의 읽기/쓰기 모두 차단** | ❌ 불가능 | ❌ 불가능 |

```sql
LOCK TABLE account IN ACCESS EXCLUSIVE MODE;
```

- `account` 테이블을 **`ACCESS EXCLUSIVE MODE`(가장 강한 락 모드)**로 잠금.
- 락이 걸린 동안 **다른 트랜잭션은 해당 테이블을 읽거나 수정할 수 없음**.

### **활용 사례**

✅ **은행 계좌 이체** → 동일한 계좌를 동시에 변경하지 않도록 락 적용

✅ **재고 관리** → 같은 상품이 동시에 업데이트되지 않도록 방지

✅ **로그 테이블 보호** → 데이터 수정 없이 읽기 전용 유지

✅ **즉, `LOCK TABLE`을 사용하면 동시성 제어가 가능하지만, 너무 강한 락을 사용하면 성능 저하가 발생할 수 있음!** 

## **8.2 병렬 쿼리 실행**

- **하나의 SQL 쿼리를 여러 CPU 코어를 사용하여 동시에 실행하는 기술**.
- **PostgreSQL은 기본적으로 단일 프로세스가 하나의 쿼리를 처리하지만**, 병렬 실행을 사용하면 **쿼리 성능을 향상**시킬 수 있음.

| 설정 변수 | 설명 | 기본값 |
| --- | --- | --- |
| **max_parallel_workers_per_gather** | 병렬 실행  사용할 최대 병렬 작업(worker) 수 | 2 |
| **parallel_tuple_cost** | 병렬 작업 간의 데이터 전송 비용 조정 | 0.1 |
| **parallel_setup_cost** | 병렬 실행을 설정하는 비용 | 1000 |
| **force_parallel_mode** | 강제로 병렬 실행을 활성화할지 여부 (`off` 또는 `on`) | off |

```sql
SET max_parallel_workers_per_gather = 4;
```

- 병렬 실행 시 **쿼리당 사용할 최대 병렬 작업(worker) 수를 4로 설정**.
- **PostgreSQL은 `GATHER` 노드를 사용하여 병렬 작업을 조정**하며, 이 값이 크면 더 많은 CPU를 활용할 수 있음.

---

# 9️⃣ 정리

- **트랜잭션**을 활용하여 데이터 정합성을 유지
- **JOIN, 서브쿼리, DISTINCT, GROUP BY, HAVING** 등의 SQL 기능을 활용하여 데이터를 효과적으로 조회
- **트리거 및 저장 프로시저**를 활용하여 데이터 변경 시 자동 처리 기능을 구현
- **고급 SQL 테크닉** (JSONB 처리, 병렬 쿼리 실행, 테이블 락) 등을 활용하여 데이터베이스 성능을 최적화