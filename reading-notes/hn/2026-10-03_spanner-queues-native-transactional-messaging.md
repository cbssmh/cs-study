1. Title

Spanner Queues: Native Transactional Messaging for Agentic Workloads and Beyond

2. Source

Author / Organization: Google Cloud Databases Team / Google Cloud
Link: https://cloud.google.com/blog/products/databases/spanner-queues-provide-native-transactional-messaging
Date: 2026-10-03

3. One-line Summary

Google Cloud가 Spanner 내부에 기본 트랜잭션 메시징 기능인 'Spanner queues'를 정식 출시하여, AI 에이전트 및 분산 시스템의 상태 전이와 비동기 작업 발행을 단일 ACID 트랜잭션 내에서 원자적으로 결합함으로써 듀얼 라이트(Dual-Write) 문제와 Outbox 패턴의 신뢰성 비용을 제거했다.

4. Key Points

• 분리된 커밋 지점의 위험성: 자율형 에이전트 시스템에서 상태를 기록하는 운영 데이터베이스와 비동기 작업을 발행하는 외부 메시지 큐를 이원화할 경우, 상태 저장 후 큐 발행 실패 또는 큐 발행 후 트랜잭션 롤백으로 인한 심각한 데이터 정합성 붕괴가 발생한다.
• 네이티브 트랜잭션 큐잉: Spanner의 읽기-쓰기 트랜잭션 내에서 메시지 엔큐(Enqueue)를 일반 행 쓰기 작업으로 처리하여, Spanner의 엄격한 직렬화 가능성(Strict Serializability)과 글로벌 외부 일관성을 바탕으로 상태 변경과 실행 의도가 100% 원자적으로 커밋된다.
• 스트리밍 SQL 기반 태스크 소비: 워커는 `RECEIVE_<QueueName>` 테이블 값 함수(TVF)를 통해 gRPC 스트리밍 SQL(`ExecuteStreamingSql`)로 작업을 동적으로 소비하며, Spanner가 고유 임대 토큰(`SpannerLeaseToken`)과 만료 시간을 자동으로 관리한다.
• 동적 임대 갱신(Lease Renewal): 다중 턴 LLM 추론이나 외부 API 호출이 기본 임대 시간을 초과할 경우 워커가 `RENEWLEASE_<QueueName>` 함수를 통해 임대 기간을 능동적으로 연장하여 조기 타임아웃에 의한 중복 실행을 방지한다.
• 경쟁 상태 방지 원자적 확인(ACK): 작업 완료 후 읽기-쓰기 트랜잭션에서 큐 행을 삭제하면서 `ASSERT_ROWS_MODIFIED 1` 구문을 실행하여, 네트워크 지연으로 임대가 만료되어 타 워커가 이미 인수한 태스크에 대한 유령 덮어쓰기(Ghost Overwrite)를 원천 차단한다.
• 시간 기반 스케줄링 및 지연 취소: 메시지 발행 시 시스템 컬럼 `DeliverTime`을 지정해 지연 실행, SLA 에스컬레이션 타이머, 지연 재시도를 외부 크론(Cron) 인프라 없이 구현하며, 상위 트랜잭션에서 표준 SQL `DELETE`로 대기 중인 에스컬레이션을 원자적으로 취소할 수 있다.
• 에피소딕 메모리 비동기 동기화: 실시간 대화 루프를 차단하지 않고 에이전트의 장기 에피소딕 메모리 요약 및 서브 에이전트 간 컨텍스트 핸드오프를 큐를 통해 비동기적이면서도 트랜잭션 보장 하에 영속화한다.
• SQL 네이티브 관찰성: 독자적인 블랙박스 큐 저장소와 달리 표준 SQL 쿼리를 통해 대기 큐 인스펙션, 백로그 모니터링, 에이전트 실행 감사 이력을 단일 쿼리 인터페이스로 조회할 수 있다.

5. Deep Dive (Structured Understanding)

