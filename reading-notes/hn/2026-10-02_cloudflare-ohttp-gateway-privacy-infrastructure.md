1. Title

Announcing Cloudflare OHTTP Gateway – Expanding Access to Cloudflare's Privacy-Preserving Infrastructure

2. Source

Author / Organization: Lara Schull and Akshat Mahajan / Cloudflare
Link: https://blog.cloudflare.com/announcing-cloudflare-ohttp-gateway/
Date: 2026-10-02

3. One-line Summary

클라우드플레어가 IETF 표준 기반의 Oblivious HTTP(OHTTP) Gateway를 관리형 서비스로 출시하여, 백엔드 애플리케이션이 클라이언트의 IP 주소와 네트워크 지문을 수집하지 않고도 안전하게 HTTP 트래픽을 수신할 수 있는 신뢰 분리형(Separation of Trust) 프라이버시 인프라를 완성했다.

4. Key Points

• 이중 홉(Two-Hop) 신뢰 분리 모델: 클라이언트와 오리진 서버 사이에 Relay(클라이언트 IP 및 TLS 지문만 확인, 페이로드는 암호화 상태로 전달)와 Gateway(페이로드 복호화 수행, 클라이언트 IP는 미인지)를 상호 비공모 독립 주체가 운영하여 신원과 요청 내용을 단일 주체가 결합할 수 없는 Double-Blind 구조를 형성한다.
• OHTTP 제품 포트폴리오의 완성: 2022년 출시된 OHTTP Relay(구 Privacy Gateway)에 이어 관리형 OHTTP Gateway를 추가함으로써, Cloudflare CDN 및 Workers 상의 백엔드가 제3자 Relay와 연동하여 표준을 준수하는 엔드 투 엔드 OHTTP 수신 체계를 구성할 수 있게 되었다.
• 엣지 기반 고성능 암복호화 처리: 자체 Gateway 구축 시 발생하는 프록시 홉 지연과 암복호화 연산 병목을 Cloudflare의 글로벌 Anycast 엣지 네트워크에서 처리하며, 오리진 서버와 동일 엣지 머신에서 복호화 및 서브리퀘스트 라우팅을 지원한다.
• Hybrid Public Key Encryption(HPKE) 키 관리 자동화: Gateway가 HPKE 공개키 생성 및 갱신을 완전 관리형으로 제공하며, 클라이언트는 표준화된 `/.well-known/ohttp-gateway` 엔드포인트를 통해 최신 공개키를 안전하게 조회한다.
• Chunked OHTTP 지원: 스트리밍 및 대용량 처리를 위한 Chunked OHTTP를 지원하여 요청/응답 전체를 메모리에 버퍼링하지 않고 청크 단위로 점진적 복호화 및 스트리밍 전송을 수행한다.
• 제로 트러스트(Cloudflare Access) 연동: Gateway가 클라이언트의 신원을 식별할 수 없는 구조적 특성상 Relay에 대한 신뢰가 필수적이므로, 복호화 전 단계에서 mTLS나 서비스 토큰 기반 Access 정책을 적용해 비인가 Relay의 트래픽을 차단한다.
• 공모 방지(Anti-Collusion) 안전장치: Cloudflare가 단독으로 Relay와 Gateway를 모두 호스팅하여 신뢰 분리 원칙이 무력화되는 구성을 방지하기 위해, Cloudflare Workers나 내부 프록시 호스트에서 인입되는 복호화 요청은 Gateway 레벨에서 구조적으로 거부한다.
• 도메인 바인딩을 통한 오픈 프록시 오남용 방지: Gateway를 특정 도메인 존(Zone)에 종속시켜, 권한 없는 클라이언트가 임의의 외부 서드파티 도메인으로 트래픽을 우회 릴레이하는 오픈 프록시 악용을 차단한다.

5. Deep Dive (Structured Understanding)

