---
tags:
  - cs/ai
  - type/interview-question
  - interview/backend
  - ai/vector-db
  - ai/embedding
  - db/index
importance: high
status: seed
source:
  - user
aliases:
  - Vector DB
  - Vector Database
  - 벡터 데이터베이스
  - 벡터DB
---

# Vector DB는 무엇이고 유사도 검색은 어떻게 동작하는가?

## 질문 의도

이 질문은 단순히 "AI에서 쓰는 DB"를 아는지보다, 임베딩 벡터를 왜 저장하는지, 기존 DB/검색 엔진과 무엇이 다른지, 유사도 검색이 어떻게 동작하는지, 그리고 RAG나 추천 시스템에서 어떤 운영 문제가 생기는지 이해하는지를 확인한다.

## 키워드

- Vector DB
- Embedding
- Similarity Search
- Approximate Nearest Neighbor, ANN
- HNSW
- Cosine Similarity
- Dot Product
- Euclidean Distance
- Metadata Filtering
- Hybrid Search
- RAG
- Re-indexing

## 전체 내용 정리

### 1. 개념

Vector DB는 텍스트, 이미지, 오디오, 상품, 문서 같은 데이터를 임베딩 모델로 변환한 고차원 벡터를 저장하고, 입력 쿼리 벡터와 의미적으로 가까운 벡터를 빠르게 찾는 데이터베이스다.

예를 들어 "환불 정책 알려줘"라는 질문을 임베딩하면 수백~수천 차원의 숫자 배열이 된다. Vector DB는 이 벡터와 가까운 문서 벡터를 찾아 "환불", "결제 취소", "반품 규정"처럼 단어가 정확히 같지 않아도 의미가 비슷한 데이터를 검색할 수 있게 한다.

보통 Vector DB에는 벡터만 저장하지 않고 다음을 함께 저장한다.

| 데이터 | 예시 | 역할 |
| --- | --- | --- |
| id | `doc_123_chunk_4` | 검색 결과 식별 |
| vector | `[0.12, -0.03, ...]` | 유사도 계산 대상 |
| text/source | 원문 chunk, 문서 URL | 답변 근거 또는 후처리 |
| metadata | tenant, role, category, updated_at | 필터링, 권한 제어, 정렬 |

### 2. 등장 배경 / 필요한 이유

기존 키워드 검색은 단어가 정확히 일치하거나 비슷한 형태로 등장해야 검색이 잘 된다. 하지만 사람의 질문은 표현이 다양하다. "비밀번호를 잊었어", "로그인이 안 돼", "계정 복구 방법"은 단어는 다르지만 의미적으로 가까울 수 있다.

Vector DB는 이런 의미 기반 검색을 위해 등장했다. 특히 RAG에서는 LLM이 답변하기 전에 관련 문서를 찾아야 하므로, 문서를 임베딩해서 Vector DB에 넣고 질문과 가까운 문서를 top-k로 검색한다.

다만 Vector DB는 "정답을 이해하는 DB"가 아니다. 임베딩 모델이 만든 벡터 공간에서 가까운 항목을 찾는 시스템이다. 따라서 임베딩 품질, chunking, metadata, index 구조, reranking이 검색 품질을 크게 좌우한다.

### 3. 동작 원리

일반적인 Vector DB 검색 흐름은 다음과 같다.

1. 문서를 수집하고 적절한 단위로 chunking한다.
2. 각 chunk를 embedding model로 벡터화한다.
3. Vector DB에 `id`, `vector`, `text`, `metadata`를 저장한다.
4. 사용자의 질문도 같은 embedding model로 벡터화한다.
5. metadata filter가 있으면 검색 범위를 좁힌다.
6. 벡터 간 유사도를 계산해 가까운 후보를 찾는다.
7. top-k 결과를 반환한다.
8. 필요하면 reranker나 keyword search 결과와 결합한다.

벡터 간 가까움을 계산하는 대표 방식은 다음과 같다.

| 방식 | 의미 | 주로 보는 관점 |
| --- | --- | --- |
| Cosine Similarity | 두 벡터의 방향이 얼마나 비슷한가 | 문장 의미 유사도 |
| Dot Product | 방향과 크기를 함께 반영한 내적 값 | 모델이 dot product 기준으로 학습된 경우 |
| Euclidean Distance | 벡터 공간에서 실제 거리가 얼마나 가까운가 | 거리 기반 검색 |

데이터가 적으면 모든 벡터와 비교하는 exact search도 가능하다. 하지만 벡터가 수백만~수십억 개가 되면 모든 벡터를 매번 비교하기 어렵다. 그래서 실무 Vector DB는 보통 ANN, Approximate Nearest Neighbor 검색을 사용한다. ANN은 정확도를 조금 양보하고 훨씬 빠르게 가까운 후보를 찾는다.

### 4. 구성 요소

