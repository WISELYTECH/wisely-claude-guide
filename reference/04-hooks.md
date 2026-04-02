# Hooks 참조

## 개요

- Claude Code 라이프사이클에 개입하는 자동화 스크립트
- `settings.json`의 `hooks` 키에 설정
- Claude가 아닌 **하네스(harness)** 가 직접 실행 — Claude 응답 없이 동작
- 핸들러 타입: `command`, `http`, `prompt`, `agent`

---

## 라이프사이클 이벤트

| 이벤트 | 시점 | 주요 용도 |
|--------|------|-----------|
| `SessionStart` | 세션 시작 / compact 후 재개 | 컨텍스트 주입, 환경 로드 |
| `SessionEnd` | 세션 종료 | 정리 작업, 로그 저장 |
| `PreToolUse` | 도구 실행 전 | 검증, 차단, 리다이렉트 |
| `PostToolUse` | 도구 성공 후 | 자동 포맷, 린트, 로그 |
| `PostToolUseFailure` | 도구 실패 후 | 에러 처리, 알림 |
| `PermissionRequest` | 권한 대화상자 전 | 자동 승인 / 거부 |
| `Stop` | Claude 응답 완료 | 완료 검증, 후처리 |
| `PreCompact` | 컨텍스트 압축 전 | 중요 컨텍스트 보존 |
| `PostCompact` | 컨텍스트 압축 후 | 압축 후 상태 복원 |
| `SubagentStart` | 서브에이전트 시작 | 서브에이전트 컨텍스트 주입 |
| `SubagentStop` | 서브에이전트 종료 | 서브에이전트 결과 처리 |
| `TeammateIdle` | 팀원 유휴 상태 | 팀 작업 재배분 |
| `TaskCreated` | 태스크 생성 | 태스크 초기화 |
| `TaskCompleted` | 태스크 완료 | 완료 후 후처리 |
| `FileChanged` | 파일 변경 감지 | 파일 감시, 자동 반응 |
| `CwdChanged` | 작업 디렉토리 변경 | 환경 갱신, 컨텍스트 재로드 |
| `Notification` | 알림 필요 시점 | 데스크톱 알림 전송 |

---

## Matcher

- 도구명 기반 정규식 매칭 (대소문자 구분)
- 빈 문자열 `""` — 모든 이벤트에 매칭

**패턴 예시:**

| 패턴 | 매칭 대상 |
|------|-----------|
| `"Edit\|Write"` | Edit 또는 Write 도구 |
| `"Bash"` | Bash 도구만 |
| `"mcp__github__.*"` | GitHub MCP 전체 |
| `"Read\|Edit\|Write\|Bash"` | 주요 파일/실행 도구 |
| `".envrc\|.env"` | FileChanged 이벤트 파일명 매칭 |
| `"compact"` | SessionStart — compact 후 재개만 |

---

## 설정 구조

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write",
            "async": false
          }
        ]
      }
    ]
  }
}
```

**필드 설명:**

| 필드 | 타입 | 설명 |
|------|------|------|
| `matcher` | string | 도구명 / 파일명 정규식 |
| `type` | string | `command` / `http` / `prompt` / `agent` |
| `command` | string | 실행할 쉘 명령어 (type=command) |
| `async` | boolean | `true` — 비동기 실행 (결과 무시), `false` — 동기 |
| `timeout` | number | 타임아웃 (ms), 기본값 없음 |

---

## Exit 코드 규칙

| 코드 | 의미 | 동작 |
|------|------|------|
| `0` | 성공 | stdout 내용이 Claude 컨텍스트에 추가됨 |
| `2` | 차단 | stderr가 Claude 피드백으로 전달, 도구 실행 중단 |
| 기타 (`1`, `3`+) | 비차단 오류 | 실행 허용, stderr는 로그에만 기록 |

- `PreToolUse`에서 exit 2 → 해당 도구 호출 차단
- `PostToolUse`에서 exit 2 → Claude에게 에러 피드백 전달
- stdout은 항상 Claude 컨텍스트에 주입 (exit 0일 때)

---

## PreToolUse 결정 (구조화 출력)

JSON 출력으로 도구 실행 허용/거부를 제어한다.

| 결정 | 동작 |
|------|------|
| `allow` | 도구 실행 허용 |
| `deny` | 도구 실행 거부 (사유 Claude에게 전달) |
| `ask` | 사용자에게 직접 질문 |
| `defer` | 기본 동작에 위임 |

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",
    "permissionDecisionReason": "grep 대신 rg를 사용하세요"
  }
}
```

---

