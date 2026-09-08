# Free Claude Code (FCC) 분석 정리 노트

> 카리나와 함께 정리한 FCC 완전 정복 가이드 ✨
> 작성일: 2026-09-08

## 📎 관련 저장소

| 구분 | 주소 |
| --- | --- |
| **내 포크 (이 저장소)** | https://github.com/bmshin94/free-claude-code |
| **원본 저장소** | https://github.com/Alishahryar1/free-claude-code |
| 이슈 / 기능 요청 | https://github.com/Alishahryar1/free-claude-code/issues |

> ⚠️ README의 설치 스크립트는 전부 **원본 저장소**를 가리킵니다.
> 포크 기준으로 설치하려면 URL의 `Alishahryar1`을 `bmshin94`로 바꿔야 합니다.

---

## 1. 이게 뭐하는 물건이야?

### 한 줄 요약

> **Claude Code는 그대로 쓰면서, 안에 들어가는 AI 두뇌만 공짜로 바꿔치기하는 로컬 프록시 서버**

`pyproject.toml`의 공식 설명:
> *"Local proxy connecting coding agents to OpenAI-compatible AI providers"*

### 동작 흐름

```
[Claude Code CLI]  →  [FCC 로컬서버 :8082]  →  [NVIDIA NIM / OpenRouter / Gemini / Ollama ...]
   Anthropic 형식        형식 번역 + 라우팅            실제로 답변을 만드는 모델
      요청 전송        (스트리밍/툴콜/이미지 유지)
```

Claude Code는 `ANTHROPIC_BASE_URL`만 바꾸면 "여기가 Anthropic이구나" 하고 인식합니다.
FCC는 그 사이에서 **Anthropic Messages API ↔ OpenAI Chat/Responses API 형식을 실시간 번역**합니다.

### 쉬운 비유

닌텐도 스위치(= Claude Code)에 정품 게임팩(= Anthropic Claude)만 꽂히는데,
FCC는 **다른 게임팩(= 무료 AI)도 꽂히게 해주는 어댑터**입니다.
본체와 조작법은 그대로, 게임팩만 바뀝니다.

---

## 2. 프로젝트 기본 정보

| 항목 | 값 |
| --- | --- |
| 버전 | 6.1.3 |
| 언어 | Python 3.14+ |
| 웹 프레임워크 | FastAPI + Uvicorn |
| 패키지 매니저 | uv (>= 0.11.16) |
| 라이선스 | MIT |
| 전체 파일 수 | 약 708개 |
| 테스트 파일 수 | 280개 |
| 기본 포트 | 8082 |
| 지원 제공자 | 약 50개 |
| 지원 코딩 에이전트 | 10개 |

---

## 3. 폴더 구조 분석

| 폴더 | 역할 | 설명 |
| --- | --- | --- |
| `src/free_claude_code/core/anthropic/` | 🧠 **핵심 번역 엔진** | `conversion.py`, `native_stream.py`, `thinking.py`, `tool_schema.py` — Anthropic 포맷을 재현하는 심장부. thinking(사고과정)까지 그대로 유지 |
| `src/free_claude_code/providers/` | 🔌 **50개 제공자 어댑터** | `nvidia_nim/`, `gemini/`, `groq/`, `github_copilot/`, `ollama`, `lmstudio/`, `vertex/` 등 각각 폴더로 분리 |
| `src/free_claude_code/api/` | 🖥️ **FastAPI 서버 + Admin UI** | `admin_routes.py`, `admin_static/` — 브라우저 관리 화면 |
| `src/free_claude_code/cli/launchers/` | ▶️ **실행 런처** | `fcc-claude`, `fcc-codex` 등. 환경변수를 세팅해 에이전트를 대신 실행 |
| `src/free_claude_code/messaging/` | 📱 **디스코드 / 텔레그램 봇** | `voice.py`, `transcription.py` — 음성 노트로 코딩 지시 |
| `src/free_claude_code/runtime/` | ⚙️ **서버 부팅 / 세션 관리** | `code_sessions_sqlite.py`, 트레이 아이콘 등 |
| `providers/failure_policy.py`<br>`providers/stream_recovery.py` | 🔄 **자동 폴백** | 한 모델이 실패하면 다음 모델로 자동 전환 |
| `tests/` | ✅ **테스트** | 280개 파일. 관리 품질이 좋다는 지표 |
| `.env.example` | 📋 **설정 카탈로그** | 46개 설정 + 제공자별 키 목록 (15KB) |