Problem
자율형 AI 에이전트는 환불 집행, 재고 변경, 다단계 에이전트 간 핸드오프(A2A) 등 실질적인 비즈니스 액션을 비동기로 수행한다. 기존 아키텍처에서는 에이전트의 내부 추론 상태와 메모리를 DB에 기록하고, 외부 액션은 분리된 메시지 브로커(Kafka, RabbitMQ, SQS 등)로 발행하는 구조를 가졌다. 그러나 이 구조는 고전적인 분산 시스템의 듀얼 라이트(Dual-Write) 결함을 유발한다. DB 상태 저장은 성공했으나 액션 발행이 실패하면 에이전트가 결정만 내리고 실행하지 못하며, 반대로 액션이 발행된 후 DB 트랜잭션이 롤백되면 유효하지 않은 상태를 기반으로 외부 액션이 영구 실행되는 치명적인 불일치가 발생한다. 이를 방지하기 위해 개발자들은 트랜잭셔널 아웃박스(Transactional Outbox) 패턴, CDC 리더 파이프라인, 복잡한 멱등성 계층 및 정합성 보정 워커(Reconciliation Workers)를 구축해야 했고, 이는 에이전트 시스템에 막대한 '신뢰성 비용(Reliability Tax)'을 초래했다.

Approach
Spanner 내부에 메시지 큐를 일급 관계형 구조(First-class Relational Structure)로 임베딩했다. 메시지 생성은 분리된 네트워크 호출이 아니라 Spanner의 일반적인 행(Row) 쓰기 작업으로 취급된다. 워커 소비는 스트리밍 SQL 인터페이스(`RECEIVE_TVF`)를 통해 이루어지며, 작업 처리가 진행되는 동안 임대 갱신(`RENEWLEASE`)과 작업 완료 확인(`DELETE with ASSERT_ROWS_MODIFIED 1`)이 데이터베이스 레벨에서 지원된다. 스케줄링 또한 `DeliverTime` 컬럼을 통해 시간 지연 큐잉을 지원하며 필요 시 표준 SQL로 손쉽게 취소할 수 있도록 구성했다.

Key Insight
1. 단일 커밋 지점을 통한 원자적 Decide-and-Act: Spanner의 TrueTime 기반 엄격한 직렬화 하에서 에이전트의 메모리/상태 전이와 다운스트림 액션 디스패치가 단일 트랜잭션으로 커밋되므로 상태 불일치가 구조적으로 발생할 수 없다.
2. 장기 실행 LLM 워크로드를 위한 적응형 임대 모델: 분 단위 이상 소요될 수 있는 LLM의 비결정적 추론 시간을 고려하여, 고정 타임아웃 대신 능동적 임대 연장(`RENEWLEASE`)과 낙관적 검증 성격의 `ASSERT_ROWS_MODIFIED`를 결합하여 스톨(Stall)된 워커에 의한 데이터 오염을 차단했다.
3. 아웃박스 패턴 및 외부 인프라의 완전한 제거: 외부 브로커, CDC 데몬, 분산 스케줄러(Cron) 등의 파편화된 컴포넌트를 하나의 글로벌 분산 데이터베이스 인터페이스로 수렴시켜 인프라 복잡도를 획기적으로 축소했다.

Result / Impact
AI 에이전트 및 미션 크리티컬 트랜잭션 시스템에서 상태 저장과 비동기 태스크 실행 간 '최소 1회 전달(At-least-once delivery) + 최대 1회 ACK(At-most-once ACK)' 조합을 통해 완벽한 Exactly-Once 처리 시맨틱을 달성할 수 있는 인프라 기반을 제공한다.

6. Why It Matters

