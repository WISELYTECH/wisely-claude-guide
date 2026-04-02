# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 저장소 개요

Wisely 전 구성원(엔지니어 + 비엔지니어)을 위한 **Claude 활용 가이드 문서 저장소**. 코드가 아닌 마크다운 문서로 구성되며, Claude Desktop, Cowork, Connector, Claude Code 사용법을 다룬다.

## 문서 구조

- `docs/` — 사용자 가이드 (시작하기, 컨텍스트 관리, 비엔지니어 가이드)
- `reference/` — 기능별 상세 레퍼런스 (CLAUDE.md 작성법, Skills, Hooks, Settings, MCP, IDE, Agent Teams)
- `skill/claude-guide/` — `/claude-guide` Skill 정의 (SKILL.md + 참조 문서 번들)

## `/claude-guide` Skill

이 저장소의 핵심 산출물. 다른 프로젝트에서 심볼릭 링크로 설치하여 사용:
```bash
ln -s /path/to/wisely-claude-guide/skill/claude-guide .claude/skills/claude-guide
```
- `SKILL.md`가 `reference/` 디렉터리의 문서를 `@reference/...` 로 참조
- 사용자가 `/claude-guide <질문>`으로 호출하면 참조 문서 기반으로 답변

## 문서 작성 규칙

- 언어: 한국어 (명령어/코드는 영어 유지)
- 포맷: 마크다운, 코드블록에 언어 태그 필수
- 대상 독자: 비엔지니어 포함 — 기술 용어는 간단히 풀어서 설명
- 각 문서 하단에 `**마지막 업데이트**: YYYY-MM-DD` 포함
