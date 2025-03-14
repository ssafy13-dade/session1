# 1️⃣ Elasticsearch 개요

## **1.1 Elasticsearch란?**

> **대량의 데이터를 실시간으로 검색, 분석, 저장** 할 수 있는 **오픈 소스 검색 엔진**
> 
- **Apache Lucene**을 기반으로 개발되었으며, **빠르고 확장 가능한(Scalable) 검색 및 데이터 분석**

기능을 제공

- 대부분의 **로그 분석, 추천 시스템, 검색 엔진**에서 Elasticsearch를 활용

- **오픈 소스 검색 엔진**으로, Apache Lucene 기반.
- **확장 가능(Scalable)**한 데이터 검색 시스템.
- **역색인(Inverted Index) 기반**의 전체 텍스트 검색 지원.
    - Elasticsearch는 **Inverted Index(역색인) 구조**를 사용하여 **빠른 검색**
    - 데이터가 저장될 때, **검색을 빠르게 하기 위한 인덱스(Index)를 자동 생성**.
- **랭킹(Relevance) 및 추천 시스템** 기능 제공.
- **NoSQL 데이터베이스 기능**을 포함.
- **Google과 유사한 검색 기능 제공**.

## **1.2 Elasticsearch의 역사**

- 기존의 검색 엔진을 **오픈 소스로 제공하고자 하는 목적**에서 개발됨.
- 초창기에는 단순 검색 엔진이었으나, **분산 데이터베이스로 발전**.
- 현재는 ELK Stack (Elasticsearch, Logstash, Kibana)의 핵심 구성 요소.

---

# 2️⃣ Elasticsearch의 라이선스 및 적용 사례

## **2.1 라이선스**

- **"오픈 코어(Open Core)" 모델**을 따름
- 핵심 기능은 **Apache 2.0 라이선스**로 제공
- Elastic 회사는 **추가적인 서비스(호스팅, 컨설팅, 프리미엄 기능)**을 판매

## **2.2 대표적인 활용 사례**

### 1) 학습 관리 시스템(LMS) - Sakai

- 대학 및 교육 기관에서 **온라인 강의 자료, 시험, 과제 관리** 등에 사용됨.
- **Elasticsearch를 검색 엔진으로 활용하여 학생들이 강의 콘텐츠를 쉽게 찾을 수 있도록 함**.
- 키워드를 입력하면 **강의자료, 토론게시판, 과제 등에서 관련 내용을 빠르게 검색**.

### **2) ELK Stack**

- **Elasticsearch를 활용한 데이터 분석 및 로그 모니터링 시스템**
- **E → Elasticsearch**: 분산형 NoSQL 데이터베이스.
- **L → Logstash**: 실시간 데이터 수집 및 변환.
- **K → Kibana**: 데이터 시각화 및 대시보드 제공.

```bash
# Kibana 대시보드 링크
<https://www.elastic.co/guide/en/kibana/current/dashboard.html>
```

---

# 3️⃣ Elasticsearch 아키텍처

## **3.1 기본 아키텍처**

- **RESTful API** 기반의 웹 서비스.
    - Elasticsearch는 **HTTP RESTful API**를 통해 데이터를 저장하고 검색할 수 있음.
    - 즉, **SQL 쿼리 대신 HTTP 요청(GET, POST, PUT, DELETE)을 사용**하여 데이터를 처리.
- **JSON 형식의 데이터 저장 및 검색**.
    - Elasticsearch는 **모든 데이터를 JSON 형식으로 저장**.
    - NoSQL 데이터베이스처럼 **문서(Document) 기반 저장 방식**을 사용.
- **이벤트 기반(Eventual Consistency)** 데이터 모델.
    - Elasticsearch는 **즉각적인 데이터 일관성(Strong Consistency)을 보장하지 않음**.
    - 대신, **최종적 일관성(Eventual Consistency)**을 따름.
        - 즉, 데이터를 여러 노드에 복제할 때 시간이 걸릴 수 있음.
        - 하지만 일정 시간이 지나면 모든 노드에서 일관된 데이터를 유지하게 됨.

## **3.2 주요 개념**

### **인덱스(Index)**

> 데이터가 저장되는 기본 단위.
> 
- **Elasticsearch에서 데이터를 저장하는 기본 단위**.
- **SQL 데이터베이스의 "테이블(Table)"과 유사한 개념**.
- 하나의 인덱스에는 **여러 개의 도큐먼트(Document)**가 저장될 수 있음.

