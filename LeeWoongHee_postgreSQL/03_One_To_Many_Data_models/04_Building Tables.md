# Building Tables
- SQL로 실습 진행.

## 1. 테이블 설정
```SQL
CREATE TABLE artist (
    id SERIAL, -- SERIAL을 통해 `자동 증가 숫자 컬럼`을 생성.
    name VARCHAR(128) UNIQUE, -- LOGICAL KEY, 중복되면 안 될 때 UNIQUE 설정. (UNIQUE는 내부적으로 idx 생성).
    PRIMARY KEY(id) -- PRIMARY KEY로 지정 -> idx가 생성되어 빠른 접근 가능.
);

CREATE TABLE album (
    id SERIAL,
    title VARCHAR(128) UNIQUE, -- Logical key
    artist_id INTEGER REFERENCES artist(id) ON DELETE CASCADE, -- CASCADE는 Freigh key가 참조하는 primary가 삭제되면, 해당 Foreign key에 해당된 정보 또한 함께 삭제하는 옵션.
    PRIMARY KEY(id)
);
```
- 위 코드 특히 `artist` 테이블을 구성하는 것은 일반적인 패턴임
    1. Primary key로 사용할 컬럼에 SERIAL을 주고,
    2. 마지막에 PRIMARY KEY로 설정.
    3. String 기반 logical key에는 UNIQUE를 사용.
- album 테이블의 구조도 동일.
- 이후 Foreign key를 추가. 
    - Foreign key는 참조하는 primary key로 향하는 화살표의 `시작점`임을 기억.

## 2. 다른 테이블 생성
```SQL
CREATE TABLE genre (
    id SERIAL,
    name VARCHAR(128) UNIQUE,
    PRIMARY KEY(id)
);

CREATE TABLE track (
    id SERIAL,
    title VARCHAR(128),
    len INTEGER,
    rating INTEGER,
    count INTEGER,
    album_id INTEGER REFERENCES album(id) ON DELETE CASCADE,
    genre_id INTEGER REFERENCES genre(id) ON DELETE CASCADE,
    -- 같은 album 내에서는 동일 제목 트랙 중복 불가, 그러나 다른 앨범에서는 가능.
    -- 따라서 `title과 album_id`를 묶어서 사용.
    UNIQUE(title, album_id), 
    PRIMARY KEY(id)
);
```
- track 테이블의 `UNIQUE(title, album_id)`는 두 컬럼의 조합으로 UNIQUE를 사용한다.
    - 즉, 두 컬럼의 `조합의 유일성`을 보장.