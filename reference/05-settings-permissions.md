# Settings & Permissions 참조

> 에이전트 참조 문서 — 구조화된 팩트 중심, 서술형 최소화

---

## 개요

- Claude Code는 계층형 설정 스코프 시스템을 사용
- 상위 스코프가 하위 스코프를 덮어씀
- Permissions: Claude가 실행할 수 있는 도구/명령어 범위를 선언적으로 제어

---

## 설정 스코프 시스템 (우선순위 높은 순)

| 스코프 | 파일 위치 | git 공유 | 용도 |
|--------|----------|----------|------|
| Managed | 시스템 레벨 managed-settings.json 또는 plist/registry | IT 관리 | 조직 전체 정책 강제 |
| Project | `.claude/settings.json` | 커밋 포함 | 프로젝트 팀 공통 규칙 |
| Local | `.claude/settings.local.json` | gitignore | 머신별 오버라이드 |
| User | `~/.claude/settings.json` | 개인 | 개인 기본 설정 |

- Project settings는 반드시 git에 포함하여 팀 전체 동일 적용
- Local settings는 `.gitignore`에 추가하여 개인 머신별 설정 분리

---

## settings.json 전체 구조

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "allow": ["Bash(git *)", "Bash(npm *)", "Read", "Edit"],
    "deny": ["Bash(rm *)", "Edit(.env*)"]
  },
  "hooks": {},
  "mcpServers": {},
  "env": {
    "NODE_ENV": "development"
  },
  "autoMemoryEnabled": true,
  "claudeMdExcludes": ["**/other-team/CLAUDE.md"],
  "initialPermissionMode": "plan"
}
```

---

## Permission 패턴 문법

| 패턴 예시 | 의미 |
|-----------|------|
| `Bash(git *)` | `git`으로 시작하는 모든 bash 명령 허용/거부 |
| `Bash(npm run *)` | `npm run`으로 시작하는 명령만 |
| `Edit(src/**/*.ts)` | `src/` 하위 `.ts` 파일 편집만 |
| `Edit(.env*)` | `.env`로 시작하는 파일 편집 거부 |
| `Read` | 모든 파일 읽기 허용 |
| `WebFetch(domain:github.com)` | `github.com` 도메인만 접근 허용 |
| `mcp__github__*` | 모든 GitHub MCP 도구 허용 |

- `allow`와 `deny` 둘 다 같은 패턴 문법 사용
- `deny`가 `allow`보다 우선 적용됨

---

## Permission Modes

| 모드 | 동작 | 시작 방법 |
|------|------|-----------|
| `default` | 매 액션마다 사용자 확인 요청 | `claude` |
| `plan` | 변경 계획 미리보기 후 일괄 실행 | `claude --mode plan` |
| `acceptEdits` | 파일 편집 자동 승인, 명령어 실행만 확인 | `claude --mode acceptEdits` |
| `auto` | 분류기 모델이 명령어 위험도 자동 검토 (Team/Enterprise 전용) | `claude --mode auto` |
| `bypassPermissions` | 모든 확인 건너뜀 (샌드박스 환경 전용) | `claude --mode bypassPermissions` |

기본 모드 설정:
```json
{
  "initialPermissionMode": "plan"
}
```

- `bypassPermissions`는 신뢰된 샌드박스 환경에서만 사용, 프로덕션 금지

---

## 환경변수 주입

```json
{
  "env": {
    "NODE_ENV": "development",
    "API_BASE_URL": "http://localhost:3000",
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

- `env` 블록의 변수는 Claude Code 실행 시 환경에 자동 주입
- 실험적 기능 플래그 활성화에 활용

---

## claudeMdExcludes

```json
{
  "claudeMdExcludes": [
    "**/other-team/CLAUDE.md",
    "**/legacy/**"
  ]
}
```

- 지정된 glob 패턴의 CLAUDE.md 파일을 컨텍스트 로드에서 제외
- 여러 팀이 공존하는 모노레포에서 관련 없는 팀 규칙 차단에 사용

---

## Wisely 실전 사례

### commerce-web: git push 보호

```json
{
  "permissions": {
    "allow": [
      "Bash(git push origin HEAD --force-with-lease)",
      "Bash(git push origin HEAD)"
    ],
    "deny": [
      "Bash(git push --force *)",
      "Bash(git push -f *)"
    ]
  }
}
```

- 일반 push와 `--force-with-lease`(안전한 강제 push)만 허용
- 무조건 `--force` / `-f` 강제 push 차단
- 실수로 인한 원격 브랜치 히스토리 손상 방지

### 실험적 에이전트 팀 기능 활성화

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

### settings.local.json 활용 패턴

```json
{
  "env": {
    "DATABASE_URL": "postgresql://localhost:5432/mydb_local",
    "AWS_PROFILE": "wisely-dev"
  },
  "permissions": {
    "allow": [
      "Bash(psql *)"
    ]
  }
}
```

- `.gitignore` 대상이므로 개인 API 키, 로컬 DB URL, 머신별 경로 등 저장
- 팀 공용 설정(`.claude/settings.json`)과 개인 설정 분리

---

## MCP 서버 설정

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/project"]
    }
  }
}
```

- `mcpServers`는 User 또는 Project 스코프 settings.json에 정의
- 환경변수 참조(`${VAR}`) 지원

---

## Hooks 설정 구조

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Bash tool about to run'"
          }
        ]
      }
    ],
    "PostToolUse": [...],
    "Stop": [...],
    "SubagentStop": [...]
  }
}
```

- Hooks는 별도 참조 문서(`04-hooks.md`) 참고
- settings.json의 `hooks` 블록에 정의하거나 SKILL.md 프론트매터에 스코프 지정 가능

---

## 주요 필드 빠른 참조

| 필드 | 타입 | 용도 |
|------|------|------|
| `permissions.allow` | `string[]` | 자동 허용 도구 패턴 목록 |
| `permissions.deny` | `string[]` | 명시적 거부 도구 패턴 목록 |
| `hooks` | `object` | 라이프사이클 훅 정의 |
| `mcpServers` | `object` | MCP 서버 연결 설정 |
| `env` | `object` | 환경변수 주입 |
| `autoMemoryEnabled` | `boolean` | Auto Memory 기능 on/off |
| `claudeMdExcludes` | `string[]` | 로드 제외할 CLAUDE.md glob 패턴 |
| `initialPermissionMode` | `string` | 기본 Permission Mode 설정 |

---

## 자주 하는 실수

- `deny` 없이 `allow`만 설정 → 나머지 도구는 여전히 프롬프트 확인 필요 (금지가 아님)
- `settings.local.json`을 git에 커밋 → 개인 API 키 노출 위험, `.gitignore`에 추가 필수
- `bypassPermissions`를 로컬 개발에 상시 사용 → 실수로 위험 명령어 실행 가능, 샌드박스 전용
- Project와 User 스코프 동일 키 중복 설정 → Project가 User를 덮어씀, 의도치 않은 오버라이드 주의
- `env` 블록에 시크릿 하드코딩 후 git 커밋 → `settings.local.json` 또는 시스템 환경변수 활용

---

**마지막 업데이트**: 2026-04-02
