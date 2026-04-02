# Claude 가이드 — Wisely

> Wisely 전 구성원을 위한 Claude 활용 가이드 (Desktop, Cowork, Claude Code, Connector)

## 이 가이드에 대해

- **대상**: Wisely 전 구성원 (엔지니어 + 비엔지니어)
- **언어**: 한국어
- **최종 업데이트**: 2026-04-02

Claude는 코딩 도구가 아닙니다. **업무를 자동화하고**, Connector로 외부 도구와 연결하고, Cowork로 결과물(엑셀, PPT, 문서)을 만들고, 필요하면 코드를 작성해서 더 복잡한 자동화도 할 수 있는 **업무 도구**입니다.

## 가이드 문서

| 문서 | 설명 |
|------|------|
| [Claude 활용 가이드](docs/10-non-engineer-guide.md) | Cowork, Connector, Claude Code 소개 + 직군별 활용 사례 |
| [시작하기 (Claude Code)](docs/01-getting-started.md) | Claude Code 설치, 인증, 첫 실행, 핵심 명령어 |
| [컨텍스트 관리](docs/07-context-management.md) | 사용량 확인, 토큰 절약, 세션 규칙 |

## /claude-guide 설치 방법

위 가이드 외에, Claude Code에게 직접 질문할 수 있는 `/claude-guide` Skill을 제공합니다.

```bash
# 1. 이 리포를 clone
git clone git@github.com:anthropics-wisely/wisely-claude-guide.git

# 2. 프로젝트에 심볼릭 링크 생성
cd your-project
ln -s /path/to/wisely-claude-guide/skill/claude-guide .claude/skills/claude-guide

# 3. Claude Code에서 사용
claude
> /claude-guide hooks 어떻게 설정해?
> /claude-guide CLAUDE.md 베스트 프랙티스
> /claude-guide MCP 서버 추가하는 법
```

## 기여하기

- 오류 발견 시: 이슈 등록 또는 직접 PR
- 문서 컨벤션: 한국어, 마크다운, 코드블록에 언어 태그

## 변경 이력

| 날짜 | 내용 |
|------|------|
| 2026-04-02 | 초판 발행 |
