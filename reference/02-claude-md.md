# CLAUDE.md 작성 참조

> 에이전트 참조 문서 — 구조화된 팩트 중심, 서술형 최소화

---

## 개요

- CLAUDE.md: 프로젝트별 지시사항 파일, 매 세션 시작 시 자동 로드
- 두 시스템이 공존:
  - **CLAUDE.md** — 사람이 직접 작성하는 프로젝트 설정
  - **Auto Memory** — Claude가 세션 간 학습 내용을 자동으로 기록 (`~/.claude/projects/<project>/memory/`)

---

## 스코프 계층 (우선순위 높은 순)

| 우선순위 | 종류 | 경로 | 특징 |
|----------|------|------|------|
| 1 | Managed | `/Library/Application Support/ClaudeCode/CLAUDE.md` (macOS) | IT 관리, 전사 적용 |
| 2 | Project | `./CLAUDE.md` 또는 `./.claude/CLAUDE.md` | git 공유, 팀 전체 |
| 3 | User | `~/.claude/CLAUDE.md` | 개인 전용, git 미포함 |
| 4 | Local | `.claude/settings.local.json` | 머신별 오버라이드 |

- 상위 스코프가 하위 스코프를 덮어씀
- Project CLAUDE.md는 반드시 git에 포함하여 팀 전체가 동일한 지시를 받도록 함

---

## 베스트 프랙티스

### 분량
- **200줄 이하 권장** — 길수록 준수율 저하, 컨텍스트 낭비
- 200줄 초과 시 `.claude/rules/`로 분리하여 필요 시점에만 로드

### 구조
- 마크다운 헤더(`##`, `###`)와 불릿(`-`)으로 구조화
- 테이블 활용 — 허용/금지 규칙, 의존성 규칙 등
- 코드블록에 언어 태그 명시

### 지시사항 작성 원칙
| 좋음 | 나쁨 |
|------|------|
| `2-space 들여쓰기 사용` | `코드를 깔끔하게 작성` |
| `jest로 테스트 실행: npm test` | `테스트를 잘 작성할 것` |
| `브랜치 네이밍: feat/JIRA-123-short-desc` | `좋은 브랜치 이름 사용` |

- **구체적이고 검증 가능한** 지시만 포함
- 모순 금지 — CLAUDE.md, sub-directory CLAUDE.md, `.claude/rules/` 간 지시가 충돌하면 안 됨
- 충돌 가능성이 있으면 `# Rule Priority` 섹션으로 우선순위 명시

### 생성 방법
```bash
# 프로젝트 루트에서
claude
> /init
```
`/init`: 프로젝트 구조 자동 분석 후 CLAUDE.md 초안 생성 → 팀 규칙에 맞게 커스터마이징

---

## 포함할 것 vs 제외할 것

| 포함 | 제외 |
|------|------|
| Claude가 추측 불가한 빌드 커맨드 | 코드에서 유추 가능한 정보 |
| 기본값과 다른 코드 스타일 규칙 | 언어 표준 컨벤션 |
| 테스트 러너 명령어 | 긴 설명이나 튜토리얼 |
| 브랜치 네이밍/PR 컨벤션 | 파일별 코드베이스 설명 |
| 필수 환경변수 목록 | 자주 변하는 정보 |
| 비직관적 동작이나 gotchas | 자명한 관행 |
| 레이어 간 의존성 규칙 | IDE 설정 (개인 취향) |

---

## 임포트 구문

```markdown
@path/to/file
```

- 다른 파일을 CLAUDE.md 컨텍스트에 포함시키는 구문
- 최대 5단계 깊이까지 재귀 임포트 지원
- 공통 규칙 파일을 여러 프로젝트에서 공유할 때 활용

**HTML 주석 주의:**
```html
<!-- 이 내용은 컨텍스트에서 제거됨 -->
```
- HTML 주석(`<!-- ... -->`)은 Claude 컨텍스트에서 자동 제거됨
- 사람용 주석에만 사용, 에이전트 지시사항에는 사용 금지

---

## .claude/rules/ 시스템