## PermissionRequest 결정 (구조화 출력)

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PermissionRequest",
    "decision": {
      "behavior": "allow"
    }
  }
}
```

- `behavior`: `"allow"` / `"deny"` / `"ask"`

---

## Hook 환경변수

| 변수 | 설명 |
|------|------|
| `CLAUDE_PROJECT_DIR` | 프로젝트 루트 절대 경로 |
| `CLAUDE_PLUGIN_ROOT` | 플러그인 루트 경로 |
| `CLAUDE_ENV_FILE` | SessionStart에서 env 변수를 영속화할 파일 경로 |
| `CLAUDE_HOOK_EVENT` | 현재 실행 중인 이벤트명 |
| `CLAUDE_TOOL_NAME` | 현재 도구명 (PreToolUse, PostToolUse) |

- `CLAUDE_ENV_FILE`에 `KEY=VALUE` 형태로 쓰면 세션 전체에서 유지됨
- 스크립트 내에서 `$CLAUDE_PROJECT_DIR`로 절대 경로 참조 권장

---

## 실전 예시 패턴

### 자동 포맷 (PostToolUse)

파일 편집 후 Prettier 자동 실행:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write"
          }
        ]
      }
    ]
  }
}
```

### Python 린팅 자동 실행 (PostToolUse)

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | grep '\\.py$' | xargs ruff check --fix 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

### macOS 알림 (Notification)

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude needs input\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```

### 자동 권한 승인 (PermissionRequest)

ExitPlanMode 권한 자동 허용:

```json
{
  "hooks": {
    "PermissionRequest": [
      {
        "matcher": "ExitPlanMode",
        "hooks": [
          {
            "type": "command",
            "command": "echo '{\"hookSpecificOutput\": {\"hookEventName\": \"PermissionRequest\", \"decision\": {\"behavior\": \"allow\"}}}'"
          }
        ]
      }
    ]
  }
}
```

### 컨텍스트 재주입 (SessionStart — compact 후)

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "compact",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Reminder: pnpm 사용. 커밋 전 테스트 실행.'"
          }
        ]
      }
    ]
  }
}
```

### grep 차단 — rg 사용 강제 (PreToolUse)

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.command' | grep -q '^grep' && echo '{\"hookSpecificOutput\":{\"hookEventName\":\"PreToolUse\",\"permissionDecision\":\"deny\",\"permissionDecisionReason\":\"grep 대신 rg(ripgrep)를 사용하세요\"}}' || true"
          }
        ]
      }
    ]
  }
}
```

### Stop hook — 빌드 검증

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/pre-stop-check.sh",
            "async": false
          }
        ]
      }
    ]
  }
}
```

---

## Wisely 실전 사례

| 프로젝트 | Hook | 동작 |
|----------|------|------|
| `commerce-web-api` | `PreToolUse` / `Stop` | `pre-pr-build-check.sh` — PR 생성 전 빌드 검증 |
| `wisely-search` | `PostToolUse` + `Stop` | Python ruff 린팅 자동 실행 |
| `commerce-web` | `PostToolUse` | agent-routing 파이프라인 — 코드 변경 시 자동 리뷰/QA |

---

## stdin으로 전달되는 JSON 구조

Hook 스크립트는 stdin으로 이벤트 컨텍스트를 받는다.

**PreToolUse / PostToolUse:**
```json
{
  "tool_name": "Edit",
  "tool_input": {
    "file_path": "/path/to/file.ts",
    "old_string": "...",
    "new_string": "..."
  },
  "tool_response": { }
}
```

**SessionStart:**
```json
{
  "trigger": "compact"
}
```

**FileChanged:**
```json
{
  "file_path": "/path/to/.envrc"
}
```

---

## 트러블슈팅

| 증상 | 원인 | 해결 |
|------|------|------|
| Hook 미실행 | matcher 불일치 | `/hooks`로 등록 확인, 대소문자 확인 |
| 스크립트 오류 | stdin 파싱 실패 | `echo '{"tool_name":"Bash"}' \| ./script.sh` 로 수동 테스트 |
| 실행 권한 오류 | 스크립트 미실행 | `chmod +x script.sh` |
| 경로 오류 | 상대 경로 사용 | `"$CLAUDE_PROJECT_DIR/.claude/hooks/script.sh"` 절대 경로 사용 |
| async 동작 불일치 | async 값 혼동 | `async: false` — 결과 대기, `async: true` — 백그라운드 |
| exit 2 미작동 | stdout에 JSON 출력 | 구조화 출력은 stdout, 에러 메시지는 stderr로 분리 |

**수동 테스트 방법:**

```bash
# 스크립트 직접 테스트
echo '{"tool_name":"Edit","tool_input":{"file_path":"test.ts"}}' | bash .claude/hooks/my-hook.sh

# jq로 stdin 파싱 확인
echo '{"tool_input":{"file_path":"foo.py"}}' | jq -r '.tool_input.file_path'
```

**settings.json 위치:**
- 프로젝트 수준: `.claude/settings.json`
- 글로벌: `~/.claude/settings.json`
- 프로젝트 설정이 글로벌보다 우선 적용

---

## type별 핸들러 요약

| type | 동작 | 주요 사용처 |
|------|------|------------|
| `command` | 쉘 명령어 실행, stdin으로 컨텍스트 전달 | 포맷, 린트, 알림 |
| `http` | HTTP 엔드포인트 POST | 웹훅, 외부 서비스 연동 |
| `prompt` | Claude에게 프롬프트 주입 | 동적 지시 추가 |
| `agent` | 서브에이전트 실행 | 복잡한 자동화 파이프라인 |
