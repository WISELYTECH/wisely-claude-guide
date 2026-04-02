# MCP Servers

## MCP란?

- **Model Context Protocol** — Claude Code에 외부 도구, DB, API, 서비스를 연결하는 표준 프로토콜
- MCP 서버가 Claude가 사용할 수 있는 도구(tool) 목록을 제공
- Claude는 MCP 서버를 통해 코드베이스 외부 시스템과 직접 상호작용 가능
- 표준화된 인터페이스: Claude ↔ MCP 서버 ↔ 외부 서비스

---

## 설정 방법

### CLI로 추가

```bash
# stdio 서버 (로컬 프로세스)
claude mcp add <name> <command>

# HTTP 서버 (원격 엔드포인트)
claude mcp add <name> http://host:port/mcp

# 예시
claude mcp add github npx -y @anthropic-mcp/server-github
claude mcp add postgres npx -y @anthropic-mcp/server-postgres postgresql://localhost/mydb
```

### 수동 설정 (settings.json 또는 mcp.json)

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@anthropic-mcp/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "your-token"
      }
    },
    "slack": {
      "command": "npx",
      "args": ["-y", "@anthropic-mcp/server-slack"],
      "env": {
        "SLACK_BOT_TOKEN": "xoxb-...",
        "SLACK_TEAM_ID": "T..."
      }
    },
    "postgres": {
      "command": "npx",
      "args": [
        "-y",
        "@anthropic-mcp/server-postgres",
        "postgresql://user:pass@localhost/dbname"
      ]
    }
  }
}
```

---

## 설치 스코프

| 스코프 | 위치 | 범위 |
|--------|------|------|
| User | `~/.claude/mcp.json` | 모든 프로젝트에서 사용 가능 |
| Project | `.claude/mcp.json` | 해당 프로젝트 전용 (팀 공유 가능) |
| Local | `.claude/settings.local.json` | 로컬 전용 (git 제외 권장) |

- User 스코프: 개인 인증 토큰이 포함되는 서버에 적합
- Project 스코프: 팀 전체가 동일하게 사용하는 서버에 적합
- Local 스코프: 개인 설정이지만 특정 프로젝트에만 한정할 때

---

## 주요 MCP 서버

| 서버 | 패키지 / 명령 | 용도 |
|------|--------------|------|
| GitHub | `@anthropic-mcp/server-github` | 리포 검색, 이슈/PR 관리, 코드 읽기 |
| Slack | `@anthropic-mcp/server-slack` | 메시지 전송, 채널/스레드 읽기 |
| Figma | `@anthropic-mcp/server-figma` | 디자인 읽기, 컴포넌트 코드 연결 |
| Notion | `@anthropic-mcp/server-notion` | 페이지 읽기/쓰기, DB 쿼리 |
| Sentry | `@anthropic-mcp/server-sentry` | 에러 모니터링, 이슈 조회 |
| PostgreSQL | `@anthropic-mcp/server-postgres` | DB 쿼리, 스키마 탐색 |
| MySQL | `@anthropic-mcp/server-mysql` | DB 쿼리, 테이블 조회 |
| Google Calendar | `@anthropic-mcp/server-google-calendar` | 일정 조회/생성/수정 |
| Filesystem | `@anthropic-mcp/server-filesystem` | 파일시스템 확장 접근 |
| Brave Search | `@anthropic-mcp/server-brave-search` | 웹 검색 |

---

## 권한 관리

### allowedTools로 자동 승인 설정

```json
{
  "allowedTools": [
    "mcp__github__search_repositories",
    "mcp__github__create_issue",
    "mcp__slack__send_message",
    "mcp__postgres__query"
  ]
}
```

- 패턴: `mcp__<server>__<tool>`
- 나열된 도구는 사용 시마다 확인 없이 자동 실행
- 나열되지 않은 도구는 실행 전 사용자 승인 필요

### 도구 차단 설정

```json
{
  "deniedTools": [
    "mcp__postgres__execute",
    "mcp__github__delete_repository"
  ]
}
```

---

## 관리 명령어

| 명령어 | 동작 |
|--------|------|
| `/mcp` | Claude 세션 내 연결된 서버 목록 및 상태 확인 |
| `claude mcp list` | CLI에서 등록된 서버 목록 출력 |
| `claude mcp remove <name>` | 서버 제거 |
| `claude mcp add <name> <cmd>` | 새 서버 등록 |

---

## 환경 변수 관리

```bash
# .env 파일에 토큰 보관 (git 제외)
GITHUB_PERSONAL_ACCESS_TOKEN=ghp_...
SLACK_BOT_TOKEN=xoxb-...
NOTION_API_KEY=secret_...

# mcp.json에서 참조
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@anthropic-mcp/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_PERSONAL_ACCESS_TOKEN}"
      }
    }
  }
}
```

- 토큰은 `.gitignore`에 포함된 파일에 보관
- Project 스코프 `mcp.json`에 실제 토큰 값 직접 기입 금지

---

## 트러블슈팅

| 증상 | 원인 | 해결 |
|------|------|------|
| 서버 미연결 | 명령 경로 오류, 패키지 미설치 | `claude mcp list`로 상태 확인, `npx` 경로 점검 |
| 인증 실패 | env 변수 누락 또는 만료 토큰 | env 값 재확인, 토큰 갱신 |
| 도구 미표시 | 서버 초기화 실패 | Claude 세션 재시작, 서버 재등록 |
| 도구 실행 차단 | `deniedTools` 또는 권한 설정 | `allowedTools` 확인, 설정 수정 |
| HTTP 서버 연결 불가 | 포트/호스트 오류, 방화벽 | 엔드포인트 URL 재확인 |

### 디버그 순서
1. `claude mcp list` — 서버 등록 여부 확인
2. `/mcp` — 세션 내 연결 상태 확인
3. 서버 명령을 터미널에서 직접 실행해 오류 메시지 확인
4. env 변수 값 직접 출력해 로드 여부 확인
5. Claude 세션 종료 후 재시작

---

## 관련 파일 위치

```
~/.claude/
  mcp.json              # User 스코프 MCP 설정
  settings.json         # 전역 설정 (allowedTools 포함)

<project>/
  .claude/
    mcp.json            # Project 스코프 MCP 설정
    settings.local.json # Local 스코프 설정 (git 제외 권장)
```