---

## 4. 킬러 기능

1. **50개 제공자 · 월 13억+ 무료 토큰** — NVIDIA NIM, OpenRouter, Groq, Gemini, Cerebras 등 무료 티어 총동원
2. **10개 코딩 에이전트 공용** — Claude Code, Codex, Pi, OpenCode, Cline, Hermes, DeepSeek Harness, Grok Build, Muse Code, Aider
3. **모델 티어별 라우팅** — `MODEL_OPUS`는 고성능 모델, `MODEL_HAIKU`는 로컬 모델처럼 분리 배정
4. **터미널 출력 토큰 최대 90% 절감** — RTK + FCC 내부 최적화 5종
   (`FAST_PREFIX_DETECTION`, `ENABLE_TITLE_GENERATION_SKIP`, `ENABLE_SUGGESTION_MODE_SKIP`,
   `ENABLE_FILEPATH_EXTRACTION_MOCK`, `ENABLE_NETWORK_PROBE_MOCK`)
5. **로컬 모델 지원** — Ollama, LM Studio, llama.cpp 연결 시 코드가 외부로 나가지 않음
6. **자동 폴백** — 재시도 소진 후 다음 모델로 자동 전환 (턴 재시작 불필요)

---

## 5. 언제 쓰면 좋은가

### 추천 ✅
- Claude 사용량 한도를 자주 초과할 때
- 구독료를 아끼고 싶을 때
- 회사 코드를 외부로 보낼 수 없어 로컬 모델이 필요할 때
- GitHub Copilot / ChatGPT 구독을 이미 갖고 있어 재활용하고 싶을 때
- 여러 모델 성능을 같은 워크플로우에서 비교하고 싶을 때
- 폰(디스코드/텔레그램)으로 원격 코딩 지시를 하고 싶을 때

### 비추천 ❌
- 성능이 최우선이고 이미 Claude를 잘 쓰고 있을 때
- 초기 설정을 만지는 것이 부담스러울 때

---

## 6. 설치 가이드

### 사전 준비

설치 스크립트가 **uv와 Python 3.14.0을 자동으로 설치**하므로 별도 준비물은 사실상 없습니다.

### 1단계: 설치

**macOS / Linux**
```bash
curl -fsSL "https://raw.githubusercontent.com/Alishahryar1/free-claude-code/main/scripts/install.sh" | sh
```

**Windows (PowerShell)**
```powershell
& ([scriptblock]::Create((irm "https://raw.githubusercontent.com/Alishahryar1/free-claude-code/main/scripts/install.ps1")))
```

설치 중 에이전트를 하나씩 Y/N으로 물어봅니다. 잘 모르겠으면 **Claude Code만 Y**, 나머지는 N이면 충분합니다.

```
Install or verify Claude Code for fcc-claude?     [Y/n]  ← Y (필수)
Install or verify Codex for fcc-codex?            [Y/n]  ← n
... (Pi, OpenCode, Cline, Hermes, DSH, Grok, Muse, Aider)
Enable RTK token optimization globally?           [y/N]  ← y 권장
```

**설치 스크립트 옵션**

| 옵션 | 설명 |
| --- | --- |
| `--rtk` | RTK 토큰 최적화 설치 및 설정 |
| `--voice-nim` | NVIDIA NIM 음성 인식 |
| `--voice-local` | 로컬 Whisper 음성 인식 |
| `--voice-all` | 음성 백엔드 전부 |
| `--torch-backend cu130` | CUDA 13.0용 PyTorch |
| `--dry-run` | 실행 없이 명령만 출력 |

예시:
```bash
curl -fsSL "https://raw.githubusercontent.com/Alishahryar1/free-claude-code/main/scripts/install.sh" | sh -s -- --rtk
```

### 2단계: 무료 API 키 발급 (NVIDIA NIM 추천)

