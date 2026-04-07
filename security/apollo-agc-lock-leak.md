# 🚀 Apollo 11 AGC Bug (57년 만에 발견된 문제)

1969년 Apollo 11에 사용된 **AGC (Apollo Guidance Computer)** 코드에서  
57년 동안 발견되지 않았던 버그가 발견됨.

이 버그는 단순한 코드 실수가 아니라  
**리소스 관리 실패 (lock leak)** 문제로,  
특정 상황에서 항법 시스템(IMU)이 영구적으로 동작하지 않는 상태를 유발할 수 있음.

---

## 🧩 Key Finding

### 🔥 Bug Summary

- Gyroscope 제어 시 `LGYRO`라는 lock 사용
- 정상 경로에서는 lock 해제됨
- 하지만 **에러 경로(BADEND)** 에서 lock 해제 누락

```assembly
CAF ZERO
TS  LGYRO
; 👉 위 2줄이 빠져 있음 (lock release 누락)
```

---

## 🧠 Root Cause

### 1. Shared Resource Lifecycle 누락

- lock acquire: 정상적으로 수행
- lock release: 일부 경로에서 누락

👉 즉,
> "모든 경로에서 release되어야 한다"는 보장이 없음

---

### 2. Error Path 설계 문제

- 정상 종료 → `STRTGYR2` → release 수행  
- 에러 종료 → `BADEND` → release 없음  

👉 공통 cleanup 루틴이  
특정 리소스(`LGYRO`)를 인지하지 못함

---

## ⚠️ Impact Scenario

### 🚨 실제 발생 가능 상황

1. Gyro torque 실행 중  
2. IMU가 “caged” 상태로 전환됨 (비상 보호)  
3. `BADEND` 경로로 종료  
4. `LGYRO` lock 해제 안됨  
5. 이후 모든 gyro 작업 block  

### 📉 결과

- 항법 재정렬 불가  
- star alignment 실패  
- 시스템은 "정상처럼 보이지만 아무 작업도 안함"  

👉 디버깅 불가능한 상태

---

## ❓ Why It Was Missed

### 기존 접근 방식의 한계

- 코드 리뷰 ✔  
- 에뮬레이션 ✔  
- 테스트 ✔  

하지만:

> ❌ "모든 실행 경로" 검증 없음

---

## 🤖 How It Was Found

### AI + Specification 기반 접근

- 130,000 lines → 12,500 lines spec 변환  
- 행동 기반 명세 (Allium)  

```text
if gyros_busy = true
→ 반드시 false로 돌아와야 함
```

👉 모든 경로 추적 → 누락 발견

---

## 💡 Key Insight

🔥 핵심 교훈

> "코드를 테스트하는 것"과  
> "시스템의 의도를 검증하는 것"은 다르다

---

## 🧱 Modern Perspective

### 현재 언어에서의 해결 방식

- Go → `defer`  
- Java → `try-with-resources`  
- Python → `with`  
- Rust → ownership  

👉 하지만 여전히 발생:

- DB connection  
- distributed lock  
- file handle  
- infra resource  

---

## 🧠 Engineering Takeaways

### 1. Resource Lifecycle은 명시적으로 관리해야 한다

- acquire → release 보장 필요  

---

### 2. Error Path가 더 위험하다

- 대부분 버그는 정상 흐름이 아니라 예외 흐름에서 발생  

---

### 3. Defensive coding은 문제를 숨길 수 있다

- AGC는 restart 시 자동 복구됨  
→ 버그가 드러나지 않음  

---

### 4. Specification 기반 검증 필요

- “이 코드는 무엇을 해야 하는가?”를 정의해야 함  

---

## 🔗 Relevance to Backend / System Design

이 사례는 현대 백엔드에서도 동일하게 적용됨:

- DB connection leak  
- distributed lock deadlock  
- transaction rollback 누락  

👉 즉,

> "리소스 정리는 기능이 아니라 시스템 안정성의 핵심이다"

---

## 📝 One-line Summary

> 57년 된 Apollo 코드에서도 발생한 문제는  
> 오늘날 백엔드 시스템에서도 그대로 반복된다.
