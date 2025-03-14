# 1️⃣ 데이터 직렬화

## **1.1 데이터 직렬화란?**

각 프로그래밍 언어는 데이터를 표현하기 위한 고유한 방법을 가지고 있습니다. 이러한 데이터를 다른 프로그램이나 시스템과 공유하려면 언어에 독립적인 형식으로 변환

| 언어 | 선형 구조 표현 | 키/값 구조 표현 |
| --- | --- | --- |
| Python | `list()` `[1,2,3]` | `dict()` `{'a':1, 'b': 2}` |
| JavaScript | `Array` `[1,2,3]` | `Object` `{'a':1, 'b': 2}` |
| PHP | `Array` `array(1,2,3)` | `Array` `array('a' => 1, 'b' => 1)` |
| Java | `ArrayList` | `HashMap` |

과거에는 XML이 이러한 데이터 구조를 표현하는 데 사용

그러나 XML은 리스트나 딕셔너리와 같은 구조를 표현하기에 적합하지 않았고, 이에 따라 JSON(JavaScript Object Notation)이 등장

---

# 2️⃣ JSON - JavaScript 객체 표기법

## **2.1 JSON의 등장 배경**

2000년대 초반, 브라우저의 JavaScript와 서버 간의 데이터 전송이 증가하면서, XML을 사용하는 AJAX(Asynchronous JavaScript And XML)가 등장

그러나 XML은 데이터 구조를 표현하기에 비효율적이었고, 이를 대체하기 위해 JSON이 도입

JSON은 JavaScript의 객체 표기법을 기반으로 하여, 가볍고 읽기 쉬운 형식으로 데이터를 표현

```jsx
<script type="text/javascript">
who = {
    "name": "Chuck",
    "age": 29,
    "college": true,
    "offices" : [ "3350DMC", "3437NQ" ],
    "skills" : { "fortran": 10, "C": 10, "C++": 5, "python" : 7 }
};
console.log(who);
</script>
```

---

# 3️⃣ Python에서의 JSON 활용

Python은 JSON 데이터를 처리하기 위한 `json` 라이브러리를 제공

## **3.1 JSON 직렬화**

Python 객체를 JSON 문자열로 변환하려면 `json.dumps()` 함수를 사용

```python
import json

data = {
    'name': 'Chuck',
    'phone': {
        'type': 'intl',
        'number': '+1 734 303 4456'
    },
    'email': {
        'hide': 'yes'
    }
}

# 직렬화
print(json.dumps(data, indent=4))
```

출력:

```json
{
    "name": "Chuck",
    "phone": {
        "type": "intl",
        "number": "+1 734 303 4456"
    },
    "email": {
        "hide": "yes"
    }
}
```

## **3.2 JSON 역직렬화**

JSON 문자열을 Python 객체로 변환하려면 `json.loads()` 함수를 사용

```python
import json

data = '''
{
  "name" : "Chuck",
  "phone" : {
    "type" : "intl",
    "number" : "+1 734 303 4456"
   },
   "email" : {
     "hide" : "yes"
   }
}'''

info = json.loads(data)
print('Name:', info["name"])
print('Hide:', info["email"]["hide"])
```

출력:

```
Name: Chuck
Hide: yes
```

---

# 4️⃣ PostgreSQL에서의 JSON 지원

## **4.1 HSTORE**

> **키-값(key-value) 쌍을 저장**하기 위한 데이터 타입
> 
- **단순한 키-값 저장**: 문자열 기반의 키-값 쌍 저장.
- **중첩 구조 불가능**: JSON과 달리 중첩된 데이터(객체/배열)를 저장할 수 없음.
- **빠른 조회**: 단순한 구조 덕분에 키-값 검색이 빠름.
- **JSON과 유사하지만 더 제한적**: 데이터 구조가 단순하기 때문에 복잡한 데이터 표현이 어려움.

```sql
SELECT 'a=>1,b=>2'::hstore;
```

```
        hstore
--------------------
 "a"=>"1", "b"=>"2"
```

### **📌 HSTORE 사용 예제**

```sql
SELECT 'a=>1, b=>2'::hstore;
```

**출력 결과**

```
        hstore
---------------------
 "a"=>"1", "b"=>"2"
```

- `'a=>1, b=>2'` 형태로 데이터를 저장.
- `a`라는 키에는 `"1"`, `b`라는 키에는 `"2"` 값이 저장됨.