1. https://build.nvidia.com/settings/api-keys 접속
2. NVIDIA 계정 로그인 (무료 가입, 카드 등록 불필요)
3. **Generate API Key** 클릭
4. `nvapi-...` 로 시작하는 키 복사

> 🔐 이 키는 비밀번호와 같습니다. 공개 저장소에 커밋하지 마세요.

### 3단계: 서버 실행

- **Windows / macOS**: 바탕화면 또는 응용 프로그램의 **Free Claude Code** 아이콘 실행 → 트레이 아이콘 생성
- **Linux**: `fcc-server` (이 터미널 창은 계속 열어둬야 함)

Admin UI 주소:
```
http://127.0.0.1:8082/admin
```

### 4단계: Admin UI 설정

1. `NVIDIA_NIM_API_KEY` 칸에 키 붙여넣기
2. `MODEL`은 기본값 유지 (`nvidia_nim/nvidia/nemotron-3-super-120b-a12b`)
3. **Apply** 버튼 클릭

### 5단계: 실행

```bash
fcc-claude      # Claude Code
fcc-codex       # Codex
fcc-cline       # Cline
fcc-aider       # Aider
fcc-opencode    # OpenCode
fcc-pi          # Pi
fcc-hermes      # Hermes
fcc-dsh         # DeepSeek Harness
fcc-grok        # Grok Build
fcc-muse        # Muse Code
```

Claude Code 안에서 `/model`을 입력해 FCC 모델 목록이 뜨면 연결 성공입니다.

### 런처가 자동으로 설정하는 환경변수

`src/free_claude_code/cli/claude_env.py` 기준:

```
ANTHROPIC_BASE_URL                        = http://127.0.0.1:8082
ANTHROPIC_AUTH_TOKEN                      = freecc
CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY = 1
CLAUDE_CODE_AUTO_COMPACT_WINDOW           = 190000
DISABLE_AUTOUPDATER                       = 1
DISABLE_FEEDBACK_COMMAND                  = 1
DISABLE_ERROR_REPORTING                   = 1
```

---

## 7. 고급 설정

### 폴백 모델 (가장 유용)

**Admin UI → Model Config → Fallback Models**

순서대로 등록하면 앞의 모델이 실패했을 때 자동으로 다음 모델로 넘어갑니다.

```
1순위: nvidia_nim/nvidia/nemotron-3-super-120b-a12b
2순위: open_router/openrouter/free
3순위: groq/llama-3.3-70b-versatile
```

> ⚠️ 실패한 요청이 여러 제공자의 사용량을 동시에 소모할 수 있습니다.

### 모델 등급별 라우팅

| 설정 | 용도 |
| --- | --- |
| `MODEL` | 전체 기본값 |
| `MODEL_FABLE` | Fable 티어 |
| `MODEL_OPUS` | 고난이도 작업 → 가장 똑똑한 모델 |
| `MODEL_SONNET` | 일반 작업 → 중간 모델 |
| `MODEL_HAIKU` | 잡일 → 가볍고 빠른 모델 |

### 추론(Reasoning) 강도

**Admin UI → Model Config → Reasoning**

| 선택 | 동작 |
| --- | --- |
| From client (기본) | 클라이언트가 보낸 effort 사용 |
| Off | 추론 비활성화 요청 |
| Low / Medium / High / X-High / Max | 클라이언트 설정 무시하고 고정 |
| Inherit | 루트 Reasoning 설정 상속 (티어별 설정에만 존재) |

### 프록시 보안

`PROXY_AUTH_ENABLED=true`로 설정하면 토큰 없이는 접근 불가. 기본 토큰은 `freecc`.

---

## 8. 자주 발생하는 문제

### Claude Code가 계속 로그인을 요구할 때

설정 파일을 열어(없으면 생성) 아래 항목을 병합합니다. 기존 내용은 지우지 마세요.

- Windows: `%USERPROFILE%\.claude.json`
- macOS / Linux / WSL: `~/.claude.json`

```json
{
  "hasCompletedOnboarding": true
}
```

저장 후 Claude Code 또는 IDE 재시작.

### 연결이 안 될 때

