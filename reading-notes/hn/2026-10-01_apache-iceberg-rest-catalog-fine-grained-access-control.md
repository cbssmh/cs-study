1. Title

Standardizing Fine-Grained Access Control in Apache Iceberg's REST Catalog

2. Source

Author / Organization: Talat Uyarer & Sung Yun / Google Cloud (Google Open Source Blog)
Link: https://opensource.googleblog.com/2026/10/standardizing-fine-grained-access-control-in-apache-icebergs-rest-catalog.html
Date: 2026-10-01

3. One-line Summary

Apache Iceberg의 REST Catalog 사양에 엔진 중립적인 읽기 제한(`read-restrictions`) 규격을 도입하여, 스토리지 직접 읽기 성능을 해치지 않으면서 카탈로그(PDP)와 신뢰할 수 있는 쿼리 엔진(PEP) 간의 미세 단위 접근 제어(Row 필터링 및 Column 마스킹)를 개방형 표준으로 공식화했다.

4. Key Points

• 문제 배경: 전통적인 단일 데이터베이스 서버와 달리 레이크하우스는 쿼리 엔진이 오브젝트 스토리지의 Parquet 파일을 직접 읽기 때문에 파일 단위 이하의 거버넌스 강제 지점(Enforcement Point)이 존재하지 않았다.
• 기존 한계: 미세 단위 접근 제어(FGAC)를 위해 벤더 전용 클라이언트를 도입하거나 스토리지 읽기 경로에 프록시를 둬야 했고, 이는 다중 엔진 상호운용성과 직접 스토리지 읽기 성능을 훼손했다.
• 페이로드 표준화: REST Catalog의 `loadTable` 응답에 `read-restrictions` 객체가 추가되었으며, 이는 `required-row-filter`(Iceberg 술어식)와 `required-column-projections`(필드 ID 기반 마스킹 변환 목록)로 구성된다.
• 평가 순서 보장: 원본(Untransformed) 컬럼 값을 기준으로 `required-row-filter`를 먼저 평가하고, 필터를 통과한 행에만 마스킹 프로젝션을 적용하여 두 기능의 결합성(Composability)을 보장한다.
• 9가지 폐쇄형 마스킹 액션: `mask-alphanum`, `show-first-4`, `show-last-4`, `replace-with-null`, `mask-to-fixed-value`, `truncate-to-year`, `truncate-to-month`, `sha-256-global`, `sha-256-query-local`의 9가지 표준 액션을 정의하여 엔진 간 바이트 단위 결과 일관성을 확보한다.
• 타입 보존성: 모든 마스킹 변환 결과는 원본과 동일한 데이터 타입을 유지하므로 엔진 쿼리 플래너의 스키마 변경이나 재작성이 필요하지 않다.
• 철저한 Fail-Closed 설계: 리더가 알 수 없는 마스킹 액션이나 처리 불가능한 필터를 수신하면 부분적 데이터 반환 없이 즉시 쿼리를 실패 처리하여 데이터 유출 위험을 차단한다.
• 식별자 안정성: 정책 적용 대상을 컬럼 이름이 아닌 불변 필드 ID(Field ID)로 지정하여 스키마 컬럼 이름 변경 시 정책이 무력화되는 취약점을 방지했다.
• 생태계 합의: Apache Iceberg 개발자 메일링 리스트에서 반대 없이 8개의 구속력 있는 찬성(+1) 투표를 얻어 공식 채택되었다.

5. Deep Dive (Structured Understanding)

Problem
레이크하우스 아키텍처는 Spark, Trino, BigQuery, DuckDB 등 다양한 분산 엔진이 오브젝트 스토리지의 Parquet 파일을 직접 읽는 구조 덕분에 성능과 유연성을 얻었다. 그러나 읽기 경로에 중앙 제어 서버가 없기 때문에, 카탈로그 수준의 Credential Vending이나 파일 단위 Scan Planning을 넘어서는 "이메일 마스킹" 또는 "특정 지역(Region) 행만 조회" 같은 미세 단위 접근 제어(FGAC)를 오픈 표준 규격으로 표현할 방법이 없었다. 그 결과 사용자는 벤더 종속적인 보안 프록시나 전용 드라이버를 써야만 했고, 오픈 테이블 포맷의 핵심 가치인 엔진 중립성이 훼손되었다.

Approach
REST Catalog의 `loadTable` API 응답 규격에 `read-restrictions`를 정의했다. 카탈로그는 호출자의 인증 토큰을 통해 내부의 복잡한 거버넌스 정책(RBAC, ABAC, 태그 기반 정책)을 자체 평가한 후, 엔진에게는 구체적인 평가 결과인 행 필터 술어식과 9개의 표준화된 열 마스킹 액션 목록만을 반환한다. 엔진은 거버넌스 모델을 이해할 필요 없이 단 9개의 정형화된 액션과 술어 평가기만 구현하면 된다. 이를 통해 카탈로그와 엔진 간 통합 복잡도를 N×M에서 N+M의 Narrow Waist 구조로 단순화했다.