Problem
일반적인 클라이언트-서버 통신에서 백엔드는 IP 패킷 헤더의 출발지 IP 주소와 TLS 핸드셰이크 특성(Cipher suite, TLS 버전 등)을 필연적으로 확인하게 된다. 이는 사용자 추적을 피하려는 엔드유저에게 VPN이나 광고 차단기 설치를 강요할 뿐만 아니라, 민감 데이터를 다루는 헬스케어 앱(Flo Health 등)이나 AI 추론 서비스(Apple Private Cloud Compute 등) 백엔드 운영자에게도 불필요한 개인식별정보(PII) 보관에 따른 규제 준수 및 보안 침해 책임(Data Liability)을 전가한다. 반면 개별 기업이 자체적으로 OHTTP Gateway를 구축해 운영하려면 다중 홉 프록시로 인한 레이턴시 증가, HPKE 암복호화 연산 오버헤드, 공개키 배포 및 악의적 Relay 필터링 등의 복잡한 인프라 관리 부담이 발생한다.

Approach
Cloudflare는 IETF RFC 9458(OHTTP) 표준을 구현한 완전 관리형 OHTTP Gateway를 자사 엣지 플랫폼에 통합했다. 고객은 도메인 설정에서 Gateway 기능을 활성화하기만 하면 `/.well-known/ohttp-gateway` 엔드포인트를 통해 즉시 OHTTP 트래픽을 수신할 수 있다. 클라이언트가 제3자 독립 Relay를 통해 전송한 암호화 요청은 Cloudflare 엣지에서 수신되며, 제로 트러스트 Access 계층에서 Relay 인증을 거친 뒤 HPKE 복호화되어 오리진 애플리케이션으로 일반 HTTP 서브리퀘스트 형태로 전달된다.

Key Insight
1. 엄격한 신뢰 분리(Separation of Trust)의 상용화: OHTTP의 보안 모델은 Relay와 Gateway가 서로 다른 비공모 주체에 의해 운영될 때만 성립한다. 기존에는 백엔드가 Cloudflare에 있으면 Cloudflare Relay를 사용할 수 없어 OHTTP 도입이 막혀 있었으나, 관리형 Gateway를 제공함으로써 "외부 독립 Relay + Cloudflare Gateway"의 안전한 조합을 가능하게 만들었다.
2. 전송 계층 메타데이터와 애플리케이션 페이로드의 분리: OHTTP는 IP 주소와 TLS 지문 같은 네트워크 레벨의 메타데이터를 클라이언트 신원과 분리하는 프로토콜이다. 본문(Body) 내부의 인증 토큰이나 식별 데이터는 그대로 유지되므로, 비인가 텔레메트리 수집, 익명 모드, 비식별 AI 질의 등 데이터 최소화가 필요한 워크로드에 최적의 보안 경계를 제공한다.
3. 엣지 네이티브 암호화 오프로딩: Anycast 네트워크를 통해 클라이언트와 가장 가까운 Relay 및 Gateway 간 경로를 최적화하고, 동일 인프라 내에서 복호화와 오리진 통신을 병합하여 OHTTP 특유의 다중 홉 레이턴시 페널티를 대폭 상쇄했다.

Result / Impact
기업과 개발팀이 독자적인 암호화 서버 플릿을 구축하거나 관리할 필요 없이 클릭 몇 번으로 IP 비식별화 HTTP 엔드포인트를 배포할 수 있게 되었다. Apple LiveCallerID SDK 연동이나 프라이버시 보존형 AI 추론 파이프라인 구축의 진입 장벽이 획기적으로 낮아졌다.

6. Why It Matters

• Security / DevSecOps & IT Risk / Governance: "수집하지 않은 데이터는 유출될 위험도 없다"는 데이터 최소화(Data Minimization) 원칙의 실질적 구현이다. 백엔드 로그에 클라이언트 IP 주소가 아예 기록되지 않으므로, 데이터 유출 사고나 법적 데이터 제출 요구 시에도 사용자의 신원과 활동 내역이 결합되지 않아 GDPR, HIPAA 등 글로벌 프라이버시 규제 대응 리스크를 원천 차단한다.
• Infrastructure & Distributed Systems: 글로벌 Anycast 엣지 네트워크에서 암호화 프록시 체인을 대규모로 호스팅할 때 마주하는 레이턴시와 컴퓨팅 부하를 엣지 플랫폼 서비스로 추상화한 대표적 아키텍처 사례다.
• AI Engineering / Secure AI: 대규모 언어 모델(LLM) 기반의 추론 시스템에서 사용자 프롬프트와 지리적 위치/기기 지문을 분리하는 것이 핵심 보안 요구사항으로 부상함에 따라, 프라이버시 보존형 AI 게이트웨이의 필수 기저 인프라로 작동한다.