1. `fcc-server` 프로세스가 살아있는지 확인
2. `http://127.0.0.1:8082/admin` 접속 확인
3. Admin UI에서 **Apply**를 눌렀는지 확인

### 답변이 이상하거나 도구를 못 쓸 때

해당 모델이 **툴 호출(tool calling)** 을 지원하지 않을 수 있습니다.
코딩 에이전트용으로는 툴 지원 모델이 필수이며, 로컬 모델은 시스템 프롬프트와 툴 정의를
담을 만큼 **충분한 컨텍스트 길이**도 필요합니다.

---

## 9. 주요 파일 위치

| 대상 | 경로 |
| --- | --- |
| 설정 파일 | `~/.fcc/.env` |
| 로그 파일 | `~/.fcc/logs/server.log` |
| Codex 모델 카탈로그 | `~/.fcc/codex-model-catalog.json` |
| 코드 세션 DB | `~/.fcc/code/code.db` |
| 인증 정보 | `~/.fcc/auth/` |
| Admin UI | `http://127.0.0.1:8082/admin` |
| 버전 확인 | `fcc-server --version` |

---

## 10. 업데이트 / 삭제

**업데이트**: 설치 명령어를 그대로 다시 실행

**삭제** (실행 중인 FCC 프로세스를 모두 종료한 뒤)

```bash
# macOS / Linux
curl -fsSL "https://raw.githubusercontent.com/Alishahryar1/free-claude-code/main/scripts/uninstall.sh" | sh
```
```powershell
# Windows
& ([scriptblock]::Create((irm "https://raw.githubusercontent.com/Alishahryar1/free-claude-code/main/scripts/uninstall.ps1")))
```

- **삭제됨**: FCC 본체, 데스크톱 런처, `~/.fcc/`
- **유지됨**: uv, Python, Claude Code, Codex, 기타 에이전트, RTK, 공유 PATH

---

## 11. React / PHP로 만들 수 있을까?

### 결론 요약

| 기술 | 가능 여부 | 한 줄 평 |
| --- | --- | --- |
| **React 단독** | ❌ 불가능 | 브라우저 안에서만 도는 UI 라이브러리 |
| **Node.js + React** | ✅ 가능 (추천) | Node가 서버, React가 Admin UI |
| **PHP** | ⚠️ 이론상 가능, 실무상 비추천 | 스트리밍/동시성/배포에서 전부 불리 |

### React가 단독으로 안 되는 이유

FCC는 UI가 아니라 **서버**입니다. 필요한 일들이 브라우저에서 불가능합니다.

| FCC가 하는 일 | 브라우저 React |
| --- | --- |
| 8082 포트를 열고 요청 수신 | ❌ 포트 개방 불가 |
| 파일 읽기/쓰기 (`~/.fcc/.env`) | ❌ 파일시스템 접근 불가 |
| 프로세스 실행 (`fcc-claude` 런처) | ❌ 불가능 |
| API 키를 안전하게 보관 | ❌ 브라우저에 두면 그대로 노출 |
| SQLite 세션 DB | ❌ 불가능 |
| 외부 API 직접 호출 | ❌ CORS에 막힘 |

> 참고: 현재 FCC의 Admin UI는 React가 아니라 **순수 바닐라 JS**입니다.
> (`src/free_claude_code/api/admin_static/` — `admin.js`, `model_combobox.js`, `session_ui.js`)
> 따라서 **Admin UI만 React로 갈아끼우는 작업은 충분히 현실적입니다.**

### Node.js로 다시 만든다면 (가장 현실적인 대안)

| FCC (Python) | Node.js 대응 |
| --- | --- |
| FastAPI + Uvicorn | Hono / Fastify / Express |
| Pydantic | Zod |
| httpx | undici / fetch |
| asyncio 스트리밍 | Web Streams API |
| Admin UI (바닐라 JS) | React + Vite |
| uv | pnpm |
| pytest | Vitest |

**장점**
- Claude Code, Codex 등 대상 에이전트 대부분이 이미 Node 생태계
- `npx`, `npm install -g`로 배포가 훨씬 간단
- SSE 스트리밍 처리가 자연스러움
- Admin UI와 서버가 같은 언어 → 타입 공유 가능

