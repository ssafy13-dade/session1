# Keys?
- Key는 데이터베이스에서 `행과 행을 연결`한다.
- 혹은 특정 행을 찾기 위한 `식별자 역할`을 한다.
- 즉, `테이블 간 관계 형성`의 핵심이다.

## 1. 세 가지 주요 Key
- Primary key (기본 키): 한 테이블 내의 각 행(row)을 `고유`하게 식별 \
( e.g. user_id, artist.id ) 
    - DB가 내부적으로 row을 탐색할 때 사용 (문자열이 아닌 숫자 key 기반 처리)
    - Why? 숫자인지
        - 문자열은 `변경`될 수 있으며 또한 `저장 공간`이 크고 느리다.
- Logical Key (논리 키): UI 등에서 `인식`하는 식별자\
( e.g. email 주소, track 제목 )
    - 사람이 `인식`하고 `검색`하는 키
    - 단, `변경`될 가능성이 있어 primary key로는 적합하지 않다.
    - 직접적으로 primary key를 보는 경우 직관적인 이해가 어렵기에 logical key를 사용.
- Foreign Key (외래 키): 한 테이블에서 다른 테이블을 `참조`하는 키.
    - 보통 `_id`로 끝나는 명명 규칙을 사용한다.


### 1-1. 명명 규칙 예시
- Primary key: id (현재 테이블의 고유 ID)
- Foreign key: artist_id (artist 테이블 id 참조)
- Logical key: email, title (사람이 이해할 수 있는 값)

## 2. 요약
![alt text](image-1.png)
- Primary key: 내부 식별용
- Logical key: 외부 식별용 (사람)
- Foreign key: 테이블 간 연결
    - Foreign key는 참조하는 테이블 뒤에 `_id`를 붙이는 식으로 하면 구조가 명확하다.
