# Laws of Software Engineering (HN Reading Note)

> Source: Laws of Software Engineering 
> Hacker News Top Post

---

## Overview

소프트웨어 개발 전반에서 반복적으로 나타나는 법칙들을 정리한 자료.
설계, 팀, 일정, 의사결정 등 다양한 영역에서 공통적으로 적용되는 경험 기반 원칙을 다룬다.

이 문서는 이론 정리보다는
**실무에서 발생하는 문제를 설명하는 관점**에 가깝다.

---

## Key Insights

### 1. Complexity is inevitable

* Gall's Law
  잘 동작하는 복잡한 시스템은 단순한 시스템에서 진화한다.

* Tesler's Law
  복잡성은 제거할 수 없고, 다른 곳으로 이동할 뿐이다.

정리
복잡한 구조를 처음부터 설계하기보다
작은 단위에서 점진적으로 확장하는 접근이 현실적이다.

---

### 2. Abstractions are imperfect

* Law of Leaky Abstractions
  모든 추상화는 일정 수준에서 내부 구현이 드러난다.

* Hyrum's Law
  사용자가 많아질수록 모든 동작이 의존성이 된다.

정리
추상화에 과도하게 의존하면 예상하지 못한 문제로 이어질 수 있다.

---

### 3. Planning is unreliable

* Hofstadter's Law
  예상보다 항상 더 오래 걸린다.

* Ninety-Ninety Rule
  마지막 단계가 전체 시간의 상당 부분을 차지한다.

정리
일정은 본질적으로 불확실하며, 여유를 고려한 계획이 필요하다.

---

### 4. Team structure influences system design

* Conway's Law
  조직 구조는 시스템 구조에 반영된다.

* Brooks's Law
  지연된 프로젝트에 인력을 추가하면 더 지연된다.

정리
문제의 원인이 코드가 아니라 팀 구조일 가능성도 고려해야 한다.

---

### 5. Simplicity is a core principle

* KISS
  가능한 한 단순하게 유지해야 한다.

* YAGNI
  필요하지 않은 기능은 구현하지 않는다.

정리
미래를 과도하게 가정한 설계는 기술 부채로 이어질 가능성이 높다.

---

### 6. Human factors affect decisions

* Dunning-Kruger Effect
  지식이 부족할수록 과신하는 경향이 있다.

* Sunk Cost Fallacy
  이미 투자한 비용 때문에 비효율적인 선택을 지속한다.

정리
개발 과정에서의 의사결정은 기술적 요소뿐 아니라 인지 편향의 영향을 받는다.

---

## Personal Notes

* 확장성을 우선 고려한 설계가 실제로는 불필요한 복잡성을 만든 경우가 많았다.
* 시스템 복잡도는 설계 실패라기보다 점진적 변화의 결과일 수 있다.
* 일정 지연은 예외적인 상황이 아니라 일반적인 현상에 가깝다.

---

## Practical Applications

* 기능 추가 전 필요성 검증 (YAGNI)
* 추상화 설계 시 한계 고려
* 일정 산정 시 여유 포함
* 팀 구조와 시스템 구조의 관계 점검

---

## Summary

소프트웨어 개발은 기술적 문제를 넘어서
복잡성, 사람, 시간의 상호작용 속에서 이루어진다.
