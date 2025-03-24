## 1. 프로세스 아키텍처

PostgreSQL은 단일 호스트에서 실행되는 다중 프로세스 아키텍처를 갖춘 클라이언트/서버 관계형 데이터베이스 관리 시스템(RDBMS)이다. 데이터베이스 클러스터를 관리하는 여러 프로세스가 협력하여 동작하며, 이를 통칭하여 **PostgreSQL 서버**라고 한다.

<img src=".\images\chapter2\process_architecture.png">

### 1.1 주요 프로세스 구성 요소

- **Postgres Server Process**: 데이터베이스 클러스터를 관리하는 모든 프로세스의 부모 프로세스
- **Backend Process**: 클라이언트의 연결을 처리하고, 쿼리 실행을 담당하는 개별 프로세스
- **Background Processes**: VACUUM, CHECKPOINT 등의 관리 작업을 수행하는 프로세스
- **Replication-associated Processes**: 스트리밍 복제 기능을 담당하는 프로세스
- **Background worker Processes**: 사용자가 구현한 추가 작업을 수행하는 프로세스

### 1.2 **Postgres Server Process**

Postgres 서버 프로세스는 PostgreSQL 서버 내의 모든 프로세스를 관리하는 부모 프로세스이다. `pg_ctl` 유틸리티를 사용하여 실행되며, 클라이언트의 연결을 수락하고 백엔드 프로세스를 생성한다.

기본적으로 PostgreSQL 서버는 **포트 5432**에서 수신 대기하며, 여러 서버를 운영하려면 각기 다른 포트(예: 5433, 5434 등)를 사용해야 한다.

### 1.3 **Backend Process**

백엔드 프로세스는 클라이언트가 연결할 때마다 생성되며, 해당 클라이언트가 실행하는 모든 쿼리를 처리한다. PostgreSQL의 최대 동시 연결 수는 `max_connections` 설정 값(기본값: 100)에 의해 제한된다.

백엔드 프로세스의 수가 증가하면 성능 저하가 발생할 수 있으므로, `pgbouncer` 또는 `pgpool-II`와 같은 연결 풀링(Connection Pooling) 솔루션을 사용하는 것이 일반적이다.

### 1.4 백그라운드 프로세스

PostgreSQL은 다양한 관리 작업을 수행하는 백그라운드 프로세스를 포함하고 있다.

| 프로세스 | 역할 |
| --- | --- |
| Background Writer | 공유 버퍼의 Dirty pages를 주기적으로 디스크에 기록 |
| Checkpointer | 데이터베이스의 체크포인트를 수행 |
| Autovacuum Launcher | VACUUM 작업을 트리거 |
| WAL Writer | WAL 데이터를 디스크에 주기적으로 기록 |
| Statistics Collector | 성능 및 쿼리 실행 정보를 수집 |
| Logging Collector (logger) | 오류 및 로그 메시지를 기록 |
| Archiver | WAL 로그를 아카이빙 |

### 1.5 프로세스 예시 (`pstree` 출력)

```bash
$ pstree -p 9687
-+= 00001 root /sbin/launchd
 \-+- 09687 postgres /usr/local/pgsql/bin/postgres -D /usr/local/pgsql/data
   |--= 09688 postgres postgres: logger process
   |--= 09690 postgres postgres: checkpointer process
   |--= 09691 postgres postgres: writer process
   |--= 09692 postgres postgres: wal writer process
   |--= 09693 postgres postgres: autovacuum launcher process
   |--= 09694 postgres postgres: archiver process
   |--= 09695 postgres postgres: stats collector process
   |--= 09697 postgres postgres: postgres sampledb 192.168.1.100(54924) idle
   \--= 09717 postgres postgres: postgres sampledb 192.168.1.100(54964) idle in transaction

```
&nbsp;
## 2. 메모리 아키텍처

PostgreSQL의 메모리 아키텍처는 **로컬 메모리**(Local Memory)와 **공유 메모리**(Shared Memory)로 구분된다.

<img src=".\images\chapter2\memory_architecture.png">

### 2.1 로컬 메모리 영역

각 백엔드 프로세스는 쿼리 처리를 위해 독립적인 로컬 메모리를 할당한다.

| 메모리 영역 | 설명 |
| --- | --- |
| **작업 메모리** | 정렬 및 조인 연산 시 사용 (ORDER BY, DISTINCT, Hash Join) |
| **유지보수 작업 메모리** | VACUUM, REINDEX 등의 작업 수행 시 사용 |
| **임시 버퍼** | 임시 테이블을 저장하는 용도로 사용 |

### 2.2 공유 메모리 영역

공유 메모리는 PostgreSQL 서버의 모든 프로세스가 함께 사용하는 메모리 영역으로, 서버 시작 시 할당된다.

| 메모리 영역 | 설명 |
| --- | --- |
| **공유 버퍼 풀** | 테이블 및 인덱스 페이지를 캐싱하여 성능 최적화 |
| **WAL 버퍼** | WAL 데이터를 영구 저장소에 쓰기 전에 임시 저장 |
| **커밋 로그 (CLOG)** | 트랜잭션 상태(진행 중, 커밋됨, 롤백됨)를 저장 |