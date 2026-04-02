# Agent Teams 규모 최적화 가이드

> 심화 주제 — [09-agent-teams.md](../reference/09-agent-teams.md) 기본 가이드를 먼저 읽고 오세요.

---

## 개요

Agent Teams를 실무에서 운영할 때 **팀 규모, 토큰 비용, 작업 유형**에 따른 효율성 차이를 정리한 문서입니다. Anthropic 공식 문서와 공신력 있는 연구를 기반으로 합니다.

---

## 팀 크기 권장사항

공식 문서 원문 (`Best Practices > Choose an appropriate team size`):

> "**Start with 3-5 teammates** for most workflows. This balances parallel work with manageable coordination."

> "**Diminishing returns**: beyond a certain point, additional teammates don't speed up work proportionally"

> "**Coordination overhead increases**: more teammates means more communication, task coordination, and potential for conflicts"

> "Three focused teammates often outperform five scattered ones."

### 요약

| 항목 | 내용 |
|------|------|
| **최적 팀 크기** | 3-5명 |
| **추가 인원 효과** | 비례하지 않음 (diminishing returns) |
| **핵심 병목** | Coordination overhead — 통신량, 태스크 조율, 충돌 가능성 증가 |
| **토큰 비용** | 팀원 수에 비례하여 선형 증가 (각 팀원이 독립 컨텍스트 윈도우 사용) |
| **파일 충돌** | 같은 파일을 여러 팀원이 수정하면 덮어쓰기 발생 |

---

## 멀티 에이전트는 언제 효과적인가

Anthropic이 자사 멀티 에이전트 리서치 시스템을 구축하며 공유한 데이터에 따르면, 멀티 에이전트 구성은 **모든 작업에 효과적인 것이 아니라 특정 조건에서만 유의미한 성능 향상**을 보입니다.

### 성능과 비용

| 구성 | 성능 (내부 평가 기준) | 토큰 소모 (채팅 대비) |
|------|---------------------|---------------------|
| 싱글 에이전트 (Opus) | 기준선 | ~4x |
| 멀티 에이전트 (Opus + Sonnet) | **+90.2%** | **~15x** |

멀티 에이전트가 싱글 대비 약 90% 성능 향상을 달성했지만, 토큰 소모는 채팅 대비 약 15배입니다. 토큰 사용량 단독으로 BrowseComp 성능 분산의 80%를 설명합니다.

### 멀티 에이전트가 잘 맞는 작업

- 대규모 **병렬화**가 가능한 작업 (3-5개 서브에이전트 동시 실행 → 리서치 시간 최대 90% 단축)
- 단일 컨텍스트 윈도우에 담기지 않는 대량 정보 처리
- 다수의 복잡한 외부 도구를 동시에 사용하는 작업

### 멀티 에이전트가 부적합한 작업

- 에이전트 간 **공유 컨텍스트**가 필요한 작업 (예: 대부분의 코딩 작업)
- 높은 상호 의존성이 있는 작업
- 실시간 조율이 필요한 작업

---

## "단순한 것부터 시작하라"

Anthropic은 에이전트 아키텍처 가이드에서 다음 원칙을 강조합니다:

> "Find the simplest solution possible, and only increase complexity when needed. This might mean not building agentic systems at all."

멀티 에이전트 시스템은 **레이턴시와 비용을 희생하여 더 나은 태스크 성능을 얻는 트레이드오프**입니다. 이 트레이드오프가 정당한지 신중하게 평가해야 합니다.

### Anthropic이 정의한 5가지 아키텍처 패턴

| 패턴 | 설명 | 적합한 상황 |
|------|------|------------|
| **Chaining** | 순차 처리, 각 단계 출력이 다음 입력 | 정해진 단계가 있는 작업 |
| **Routing** | 입력 분류 후 전문 처리로 분기 | 입력 유형이 다양한 작업 |
| **Parallelization** | 독립 서브태스크 동시 실행 | 분할 가능한 작업 |
| **Orchestrator-Workers** | 중앙 LLM이 동적으로 태스크 분해·위임·통합 | 예측 불가능한 문제 공간 |
| **Evaluator-Optimizer** | 하나가 생성, 다른 하나가 평가하는 피드백 루프 | 품질 기준이 명확한 작업 |

Agent Teams는 이 중 **Orchestrator-Workers** 패턴에 해당합니다. 단순한 작업이라면 Chaining이나 단일 에이전트로 충분할 수 있습니다.

---

## 대규모 실험이 말하는 것

Google DeepMind의 연구 "Towards a Science of Scaling Agent Systems"(2024)는 **180개 에이전트 설정**을 5가지 아키텍처, 3개 LLM 패밀리에 걸쳐 실험했습니다.

### 핵심 결과

| 작업 유형 | 멀티 에이전트 효과 |
|-----------|-------------------|
| **병렬화 가능한 작업** | 중앙 조율 시 **+80.9%** 성능 향상 |
| **순차 추론 작업** | 모든 멀티 에이전트 변형이 **39-70% 성능 저하** |

### 에러 증폭 문제

| 조율 방식 | 에러 증폭 배율 |
|-----------|---------------|
| 독립 에이전트 (조율 없음) | **17.2x** |
| 중앙 조율 (Orchestrator) | **4.4x** |

조율 없이 에이전트를 늘리면 에러가 17배까지 증폭됩니다. Agent Teams의 Orchestrator 패턴은 이를 억제하는 역할을 합니다.

### 추가 발견

- **도구 16개 이상**을 사용하는 작업에서는 조율 비용이 멀티 에이전트 이점을 상쇄
- 싱글 에이전트 성능이 이미 **~45% 이상**이면 멀티 에이전트 추가 효과가 급감 (capability saturation)
- 태스크의 **도구 밀도(tool density)**와 **분해 가능성(decomposability)**으로 최적 아키텍처를 87% 정확도로 예측 가능

---

## 작업 배분 원칙

공식 문서의 핵심 원칙 두 가지:

1. **독립적 작업으로 분할** — 팀원 간 의존성을 최소화
2. **파일 충돌 회피** — 같은 파일을 여러 팀원이 동시에 수정하지 않도록 배분

| 작업 유형 | Agent Teams 적합도 | 이유 |
|-----------|-------------------|------|
| Code review, bug tracing | ✅ 좋음 | 읽기 위주, 파일 충돌 없음 |
| Architecture analysis | ✅ 좋음 | 독립적 분석 가능 |
| Refactoring shared types | ⚠️ 주의 | 같은 파일 수정 → 충돌 가능 |
| DB schema changes | ⚠️ 주의 | 의존성 높음, 순차 처리가 안전 |
| API contract updates | ⚠️ 주의 | 변경 영향 범위가 넓음 |

---

## 출처

- [Agent Teams — Anthropic 공식 문서](https://docs.anthropic.com/en/docs/claude-code/agent-teams) — Best Practices, Limitations 섹션
- [How We Built Our Multi-Agent Research System — Anthropic Engineering](https://www.anthropic.com/engineering/multi-agent-research-system) — 멀티 에이전트 성능/비용 데이터
- [Building Effective Agents — Anthropic Research](https://www.anthropic.com/research/building-effective-agents) — 에이전트 아키텍처 패턴, "단순한 것부터" 원칙
- [Towards a Science of Scaling Agent Systems — Google DeepMind (2024)](https://arxiv.org/abs/2512.08296) — 180개 설정 대규모 실험, 에러 증폭, 병렬 vs 순차

---

**마지막 업데이트**: 2026-04-03
