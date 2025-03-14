# 1️⃣ 데이터베이스 정규화 (3NF)

## **1.1 정규화 개요**

- 데이터베이스 정규화는 **데이터 중복을 최소화하고 데이터 무결성을 유지**하기 위한 과정입니다.
- 중복 데이터를 저장하는 대신 **참조(reference)**를 통해 데이터를 연결합니다.
- 각 테이블에 **고유한 키(primary key)** 를 추가하여 참조 무결성을 보장합니다.

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255),
    email VARCHAR(255) UNIQUE
);
```

---

# 2️⃣ 관계형 vs. 비관계형 데이터베이스

## **2.1 관계형 데이터베이스 (SQL)**

- 데이터를 **행(Row)과 열(Column)** 형식으로 저장합니다.
- 데이터 정합성을 유지하기 위해 **ACID 원칙**을 따릅니다.
- 대표적인 SQL 데이터베이스:
    - **PostgreSQL**
    - **MySQL**
    - **Oracle**
    - **SQL Server**

## **2.2 비관계형 데이터베이스 (NoSQL)**

- 데이터를 **문서(Document), 키-값(Key-Value), 그래프(Graph)** 형태로 저장합니다.
- 데이터 정합성보다 **확장성과 속도**에 초점을 둡니다.
- 대표적인 NoSQL 데이터베이스:
    - **MongoDB** (문서 기반)
    - **Cassandra** (키-값 기반)
    - **BigTable** (구글의 대용량 분산 데이터베이스)

---

# 3️⃣ ACID vs. BASE

## **3.1 ACID 모델 (신뢰성이 중요한 시스템)**

ACID는 데이터의 **일관성과 무결성**을 보장하기 위한 원칙입니다.

- **A (Atomicity, 원자성)**: 트랜잭션이 **완전히 수행되거나 완전히 롤백**됨.
- **C (Consistency, 일관성)**: 데이터베이스가 **일관된 상태**를 유지.
- **I (Isolation, 고립성)**: 트랜잭션 간의 **간섭 방지**.
- **D (Durability, 지속성)**: 트랜잭션이 **성공적으로 저장되면 영구 보존**.

ACID를 지원하는 데이터베이스 예시:

- **PostgreSQL**
- **Oracle**
- **MySQL**
- **SQLite**

## **3.2 BASE 모델 (대규모 분산 시스템)**

BASE는 확장성을 강조하는 **NoSQL 데이터베이스의 원칙**입니다.

- **BA (Basically Available, 기본적인 가용성)**: 일부 장애가 발생해도 **서비스 지속 가능**.
- **S (Soft-state, 소프트 상태 유지)**: 데이터가 즉시 일관되지 않을 수 있음.
- **E (Eventual Consistency, 최종적 일관성)**: 시간이 지나면 **일관된 상태**를 유지.

BASE를 지원하는 데이터베이스 예시:

- **MongoDB**
- **Cassandra**
- **BigTable**

---

# 4️⃣ 데이터베이스 확장 기법

## **4.1 수직 확장 (Vertical Scaling)**

- 기존 **하나의 서버에서 CPU, RAM, 디스크 등의 성능을 업그레이드**하는 방식.
- 서버 자체의 성능이 올라가므로 **구조 변경 없이 확장**이 가능함.
- 간단하지만, **비용이 급격히 증가**하는 한계가 있음.

| 장점 | 단점 |
| --- | --- |
| **구현이 간단함** → 추가적인 네트워크 설정이 필요 없음 | **비용이 급격히 증가** → 고성능 하드웨어는 매우 비쌈 |
| **데이터 일관성(Consistency) 유지** → 단일 서버에서 처리 | **확장 한계 존재** → 한 서버의 성능에는 물리적 한계가 있음 |
| **트랜잭션 처리 안정성 유지** → ACID 보장 용이 | **단일 장애점(Single Point of Failure, SPOF)** → 서버가 다운되면 전체 시스템이 멈춤 |

## **4.2 수평 확장 (Horizontal Scaling)**

- 서버를 여러 대 추가하여 **부하를 분산(Load Balancing)** 하는 방식.
- 데이터베이스의 데이터를 **여러 서버에 나누어 저장하거나 복제(Replication)하여 운영**.
- **마스터-슬레이브 복제(Master-Slave Replication)**, **샤딩(Sharding)** 등의 기법을 사용.

### 마스터-슬레이브 복제 (Master-Slave Replication)

- **하나의 마스터(Master) 서버가 쓰기(Write) 연산을 담당**하고,**여러 개의 슬레이브(Slave) 서버가 읽기(Read) 연산을 담당하는 구조**.
- 데이터를 **자동으로 복제(Sync)** 하여 **읽기 성능을 향상**시킬 수 있음

- **클라이언트가 마스터 서버에 데이터를 저장 (INSERT, UPDATE, DELETE)**
- **슬레이브 서버는 마스터 서버의 변경 사항을 자동으로 복제 (Read 전용)**
- **클라이언트는 슬레이브 서버에서 데이터를 읽을 수 있음 (SELECT)**

```sql
-- 마스터-슬레이브 복제 예시
SELECT * FROM replica_db.users;
```

**✅ 장점**

- **읽기 성능 향상** → 슬레이브 서버를 늘려 부하를 분산 가능.
- **백업 및 장애 복구 용이** → 마스터가 다운되면 슬레이브를 마스터로 승격 가능.

**⚠️ 단점**

- **쓰기 성능은 개선되지 않음** → 마스터 서버가 여전히 쓰기 연산을 담당.
- **일관성 문제 발생 가능** → 데이터가 실시간으로 동기화되지 않을 수 있음.

📌 **실제 사례**

- **은행 시스템**: 읽기 요청(잔액 조회)은 많지만, 쓰기 요청(송금)은 상대적으로 적음.
- **검색 엔진**: 사용자 검색 요청을 여러 개의 슬레이브 서버에서 처리하여 속도 향상.

### 샤딩 (Sharding)

- 데이터를 여러 개의 **서버(샤드, Shard)에 분산 저장**하여 확장하는 방식.
- 특정 기준(예: 사용자 ID, 지역, 날짜)에 따라 **데이터를 나누어 저장**.

- **각 샤드(Shard)는 서로 다른 데이터 조각을 저장**.
- **클라이언트는 샤드 키(Shard Key)를 기준으로 적절한 서버에서 데이터를 검색**.
- **데이터가 증가하면 샤드를 추가하여 확장 가능**.

```sql
-- 샤드1 (유저 ID 1~1000 저장)
CREATE TABLE users_shard1 (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);

