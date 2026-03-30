# DuckDB HNSW + ACORN-1 Filtered Vector Search

## 1. 한 줄 요약
Vector search에서 filter 조건까지 정확하게 반영하기 위해, HNSW 탐색 과정에 필터를 포함시키는 방식(ACORN-1)을 적용한 구조

---

## 2. 문제 정의

기존 DuckDB VSS(HNSW 기반)의 구조적 한계:

- WHERE 조건이 검색 이후(post-filter) 적용됨
- top-K 후보 대부분이 필터로 제거됨

→ 결과:
- LIMIT 10 요청에도 0~2개만 반환
- 즉, **recall 붕괴 발생**

---

## 3. 핵심 아이디어

👉 "검색 중에 필터를 적용하자"

- 기존:
  HNSW 탐색 → 결과 → filter 적용

- 개선:
  HNSW 탐색 중 → filter 조건 반영

---

## 4. ACORN-1 알고리즘

### 핵심 개념

1. 필터 조건을 만족하지 않는 노드가 있어도
2. 그 노드를 통해 연결된 다른 노드를 탐색 (2-hop 확장)
3. 그래프 연결성 유지

👉 결과:
- 필터 조건이 있어도 충분한 후보 탐색 가능

---

## 5. Selectivity 기반 전략

필터 선택도에 따라 전략 변경

- > 60%
  → 일반 HNSW (post-filter)

- 1% ~ 60%
  → ACORN-1 적용

- < 1%
  → brute-force (정확 탐색)

👉 이유:
- 필터가 너무 강하면 index보다 full scan이 더 효율적

---

## 6. 성능 결과 (핵심)

예: 영화 데이터셋 (228k, 768-dim)

| 필터 | 기존 | ACORN |
|------|------|------|
| 일본어 (~3%) | 0~1/10 | 10/10 |
| 한국어 (~1%) | 0/10 | 10/10 |
| 평점 >=8 (~5%) | 0/10 | 10/10 |

👉 핵심:
**필터가 있을 때 recall 문제 해결**

---

## 7. 기술 포인트

- HNSW 기반 vector index
- filter pushdown
- adaptive query planning
- brute-force fallback

---

## 8. 한계

- subquery vector 사용 시 index 미사용
- index 전체가 메모리에 올라가야 함
- 업데이트/삭제 후 성능 저하 가능 (re-compaction 필요)

---

## 9. 내가 얻은 인사이트

- vector search는 단순 거리 계산 문제가 아니라
  "조건 + 검색" 문제다

- 실제 서비스에서는
  filter 조건이 거의 항상 존재함

- 따라서
  "index + filter 통합 설계"가 중요

---

## 10. 내 프로젝트 적용 아이디어

### AI Job Scout
- 직무 필터 + 유사도 검색 결합 가능

### 추천 시스템
- 카테고리 필터 + embedding 검색

### 데이터 분석
- DuckDB 기반 local vector search 가능성
