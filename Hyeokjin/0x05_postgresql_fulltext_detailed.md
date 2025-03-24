# 1️⃣ 테이블 설계 및 데이터 삽입

## **테이블 생성**

```sql
CREATE TABLE messages (
    id SERIAL PRIMARY KEY,
    email TEXT,
    sent_at TIMESTAMPTZ,
    subject TEXT,
    headers TEXT,
    body TEXT
);
```

- **컬럼 설명**:
    - `id`: 고유 식별자
    - `email`: 발신자 이메일 주소
    - `sent_at`: 메일 발송 시간
    - `subject`: 메일 제목
    - `headers`: 메일 헤더 정보
    - `body`: 메일 본문

## **데이터 삽입 예시**

```sql
INSERT INTO messages (email, sent_at, subject, headers, body) VALUES
('user@example.com', '2025-03-10 10:00:00', 'Hello World', 'Header info', 'This is the body of the email.');
```

---

# 2️⃣ 전문 검색 설정 (Full-Text Search)

- **TSVECTOR** : **문서를 색인할 때 사용하는 데이터 타입**. 텍스트를 토큰화(tokenization)하고 정규화(normalization)하여 저장.
- **TSQUERY** : **검색어를 표현하는 데이터 타입**. 특정 단어나 구문을 검색할 때 사용.

### **`tsvector` 생성**

```sql
SELECT to_tsvector('english', 'This is a sample document.');
```

- **불용어(Stop Words) 제거** → "is", "a" 같은 의미 없는 단어 삭제됨.
- **어간 추출(Stemming) 적용** → "sample" → "sampl", "this" → "thi" 등으로 변환.
- **단어별 위치 저장** → `'sampl':4`는 **4번째 단어가 "sample"이었음**을 의미.
- 결과: `'document':5 'sampl':4 'thi':1`

### **`tsquery` 생성**

```sql
SELECT to_tsquery('english', 'sample & document');
```

- **"sample"이 "sampl"로 변환됨** → **어간 추출 적용**.
- **"&"는 AND 연산자 역할** → "sample"과 "document"가 **둘 다 포함된 문서를 검색**.
- 결과: `'sampl' & 'document'`

---

# 3️⃣ 전문 검색 인덱스 생성

## **GIN 인덱스 생성**

```sql
CREATE INDEX idx_messages_body ON messages USING GIN(to_tsvector('english', body));
```

- **GIN(Generalized Inverted Index)**: 다수의 값이 포함된 컬럼에 효율적인 인덱스

---

# 4️⃣ 전문 검색 쿼리 실행

```sql
SELECT * FROM messages
WHERE to_tsvector('english', body) @@ to_tsquery('english', 'sample & document');
```

- **`@@` 연산자**: `tsvector`와 `tsquery`를 비교하여 일치 여부를 반환

**`messages` 테이블 데이터**

| id | body |
| --- | --- |
| 1 | This is a sample document. |
| 2 | This document contains a sample text. |
| 3 | This is another text without sample. |

🔍 **결과**

| id | body |
| --- | --- |
| 1 | This is a sample document. |
| 2 | This document contains a sample text. |

✅ **"sample"과 "document"가 포함된 문서만 검색**

---

# 5️⃣ 가중치 및 순위 설정

## **가중치 부여**

```sql
SELECT setweight(to_tsvector('english', coalesce(subject, '')), 'A') ||
       setweight(to_tsvector('english', coalesce(body, '')), 'B') AS document
FROM messages;
```

- **`setweight` 함수**: 텍스트에 가중치를 부여하여 검색 결과의 우선순위를 설정
    - 'A' > 'B' > 'C' > 'D' 순으로 높은 가중치

| document |
| --- |
| 'hello':1A 'sample':2B 'document':3B |

→ 
제목(`subject`)은 높은 우선순위(`A`), 본문(`body`)은 그보다 낮은 우선순위(`B`)를 가지게 됨!

## **순위 계산**

```sql
SELECT ts_rank_cd(to_tsvector('english', body), to_tsquery('english', 'sample & document')) AS rank,
       *
FROM messages
WHERE to_tsvector('english', body) @@ to_tsquery('english', 'sample & document')
ORDER BY rank DESC;
```

- **`ts_rank_cd` 함수**: 문서와 검색어의 일치 정도를 계산하여 순위를 매김

| rank | id | body |
| --- | --- | --- |
| 0.75 | 1 | This is a sample document. |
| 0.60 | 2 | This document contains a sample text. |
| 0.30 | 3 | This is another document example. |

