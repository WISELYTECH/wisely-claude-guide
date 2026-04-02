# IDE 통합

## VS Code 확장

### 설치

- Extensions (`Cmd+Shift+X`) → "Claude Code" 검색 → Install
- 최소 요구 버전: VS Code **1.98.0** 이상
- 확장 설치 후 터미널에서 `claude` 세션 시작 시 자동 연결

### 핵심 기능

| 기능 | 설명 |
|------|------|
| 인라인 diff 리뷰 | 변경 전후를 나란히 표시, 개별 수락/거절 가능 |
| `@`-mention 파일 참조 | `@auth.ts`, `@auth.ts#5-10` 형식으로 특정 파일/줄 참조 |
| Plan mode | 계획을 마크다운으로 VS Code에서 열고 인라인 코멘트 작성 |
| 대화 히스토리 | 이전 세션 검색 및 재개 |
| 체크포인트/되감기 | 특정 시점으로 대화 복원 |
| 대화 포크 | 현재 시점에서 새 분기 생성 |
| 상태바 표시 | 현재 세션 이름, 컨텍스트 사용량(토큰) 표시 |

### 단축키

| 단축키 (Mac / Windows·Linux) | 동작 |
|------------------------------|------|
| `Cmd+Esc` / `Ctrl+Esc` | 에디터 ↔ Claude 패널 포커스 전환 |
| `Cmd+Shift+Esc` / `Ctrl+Shift+Esc` | 새 Claude 대화 탭 열기 |
| `Option+K` / `Alt+K` | 커서 위치에 `@`-mention 삽입 |
| `Cmd+N` / `Ctrl+N` | 새 대화 시작 (Claude 패널 포커스 시) |

### 설정

- VS Code: **Settings** → Extensions → Claude Code
- 주요 설정 항목:

| 설정 키 | 타입 | 설명 |
|---------|------|------|
| `initialPermissionMode` | string | 기본 권한 모드 (`default` / `acceptEdits` / `bypassPermissions`) |
| `useCtrlEnterToSend` | boolean | `Ctrl+Enter`로 메시지 전송 (기본: `false`) |
| `autosave` | boolean | Claude 편집 후 파일 자동 저장 |

### IDE MCP 서버 (자동 실행)

확장이 로컬 MCP 서버를 자동으로 실행해 IDE 기능을 Claude에 노출:

- 바인드 주소: `127.0.0.1`, 랜덤 포트 (로컬 전용)
- Claude 세션 시작 시 자동 등록, 종료 시 자동 해제

| 도구 | 설명 |
|------|------|
| `mcp__ide__getDiagnostics` | 린트 오류, 타입 에러 읽기 |
| `mcp__ide__executeCode` | Jupyter 노트북에서 코드 셀 실행 |
| `mcp__ide__getOpenFiles` | 현재 열린 파일 목록 반환 |
| `mcp__ide__getCurrentSelection` | 에디터에서 선택된 텍스트 반환 |

---

## JetBrains 플러그인

### 지원 IDE

- IntelliJ IDEA
- PyCharm
- Android Studio
- WebStorm
- PhpStorm
- GoLand

### 설치

- **JetBrains Marketplace** → "Claude Code" 검색 → Install
- IDE 재시작 후 사이드바에 Claude 패널 표시

### 핵심 기능

| 기능 | 설명 |
|------|------|
| 에디터에서 Claude 열기 | `Cmd+Esc` (Mac) / `Ctrl+Esc` (Win/Linux) |
| 네이티브 diff 뷰어 | JetBrains 내장 diff UI로 변경 사항 표시 |
| 현재 선택/탭 자동 공유 | 에디터에서 선택한 코드를 Claude 컨텍스트에 자동 포함 |
| 파일 참조 삽입 | `Cmd+Option+K` (Mac) / `Alt+Ctrl+K` (Win/Linux) |
| IDE 진단 자동 공유 | 에러, 경고, 린트 결과를 Claude에 자동으로 전달 |

### 단축키

| 단축키 (Mac / Windows·Linux) | 동작 |
|------------------------------|------|
| `Cmd+Esc` / `Ctrl+Esc` | 에디터에서 Claude 패널 열기/전환 |
| `Cmd+Option+K` / `Alt+Ctrl+K` | 파일/심볼 참조 삽입 |

### 원격 개발 주의사항

- 원격 개발(SSH, Dev Container) 환경에서는 플러그인을 **원격 호스트**에 설치
- 로컬 IDE에만 설치하면 Claude가 원격 파일시스템에 접근 불가
- JetBrains Gateway 사용 시: Remote IDE 플러그인 목록에서 설치

---

## 터미널 vs IDE 비교

| 항목 | 터미널 | IDE |
|------|--------|-----|
| 속도 | 빠름 (경량, 추가 프로세스 없음) | 약간 느림 (확장 오버헤드) |
| Diff 리뷰 | 텍스트 기반 (유니코드 diff) | 시각적 (사이드 바이 사이드) |
| 파일 참조 | 경로 직접 입력 | `@`-mention, 드래그 앤 드롭 |
| 진단 정보 | 수동 붙여넣기 필요 | 에러/린트 자동 공유 |
| 적합 용도 | 빠른 작업, 스크립팅, CI/CD | 코드 리뷰, 대규모 변경, 리팩터링 |
| 체크포인트 | 미지원 | 지원 (VS Code) |

---

## 키보드 단축키 커스터마이징

### 파일 위치

```
~/.claude/keybindings.json
```

### 편집 방법

```bash
# Claude 세션 내에서
/keybindings
```

### 형식

```json
[
  {
    "key": "ctrl+shift+c",
    "command": "claude.newConversation"
  },
  {
    "key": "ctrl+k ctrl+s",
    "command": "claude.focusPanel"
  }
]
```

### 수정자 키 (modifier keys)

| 키 이름 | 설명 |
|---------|------|
| `ctrl` | Control |
| `alt` | Option (Mac) / Alt (Win/Linux) |
| `shift` | Shift |
| `meta` | Cmd (Mac) / Win (Windows) |

### 코드 시퀀스 (chord binding)

- 공백으로 구분해 두 단계 단축키 설정 가능
- 예: `"ctrl+k ctrl+s"` — `Ctrl+K` 누른 후 `Ctrl+S`

---

## 관련 파일 위치

```
~/.claude/
  keybindings.json      # 단축키 커스터마이징

<project>/
  .claude/
    settings.json       # IDE 관련 프로젝트 설정
```
