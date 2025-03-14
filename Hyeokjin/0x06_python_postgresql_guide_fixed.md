## 1️⃣ Python과 PostgreSQL 연동

### **1.1 psycopg2 라이브러리 설치**

Python에서 PostgreSQL과 상호작용하기 위해 `psycopg2` 라이브러리를 설치

```bash
pip install psycopg2
```

### **1.2 데이터베이스 연결**

PostgreSQL에 연결

```python
import psycopg2

conn = psycopg2.connect(
    host='localhost',
    database='mydatabase',
    user='myuser',
    password='mypassword',
    connect_timeout=3
)
```

### **1.3 커서 생성 및 SQL 실행**

커서를 사용하여 SQL 문을 실행할 수 있습니다.

```python
cur = conn.cursor()

# 테이블 삭제 및 생성
cur.execute('DROP TABLE IF EXISTS pythonfun CASCADE;')
cur.execute('CREATE TABLE pythonfun (id SERIAL, line TEXT);')

# 변경사항 저장
conn.commit()
```

---

## 2️⃣ 예제: 책 텍스트 로드 및 처리

### **2.1 프로젝트 구텐베르크 책 다운로드**

다음 명령어를 사용하여 프로젝트 구텐베르크에서 책을 다운로드

```bash
wget <http://www.gutenberg.org/cache/epub/19337/pg19337.txt>
```

### **2.2 책 데이터베이스에 로드**

책의 각 단락을 데이터베이스에 저장하는 `loadbook.py` 스크립트를 실행

```python
import psycopg2  # PostgreSQL과 연결하기 위한 라이브러리
import hidden    # 데이터베이스 접속 정보(비밀값 저장)
import time      # 처리 속도 조절을 위한 라이브러리

# 사용자로부터 입력 파일명 받기
bookfile = input("Enter book file (i.e. pg19337.txt): ")
if bookfile == '':  # 입력이 없으면 기본 파일 사용
    bookfile = 'pg19337.txt'
base = bookfile.split('.')[0]  # 확장자를 제거한 파일명을 데이터베이스 테이블명으로 사용

# 파일 열어서 읽기 가능 여부 확인
fhand = open(bookfile)

# 데이터베이스 접속 정보 가져오기
secrets = hidden.secrets()

# PostgreSQL 데이터베이스에 연결
conn = psycopg2.connect(
    host=secrets['host'],
    port=secrets['port'],
    database=secrets['database'],
    user=secrets['user'],
    password=secrets['pass'],
    connect_timeout=3  # 3초 동안 연결 시도 후 실패 시 오류 발생
)

cur = conn.cursor()  # 커서 객체 생성 (SQL 실행을 위한 객체)

# 기존 테이블 삭제 (같은 테이블이 존재하면 삭제하고 새로 생성하기 위해)
sql = 'DROP TABLE IF EXISTS ' + base+' CASCADE;'
print(sql)
cur.execute(sql)

# 새로운 테이블 생성 (id: 자동 증가, body: 문단 저장)
sql = 'CREATE TABLE ' + base + ' (id SERIAL, body TEXT);'
print(sql)
cur.execute(sql)

# 데이터 저장을 위한 변수 초기화
para = ''   # 문단 저장
chars = 0   # 전체 문자 개수 카운트
count = 0   # 총 라인 수 카운트
pcount = 0  # 문단 개수 카운트

# 파일을 한 줄씩 읽어서 처리
for line in fhand:
    count += 1
    line = line.strip()  # 양 끝 공백 제거
    chars += len(line)  # 전체 문자 개수 카운트
    
    # 빈 줄이 연속되면 무시
    if line == '' and para == '':
        continue
    
    # 빈 줄이 나오면 하나의 문단이 끝났다는 의미 → DB에 저장
    if line == '':
        sql = 'INSERT INTO '+base+' (body) VALUES (%s);'
        cur.execute(sql, (para, ))  # SQL 실행 (문단 저장)
        pcount += 1  # 문단 개수 증가
        
        # 50개 문단마다 commit() → 데이터 확정하여 효율적 저장
        if pcount % 50 == 0:
            conn.commit()
        
        # 100개 문단마다 진행 상황 출력 및 1초 대기 (서버 부담 완화)
        if pcount % 100 == 0:
            print(pcount, 'loaded...')
            time.sleep(1)
        
        para = ''  # 문단 초기화
        continue  # 다음 줄 읽기
    
    # 빈 줄이 아니라면 문단을 이어서 저장
    para = para + ' ' + line

# 마지막 commit() → 남아있는 데이터 반영
conn.commit()
cur.close()  # 커서 닫기

# 최종 결과 출력
print(' ')
print('Loaded', pcount, 'paragraphs', count, 'lines', chars, 'characters')

# 검색 최적화를 위한 GIN 인덱스 생성 SQL 출력
sql = "CREATE INDEX TABLE_gin ON TABLE USING gin(to_tsvector('english', body));"
sql = sql.replace('TABLE', base)  # 테이블명 적용
print(' ')
print('Run this manually to make your index:')
print(sql)
```

