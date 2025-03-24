# 학습 목표
- `검색 속도`를 극대화하는 `인덱스(INDEX)`에 대해 배운다.
- AUTO_INCREMENT 설정은 postgreSQL에서는 `SERIAL`이다.
```SQL
CREATE TABLE users (
    id SERIAL, -- 자동 증가 설정.
    name VARCHAR(128),
    email VARCHAR(128) UNIQUE, -- UNIQUE는 '중복값'이 있으면 안됨을 의미.
    PRIMARY KEY(id) -- primary key 지정
);
```

## 1. Key
- DB 내의 하나의 테이블에서 특정 행을 `고유하게 식별`하는 값.
    - Key는 index와는 다른 개념.
    - Index는 데이터를 빠르게 검색하기 위한 `보조 도구`.

### 1-1. Primary Key (기본 키)
- 각 행을 고유하게 식별.
- 자동으로 index 생성
    - 검색 속도 매우 빠름.
- 보통 `SERIAL`과 함께 사용
```SQL
CREATE TABLE users (
    -- 1. SERIAL: 자동 증가
    -- 2. PRIMARY KEY: 해당 열을 기준으로 자동 idx 생성.
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(255)
);
```

### 1-2. Unique Key (고유 키)
- `중복`을 허용하지 않는 키, 즉 동일한 컬럼 내에 `중복 값`이 들어갈 수 없다.
- PRIMARY KEY와의 차이는 `여러 개`의` 컬럼에 지정이 가능하다.
- UNIQUE KEY 또한 `자동 idx`가 생성되어 검색 속도가 향상된다.
```SQL
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE -- 같은 이메일 중복 저장 불가
);
```

## 2. Index
- 검색을 빠르게 하기 위한 추가적인 데이터 구조.
- 데이터를 `정렬된 상태`로 저장
    - 빠른 조회 가능.
- 대용량 데이터에 매우 중요한 성능 최적화 요소
- 즉, 완탐 안해도 된다는 의미.
- 강의에서는 가장 많이 사용되는 `B-Tree`와 `Hash`를 다룸.

### 2-1. B-Tree
- PostgreSQL의 `기본 idx`이다.
    - `Balanced Tree`로 데이터를 정렬된 상태로 여러 `level의 블록`에 저장한다.
        - 트리 구조이기에 탐색시 `루트 - 중간 노드 - 리프 노드 - 데이터 위치`와 같이 접근한다. (빠른 조회)
    - 정렬, 범위 검색, 접두어 검색, 정렬 최적화에 유용하다.
    - PRIMARY KEY, UNIQUE, ORDER BY, LIKE 'abc%', BETWEEN 등에 적합.
- < 구조 >
    - 데이터를 `정렬된 구조(트리)`로 저장한다.
        - 하나의 idx가 여러 `층`으로 구성된다. 따라서 탐색 시 적은 횟수로 목표에 도달한다. (알고리즘의 탐색 같은 느낌)
- < 장점 >
    - 범위 조회: WHERE age BETWEEN 20 AND 30
    - 정렬 속도 향상
    - 접두어 검색: LIKE 'abc%'

### 2-2. Hash
- `정확히 일치하는 값`을 찾는데 특화 되었다. (e.g. WHERE id = 12345).
    - B-tree 처럼 정렬, 범위 검색, 접두어에 적합 X
- 내부적으로 문자열이나 숫자를 `수학적`으로 변환해 `숫자 하나`로 압축한다.
    - 정렬되어 있지는 않다.
    - 즉, LIKE, ORDER BY, BETWEEN과 같은 쿼리에서 유용하지 않다.
    - 대신 `id, token, GUID` 등에서는 유용하다.
```SQL
CREATE INDEX idx_id_hash ON users USING HASH (id);
```

## 3. 마무리
- 어떤 idx를 사용하는가는 PostgreSQL이 알아서 판단해준다.
    - PRIMARY KEY: `정수형` 기반인 경우 `HASH`
    - UNIQUE: `문자열`이라면 `B-tree`
- 즉, 데이터베이스가 알아서 최적화 한다.
- 너무 많은 idx는 `쓰기 성능 저하`의 가능성이 있다.