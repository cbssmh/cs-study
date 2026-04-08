# VeraCrypt Windows 배포 중단 이슈 정리

## 📌 개요
VeraCrypt 개발자인 Mounir Idrassi가 Microsoft 계정이 예고 없이 종료되면서  
Windows용 업데이트 배포가 불가능해진 상황 발생.

- 사전 경고 없음
- 이메일 안내 없음
- 이의 제기 불가 메시지
- Microsoft와 직접 संपर्क 실패 (봇 응답만 수신)

👉 Windows 플랫폼 업데이트 중단 상태

---

## 🔥 핵심 문제

### 1. Microsoft 계정 종료
- Windows 드라이버 및 부트로더 서명에 사용하던 계정 삭제
- Partner Center 접근 불가
- 복구 경로 없음

> "Microsoft did not send me any emails or prior warnings." :contentReference[oaicite:0]{index=0}  

---

### 2. Windows 배포 차단
- Windows 업데이트 불가능
- Linux / macOS는 정상 배포 가능
- 주요 사용자층이 Windows → 치명적 영향

---

## ⚠️ 기술적 영향

### Secure Boot 관련 이슈
- 현재 버전 (1.26.24) → 2011 CA 서명 사용
- 인증서 만료 예정

예상 영향:
- Secure Boot 환경에서 설치 문제 가능성
- unsigned build 사용 시 보안 경고
- portable 사용에도 영향 가능성

---

## 💬 커뮤니티 논의 요약

### 🧠 원인 추정
- 계정 신고 (암호화 소프트웨어 특성)
- 지역 변경 (일본 relocation)
- Microsoft 자동 검증 실패 (WHOIS / 도메인 문제 가능성)

---

### 🛠 해결 시도 제안

#### 1. Microsoft 직접 접근
- 계정 복구 페이지 활용
- 고객센터 연결 시도
- CEO 이메일 연락 제안

#### 2. 우회 전략
- 새 계정 생성 후 재인증
- 커뮤니티/지인 통해 내부 연결

#### 3. 외부 확산
- Reddit / X (Twitter) 공유
- 언론 제보 (Ars Technica, Wired 등)

---

### 🧩 기술적 대안
- 비시스템 볼륨만 지원하는 기능 추가
- 서명 없이 사용 가능한 제한 버전 고려
- archive-like 기능 도입 제안

---

## 🧑‍💻 유사 사례

- Rufus 개발자도 동일 문제 경험
- 원인: Microsoft 자동 도메인 검증 실패
- 해결: 도메인 등록 정보 제출 후 복구

---

## 📉 영향 평가

| 항목 | 영향 |
|------|------|
| Windows 사용자 | 매우 큼 |
| 프로젝트 지속성 | 위험 |
| 보안 신뢰성 | 저하 가능 |
| 배포 체계 | 사실상 중단 |

---

## 🧭 결론

이 사건은 단순 계정 문제가 아니라:

- 🔐 **소프트웨어 배포가 플랫폼에 종속되는 구조적 리스크**
- 🤖 **자동화된 검증 시스템의 한계**
- 🚫 **Big Tech 의존성 문제**

를 보여주는 사례이다.

---

## 📎 참고

- SourceForge Forum (VeraCrypt Discussion)
- 작성자: Mounir Idrassi
- 날짜: 2026-03-30
