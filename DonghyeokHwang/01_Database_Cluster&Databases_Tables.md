## 1. 데이터베이스 클러스터의 논리적 구조

<img src=".\images\chapter1\logical_structure_of_database_cluster.png">

### 데이터베이스 클러스터

PostgreSQL에서 데이터베이스 클러스터(Database Cluster)는 하나의 서버 인스턴스에서 관리되는 데이터베이스들의 모음을 의미한다. 단일 호스트에서 실행되며, 여러 개의 데이터베이스를 포함할 수 있다.

### 데이터베이스 객체와 OID

- 데이터베이스는 테이블, 인덱스, 뷰 등 다양한 데이터베이스 객체(Database Object)로 구성된다.
- PostgreSQL에서는 각 데이터베이스 자체도 데이터베이스 객체로 취급되며 논리적으로 분리된다.
- 모든 데이터베이스 객체는 객체 식별자(OID, Object Identifier)를 통해 관리되며, 이는 4바이트 크기의 부호 없는 정수이다.
- OID 정보는 `pg_database`와 `pg_class` 시스템 카탈로그에 저장된다.

```bash
# 특정 데이터베이스의 OID 조회
sampledb=# SELECT datname, oid FROM pg_database WHERE datname = 'sampledb';
 datname  |  oid
----------+-------
 sampledb | 16384
(1 row)

# 특정 테이블의 OID 조회
sampledb=# SELECT relname, oid FROM pg_class WHERE relname = 'sampletbl';
  relname  |  oid
-----------+-------
 sampletbl | 18740
(1 row)

```
&nbsp;
## 2. 데이터베이스 클러스터의 물리적 구조

### 기본 디렉토리 구조

PostgreSQL 데이터베이스 클러스터는 단일 디렉토리(기본 디렉토리)로 관리된다. 데이터베이스 클러스터를 생성할 때 `initdb` 유틸리티를 실행하면 이 기본 디렉토리가 생성되며, 일반적으로 환경 변수 `PGDATA`에 설정된다.

### 주요 파일 및 하위 디렉토리 설명

<img src=".\images\chapter1\database_cluster.png">

### 파일 목록

| 파일명 | 설명 |
| --- | --- |
| PG_VERSION | PostgreSQL의 주요 버전 번호를 저장하는 파일 |
| current_logfiles | 현재 사용 중인 로그 파일을 기록하는 파일 |
| pg_hba.conf | 클라이언트 인증을 제어하는 파일 |
| pg_ident.conf | 사용자 이름 매핑을 제어하는 파일 |
| postgresql.conf | PostgreSQL의 주요 설정을 정의하는 파일 |
| postgresql.auto.conf | `ALTER SYSTEM` 명령으로 변경된 설정을 저장하는 파일 |
| postmaster.opts | 서버가 마지막으로 시작된 옵션을 기록하는 파일 |

### 하위 디렉토리 목록

| 디렉토리명 | 설명 |
| --- | --- |
| base/ | 각 데이터베이스의 데이터를 저장하는 서브디렉토리 |
| global/ | 클러스터 전체에서 사용되는 테이블 및 설정 저장 |
| pg_wal/ | WAL(Write Ahead Logging) 로그를 저장하는 디렉토리 |
| pg_stat/ | 통계 정보를 저장하는 디렉토리 |
| pg_tblspc/ | 테이블스페이스를 관리하는 디렉토리 |

PostgreSQL의 테이블스페이스는 기본 디렉토리 외부에 있는 데이터 저장소를 의미한다. 이를 통해 데이터베이스 파일을 물리적으로 분산할 수 있다.

&nbsp;
## 3. 힙 테이블 파일의 내부 레이아웃
PostgreSQL의 데이터 파일은 **페이지(Page)** 또는 **블록(Block)** 단위로 저장되며, 기본 크기는 8KB(8192바이트)이다. 페이지 내부는 다음과 같이 구성된다.

<img src=".\images\chapter1\heap_table_file.png">

### 페이지 내부 레이아웃
1. **헤더 데이터 (PageHeaderData)**
    - 페이지에 대한 일반적인 정보를 저장하며, 길이는 24바이트이다.
    - 주요 변수:
        - `pd_lsn`: WAL 로그의 마지막 변경을 추적하는 LSN(Log Sequence Number)
        - `pd_checksum`: 페이지 체크섬 값
        - `pd_lower`, `pd_upper`: 데이터 저장 위치를 나타내는 포인터
        - `pd_special`: 인덱스 페이지에서 특수 데이터 영역의 시작 위치를 나타냄
2. **라인 포인터(Line Pointer)**
    - 각 힙 튜플(Heap Tuple)에 대한 포인터 역할
    - 4바이트 크기의 인덱스 배열로 구성되며, 새로운 튜플이 추가될 때마다 새로운 포인터가 추가된다.
3. **힙 튜플(Heap Tuple)**
    - 실제 레코드 데이터를 저장한다.
    - 페이지 하단에서부터 차곡차곡 쌓이는 구조


&nbsp;
## 4. 데이터 읽기 및 쓰기 방식

### 힙 튜플 쓰기

<img src=".\images\chapter1\write_heap_tuple.png">

1. 새로운 튜플이 추가되면 페이지의 빈 공간에 저장
2. 새로운 라인 포인터가 추가되어 해당 튜플을 가리킴
3. `pd_lower`와 `pd_upper` 값 업데이트

### 힙 튜플 읽기

<img src=".\images\chapter1\sequential_scan_and_index_scan.png">

PostgreSQL은 데이터를 읽을 때 두 가지 주요 접근 방식을 사용한다.

### 1. 순차 스캔(Sequential Scan)
- 모든 페이지를 순차적으로 읽으며, 각 페이지의 모든 라인 포인터를 탐색
- 테이블의 모든 데이터를 빠짐없이 조회할 때 유용하지만, 대량의 데이터를 읽을 경우 성능이 저하될 수 O

### 2. B-트리 인덱스 스캔(B-Tree Index Scan)
- 인덱스 파일을 사용하여 원하는 데이터의 위치를 빠르게 조회
- 인덱스 튜플은 (인덱스 키, 힙 튜플 위치)로 구성
- TID(Tuple Identifier)를 통해 특정 블록과 오프셋을 찾고, 해당 위치의 튜플을 직접 읽는다.

예를 들어, 특정 키 값이 있는 튜플의 TID가 `(block = 7, Offset = 2)`라면, PostgreSQL은 7번째 페이지의 2번째 튜플을 직접 검색하여 읽을 수 있다.