### 구조
```
.claude/
  rules/
    coding-standards.md      # paths 없음 → 항상 로드
    api-guidelines.md        # paths 있음 → 조건부 로드
    react-patterns.md        # paths 있음 → 조건부 로드
```

### 프론트매터 없는 파일 — 항상 로드
```markdown
# coding-standards.md
- 함수 최대 50줄
- KISS, DRY, YAGNI 원칙 준수
```

### 프론트매터 있는 파일 — 매칭 파일 작업 시에만 로드
```yaml
---
paths:
  - "src/api/**/*.ts"
---
# API 개발 규칙
- 모든 엔드포인트에 입력 검증 필수
- 응답 타입은 반드시 명시
```

```yaml
---
paths:
  - "src/**/*.tsx"
  - "src/**/*.jsx"
---
# React 컴포넌트 규칙
- wisely-ds 컴포넌트 우선 사용
- inline style 금지
```

### 장점
- CLAUDE.md 본문을 짧게 유지하면서 규칙을 풍부하게 관리
- 작업 파일과 무관한 규칙이 컨텍스트를 채우지 않음
- 규칙 파일별 독립적인 버전 관리 가능

---

## AGENTS.md

- 에이전트 역할 정의 파일 (아키텍처, 에이전트 라우팅 등)
- CLAUDE.md와 AGENTS.md 간 충돌 시 우선순위를 CLAUDE.md에서 명시

```markdown
# Rule Priority
문서 간 규칙이 충돌하면 AGENTS.md를 우선 적용
AGENTS.md에 없는 내용은 이 파일(CLAUDE.md) 따름
```

---

## Wisely 실전 사례

### commerce-web CLAUDE.md (29줄, 간결형)

```markdown
# commerce-web

## Build Commands
- dev: `npm run dev`
- build: `npm run build`
- test: `npm test`

## Rule Priority
문서 간 규칙이 충돌하면 AGENTS.md를 우선 적용

## Core Rules
- TypeScript strict mode
- 컴포넌트는 wisely-ds 우선
- PR 제목: `[JIRA-123] 한 줄 요약`
```

특징:
- 빌드 커맨드, 핵심 규칙만 간결하게
- 아키텍처 상세는 AGENTS.md에 위임
- "Rule Priority" 섹션으로 우선순위 명시

---

### commerce-web-api CLAUDE.md (154줄, 아키텍처 중심)

레이어 구조:
```
Controller → Service → Component → Entity
```

의존성 규칙 (테이블):
| 출발 | 도착 | 허용 |
|------|------|------|
| Controller | Service | O |
| Service | Component | O |
| Service | Service | X |
| Component | Entity | O |
| Module | Service export | X |

```markdown
## Dependency Rules
- Service → Service 직접 호출 금지
- Module에서 Service export 금지 (Controller에서 직접 주입)
- 컴포넌트 간 의존: 동일 모듈 내에서만 허용
```

특징:
- 레이어 구조와 의존성 규칙을 테이블로 명시
- 금지 패턴을 구체적으로 열거

---

### wisely-lambda CLAUDE.md (436줄, 포괄형)

```markdown
## Build Commands

### TypeScript (Node.js)
- build: `cd functions/typescript && npm run build`
- test: `npm test`

### Go
- build: `cd functions/go && go build ./...`
- test: `go test ./...`

### Python
- build: `cd functions/python && pip install -r requirements.txt`
- test: `pytest`

## Project Structure
각 런타임은 독립적인 디렉토리로 분리:
- `functions/typescript/` — Node.js Lambda
- `functions/go/` — Go Lambda
- `functions/python/` — Python Lambda
```

특징:
- 멀티 런타임(TypeScript/Go/Python) 빌드 커맨드 전체 포함
- 프로젝트별 독립 디렉토리 구조 명시
- 436줄로 길지만 런타임 다양성으로 불가피 (규칙 분리 고려 권장)

---

## Wisely .claude/rules/ 파일 목록