```bash
python3 loadbook.py
```

→ 이 스크립트는 `pg19337` 테이블을 생성하고, 책의 내용을 개별 행으로 삽입

### **2.3 GIN 인덱스 생성 및 검색**

검색 속도를 최적화하기 위해 **GIN 인덱스**를 생성

```sql
CREATE INDEX pg19337_gin ON pg19337 USING gin(to_tsvector('english', body));
```

특정 키워드를 검색

```sql
SELECT body FROM pg19337
WHERE to_tsquery('english', 'goose') @@ to_tsvector('english', body)
LIMIT 5;
```

---

## 3️⃣ 이메일 메시지 데이터베이스 구축

### **3.1 테이블 생성**

이메일 메시지를 저장하기 위한 테이블을 생성

```sql
CREATE TABLE IF NOT EXISTS messages (
    id SERIAL PRIMARY KEY,
    email TEXT,
    sent_at TIMESTAMPTZ,
    subject TEXT,
    headers TEXT,
    body TEXT
);
```

### **3.2 데이터 로드**

웹에서 이메일 데이터를 가져와 `messages` 테이블에 저장

```python
cur.execute(
    "INSERT INTO messages (email, sent_at, subject, headers, body) VALUES (%s, %s, %s, %s, %s)",
    ('user@example.com', '2025-03-10 10:00:00', 'Hello World', 'Header info', 'This is the email body.')
)

conn.commit()
```

### **3.3 GIN 인덱스 생성 및 활용**

이메일 본문에 대한 **전문 검색**을 위해 GIN 인덱스를 생성

```sql
CREATE INDEX messages_gin ON messages USING gin(to_tsvector('english', body));
```

검색어가 포함된 이메일 찾기

```sql
SELECT subject, email FROM messages
WHERE to_tsquery('english', 'meeting') @@ to_tsvector('english', body)
LIMIT 10;
```

---

## 4️⃣ Python을 사용한 데이터 조회 및 처리

### **4.1 데이터 조회**

Python에서 데이터베이스의 데이터를 가져오는 코드

```python
cur.execute("SELECT subject, email FROM messages WHERE body LIKE %s LIMIT 5;", ('%meeting%',))
rows = cur.fetchall()

for row in rows:
    print(row)
```

### **4.2 트랜잭션 처리**

데이터베이스 트랜잭션을 관리하는 방법

```python
try:
    cur.execute("UPDATE messages SET subject='Updated Subject' WHERE id=1;")
    conn.commit()
except Exception as e:
    conn.rollback()
    print("Error:", e)
```

---

## 5️⃣ 성능 최적화 및 인덱스 활용

### **5.1 GIN 인덱스 활용**

GIN 인덱스를 활용하면 텍스트 검색 성능이 향상

```sql
SELECT subject FROM messages
WHERE to_tsvector('english', body) @@ to_tsquery('english', 'urgent');
```

### **5.2 데이터 정규화 및 최적화**

- **중복된 데이터를 제거**하여 성능을 향상
- **인덱스를 추가**하여 검색 속도를 최적화

---

## 6️⃣ 정리

- **Python의 `psycopg2` 라이브러리를 사용하여 PostgreSQL과 연결**할 수 있습니다.
- **`psycopg2.cursor()`를 사용하여 SQL 문을 실행**하고 데이터를 저장, 수정 및 조회할 수 있습니다.
- **GIN 인덱스를 활용하여 텍스트 검색 성능을 최적화**할 수 있습니다.
- **트랜잭션 관리 및 오류 처리 기법**을 적용하여 데이터 정합성을 유지합니다.