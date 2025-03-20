## 1. 백엔드 프로세스

백엔드 프로세스는 기본적으로 클라이언트가 발행한 모든 쿼리를 처리하며, 이는 5개의 주요 하위 시스템으로 구성된다.

<img src=".\images\chapter3\query_processing.png">

1. **Parser (파서)**: SQL 명령문을 분석하여 파스 트리를 생성
2. **Analyzer (분석기)**: 구문 트리의 의미 분석을 수행하여 쿼리 트리를 생성
3. **Rewriter (리라이터)**: 규칙 시스템을 기반으로 쿼리 트리를 변환
4. **Planner (플래너)**: 최적의 실행 계획을 수립하여 플랜 트리를 생성
5. **Executor (실행자)**: 플랜 트리를 기반으로 데이터를 검색하고 처리

&nbsp;
## 2. 파서 (Parser)

파서는 SQL 명령문을 구문 분석하여 파서 트리(Parse Tree)를 생성한다.

다음과 같은 쿼리를 가정했을 때,

```sql
testdb=# SELECT id, data FROM tbl_a WHERE id < 300 ORDER BY data;
```

파서는 이 쿼리의 구문을 분석하여 아래와 같은 트리 구조로 변환한다.

<img src=".\images\chapter3\parse_tree.png">

파서의 역할은 **구문 오류(Syntax Error)를 감지**하는 것이며, 의미론적 오류(Semantic Error)는 감지하지 않는다. 즉, 존재하지 않는 테이블을 참조하더라도 파서는 오류를 반환하지 않는다. 이러한 검사는 분석기에서 수행된다.

&nbsp;
## 3. 분석기 (Analyzer)

분석기는 파서가 생성한 파스 트리를 쿼리 트리(Query Tree)로 변환한다. 이 과정에서 테이블과 컬럼의 존재 여부를 확인하고, `*` (와일드카드)를 실제 컬럼 목록으로 확장한다.

<img src=".\images\chapter3\query_tree.png">

쿼리 트리의 주요 구성 요소는 다음과 같다.

- **targetList**: 반환할 컬럼 목록을 저장
- **rtable**: 조회 대상 테이블의 정보를 저장
- **jointree**: FROM 절과 WHERE 절을 저장
- **sortClause**: ORDER BY 절을 저장

분석이 완료된 후, 리라이터가 규칙을 적용하여 쿼리를 변환할 수 있다.

&nbsp;
## 4. 리라이터 (Rewriter)

리라이터는 규칙 시스템(Rule System)을 기반으로 쿼리를 변환합니다. 규칙이 존재하는 경우, `pg_rules` 시스템 카탈로그에 저장된 정보를 활용하여 쿼리 트리를 수정한다.

예를 들어, 뷰(View)는 내부적으로 규칙 시스템을 통해 변환되며, 기본 테이블을 조회하는 방식으로 변경된다.

&nbsp;
## 5. 플래너 및 실행자 (Planner & Executor)

### 5.1 플래너 (Planner)

플래너는 최적의 실행 계획을 수립하여 플랜 트리(Plan Tree)를 생성한다. PostgreSQL의 플래너는 **비용 기반 최적화(Cost-Based Optimization, CBO)** 방식을 사용한다.

아래는 실행 계획을 확인하는 코드이다.

```bash
testdb=# EXPLAIN SELECT * FROM tbl_a WHERE id < 300 ORDER BY data;
                          QUERY PLAN
---------------------------------------------------------------
 Sort  (cost=182.34..183.09 rows=300 width=8)
   Sort Key: data
   ->  Seq Scan on tbl_a  (cost=0.00..170.00 rows=300 width=8)
         Filter: (id < 300)
(4 rows)
```

위의 실행 계획에서, **순차 검색(Sequential Scan)** 후 **정렬(Sort)** 작업이 수행되는 것을 알 수 있다.

### 5.2 실행자 (Executor)

실행자는 플랜 트리를 따라 쿼리를 실행한다. 실행자의 주요 역할은 테이블과 인덱스에 접근하여 데이터를 가져오고, 필요한 연산을 수행하는 것이다.

<img src=".\images\chapter3\plan_tree.png">

예를 들어, 위의 플랜 트리에서 실행자는 다음과 같은 단계를 수행한다.

1. 테이블 `tbl_a`를 순차 검색(Sequential Scan)하여 `id < 300` 조건을 만족하는 데이터를 필터링한다.
2. 필터링된 데이터를 `ORDER BY data` 기준으로 정렬한다.