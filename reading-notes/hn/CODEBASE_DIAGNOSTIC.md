# 🔍 Codebase Diagnostic Using Git

> 코드 분석에 들어가기 전, Git 히스토리를 기반으로 시스템 리스크를 진단한 과정

---

## 🎯 Objective

코드를 읽기 전에 Git 데이터를 활용하여
다음과 같은 핵심 리스크를 사전에 식별한다.

* 변경이 집중되는 영역 (Churn)
* 특정 개발자 의존도 (Bus Factor)
* 버그 발생 밀집 영역 (Bug Hotspot)
* 개발 속도 및 팀 상태 (Velocity)
* 운영 안정성 (Firefighting)

---

## 🛠️ Analysis Methods

### 1. High Churn Files (변경 빈도 분석)

```bash
git log --format=format: --name-only --since="1 year ago" \
| sort | uniq -c | sort -nr | head -20
```

**목적**

* 최근 1년간 가장 많이 수정된 파일 식별

**해석**

* 높은 변경 빈도 = 핵심 도메인 또는 구조적 문제 가능성
* 팀이 수정을 꺼리는 파일일 가능성 존재

**활용**

* 리팩토링 우선 대상 선정
* 변경 영향 범위가 큰 영역 사전 파악

---

### 2. Bus Factor (개발자 의존도 분석)

```bash
git shortlog -sn --no-merges
```

**목적**

* 커밋 수 기준 기여자 분포 확인

**해석**

* 특정 개발자 비중이 높을 경우 → 지식 편중 리스크
* 주요 기여자가 최근 활동하지 않으면 유지보수 위험 증가

**활용**

* 도메인 책임 분리 필요성 판단
* 팀 구조 및 유지보수 가능성 평가

---

### 3. Bug Hotspot (버그 발생 영역 분석)

```bash
git log -i -E --grep="fix|bug|broken" --name-only --format='' \
| sort | uniq -c | sort -nr | head -20
```

**목적**

* 버그 수정이 자주 발생한 파일 식별

**해석**

* 반복적으로 수정되는 영역 = 구조적 문제 가능성
* Churn과 겹치는 파일 → 최우선 리스크

**활용**

* 기술 부채 집중 영역 식별
* 안정성 개선 대상 선정

---

### 4. Commit Velocity (개발 흐름 분석)

```bash
git log --format='%ad' --date=format:'%Y-%m' \
| sort | uniq -c
```

**목적**

* 월별 커밋 수 기반 개발 속도 변화 분석

**해석**

* 급격한 감소 → 인력 변화 또는 프로젝트 정체
* 주기적 스파이크 → 배포 중심 개발 패턴

**활용**

* 팀 운영 상태 파악
* 개발 방식 (Continuous vs Batch) 분석

---

### 5. Firefighting Pattern (운영 안정성 분석)

```bash
git log --oneline --since="1 year ago" \
| grep -iE 'revert|hotfix|emergency|rollback'
```

**목적**

* 긴급 수정 및 롤백 패턴 확인

**해석**

* 빈번한 hotfix/revert → 배포 안정성 문제
* 테스트 부족 또는 CI/CD 미성숙 가능성

**활용**

* 테스트 전략 및 배포 프로세스 개선 필요성 판단

---

## 🔗 Cross Analysis (핵심)

가장 중요한 분석은 다음 두 가지의 교차이다.

* High Churn Files
* Bug Hotspot

👉 두 조건을 모두 만족하는 파일은
**시스템 내 가장 높은 리스크 영역**

---

## 🚀 Application (프로젝트 적용)

### Example: Mini Core Banking System

* Git churn 분석을 통해 `AccountService`가 높은 변경 빈도를 보이는 핵심 영역임을 확인
* Bug hotspot 분석 결과, 동일 영역에서 반복적인 수정 발생
* 이를 기반으로 도메인 책임 분리 및 트랜잭션 경계 재설계 수행

---

## 📈 Impact

* 변경 영향 범위 감소
* 트랜잭션 안정성 향상
* 유지보수성 및 가독성 개선
* 리팩토링 대상 선정의 근거 확보

---

## 🧠 Key Insight

> 코드베이스는 코드만으로 이해되지 않는다.
> Git 히스토리는 시스템의 구조적 문제와 팀의 운영 상태를 함께 보여준다.

---