### PHP가 비추천인 이유

기술적으로 **불가능하지는 않습니다.** 다만 구조적으로 불리합니다.

| 문제 | 설명 |
| --- | --- |
| **스트리밍(SSE)** | AI 응답은 토큰 단위 실시간 스트리밍이 핵심. PHP는 output buffering을 강제로 꺼야 하고 장시간 연결 유지가 까다로움 |
| **동시성 모델** | PHP-FPM은 요청 1개당 프로세스 1개. 수 분씩 열려 있는 스트리밍 연결이 워커를 계속 점유 → 금방 고갈 |
| **상주 프로세스** | FCC는 계속 떠 있는 로컬 데몬. PHP는 전통적으로 "요청 오면 실행 후 종료" 모델 |
| **CLI 배포** | `fcc-claude` 같은 런처, 트레이 아이콘, 데스크톱 앱 배포가 매우 번거로움 |
| **생태계** | AI 관련 SDK가 Python / TypeScript 중심. PHP는 커뮤니티 라이브러리 의존 |

Swoole, ReactPHP, Amp 같은 async 런타임을 쓰면 스트리밍과 동시성 문제는 해결되지만,
그 시점엔 **"굳이 PHP를 고른 이유"** 가 사라집니다.

### 추천 로드맵

```
난이도 하 ─ Admin UI만 React로 리뉴얼 (기존 Python 서버 유지)
              ↓
난이도 중 ─ Node.js로 프록시 코어만 이식 (제공자 2~3개부터)
              ↓
난이도 상 ─ 전체 재구현 (제공자 50개 + 폴백 + 메시징까지)
```

> 💡 실용적 조언: **처음부터 다시 만들기보다 기존 프로젝트에 기여하는 편이 압도적으로 효율적**입니다.
> 특히 `src/free_claude_code/providers/` 는 폴더 하나 추가하는 구조라 진입 장벽이 낮습니다.

---

## 12. 수익화 아이디어

### ⛔ 먼저, 하면 안 되는 것

| 금지 | 이유 |
| --- | --- |
| **무료 티어 재판매** (내 키로 프록시 열고 구독료 받기) | NVIDIA, Groq, Google 등 대부분 제공자의 약관 위반. 계정 정지 + 법적 리스크 |
| **`Claude` / `Claude Code` 브랜딩** | Anthropic 상표. README에도 "Anthropic과 무관"이라 명시되어 있음 |
| **계정 공유 / 구독 대행** | Kimi Code, QwenCloud 등은 "개인의 대화형 사용"으로 명시 제한 |

> 핵심 원칙: **"무료 토큰 자체"는 팔 수 없습니다. 팔 수 있는 것은 그 주변의 지식·시간·신뢰입니다.**

### ✅ 현실적인 아이디어 5가지

#### 1. 한국어 콘텐츠 (리스크 0, 즉시 시작 가능)

영어 자료는 많지만 한국어 자료는 거의 없는 상태.

- 유튜브: "Claude Code 요금 90% 아끼는 법"
- 블로그: 제공자별 실사용 후기 + 벤치마크
- 인프런 / 클래스101 강의
- 유료 뉴스레터: 주간 무료 모델 순위

| 난이도 | ⭐☆☆☆☆ | 수익성 | ⭐⭐⭐☆☆ | 리스크 | 없음 |
| --- | --- | --- | --- | --- | --- |

#### 2. 기업 대상 구축 컨설팅 (가장 큰 수익)

금융, 방산, 공공, 의료 등 **"코드가 외부로 나가면 안 되는"** 기업이 많습니다.
FCC + Ollama / LM Studio 조합이면 완전 폐쇄망 코딩 에이전트 환경 구축이 가능합니다.

- 온프레미스 AI 코딩 환경 구축
- 사내 GPU 서버 모델 세팅 + 팀 온보딩 교육
- 월 유지보수 계약

| 난이도 | ⭐⭐⭐⭐☆ | 수익성 | ⭐⭐⭐⭐⭐ | 리스크 | 낮음 (로컬 모델은 약관 이슈 없음) |
| --- | --- | --- | --- | --- | --- |