-- 샤드2 (유저 ID 1001~2000 저장)
CREATE TABLE users_shard2 (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);
```

**✅ 장점**

- **쓰기(Write) 및 읽기(Read) 성능 개선** → 데이터가 여러 서버에 분산되므로 부하 분산 가능.
- **확장성 뛰어남** → 새로운 샤드를 추가하면 쉽게 확장 가능.

**⚠️ 단점**

- **복잡한 쿼리 실행 어려움** → 여러 샤드에서 데이터를 가져와야 하는 경우 JOIN이 어려움.
- **샤드 키 설계가 중요** → 잘못 설계하면 특정 샤드에 부하가 집중됨(Hot Spot 문제).

📌 **실제 사례**

- **SNS (예: 트위터, 페이스북)**
    - 사용자 ID를 기준으로 샤딩하여 **특정 지역 또는 그룹 단위로 데이터를 분산**.
- **대형 전자상거래 (예: 아마존)**
    - 주문 데이터를 샤딩하여 **고객별로 주문 내역을 관리**.

## **수직 확장 vs. 수평 확장**

| 방식 | 수직 확장 (Vertical Scaling) | 수평 확장 (Horizontal Scaling) |
| --- | --- | --- |
| **확장 방식** | 서버 성능 업그레이드 | 서버 개수 증가 |
| **비용** | 하드웨어 비용 증가 | 서버 추가 비용 |
| **트랜잭션 일관성** | 보장 쉬움 (ACID) | 보장 어려움 (BASE) |
| **확장성** | 한계 존재 | 무제한 확장 가능 |
| **장애 대응** | SPOF(단일 장애점) 존재 | 고가용성(HA) 제공 |

**수직 확장(Vertical Scaling)이 적합한 경우**

- 트래픽이 크지 않고 **소규모 서비스**.
- 데이터 일관성이 매우 중요한 경우 (예: 은행, 기업 데이터 관리).

**수평 확장(Horizontal Scaling)이 적합한 경우**

- **대규모 트래픽**이 발생하는 경우 (예: SNS, 게임, 검색 엔진).
- 서버 장애 발생 시 **무중단 서비스**가 필요한 경우 (예: 전자상거래, 글로벌 서비스).

---

# 5️⃣ 데이터베이스 분산 구조

## **5.1 마스터-슬레이브 복제**

- **마스터(Master)**: 모든 쓰기 연산을 처리.
- **슬레이브(Slave)**: 읽기 전용으로 데이터 복제.
- **장점**: 읽기 부하 분산 가능.
- **단점**: 쓰기 연산 부하를 줄이기 어려움.

## **5.2 멀티 마스터 복제 (Multi-Master)**

- **여러 개의 마스터 노드**가 존재하며, 각각의 마스터에서 **쓰기 연산 가능**.
- **장점**: 쓰기 부하 분산.
- **단점**: 데이터 충돌 가능성이 있음.

```sql
-- Multi-Master Replication을 위한 트랜잭션 예제
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