Key Insight
1. 정직한 강제 지점(Honest Enforcement Point): 읽기 성능을 위해 직접 스토리지 읽기를 유지하는 한, 보안 강제는 스토리지 자격 증명을 쥔 '신뢰할 수 있는 쿼리 엔진(Trusted Engine)' 내부에서 실행되어야 함을 투명하게 수용했다.
2. 실행 순서의 결합성: 마스킹된 값으로 인해 필터 평가가 왜곡되지 않도록, 원본 값으로 행 필터를 먼저 수행한 후 잔여 행에 열 마스킹을 적용하는 순서를 표준 규격으로 명시했다.
3. Fail-Closed 기본값: 인식할 수 없는 액션, 파싱 불가능한 필터, 중복 필드 ID 등이 발생하면 원본 데이터를 노출하지 않고 쿼리를 즉시 중단하도록 강제하여, 사양 확장 시 구형 엔진이 보안 누출 경로가 되는 것을 방지했다.
4. 조인 가능성과 프라이버시의 균형: 결정론적 해시(`sha-256-global`)와 쿼리 단위 솔트 해시(`sha-256-query-local`)를 분리 제공하여 분석적 유용성(다중 테이블 조인 및 집계)과 사전 계산 공격 방어 간의 트레이드오프를 정책 작성자가 선택할 수 있도록 설계했다.

Result / Impact
Spark, Trino, PyIceberg 등의 엔진이 `iceberg-core`의 공통 변환 로직을 활용해 동일한 마스킹 결과를 산출할 수 있게 되었으며, 오픈 레이크하우스 환경에서 상용 솔루션 없이도 표준에 기반한 데이터 거버넌스 파이프라인을 구축할 수 있는 기틀이 마련되었다.

6. Why It Matters

• Backend Engineering & Distributed Systems: 분산 쿼리 엔진과 분리된 스토리지 계층 사이에서 단일 중앙 게이트웨이를 강제하지 않고도 거버넌스를 달성하는 탈중앙화된 보안 협약 모델을 제시했다.
• Platform Engineering & Infrastructure: Trino, Spark 등 다중 쿼리 엔진을 혼용하는 현대 데이터 플랫폼에서 엔진마다 제각기 구축하던 보안 플러그인(Apache Ranger 등)을 메타데이터 카탈로그 인터페이스 중심의 단일 체계로 통합할 수 있다.
• IT Risk / Governance & Security: GDPR, HIPAA 등 규제 준수를 위한 PII 보호 및 멀티테넌트 데이터 격리를 플랫폼 종속 없이 일관되게 적용할 수 있다. 특히 사양에서 언급하듯 `loadTableResponse`가 사용자 권한 컨텍스트에 종속되므로, 플랫폼 팀이 캐시 레이어의 권한 누수 위험을 점검해야 하는 보안 감사 포인트를 도출했다.

7. Critical Analysis

• 엔진 신뢰 검증(Attestation) 메커니즘의 부재: 보안 강제가 전적으로 리더(엔진)에 위임되어 있어, 클라이언트가 악의적이거나 제약 조건을 무시하고 원본 Parquet 파일을 직접 조회할 경우 이를 프로토콜 수준에서 검증하거나 방어할 수 없다. 따라서 최종 사용자가 스토리지 자격 증명에 직접 접근할 수 없는 중앙 집중형 클러스터 환경에서만 실질적 보안이 성립한다.
• 폐쇄형 액션의 한계: 현재는 9개의 고정된 마스킹 액션과 단순 술어 비교만 지원하여 정규식 기반 치환이나 비즈니스 맞춤형 UDF 적용이 불가능하다. 이는 2026년 중반 채택된 Iceberg Expressions 사양이 실제 엔진들에 완전히 구현되어야 해결될 과제다.
• 정책 정의 자체의 비표준화: 엔진이 수행할 '결과 의무(Obligations)'만 표준화되었을 뿐, "누가 무엇을 볼 수 있는가"를 정의하는 카탈로그 측의 정책 모델(역할, 속성, 태그)은 여전히 각 카탈로그 벤더의 독자 영역으로 남아 있어 카탈로그 간 정책 마이그레이션은 별도의 숙제로 남는다.

8. Connections

• Narrow Waist Architecture: 인터넷의 IP(Internet Protocol)처럼 다양한 상위 정책 엔진(Polaris, Unity, Nessie 등)과 다수의 하위 쿼리 엔진(Spark, Trino, Flink 등) 사이에 최소한의 명확한 중간 추상화 계층을 두어 상호운용성을 극대화했다.
• XACML / Zero Trust의 PDP-PEP 분리: 카탈로그를 정책 결정 지점(Policy Decision Point, PDP)으로, 신뢰할 수 있는 분산 엔진을 정책 강제 지점(Policy Enforcement Point, PEP)으로 명확히 역할 분리한 보안 참조 아키텍처의 전형이다.
• Cryptographic Tokenization & Pseudonymization: 전역 고정 해시와 쿼리 로컬 솔트 해시를 분리한 것은 분석 파이프라인에서 Linkability(조인 가능성)와 Unlinkability(추적 불가능성) 사이의 암호학적 상충 관계를 실용적으로 다룬 설계다.

9. Keywords

• Apache Iceberg
• REST Catalog
• Fine-Grained Access Control (FGAC)
• Row Filtering
• Column Masking
• Lakehouse Governance
• Fail-Closed
• Policy Enforcement Point (PEP)

10. TL;DR

• Apache Iceberg REST Catalog에 엔진 중립적인 읽기 제한(`read-restrictions`) 규격이 추가되어 오픈 레이크하우스의 고질적인 거버넌스 공백을 해결했다.
• 직접 스토리지 읽기 성능을 보존하기 위해 카탈로그(PDP)의 평가 결과를 신뢰할 수 있는 엔진(PEP)이 9개 표준 액션과 술어식으로 강제하며 엄격한 Fail-closed 원칙을 적용한다.
• 다중 엔진 전반의 일관된 보안 거버넌스 기반을 마련했으나, 프로토콜 수준의 엔진 증명(Attestation) 부재로 인해 엔드유저와 스토리지를 격리하는 인프라 신뢰 경계 관리가 필수적이다.