### **📌 HSTORE 데이터 삽입 및 조회**

```sql
CREATE TABLE store_example (
    id SERIAL PRIMARY KEY,
    data HSTORE
);

INSERT INTO store_example (data)
VALUES ('"name"=>"Alice", "age"=>"30"');

SELECT data->'name' FROM store_example;
```

```
 name
------
 Alice
```

- `>` 연산자를 사용하여 특정 키의 값을 조회 가능.

### **📌 HSTORE 단점**

- **중첩 데이터 저장 불가** → JSON처럼 복잡한 구조(배열, 객체)를 지원하지 않음.
- **데이터 타입 제한** → 모든 값이 **문자열로 저장됨** (숫자도 문자열로 취급).
- **JSON보다 덜 유연함** → PostgreSQL에서는 JSONB가 더 강력해서, 최근에는 HSTORE 대신 JSONB를 더 많이 사용.

## **4.2 JSON**

PostgreSQL 9.3부터 도입된 JSON 타입은 텍스트 형식으로 JSON 데이터를 저장하며, 일부 함수와 연산자를 제공

## **4.3 JSONB**

JSONB는 이진 형식으로 JSON 데이터를 저장하며, 효율적인 저장 공간 활용과 빠른 검색을 지원

현재 PostgreSQL에서 JSON 데이터를 다룰 때 권장되는 타입

---

# 5️⃣ PostgreSQL에서의 JSONB 활용

## **5.1 테이블 생성 및 데이터 삽입**

JSON 데이터를 저장하기 위한 테이블을 생성, 데이터를 삽입

```sql
CREATE TABLE IF NOT EXISTS jtrack (
	id SERIAL, 
	body JSONB
);
```

```sql
-- JSON 데이터 파일을 테이블에 삽입
\copy jtrack (body) FROM 'library.jstxt' WITH CSV QUOTE E'' DELIMITER E'';
```

- `\copy` → PostgreSQL의 **COPY 명령어**를 활용하여 데이터를 빠르게 삽입.
- `'library.jstxt'` → 삽입할 JSON 데이터 파일.
- `WITH CSV QUOTE E'' DELIMITER E''` → 파일을 CSV로 처리하지만, **특수 문자를 구분하지 않음**.

## **5.2 JSONB 데이터 조회**

JSONB 데이터에서 특정 필드를 추출하거나 조건을 지정하여 조회할 수 있습니다.

```sql
-- 'name' 필드가 'Summer Nights'인 레코드의 'count' 값을 정수로 변환하여 조회
SELECT (body->>'count')::int FROM jtrack WHERE body->>'name' = 'Summer Nights';

-- JSONB 데이터에 특정 키-값 쌍이 포함되어 있는지 확인하여 조회
SELECT (body->>'count')::int FROM jtrack WHERE body @> '{"name": "Summer Nights"}';

-- JSONB 데이터에 특정 키가 존재하는지 확인
SELECT COUNT(*) FROM jtrack WHERE body ? 'favorite';

-- 'count' 값을 기준으로 내림차순 정렬하여 'name' 필드 조회
SELECT body->>'name' AS name FROM jtrack ORDER BY (body->>'count')::int DESC;
```

## **5.3 인덱스 생성**

JSONB 데이터에 대한 효율적인 검색을 위해 인덱스를 생성

```sql
-- 'name' 필드에 대한 BTREE 인덱스 생성
CREATE INDEX jtrack_btree ON jtrack USING BTREE ((body->>'name'));

-- 전체 JSONB 데이터에 대한 GIN 인덱스 생성
CREATE INDEX jtrack_gin ON jtrack USING gin (body);

-- JSONB 경로 연산자에 대한 GIN 인덱스 생성
CREATE INDEX jtrack_gin_path_ops ON jtrack USING gin (body jsonb_path_ops);
```

---

# 6️⃣ JSON 데이터를 PostgreSQL에 저장하고 활용하기

- **Python과 PostgreSQL을 연동하여 JSON 데이터를 처리**할 수 있습니다.
- **JSONB 타입을 사용하여 효율적으로 데이터를 저장하고 검색**할 수 있습니다.
- **GIN 인덱스를 활용하여 JSONB 데이터의 검색 성능을 최적화**할 수 있습니다.