• Backend Engineering & Distributed Systems: 마이크로서비스 및 분산 아키텍처에서 지난 수십 년간 엔지니어들을 괴롭혀 온 듀얼 라이트 문제와 분산 2PC(Two-Phase Commit)의 복잡성을 글로벌 분산 DB 엔진 차원에서 흡수한 아키텍처적 전환점이다.
• Databases & Infrastructure: 관계형 테이블, 벡터 검색, 그래프를 넘어 '작업 오케스트레이션 큐'까지 통합 흡수하는 현대 데이터베이스의 다중 모달(Multi-Modal) 확장 트렌드를 대변한다.
• AI Engineering & Reliable Agentic Systems: 자율 에이전트의 비결정적 추론과 외부 API 도구 호출이 초래하는 런타임 신뢰성 붕괴(중복 환불, 유령 태스크 실행, 메모리-실행 이력 불일치)를 방어하는 결정론적 인프라 안전망을 제공한다.
• IT Risk / Governance: 에이전트의 모든 의사결정, 상태 전이, 큐잉된 작업 및 승인 대기 상태가 Spanner의 단일 감사 로그와 SQL 쿼리로 추적 가능해져 엔터프라이즈 감사성(Auditability)과 규제 컴플라이언스가 대폭 강화된다.

7. Critical Analysis

• 데이터베이스 부하 및 핫스팟 우려: 메시지 큐의 빈번한 쓰기/삭제(Enqueuing/Dequeuing) 패턴은 전통적으로 관계형 DB에서 심각한 MVCC 가비지 컬렉션 부하, 인덱스 블로트(Bloat), 특정 스플릿(Split)에 대한 쓰기 핫스팟을 유발한다. Spanner가 내부적으로 파티셔닝과 스토리지 압축을 어떻게 최적화했는지에 대한 장기 운영 데이터와 극한 워크로드 벤치마크 검증이 필요하다.
• 벤더 종속성(Lock-in)의 심화: Spanner queues는 GoogleSQL 확장 문법과 Spanner 고유의 분산 트랜잭션 엔진에 긴밀히 결합되어 있으므로, 이를 채택한 에이전트 런타임은 타 클라우드(AWS, Azure)나 온프레미스 오픈소스 환경(PostgreSQL, CockroachDB 등)으로의 이식성이 크게 제한된다.
• 고비용 인프라 오버헤드: Spanner는 엔터프라이즈급 글로벌 분산 데이터베이스로서 노드당 비용이 높다. 단순한 고처리량 메시지 버퍼링 용도로 Spanner를 사용하는 것은 Kafka나 SQS 대비 비용 효율성 측면에서 불리할 수 있으므로, 엄격한 트랜잭션 일관성이 필수적인 비즈니스 핵심 태스크에 국한하여 사용하는 아키텍처적 절제가 요구된다.

8. Connections

• Transactional Outbox Pattern & Debezium CDC: 분산 시스템에서 DB 상태와 메시지 발행의 정합성을 맞추기 위해 사용되던 고전적 아웃박스 패턴과 폴링/CDC 엔진의 인프라적 대안으로 직접 연결된다.
• PostgreSQL `SKIP LOCKED` 기반 작업 큐: 관계형 DB를 큐로 활용할 때 동시성 충돌을 피하기 위해 사용되던 `SELECT FOR UPDATE SKIP LOCKED` 패턴을 글로벌 분산 데이터베이스 환경에 맞추어 스트리밍 TVF와 자동 임대 관리 형태로 격상시킨 모델이다.
• Saga Pattern & Distributed Orchestration: 분산 트랜잭션 환경에서 보상 트랜잭션과 비동기 단계 실행을 조율하던 사가 패턴의 오케스트레이터를 데이터베이스 내부 트랜잭션 큐로 단순화했다.

9. Keywords

• Spanner Queues
• Transactional Messaging
• Dual-Write Problem
• Transactional Outbox Pattern
• Strict Serializability
• Lease Management
• Agentic Architecture
• Exactly-Once Processing

10. TL;DR

• Spanner가 상태 변경과 비동기 메시지 발행을 단일 ACID 트랜잭션으로 묶는 네이티브 트랜잭션 큐(`Spanner queues`)를 정식 출시했다.
• 듀얼 라이트 문제와 아웃박스 패턴의 복잡성을 제거하고, 스트리밍 SQL 소비, 동적 임대 갱신, 시간 기반 스케줄링을 분산 DB 수준에서 원자적으로 제공한다.
• 자율 에이전트의 신뢰성 높은 실행과 엔터프라이즈 거버넌스를 보장하지만, Spanner 고유의 비용 구조와 벤더 종속성에 대한 고려가 필요하다.

