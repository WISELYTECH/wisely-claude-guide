# Claude Code 시작하기

> 설치부터 첫 작업까지 — 5분 안에 준비하세요.

## Claude Code란?

**Claude Code**는 프로젝트 파일에 직접 접근하고 명령어를 실행할 수 있는 AI 에이전트입니다. 코드 작성뿐 아니라 데이터 분석, 문서 생성, 프로젝트 관리 등 다양한 업무를 자동화할 수 있습니다.

### Claude 제품군 비교

| | Claude Desktop | Cowork | Claude Code |
|---|---|---|---|
| 접근 방법 | 데스크톱 앱 대화 | Claude Desktop 내 에이전트 | 터미널, IDE, 데스크톱 앱 |
| 결과물 | 텍스트 답변 | 파일(엑셀, PPT, 문서) | 코드 수정, 파일 조작, 명령어 실행 |
| 외부 도구 연결 | Connector | Connector | MCP 서버 |
| 적합한 용도 | 질문, 문서 작성 | 리포트, 프레젠테이션, 파일 정리 | 코드 수정, 로컬 파일 조작, 자료 검색, 자동화, 배포 |

**언제 어떤 걸 쓸까?**
- 빠른 질문/답변 → **Claude Desktop**
- 결과물이 파일로 필요 → **Cowork** (엑셀, PPT, 정리된 문서)
- 로컬 환경에서 직접 작업 → **Claude Code** (코드 수정, 파일 정리, 검색, 자동화 등)

---

## 설치

### macOS

**방법 1: 설치 스크립트** (권장)
```bash
curl -fsSL https://claude.ai/install.sh | bash
```

**방법 2: Homebrew**
```bash
brew install --cask claude-code
```

### Windows

PowerShell을 관리자 권한으로 열고:
```powershell
irm https://claude.ai/install.ps1 | iex
```

### Linux

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

설치 후 터미널에서 `claude --version`으로 확인하세요.

---

## 인증

설치 후 처음 실행할 때:

```bash
claude
```

프롬프트가 나타나면 `/login` 입력:
```
> /login
```

브라우저가 열리고 Anthropic 계정 로그인 화면이 나옵니다. 다음 중 하나의 계정이 필요합니다:
- **Claude Pro** 또는 **Claude Max** (개인)
- **Claude Teams** (팀 관리자 통해 초대)
- **Claude Enterprise** (회사)

로그인 후 인증 코드가 터미널에 표시되고 자격 증명은 **로컬에 안전하게 저장**됩니다.

---

## 첫 실행

### 대화형 모드 (권장)

```bash
cd /path/to/your/project
claude
```

이제 Claude와 대화하며 작업할 수 있습니다:
```
> 이 프로젝트가 뭐 하는 건지 설명해줄래?
> README를 한국어로 번역해줘
> 버그 수정 좀 도와줘
```

### 한 번 실행 후 종료

초기 프롬프트와 함께 시작해서 한 번 실행 후 종료:
```bash
claude "README를 한국어로 번역해"
```

또는 플래그 사용:
```bash
claude -p "README를 한국어로 번역해"
```

### 이전 대화 이어가기

```bash
claude -c              # 가장 최근 대화 재개
claude -r session-id   # 특정 세션 재개
```

---

## 핵심 슬래시 커맨드

Claude Code 내에서 `/` 로 시작하는 커맨드를 사용합니다.

| 커맨드 | 용도 |
|--------|------|
| `/help` | 모든 커맨드 도움말 |
| `/clear` | 대화 초기화 |
| `/compact` | 긴 대화 압축 (토큰 절약) |
| `/context` | 현재 컨텍스트 사용량 시각화 |
| `/cost` | 이번 세션 토큰 비용 확인 |
| `/memory` | CLAUDE.md 및 자동 메모리 열기 |
| `/model` | 모델 변경 (claude-opus, claude-sonnet 등) |
| `/plan` | 계획 모드 진입 (큰 작업용) |
| `/init` | CLAUDE.md 자동 생성 |
| `/diff` | 미커밋 변경사항 확인 |
| `/rewind` | 특정 체크포인트로 되돌리기 |

**자주 쓰는 조합:**
- 큰 리팩터링: `/plan` → 계획 세우기 → 실행
- 토큰 절약: `/compact` → 장시간 세션 계속
- 프로젝트 처음: `/init` → CLAUDE.md 자동 생성

