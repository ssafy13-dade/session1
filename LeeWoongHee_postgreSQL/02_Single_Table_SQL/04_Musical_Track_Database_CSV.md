# 실습 코드를 주석과 함께 저장.
```SQL
-- Home directroy
pwd

-- 파일 확인 (지금은 없음)
ls -l

-- postgreSQL 실행 
-- (강의에서는 동일 클라이언트에서 실행하기에 생략)

-- URL에서 csv 파일 copy 해오기.
wget http://www.pg4e.com/tools/sql/library.csv

-- 잘 되고 있는지 파일 확인
ls -l

-- copy한 csv 파일을 vi로 txt 파일 열기
vi library.csv

-- vi란?
-- - Visual Editor로 터미널 환경에서 실행되는 '텍스트 편집기'
-- - '입력 모드'와 '명령 모드'를 구분한다.
--     - 입력 모드: 텍스트 입력 가능. (i: insert, a: append, o: 새 줄 추가, Esc: 입력 모드 종료)
--     - 명령 모드: 저장, 종료 등

-- 실습 환경에 있는 psql 코드를 복사해와 실행.
-- password를 복사해와 접근 완료.

-- =>: 명령어의 시작 부분임을 알림.
-- ->: 명령어를 입력 중임을 알림.

-- \dt: DB의 table을 확인할 수 있는 psql의 명령어
\dt

-- 테이블 생성
CREATE TABLE track_raw
(title TEXT, artist TEXT, album TEXT, count INTEGER, rating INTEGER, len INTEGER);

-- 생성된 table 확인
\dt

-- \copy: psql 명령어로 복사를 하는 역할을 한다.
--     - track_raw: 테이블 이름.
--     - (col name1, col name2, ...): 가져올 컬럼들
--     - 'library.csv': 파일 이름
--     - DELIMITER: 구분자.
\copy track_raw(title, artist, album, count, rating, len) FROM 'library.csv' WITH DELIMITER ',' CSV;

-- 테이블 확인
SELECT * FROM track_raw;
SELECT count(*) FROM track_raw;

-- \q: 종료
\q
```