| 구성 요소 | 역할 |
| --- | --- |
| Embedding Model | 원문과 쿼리를 같은 의미 공간의 벡터로 변환 |
| Vector Store | 벡터, 원문, metadata 저장 |
| Vector Index | 빠른 근접 이웃 검색을 위한 자료구조 |
| Similarity Metric | cosine, dot product, euclidean 등 점수 계산 기준 |
| Metadata Filter | 권한, 카테고리, 날짜, tenant 기준으로 검색 범위 제한 |
| Retriever | 검색 요청을 만들고 top-k 결과를 가져오는 계층 |
| Reranker | 1차 검색 결과를 더 정밀하게 재정렬 |
| Re-index Pipeline | 문서 변경, 임베딩 모델 변경, 삭제를 인덱스에 반영 |

대표적인 인덱스 방식에는 HNSW, IVF, PQ 등이 있다. HNSW는 그래프를 따라 가까운 벡터를 탐색하는 방식으로 검색 속도와 recall이 좋아 많이 쓰인다. 대신 메모리 사용량이 커질 수 있고, 삽입/삭제/튜닝 비용을 고려해야 한다.

### 5. 예시

사내 문서 검색 챗봇을 만든다고 가정하자.

1. 사내 위키 문서를 수집한다.
2. 문서를 제목, 섹션, 문단 단위로 chunking한다.
3. 각 chunk를 embedding한다.
4. Vector DB에 chunk vector와 원문, 문서 URL, 부서, 권한 정보를 저장한다.
5. 사용자가 "배포 실패 시 롤백 절차 알려줘"라고 질문한다.
6. 질문을 벡터화하고 `role=backend`, `service=payment` 같은 metadata filter를 적용한다.
7. Vector DB에서 의미적으로 가까운 runbook chunk를 top-k로 찾는다.
8. 검색 결과를 LLM 컨텍스트에 넣어 답변하게 한다.

이때 권한 metadata가 빠지면 사용자가 볼 수 없는 문서가 답변에 섞일 수 있다. 반대로 chunk가 너무 작거나 임베딩 모델이 도메인 용어를 잘 모르면 정답 문서가 검색되지 않을 수 있다.

### 6. 장단점 / 트레이드오프

| 관점 | 장점 | 단점 / 주의점 |
| --- | --- | --- |
| 검색 품질 | 단어가 달라도 의미가 비슷하면 검색 가능 | 의미는 비슷하지만 답은 아닌 문서도 높게 나올 수 있음 |
| 성능 | ANN 인덱스로 대규모 벡터를 빠르게 검색 | recall과 latency 사이의 튜닝이 필요 |
| RAG 연동 | 문서 기반 LLM 답변의 핵심 검색 저장소로 사용 가능 | Vector DB만 붙인다고 RAG 품질이 보장되지 않음 |
| 운영 | metadata와 함께 저장해 필터링 가능 | 권한, 삭제, 최신성 반영을 놓치면 위험 |
| 비용 | 의미 검색 기능을 빠르게 구축 가능 | 벡터 저장 공간, 메모리, 인덱스 재구축 비용 발생 |
| 변경 대응 | 문서 추가는 비교적 쉬움 | embedding model 변경 시 전체 재임베딩/재인덱싱 필요 |

### 7. 헷갈리기 쉬운 지점

- Vector DB는 LLM 자체가 아니다. 벡터를 저장하고 가까운 데이터를 찾는 검색 저장소다.
- 벡터 유사도가 높다고 항상 정답 근거라는 뜻은 아니다.
- 키워드 검색을 완전히 대체하지 않는다. 정확한 상품명, 에러 코드, 고유명사는 BM25 같은 keyword search가 더 강할 수 있다.
- `top-k`를 크게 하면 recall은 늘 수 있지만 잡음, 비용, context 낭비도 늘어난다.
- 임베딩 모델이 바뀌면 기존 벡터와 새 쿼리 벡터의 의미 공간이 달라질 수 있어 재인덱싱이 필요하다.
- metadata filter는 성능과 recall에 영향을 줄 수 있다. 특히 필터를 먼저 적용할지, ANN 검색 후 적용할지에 따라 결과가 달라질 수 있다.

### 8. 실무 연결

실무에서 Vector DB를 운영할 때는 검색 품질보다 먼저 데이터 동기화와 권한 제어가 문제가 되는 경우가 많다. 문서가 삭제되었는데 벡터 인덱스에 남아 있거나, 권한 변경이 metadata에 반영되지 않으면 잘못된 정보가 검색될 수 있다.

또한 Vector DB는 "넣으면 알아서 잘 찾는" 시스템이 아니다. 좋은 검색 품질을 위해서는 chunking 전략, embedding model 선택, metadata 설계, hybrid search, reranking, 평가셋 관리가 필요하다.

RAG 서비스에서는 다음 지표를 함께 본다.

| 지표 | 의미 |
| --- | --- |
| Recall@k | 정답 근거 문서가 top-k 안에 들어왔는가 |
| Precision@k | 검색 결과 중 실제 관련 문서 비율 |
| Latency | 검색 응답 시간이 서비스 요구사항을 만족하는가 |
| Index Freshness | 최신 문서 변경이 인덱스에 반영되었는가 |
| Faithfulness | 생성 답변이 검색 근거에 충실한가 |