---

## 권장 첫 워크플로우

처음 시작하는 엔지니어라면 이 순서로 진행하세요:

### 1단계: 설치 및 로그인
```bash
curl -fsSL https://claude.ai/install.sh | bash
claude
> /login
```

### 2단계: 프로젝트 디렉토리로 이동
```bash
cd your-project
claude
```

### 3단계: CLAUDE.md 생성
```
> /init
```

프로젝트의 구조, 기술 스택, 컨벤션을 자동으로 분석해서 CLAUDE.md를 생성합니다.

### 4단계: 프로젝트 이해하기
```
> 이 프로젝트의 목적이 뭐야?
> 주요 디렉토리 구조 설명해줄래?
```

### 5단계: 간단한 작업 시도
```
> README.md를 한국어로 번역해줄래?
> 타이포 있는 곳 찾아서 고쳐줄래?
> 이 함수의 테스트 코드 작성해줄래?
```

성공하면 자신감 생기고, `/diff`로 변경사항 확인한 후 git에 커밋하면 됩니다.

---

## 비엔지니어도 사용할 수 있을까?

**네.** Claude Code 데스크톱 앱을 설치하면 터미널 없이 GUI로 접근 가능합니다. 프로젝트 파일에 직접 접근하고, 코드 분석·데이터 정리·문서 작성 등을 자연어로 요청할 수 있습니다.

일반 대화나 파일 업로드 기반의 가벼운 질문은 **Claude Desktop**으로도 충분합니다.

**상세 활용법은** [활용 가이드](10-non-engineer-guide.md)를 참고하세요.

---

## 다음 단계

이제 Claude Code의 기본을 알았습니다. 다음은 상황에 따라:

### 추천 다음 문서
- **Claude 활용 가이드**: [활용 가이드](10-non-engineer-guide.md) → Cowork, Connector, Skills, CLAUDE.md 등 핵심 기능
- **컨텍스트 관리**: [컨텍스트 관리 가이드](07-context-management.md) → 사용량 확인, 5시간/주간 한도, 토큰 절약법

### 기능별 상세 레퍼런스
- [CLAUDE.md 작성법](../reference/02-claude-md.md) — 프로젝트 설정 파일의 모든 것
- [Skills](../reference/03-skills.md) — 커스텀 명령어 만들기
- [Hooks](../reference/04-hooks.md) — 자동화 규칙 설정
- [Settings & Permissions](../reference/05-settings-permissions.md) — 권한 및 설정
- [MCP Servers](../reference/06-mcp-servers.md) — 외부 도구 연결
- [IDE 통합](../reference/08-ide-integration.md) — VS Code, JetBrains
- [Agent Teams](../reference/09-agent-teams.md) — 멀티 에이전트

### 지금 해볼 만한 것
- `/help`로 모든 커맨드 목록 보기
- `/cost`로 토큰 사용량 모니터링
- `/memory`로 CLAUDE.md 내용 확인 (프로젝트마다 다름)

---

## 자주 묻는 질문

**Q: Claude Code는 무료인가요?**  
A: Claude Pro/Max 구독이 필요합니다. 회사 계정(Teams/Enterprise)을 사용하면 개인 비용 없이 가능합니다.

**Q: 내 코드가 Anthropic 서버로 올라가나요?**  
A: 네, API 호출 시 프롬프트와 코드가 Anthropic 서버로 전송됩니다. 회사 정책을 확인하세요.

**Q: 오프라인에서도 쓸 수 있나요?**  
A: 아니요. API 호출이 필요하므로 인터넷 연결이 필수입니다.

**Q: 터미널이 너무 어려워요. GUI 버전은?**  
A: Homebrew로 설치하면 데스크톱 앱으로도 실행됩니다. 또는 Claude Desktop 앱을 사용하세요.

**Q: 세션이 자동 저장되나요?**  
A: 네. 대화 기록은 자동 저장되고, `/rewind`로 특정 지점으로 돌아갈 수 있습니다.

---

## 도움말

- 커맨드 도움말: Claude Code에서 `/help` 입력
- 기술 심화: Claude Code에서 `/claude-guide` 설치 후 질의
- 문제 신고: 이 리포지토리 이슈 등록

**마지막 업데이트**: 2026-04-02
