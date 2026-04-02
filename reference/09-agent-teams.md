# Agent Teams (Multi-Agent)

## 개요

- 여러 독립 Claude Code 세션이 공유 태스크 리스트 기반으로 병렬 작업
- 팀 리더가 팀원(teammate)을 생성하고 태스크를 할당
- 각 팀원은 완전히 독립된 컨텍스트를 가짐
- 실험적 기능 — v2.1.32 이상 필요

---

## 활성화

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

`~/.claude/settings.json` 또는 프로젝트 `.claude/settings.json`에 설정.

---

## Subagent vs Agent Teams

| 항목 | Subagent | Agent Teams |
|---|---|---|
| 컨텍스트 | 별도, 결과만 호출자에 반환 | 완전 독립 |
| 커뮤니케이션 | 메인에게만 보고 | 팀원 간 직접 메시지 가능 |
| 조율 | 메인 에이전트가 관리 | 공유 태스크 리스트, 자체 조율 |
| 토큰 비용 | 낮음 | 높음 (팀 크기에 비례) |
| 적합한 상황 | 단일 위임 작업 | 병렬 독립 작업 |

---

## 디스플레이 모드

| 모드 | 설명 | 전환 방법 |
|------|------|-----------|
| `in-process` | 단일 터미널 내 팀원 전환 | `Shift+Down` 으로 팀원 간 이동 |
| `split-panes` | tmux / iTerm2 분할 화면 | 터미널 앱에서 창 분할 |

---

## 팀 설정 파일

```
~/.claude/teams/{team-name}/config.json
```

---

## 팀 관련 Hooks

| Hook | 용도 | exit 코드 |
|------|------|-----------|
| `TeammateIdle` | 팀원 유휴 상태 감지 | `exit 2` → 계속 작업 유지 |
| `TaskCreated` | 태스크 생성 시 차단 가능 | `exit 2` → 차단 |
| `TaskCompleted` | 태스크 완료 시 후처리 가능 | `exit 2` → 차단 |

---

## .claude/agents/ 디렉토리

- 프로젝트별 에이전트 역할 정의 파일 위치
- 각 `.md` 파일이 하나의 에이전트 역할을 정의
- 팀 구성 시 해당 역할 파일을 참조하여 팀원 생성

```
.claude/agents/
├── architect.md
├── code-reviewer.md
├── qa-tester-high.md
└── ...
```

---

## 제한사항

- 세션 재개 시 `in-process` 팀원 복원 불가
- 태스크 상태 동기화 지연 가능
- 세션당 1팀 (중첩 팀 불가)
- VS Code 통합 터미널에서 `split-panes` 미지원

---

## Wisely 실전 사례

### commerce-web 에이전트 구성 (10개)

| 에이전트 | 역할 |
|----------|------|
| `architect` | 구조/의존성 방향/폴더 컨벤션 리뷰 |
| `code-reviewer` | 코드 품질 리뷰 + 리팩토링 제안 |
| `design-system-expert` | wisely-ds 디자인 시스템 전문가 |
| `qa-tester-high` | 요청사항 종합 QA |
| `tdd-guide` | TDD 가이드 |
| `e2e-runner` | E2E 테스트 실행 |
| `gate-keeper` | 머지 전 최종 검증 |
| `refactor-cleaner` | 중복/죽은 코드 정리 |
| `security-reviewer` | 보안 검토 |
| `mixpanel-tracking` | 이벤트 트래킹 검토 |

---

### agent-routing 파이프라인 (commerce-web)

코드 변경 시 `.claude/rules/agent-routing.md` 규칙에 따라 자동으로 실행되는 4단계 파이프라인.

**실행 단계:**

1. Code Review → `.claude/agents/code-reviewer.md`
2. Architecture Review → `.claude/agents/architect.md`
3. Web Vitals Review → `.claude/rules/web-vitals-review.md` (UI 변경 시에만)
4. QA → `.claude/agents/qa-tester-high.md`

**스킵 조건:**

| 변경 범위 | 실행 단계 |
|-----------|-----------|
| 단순 질의응답 (코드 변경 없음) | 전체 스킵 |
| 타입/테스트 파일만 변경 | Code Review만 |
| 1-3개 파일 (UI 없음) | Code Review + QA |
| 1-3개 파일 (UI 포함) | Code Review + Web Vitals + QA |
| 대규모 변경 (4개 파일 이상) | 4단계 모두 실행 |
