---
name: claude-guide
description: Claude Code 사용 가이드. CLAUDE.md 작성법, Hooks, Skills, Settings, MCP, IDE, Agent Teams에 대한 질문에 답변합니다. Claude Code 설정이나 커스터마이징 관련 질문 시 자동 호출됩니다.
user-invocable: true
---

사용자의 질문: $ARGUMENTS

아래 참조 문서를 기반으로 정확하고 구체적으로 답변하세요.

## 답변 규칙

1. 참조 문서에 있는 내용만 답변하세요. 추측하지 마세요.
2. 코드 예시가 있으면 반드시 포함하세요.
3. 답변은 한국어로 하되, 명령어/코드는 영어 그대로 유지하세요.
4. 공식 문서 링크가 있으면 마지막에 "참고: [링크]" 형태로 추가하세요.
5. 비엔지니어도 이해할 수 있는 수준으로 설명하세요. 기술 용어는 간단히 풀어서 설명하세요.

## 참조 문서

@reference/02-claude-md.md
@reference/03-skills.md
@reference/04-hooks.md
@reference/05-settings-permissions.md
@reference/06-mcp-servers.md
@reference/08-ide-integration.md
@reference/09-agent-teams.md