### **`ts_rank_cd` vs.  `ts_rank`**

| 함수 | 설명 |
| --- | --- |
| **`ts_rank`** | 문서 길이를 고려하여 점수를 계산 |
| **`ts_rank_cd`** | 문서 길이를 고려하지 않고 단순 계산 (속도가 빠름) |

✅ **검색 성능을 최적화하려면 `ts_rank_cd`가 더 유리**

---

# 6️⃣ 사전 및 불용어 처리

## **사용 가능한 사전 확인**

```sql
SELECT * FROM pg_ts_config;
```

- PostgreSQL에서 지원하는 텍스트 구성 목록을 확인

| oid | cfgname | cfgnamespace | cfgowner | cfgparser |
| --- | --- | --- | --- | --- |
| 3734 | english | 11 | 10 | 3722 |
| 3735 | simple | 11 | 10 | 3722 |
| 3736 | german | 11 | 10 | 3722 |

## **불용어(Stop Words) 처리**

- 불용어: 검색 시 의미가 적은 단어들(e.g., 'a', 'the', 'is')
- PostgreSQL은 언어별 불용어 목록을 제공하며, 필요에 따라 사용자 정의 가능

### **사용자 정의 불용어 목록 생성**

PostgreSQL에서는 **사용자 정의 불용어 사전을 만들 수 있음.**

1. **불용어 파일 생성**
    - PostgreSQL의 **불용어 파일은 `pg_ts_dict` 테이블에서 관리됨.**
    - 예제: `custom_stopwords` 파일 생성 (`/usr/share/postgresql/tsearch_data/` 위치)
    
    📌 **파일 내용 (`custom_stopwords.stop`)**
    
    ```
    the
    is
    a
    of
    and
    ```
    
2. **사용자 정의 불용어 사전 등록**
    
    ```sql
    CREATE TEXT SEARCH DICTIONARY custom_dict (
        TEMPLATE = simple,
        STOPWORDS = 'custom_stopwords'
    );
    ```
    
    ✅ `custom_stopwords` 목록을 불용어 사전으로 등록.
    
3. **새로운 텍스트 구성(`TS Configuration`) 생성**
    
    ```sql
    CREATE TEXT SEARCH CONFIGURATION custom_fts (COPY = english);
    ALTER TEXT SEARCH CONFIGURATION custom_fts
        ALTER MAPPING FOR asciiword, word WITH custom_dict, english_stem;
    ```
    
    ✅ `custom_fts`라는 **새로운 전문 검색 구성 생성**
    
    ✅ **기존 `english` 설정을 복사하고, 불용어 사전을 추가로 적용**
    
4. **불용어 적용 테스트**
    
    ```sql
    SELECT to_tsvector('custom_fts', 'This is a custom text for testing.');
    ```
    

🔍 **결과**

```
'custom':3 'text':4 'test':6
```

✅ `"this"`, `"is"`, `"a"` 같은 불용어가 제거

---

# 7️⃣ 전문 검색의 실제 적용

## **이메일 검색 예시**

```sql
SELECT * FROM messages
WHERE to_tsvector('english', body) @@ to_tsquery('english', 'meeting & schedule');
```

- 'meeting'과 'schedule' 단어가 모두 포함된 이메일 본문을 검색

## **제목과 본문을 함께 검색**

```sql
SELECT * FROM messages
WHERE to_tsvector('english', subject || ' ' || body) @@ to_tsquery('english', 'project & update');
```

- 제목과 본문을 합쳐서 'project'와 'update' 단어가 모두 포함된 이메일을 검색

---

# 8️⃣ 전문 검색 성능 최적화

## **인덱스 활용**

- `to_tsvector` 함수를 사용하여 GIN 인덱스를 생성하면 대량의 텍스트 데이터에서도 빠른 검색이 가능

## **정규화 및 토큰화**

- 텍스트를 소문자로 변환하고, 불필요한 문자를 제거하여 검색 효율을 향상

---

# 9️⃣ 전문 검색의 한계와 고려사항

- **언어 지원**: PostgreSQL은 여러 언어를 지원하지만, 언어별로 토큰화 및 불용어 처리가 다를 수 있음
- **복합 검색**: 여러 컬럼을 동시에 검색할 경우, 각 컬럼에 대한 `tsvector`를 생성하고 결합하여 인덱스를 생성해야 함
- **정확도 vs 성능**: 정확도를 높이기 위한 복잡한 쿼리는 성능 저하를 초래할 수 있으므로, 적절한 균형이 필요