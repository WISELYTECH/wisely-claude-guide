# Skills (커스텀 슬래시 커맨드) 참조

> 에이전트 참조 문서 — 구조화된 팩트 중심, 서술형 최소화

---

## 개요

- Skills: 재사용 가능한 프롬프트 템플릿, `/skill-name` 으로 호출
- `SKILL.md` 파일 하나가 슬래시 커맨드 하나를 생성
- Claude가 `description`을 보고 자동 호출하거나, 사용자가 직접 수동 호출

---

## Skill 위치

| 위치 | 경로 | 범위 |
|------|------|------|
| 개인 | `~/.claude/skills/<name>/SKILL.md` | 모든 프로젝트 |
| 프로젝트 | `.claude/skills/<name>/SKILL.md` | 해당 프로젝트만 |
| 플러그인 | `<plugin>/skills/<name>/SKILL.md` | 플러그인 활성화된 곳 |

---

## SKILL.md 구조

```yaml
---
name: my-skill
description: 이 스킬의 용도 설명. Claude의 자동 호출 판단에 사용됨.
---

스킬 지시사항 본문...
```

- `description`은 자동 호출 여부 결정의 핵심 — 명확하고 구체적으로 작성
- 최대 250자까지 표시됨

---

## 프론트매터 옵션 전체

| 필드 | 용도 |
|------|------|
| `name` | `/slash-command` 이름 |
| `description` | 자동 호출 판단 기준 (최대 250자 표시) |
| `disable-model-invocation: true` | 사용자만 호출 가능, Claude 자동 호출 불가 |
| `user-invocable: false` | Claude만 호출 가능 (슬래시 메뉴에서 숨김) |
| `allowed-tools` | 해당 스킬 실행 중 권한 프롬프트 없이 사용 가능한 도구 목록 |
| `model` | 모델 오버라이드 (예: `claude-opus-4`) |
| `effort` | 노력 수준 오버라이드 |
| `context: fork` | 격리된 서브에이전트에서 실행 |
| `agent` | `context: fork` 시 사용할 서브에이전트 유형 |
| `paths` | glob 패턴: 매칭 파일 작업 시에만 자동 활성화 |
| `hooks` | 스킬 라이프사이클에 스코프된 hooks |

---

## 인수 전달

| 변수 | 의미 |
|------|------|
| `$ARGUMENTS` | 슬래시 커맨드 뒤에 오는 전체 인수 문자열 |
| `$ARGUMENTS[N]` 또는 `$N` | N번째 위치 인수 (0-indexed) |
| `${CLAUDE_SESSION_ID}` | 현재 세션 ID |
| `${CLAUDE_SKILL_DIR}` | SKILL.md 파일이 위치한 디렉토리 경로 |

예시:
```markdown
/my-skill foo bar
```
- `$ARGUMENTS` → `foo bar`
- `$1` → `foo`
- `$2` → `bar`

---

## 동적 컨텍스트 주입

- `` !`command` `` — 셸 명령어 출력을 런타임에 주입
- Claude 실행이 아니라 **전처리 단계**에서 실행됨
- 실제 데이터를 스킬 본문에 포함시킬 때 사용

```yaml
---
name: review-pr
description: 현재 PR을 리뷰합니다.
---

## PR 컨텍스트

- Diff: !`gh pr diff`
- Comments: !`gh pr view --comments`
- Branch: !`git branch --show-current`

위 내용을 바탕으로 PR을 리뷰하세요.
```

---

## 서브에이전트 실행 (context: fork)

```yaml
---
name: deep-research
description: 주어진 주제를 심층 조사합니다.
context: fork
agent: Explore
---

$ARGUMENTS 를 조사하세요.
다음 항목을 포함하세요:
- 관련 파일 목록
- 핵심 패턴
- 의존성
```

- `context: fork`: 현재 컨텍스트와 격리된 서브에이전트에서 실행
- `agent`: 서브에이전트 유형 지정 (Explore, Executor 등)
- 현재 세션에 영향 없이 독립적으로 실행됨

---

## 번들 스킬 (Claude Code 기본 제공)

| 스킬 | 용도 |
|------|------|
| `/batch <instruction>` | 병렬 대규모 코드 변경 (git worktree 활용) |
| `/simplify` | 변경된 코드 리뷰 및 리팩토링 제안 |
| `/loop [interval] <prompt>` | 지정 간격으로 프롬프트 반복 실행 |
| `/debug` | 디버그 로깅 활성화 |
| `/claude-api` | Claude API 레퍼런스 컨텍스트 로드 |

---

## allowed-tools 활용 예시

```yaml
---
name: safe-deploy
description: 안전하게 배포합니다.
allowed-tools:
  - Bash(npm run build)
  - Bash(npm run deploy)
  - Read
---

빌드 후 배포를 실행합니다.
```

- `allowed-tools`에 포함된 도구는 실행 시 권한 프롬프트 생략
- 반복 실행 스킬에서 UX 개선에 유용

---

## paths 조건부 활성화

```yaml
---
name: react-review
description: React 컴포넌트 코드를 리뷰합니다.
paths:
  - "src/**/*.tsx"
  - "src/**/*.jsx"
---

이 컴포넌트가 다음 규칙을 따르는지 확인하세요:
- wisely-ds 컴포넌트 사용 여부
- inline style 미사용 여부
- 접근성 속성 포함 여부
```

- `paths` 매칭 파일 작업 시에만 Claude가 자동 활성화 고려
- 관련 없는 작업에서는 로드되지 않음

---

## Wisely 실전 사례

### `.claude/commands/` — 9개 커맨드 (commerce-web)

| 커맨드 | 용도 |
|--------|------|
| `pr-ready` | PR 생성 전 검증 워크플로 |
| `audit` | 코드베이스 보안/품질 감사 |
| `check` | 빌드 및 테스트 통합 검사 |
| `fix-all-errors` | 타입/린트 에러 일괄 수정 |
| `impl` | 스펙 기반 기능 구현 |
| `review` | PR 리뷰 워크플로 |
| `qa` | QA 체크리스트 실행 |
| `deploy-check` | 배포 전 사전 검증 |
| `ship` | 랜딩 및 배포 통합 워크플로 |

### `skills/gemini-analyze.md` — Gemini CLI 연동 (oh-my-wisely)

```yaml
---
name: gemini-analyze
description: Gemini CLI를 활용해 대규모 코드베이스를 분석합니다.
context: fork
---

!`gemini -p "$ARGUMENTS"`
```

- Gemini의 대용량 컨텍스트 윈도우를 활용한 분석
- Claude가 직접 처리하기 어려운 대규모 파일에 위임

### commerce-web-api: openspec-* 4개 스킬

| 스킬 | 용도 |
|------|------|
| `openspec-propose` | OpenAPI 스펙 기반 변경 제안 |
| `openspec-apply-change` | 제안된 변경사항 코드 적용 |
| `openspec-explore` | 스펙 탐색 모드 진입 |
| `openspec-archive-change` | 완료된 변경사항 아카이브 |

---

## 자주 하는 실수

- `description` 없이 스킬 작성 → Claude가 자동 호출 불가, 수동 호출만 가능
- `context: fork` 없이 무거운 작업 실행 → 현재 컨텍스트 오염
- 셸 명령어를 Claude 지시사항으로 착각 → `` !`command` `` 구문 사용해야 전처리 단계에서 실행됨
- `user-invocable: false` 설정 후 수동 호출 시도 → 메뉴에서 숨겨져 있어 호출 불가
- 스킬 경로 오타 (`skills/` vs `commands/`) → 로드되지 않음

---

**마지막 업데이트**: 2026-04-02