## 관련 개념 링크

- [[RAG는 무엇이고 왜 파인튜닝만으로는 부족한가?|RAG]]
- 후보: [[Embedding]], [[BM25]], [[Hybrid Search]], [[HNSW]], [[ANN 검색]], [[Reranker]]

## 꼬리 질문과 답변

### 1. Vector DB는 일반 RDB와 무엇이 다른가?

**답변**

RDB는 row, column, index, SQL을 중심으로 정확한 조건 검색과 트랜잭션 처리에 강하다. Vector DB는 고차원 벡터를 저장하고 유사도 기반으로 가까운 데이터를 찾는 데 특화되어 있다. 즉 RDB는 "조건에 맞는 데이터"를 찾는 데 강하고, Vector DB는 "의미적으로 비슷한 데이터"를 찾는 데 강하다. 다만 최근에는 RDB도 vector extension을 제공하기 때문에, 규모와 운영 요구사항에 따라 전용 Vector DB와 RDB 확장을 비교해야 한다.

### 2. Vector DB가 있는데 왜 keyword search나 BM25를 같이 쓰는가?

**답변**

벡터 검색은 의미 유사도에 강하지만 정확한 키워드, 에러 코드, API 이름, 상품 ID 같은 검색에는 약할 수 있다. 예를 들어 `ERR_CONN_RESET` 같은 문자열은 의미보다 정확한 일치가 중요하다. 그래서 실무에서는 vector search와 BM25를 결합한 hybrid search를 많이 사용한다. 이후 reranker로 최종 순서를 다시 매기면 recall과 precision을 함께 개선할 수 있다.

### 3. ANN 검색은 왜 필요하고 어떤 tradeoff가 있는가?

**답변**

벡터 수가 많아지면 모든 벡터와 쿼리 벡터를 비교하는 exact search는 비용이 너무 커진다. ANN은 정확한 최근접 이웃을 항상 보장하지 않는 대신, 매우 빠르게 가까운 후보를 찾는다. tradeoff는 recall과 latency다. 검색을 더 꼼꼼하게 하면 정답 후보를 찾을 확률은 높아지지만 응답 시간이 늘고, 더 빠르게 찾으면 일부 정답 후보를 놓칠 수 있다.

### 4. Vector DB 검색 결과가 엉뚱할 때 어디를 점검해야 하는가?

**답변**

먼저 query와 document가 같은 embedding model로 만들어졌는지 확인해야 한다. 그다음 chunking이 너무 작거나 크지 않은지, metadata filter가 정답 문서를 제외하고 있지 않은지, similarity metric이 모델에 맞는지 확인한다. 정답 문서가 top-k 안에는 있지만 순위가 낮다면 reranking이나 hybrid search가 필요할 수 있다. 문서가 오래되었거나 삭제 반영이 안 된 경우에는 인덱싱 파이프라인을 점검해야 한다.

### 5. RAG에서 Vector DB를 설계할 때 가장 중요한 운영 포인트는 무엇인가?

**답변**

권한 metadata, 인덱스 최신성, 평가셋 관리가 중요하다. 사용자가 볼 수 없는 문서가 검색되면 보안 사고가 될 수 있고, 오래된 벡터가 남아 있으면 잘못된 답변이 생성될 수 있다. 또한 검색 품질은 감으로 판단하기 어렵기 때문에 실제 질문과 정답 근거 문서로 Recall@k, MRR, nDCG 같은 지표를 측정해야 한다. 운영에서는 검색 실패와 생성 실패를 분리해서 디버깅하는 것도 중요하다.

## 마지막 요약

| 구분 | 핵심 내용 |
| --- | --- |
| 한 줄 정의 | Vector DB는 임베딩 벡터를 저장하고 의미적으로 가까운 데이터를 빠르게 찾는 데이터베이스다. |
| 왜 필요한가 | 키워드가 정확히 일치하지 않아도 의미가 비슷한 문서, 상품, 이미지, 질문을 검색하기 위해 필요하다. |
| 핵심 원리 | 원문과 쿼리를 같은 임베딩 모델로 벡터화한 뒤 similarity metric과 ANN index로 top-k 후보를 찾는다. |
| 장점 | semantic search, RAG, 추천, 중복 탐지에 유용하다. |
| 주의할 점 | 유사도는 정답 보장이 아니며, chunking, metadata, 권한, re-indexing, reranking이 품질을 좌우한다. |
| 실무 포인트 | Vector DB 단독보다 hybrid search, metadata filter, 평가셋, 모니터링을 함께 설계해야 한다. |
| 면접 포인트 | Vector DB를 "AI용 DB"라고만 말하지 말고 embedding, ANN, similarity metric, 운영 tradeoff까지 설명해야 한다. |

## 복습 로그

- 2026-07-30: Vector DB 기본 개념, 유사도 검색, ANN, RAG 운영 포인트 1차 정리.