| 파일명 | 내용 | paths 설정 |
|--------|------|-----------|
| `coding-standards.md` | KISS·DRY·YAGNI, 함수 50줄 제한 | 없음 (항상 로드) |
| `preferWiselyDs.md` | wisely-ds 컴포넌트 우선 사용 규칙 | `src/**/*.tsx` |
| `web-vitals-review.md` | CLS·LCP·INP 체크리스트 | `src/**/*.tsx` |
| `mixpanelTracking.md` | 이벤트 네이밍 룰, 추가 절차 | `src/**/tracking/**` |
| `agent-routing.md` | 코드 변경 시 자동 리뷰/QA 파이프라인 | 없음 (항상 로드) |
| `useEffectEvent.md` | React useEffectEvent 패턴 가이드 | `src/**/*.tsx` |

---

## Auto Memory

- 위치: `~/.claude/projects/<project-name>/memory/MEMORY.md`
- 세션 시작 시 **첫 200줄** 자동 로드
- Claude가 직접 작성 — 사람이 편집 가능하지만 덮어쓰기 주의
- `/memory` 커맨드로 확인 및 편집

```markdown
# Memory Index

- [프로젝트 구조](project_structure.md)
- [팀 규칙](team_conventions.md)
- [자주 실행하는 커맨드](commands.md)
```

Auto Memory vs CLAUDE.md 구분:
| 항목 | CLAUDE.md | Auto Memory |
|------|-----------|-------------|
| 작성자 | 사람 | Claude |
| 내용 | 규칙, 커맨드, 컨벤션 | 학습 내용, 이전 세션 맥락 |
| 편집 | 직접 편집 | `/memory`로 편집 |
| git 포함 | 포함 (팀 공유) | 미포함 (개인) |

---

## 세션 시작 시 로드 순서

1. 시스템 프롬프트 (~4,200 토큰)
2. Managed CLAUDE.md (존재 시)
3. User CLAUDE.md (`~/.claude/CLAUDE.md`, 존재 시)
4. Project CLAUDE.md (`./CLAUDE.md` 또는 `./.claude/CLAUDE.md`)
5. sub-directory CLAUDE.md (작업 파일 경로 기준 상위 → 하위 순)
6. `.claude/rules/` 파일 — paths 없는 파일 전체 + 현재 작업 파일 매칭 파일
7. Auto Memory 첫 200줄 (`~/.claude/projects/<project>/memory/MEMORY.md`)
8. 환경 정보 (~280 토큰): 작업 디렉토리, OS, git 브랜치
9. MCP 도구 목록 (~120 토큰)

---

## sub-directory CLAUDE.md

- 프로젝트 루트 외 하위 디렉토리에도 CLAUDE.md 배치 가능
- 해당 디렉토리 내 파일 작업 시에만 추가 로드됨
- 상위 CLAUDE.md와 충돌 시 하위 CLAUDE.md가 우선 적용

```
project/
  CLAUDE.md              # 전체 프로젝트 규칙
  src/
    api/
      CLAUDE.md          # API 디렉토리 전용 규칙 (api 작업 시에만 로드)
    components/
      CLAUDE.md          # 컴포넌트 디렉토리 전용 규칙
```

활용 사례:
- 모노레포에서 패키지별 독립 규칙 관리
- 특정 모듈(예: 결제, 인증)에 추가 보안 규칙 적용
- 하위 디렉토리 전담 팀이 별도 컨벤션 유지할 때

---

## 자주 하는 실수

- CLAUDE.md에 파일별 설명 나열 → 코드베이스가 바뀔 때마다 수동 업데이트 필요, 제거 권장
- 200줄 초과 방치 → `.claude/rules/`로 분리
- HTML 주석으로 에이전트 지시 작성 → 컨텍스트에서 제거되므로 일반 마크다운으로 작성
- CLAUDE.md와 rules/ 파일 간 모순 지시 → 반드시 Rule Priority 섹션으로 해소
- 팀 규칙을 User 스코프(`~/.claude/CLAUDE.md`)에만 작성 → 다른 팀원에게 미적용

---

**마지막 업데이트**: 2026-04-02