7. Critical Analysis

• 악성 트래픽 및 오남용 방어의 딜레마: Hacker News 커뮤니티 등에서 제기되었듯, 출발지 IP가 완전히 은닉된 Gateway는 봇넷 공격이나 크리덴셜 스터핑 같은 악성 행위자들에게도 완벽한 차폐막을 제공할 수 있다. 백엔드 방화벽(WAF)이 기존의 IP 평판 기반 차단 방식을 적용할 수 없게 되므로, 오남용 방어가 전적으로 Relay의 클라이언트 인증 능력과 애플리케이션 레벨의 챌린지 기법에 전가되는 위험이 존재한다.
• 인프라 중앙화와 트래픽 분석 공격 위험: Cloudflare가 전 세계 웹 트래픽의 상당 부분을 점유하고 있는 상황에서, 신뢰 분리 모델을 적용하더라도 소수의 대형 CDN 사업자가 프라이버시 인프라를 독점할 경우 거시적 관점의 트래픽 분석(Traffic Analysis)이나 사이드 채널 공격에 대한 우려를 완전히 불식하기 어렵다.
• 익명성 세트(Anonymity Set)의 한계: OHTTP의 익명성은 대규모의 다양한 사용자 트래픽이 동일 Relay에 뒤섞일 때만 보장된다. 특정 전용 Relay를 소규모 사용자군이나 단일 기업용으로만 구축해 사용할 경우, Relay의 IP 자체가 해당 그룹을 식별하는 또 다른 정적 핑거프린트가 될 위험이 있다.
• 애플리케이션 계층 누출에 대한 무방비성: OHTTP는 네트워크 전송 계층의 IP를 지워줄 뿐, 개발자가 요청 본문에 세션 쿠키, 사용자 ID, 고유 디바이스 식별자를 포함해 전송할 경우 프라이버시 보호가 즉시 무력화되므로 철저한 클라이언트 사이드 데이터 감사 체계가 동반되어야 한다.

8. Connections

• IETF RFC 9458 (Oblivious HTTP) & RFC 9180 (HPKE): 단순 프록시(CONNECT)와 차별화되는 하이브리드 공개키 암호화(HPKE) 기반의 캡슐화를 통해 L7 데이터 기밀성과 L3/L4 메타데이터 비식별성을 수학적으로 보장하는 웹 표준 프로토콜이다.
• Apple Private Cloud Compute (PCC) & iCloud Private Relay: MASQUE 및 OHTTP 프로토콜을 기반으로 사용자 단말에서 클라우드로 향하는 AI 프롬프트와 웹 트래픽을 통신사 및 인프라 제공자로부터 이중 차단하는 차세대 프라이버시 아키텍처 흐름의 핵심 축이다.
• Tor Network와의 트레이드오프: 다중 노드를 거치는 Onion Routing(Tor)의 극심한 지연 시간을 극복하기 위해, 상호 독립적인 2-hop(Relay-Gateway) 신뢰 분리와 최신 암호학을 결합하여 현대 웹 애플리케이션이 감내할 수 있는 레이턴시 내에서 실용적 익명성을 달성했다.

9. Keywords

• Oblivious HTTP (OHTTP)
• RFC 9458
• Hybrid Public Key Encryption (HPKE)
• Separation of Trust
• Data Minimization
• Cloudflare Access
• Anycast Edge
• Anonymity Set

10. TL;DR

• Cloudflare가 IETF 표준 OHTTP Gateway를 관리형으로 출시하여 백엔드가 사용자 IP 주소 없이 안전하게 요청을 수신하는 2-hop 프라이버시 인프라를 완성했다.
• 제3자 Relay와의 엄격한 신뢰 분리, Anycast 기반 HPKE 복호화 오프로딩, 제로 트러스트(Access) 연동 및 공모 방지 안전장치를 엣지 레벨에서 통합 구현했다.
• PII 수집 리스크와 프라이버시 침해 책임을 획기적으로 줄였으나, IP 기반 보안 방어 무력화 대응과 Relay의 충분한 익명성 세트 유지가 핵심 운영 과제로 남는다.