### **도큐먼트(Document)**

> JSON 형식으로 저장되는 개별 데이터.
> 
- **Elasticsearch에서 저장되는 개별 데이터 단위**.
- **JSON 형식으로 저장**되며, 각 도큐먼트는 **고유한 ID**를 가짐.

### **샤드(Shard)**

> 데이터를 여러 서버에 분산 저장하여 성능을 향상.
> 
- **Elasticsearch는 데이터를 여러 개의 샤드(Shard)로 나누어 저장**하여 성능을 향상.
- **샤드란?** 대량의 데이터를 작은 단위로 나누어 저장하는 방식.
- *샤딩(Sharding)**을 통해 **검색 속도 향상 및 대량 데이터 처리 가능**.

### **노드(Node)**

> 개별적인 Elasticsearch 서버 인스턴스.
> 
- **Elasticsearch에서 실행되는 개별 서버 인스턴스**.
- 여러 개의 노드가 모여 **클러스터(Cluster)를 형성**.
- Elasticsearch는 **수평 확장(Scale-out)이 가능**하여, 노드를 추가하면 성능이 향상됨.

### **클러스터(Cluster)**

> 여러 개의 노드가 모여 하나의 검색 엔진을 형성.
> 
- **여러 개의 노드가 모여 하나의 Elasticsearch 클러스터를 형성**.
- 클러스터에는 **하나의 마스터 노드(Master Node)**가 존재하며, 전체 시스템을 관리.

```json
{
    "index": "my_index",
    "type": "_doc",
    "id": "1",
    "body": {
        "title": "Elasticsearch Tutorial",
        "content": "Elasticsearch is a powerful search engine"
    }
}
```

---

# 4️⃣ Elasticsearch의 데이터 저장 및 검색

## **4.1 기본적인 검색 쿼리 (Match All Query)**

모든 문서를 검색하는 가장 기본적인 쿼리.

```json
GET /_search
{
    "query": {
        "match_all": {}
    }
}
```

**Python을 활용한 검색 예제:**

```python
import requests
import json

query_url = "<http://pg4e_86f9:*@es.py4e.com:9210/prefx/testindex/_search?pretty>"
query_body = json.dumps({"query": {"match_all": {}}})
headers = {'Content-type': 'application/json; charset=UTF-8'}

response = requests.post(query_url, headers=headers, data=query_body)
print(response.json())
```

---

# 5️⃣ Python과 Elasticsearch 연동

## **5.1 Python Elasticsearch 라이브러리 설치**

```bash
pip3 install elasticsearch
```

## **5.2 Elasticsearch 연결 및 검색**

```python
from elasticsearch import Elasticsearch

es = Elasticsearch(
    ["<http://localhost:9200>"],
    http_auth=("user", "password"),
    scheme="http"
)

# 검색 쿼리 실행
res = es.search(index="testindex", body={"query": {"match_all": {}}})
print(res)
```

---

# 6️⃣ Elasticsearch의 주요 기능

## **6.1 전체 텍스트 검색**

- 역색인(Inverted Index)을 활용하여 빠른 검색 수행.
- 자연어 처리(NLP)와 연동 가능.

## **6.2 분산 검색 및 데이터 저장**

- **샤딩(Sharding)**을 통해 대규모 데이터를 여러 노드에 저장.
- **레플리케이션(Replication)**으로 고가용성(High Availability) 보장.

## **6.3 실시간 데이터 분석**

- **ELK Stack (Elasticsearch + Logstash + Kibana)**을 통해 **로그 및 실시간 이벤트 분석** 가능.

---

# 7️⃣ Elasticsearch의 장점 및 활용

## **7.1 장점**

✅ **빠른 검색 성능**: 역색인을 활용하여 대량의 데이터를 빠르게 검색.

✅ **확장성(Scalability)**: 수평 확장이 가능하여 데이터 증가 시 손쉽게 확장 가능.

✅ **JSON 기반의 REST API**: 간편한 API 연동 가능.

✅ **NoSQL 데이터베이스 기능**: 문서 저장 및 쿼리 기능 제공.

## **7.2 Elasticsearch를 활용한 검색 시스템**

- **검색 엔진**: 웹사이트 검색, 문서 검색, 로그 분석.
- **추천 시스템**: 사용자의 관심사를 기반으로 추천.
- **실시간 모니터링**: 로그 및 보안 이벤트 감지.

```json
{
    "query": {
        "match": {
            "content": "machine learning"
        }
    }
}
```