---

# 6️⃣ 클라우드 데이터베이스 및 NoSQL

## **6.1 클라우드 데이터베이스**

- 클라우드 환경에서 제공되는 **관리형 데이터베이스 서비스**.
- **물리적인 서버 운영 없이**, 사용자는 데이터베이스 기능만 이용할 수 있음.
- 대표적인 클라우드 데이터베이스 서비스:
    - **Google Cloud SQL**
    - **Amazon RDS (Relational Database Service)**
    - **Microsoft Azure SQL Database**
    - **Firebase Firestore (NoSQL 기반)**
    - **MongoDB Atlas (NoSQL 관리형 서비스)**

| 특징 | 설명 |
| --- | --- |
| **확장성 (Scalability)** | 수직/수평 확장이 쉽고, 트래픽 증가에 대응 가능 |
| **자동 관리 (Managed Service)** | 유지보수, 백업, 패치, 성능 최적화 등이 자동화됨 |
| **고가용성 (High Availability)** | 여러 리전(Region)에 걸쳐 데이터 복제 가능 |
| **보안 (Security)** | 자동 암호화, 접근 제어, 방화벽 관리 제공 |
| **비용 효율성** | 사용한 만큼만 요금 부과 (Pay-as-you-go 모델) |

📌 **실제 사용 사례**

- **스타트업**: 인프라 운영 인력이 부족한 경우, 클라우드 DB를 활용하여 서비스 구축.
- **대기업**: 글로벌 확장이 필요한 서비스에서 클라우드 DB를 활용하여 장애 발생 시 빠른 복구 가능.
- **E-commerce (전자상거래)**: **Amazon RDS + DynamoDB**를 조합하여 대량의 주문 데이터를 실시간 처리.

## **6.2 NoSQL의 등장**

- **Not Only SQL**의 약자로, **관계형 데이터베이스(RDBMS)와 다른 구조를 가진 데이터베이스**
- 기존 RDBMS가 **스키마(테이블 구조)와 정형 데이터**에 최적화되어 있었다면,**NoSQL은 비정형 데이터, JSON 데이터, 대규모 트래픽 처리**에 적합하게 설계

| NoSQL 유형 | 설명 | 대표적인 데이터베이스 |
| --- | --- | --- |
| **Key-Value Store** | 키(Key)와 값(Value) 형태로 데이터를 저장 | **Redis, DynamoDB, Riak** |
| **Document Store** | JSON, BSON 같은 문서(Document) 형태로 저장 | **MongoDB, CouchDB, Firebase Firestore** |
| **Column-Family Store** | 컬럼 기반으로 데이터를 저장 | **Apache Cassandra, HBase, Bigtable** |
| **Graph Database** | 그래프 구조로 관계 데이터를 저장 | **Neo4j, ArangoDB** |

```json
{
    "name": "John Doe",
    "email": "johndoe@example.com",
    "orders": [
        {"item": "Laptop", "price": 1200},
        {"item": "Mouse", "price": 20}
    ]
}
```

```jsx
db.users.insertOne({
    "name": "John Doe",
    "email": "johndoe@example.com",
    "orders": [
        {"item": "Laptop", "price": 1200},
        {"item": "Mouse", "price": 20}
    ]
});
```

```jsx
db.users.find({ "name": "John Doe" });
```