#### 3. 팀용 SaaS 확장 (BYOK 방식)

FCC는 개인용 1인 프록시입니다. 기업에는 아래가 추가로 필요합니다.

- 팀원별 사용량 대시보드
- 부서별 예산 한도
- 비용 리포트 / 감사 로그

**API 키는 고객이 직접 입력(BYOK)** 하게 하면 "도구"를 파는 것이므로 약관 문제를 피할 수 있습니다.

| 난이도 | ⭐⭐⭐⭐⭐ | 수익성 | ⭐⭐⭐⭐☆ | 리스크 | 경쟁 심함 (LiteLLM, Portkey, OpenRouter) |
| --- | --- | --- | --- | --- | --- |

#### 4. 한국 특화 프로바이더 추가

현재 50개 제공자 중 **한국 서비스는 0개**입니다.

- 네이버 하이퍼클로바X
- 업스테이지 Solar
- 카카오 Kanana

`src/free_claude_code/providers/` 에 폴더를 추가하는 구조라 기여 난이도가 낮고,
원본 저장소에 PR이 머지되면 공개 포트폴리오가 됩니다.
공공기관·국내 기업 대상 컨설팅(2번)의 강력한 차별점이 됩니다.

| 난이도 | ⭐⭐⭐☆☆ | 수익성 | 간접적 | 리스크 | 없음 |
| --- | --- | --- | --- | --- | --- |

#### 5. 벤치마크 큐레이션

무료 모델은 성능이 자주 바뀌고 갑자기 사라지기도 합니다.

- 무료 코딩 모델 실시간 순위표 사이트
- 유료 비교 리포트
- 합법적 제휴(어필리에이트) 링크

| 난이도 | ⭐⭐☆☆☆ | 수익성 | ⭐⭐☆☆☆ | 리스크 | 없음 |
| --- | --- | --- | --- | --- | --- |

### 추천 실행 순서

```
[1개월]   4번 → 한국 프로바이더 PR 올리기        (실력 + 포트폴리오)
[2~3개월] 1번 → 그 과정을 블로그/영상으로 기록     (신뢰 + 인바운드 문의)
[4개월~]  2번 → 문의 온 기업에 구축 컨설팅         (실매출)
[6개월~]  3번 → 반복되는 요구사항을 SaaS로 제품화   (스케일)
```

앞 단계가 뒷 단계의 영업 도구가 되는 구조라 어느 것도 버려지지 않습니다.

### 법적 체크리스트

- FCC는 **MIT 라이선스** — 상업적 이용 가능, 단 **저작권 표시 유지 의무**
- 제품명에 `Claude` 사용 금지 (상표)
- 각 제공자 약관의 "재판매 금지", "개인 사용 한정" 조항 직접 확인
- 실제 사업화 시 변호사 상담 권장 (본 문서는 법률 자문이 아님)

---

## 13. 최종 요약

| 질문 | 답 |
| --- | --- |
| **이게 뭐야?** | Claude Code의 AI 두뇌를 무료 모델로 바꿔주는 로컬 프록시 |
| **언제 써?** | 요금 아낄 때, 사용량 한도 찼을 때, 회사 코드 보안이 필요할 때 |
| **뭐가 좋아?** | 무료 + 자동 폴백 + 로컬 모델 + 폰 원격 조작 |
| **단점은?** | 무료 모델은 진짜 Claude보다 성능이 아쉬움 |
| **React/PHP로?** | React 단독 ❌ / Node.js+React ✅ / PHP ⚠️ 비추천 |
| **돈은 어떻게?** | 토큰을 팔지 말고, **지식과 구축 서비스**를 팔 것 |

### 5줄 요약 설치법

```bash
1. 설치 명령어 실행       → Claude Code Y, RTK y
2. build.nvidia.com 에서 무료 키 발급
3. 서버 실행              → fcc-server (또는 앱 아이콘)
4. Admin UI에 키 입력      → Apply
5. fcc-claude 실행        → /model 로 확인
```

---

*이 문서는 https://github.com/bmshin94/free-claude-code 저장소를 직접 분석하여 작성되었습니다.*
