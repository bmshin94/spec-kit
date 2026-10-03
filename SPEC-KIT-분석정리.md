# Spec Kit 전수조사 분석 및 활용 정리

> 작성일: 2026-10-03
> 분석 대상 저장소: <https://github.com/bmshin94/spec-kit>
> 원본(업스트림): <https://github.com/github/spec-kit>
> 공식 문서: <https://github.github.io/spec-kit/>
> 패키지: <https://pypi.org/project/specify-cli/>

---

## 목차

1. [Spec Kit이란](#1-spec-kit이란)
2. [저장소 전수조사 결과](#2-저장소-전수조사-결과)
3. [3가지 독립 프로세스](#3-3가지-독립-프로세스)
4. [쉽게 이해하기](#4-쉽게-이해하기)
5. [설치 및 사용법](#5-설치-및-사용법)
6. [플러그인 / 스킬 / MCP 구분](#6-플러그인--스킬--mcp-구분)
7. [API 토큰 필요 여부](#7-api-토큰-필요-여부)
8. [AI 에이전트 구축에 주는 도움](#8-ai-에이전트-구축에-주는-도움)
9. [React / PHP 재구현 가능성](#9-react--php-재구현-가능성)
10. [유튜브 강의 제작 가능성](#10-유튜브-강의-제작-가능성)
11. [수익화 아이디어 상세](#11-수익화-아이디어-상세)
12. [실행 로드맵 및 리스크](#12-실행-로드맵-및-리스크)

---

## 1. Spec Kit이란

**GitHub가 공개한 "AI 코딩 에이전트용 작업 지시서 자동 생성 시스템"입니다.**

코드를 대신 작성해 주는 도구가 **아니라**, AI에게 코드를 시키기 전에
`기획서 → 설계도 → 작업목록`을 체계적으로 만들게 강제하는 **프로세스 프레임워크**입니다.

핵심 철학: **"무엇(What)과 왜(Why)를 먼저 확정한 뒤에 어떻게(How)를 정한다."**

### 기본 정보

| 항목 | 내용 |
| --- | --- |
| 원본 저장소 | `github/spec-kit` (GitHub 공식) |
| 현재 저장소 | `bmshin94/spec-kit` (포크본) |
| 패키지명 | `specify-cli` (v1.0.9.dev0) |
| 구현 언어 | Python 3.11+ |
| 소스 규모 | `.py` 325개 / `.md` 151개 / `.yml` 47개 |
| 전체 용량 | 약 14MB (CHANGELOG 2,927줄 → 매우 활발한 개발) |
| 라이선스 | MIT (상업적 이용 · 재배포 · 재구현 모두 자유) |
| 주요 의존성 | typer, click, rich, platformdirs, readchar, pyyaml, packaging, pathspec, json5 |

---

## 2. 저장소 전수조사 결과

### 디렉터리 구조

```
spec-kit/
├── src/specify_cli/          # 실제 프로그램 (Python Typer CLI)
│   ├── integrations/         # 41개 AI 에이전트 연동 모듈  ★핵심
│   ├── extensions/           # 확장 기능 설치 / 관리
│   ├── presets/              # 템플릿 변형(오버라이드) 관리
│   ├── workflows/            # 자동 실행 엔진 (gate / if-then / while / do-while)
│   ├── authentication/       # GitHub · Azure DevOps 인증
│   ├── bundler/              # 역할별 패키지 묶음
│   ├── artifacts/            # 산출물 조회
│   └── commands/             # init, event, bundle
├── templates/                # ★ 진짜 알맹이 — AI 프롬프트 템플릿
│   ├── spec-template.md         (요구사항 명세 — User Story/우선순위/Given-When-Then)
│   ├── plan-template.md         (기술 설계)
│   ├── tasks-template.md        (작업 분해)
│   ├── constitution-template.md (프로젝트 원칙)
│   ├── checklist-template.md    (품질 게이트)
│   └── commands/                (9개 슬래시 명령 프롬프트)
│       specify / plan / tasks / implement / converge /
│       analyze / clarify / constitution / checklist / taskstoissues
├── extensions/               # bug, assess, git, agent-context, template, selftest
├── presets/                  # lean, constitution-sync, scaffold, self-test
├── workflows/speckit/        # "전체 SDD 사이클" 자동화 정의 (workflow.yml)
├── scripts/                  # bash / powershell / python 3종 동일 기능 제공
├── docs/                     # 41개 공식 문서
├── examples/bundles/         # developer / product-manager / business-analyst /
│                             #   security-researcher 역할별 번들 예시
├── integrations/             # 통합 카탈로그 JSON
├── bundles/                  # 커뮤니티 번들 카탈로그
├── tests/                    # 통합별 테스트 포함 광범위한 테스트 스위트
└── .specify/memory/          # 이 저장소 자신의 "헌법(constitution)"
```

### 핵심 발견

- `templates/commands/*.md` 는 전부 **영어로 작성된 AI 지시문**입니다.
  → 이 프로젝트의 본질은 **"검증된 프롬프트 모음집 + 그것을 배포·관리하는 CLI"** 입니다.
- 스크립트가 bash / PowerShell / Python **3종으로 모두 제공**됩니다 (완전한 크로스플랫폼).
- `pyproject.toml` 의 `force-include` 로 템플릿·확장·프리셋을 패키지에 동봉 →
  **네트워크 없이(air-gapped / 사내망) 동작**합니다.
- `ruff` 린트 설정에 `S602/S604/S605`(shell=True 금지)를 강제 → 보안 자세가 명시적입니다.
- Spec Kit 저장소 스스로가 `.specify/memory/constitution.md` 를 갖고 있습니다 (자기 적용, dogfooding).

### 지원 AI 에이전트 (41종)

claude, copilot, codex, cursor_agent, gemini, qwen, grok, kiro_cli, opencode, zed,
cline, kilocode, junie, auggie, amp, droid, devin, trae, goose, agy, alquimia,
bob, codebuddy, command_code, docker_agent, dsh, firebender, forge, generic,
hermes, kimi, lingma, muse, omp, pi, qodercli, rovodev, shai, tabnine, vibe, zcode

`specify integration switch` 로 **한 번 만든 스펙을 다른 AI로 그대로 이전**할 수 있습니다.

---

## 3. 3가지 독립 프로세스

세 가지는 순차 단계가 아니라 **독립적인 진입점**입니다.

### ① Spec-Driven Development (코어 기본 탑재)

```text
/speckit-constitution   프로젝트 원칙 수립 (프로젝트당 1회)
/speckit-specify        무엇을 / 왜          → specs/NNN-기능명/spec.md
/speckit-plan           어떻게 (기술 설계)   → plan.md
/speckit-tasks          작업 분해            → tasks.md
/speckit-implement      구현
/speckit-converge       스펙 대비 미완성분 검출 → 남은 작업을 tasks.md에 추가
```

`converge` 가 **Converged** 를 보고할 때까지 `implement ↔ converge` 를 반복합니다.
추가 품질 게이트: `/speckit-clarify`(모호성 해소), `/speckit-checklist`, `/speckit-analyze`(정합성 분석).

### ② Bug fixing (옵트인 확장)

```bash
specify extension add bug
```

```text
/speckit-bug-assess "빈 비밀번호로 제출하면 로그인 폼이 깨집니다." slug=login-crash
/speckit-bug-fix    slug=login-crash
/speckit-bug-test   slug=login-crash
```

산출물: `.specify/bugs/<slug>/` · 최종 판정: `verified` / `partial` / `failed`
**검증 없는 수정은 성공으로 치지 않습니다.**

### ③ Idea assessment (옵트인 확장)

```bash
specify extension add assess
```

```text
/speckit-assess-intake "오프라인에서 작업하고 재접속 시 동기화되게 하고 싶다." slug=offline-mode
/speckit-assess-research slug=offline-mode
/speckit-assess-define   slug=offline-mode
/speckit-assess-shape    slug=offline-mode
/speckit-assess-decide   slug=offline-mode
```

산출물: `.specify/assessments/<slug>/` · 최종 결론: `go` / `needs-clarification` / `kill`
**소스 코드가 전혀 없는 프로젝트에서도 동작합니다.** `go` 는 `/speckit-specify` 로 인계됩니다.

---

## 4. 쉽게 이해하기

### 비유 — 인테리어 공사

**Spec Kit 없이 AI 사용** = 인부에게 전화로 "우리집 좀 예쁘게 고쳐줘"
→ 벽을 다 뜯어놓고 "원하시던 게 이거 맞죠?" → 아니요 → 재공사 → 비용 낭비

**Spec Kit 사용** = 공사 전에 다음 순서를 밟음

| 단계 | 공사 비유 | Spec Kit |
| --- | --- | --- |
| 1 | 집안 규칙 (친환경 자재만, 야간공사 금지) | `constitution` |
| 2 | 기획서 (4인 가족 영화 공간, 소파–TV 2.5m 이상) | `specify` |
| 3 | 설계도 (벽지 ○○, 조명 ○○W, 배선 위치) | `plan` |
| 4 | 작업지시서 (1일차 철거 / 2일차 배선 …) | `tasks` |
| 5 | 실제 시공 | `implement` |
| 6 | 준공검사 — 기획서 들고 빠진 것 체크 | `converge` |

### 반드시 알아야 할 3가지

**① 슬래시 명령은 터미널 명령이 아닙니다**

```text
터미널에 입력   →  specify init my-project --integration claude
AI 채팅에 입력  →  /speckit-specify 사진 정리 앱을 만들어줘
```

터미널에 치는 것은 `specify` 로 시작하는 것뿐입니다.
`/speckit-*` 는 **AI 에이전트 채팅창에 치는 말**입니다.

**② 결과물은 전부 평범한 마크다운 파일입니다**

```text
my-project/
├── .specify/memory/constitution.md
└── specs/001-photo-organizer/
    ├── spec.md      무엇을 / 왜
    ├── plan.md      어떻게
    └── tasks.md     할 일 체크리스트
```

특수 포맷이 아니므로 메모장으로도 열립니다.
**Spec Kit을 나중에 버려도 이 문서들은 그대로 남습니다 (락인 없음).**

**③ 왜 "무엇"과 "어떻게"를 분리하는가**

- `spec.md` 에 "React를 쓸 것"이라고 적으면 → Vue로 바꿀 때 기획서부터 다시 써야 함
- `spec.md` 에는 "사용자가 사진을 날짜별 앨범으로 본다"만 적고 → 기술 선택은 `plan.md` 에서 교체

### 가장 흔한 오해

> **"Spec Kit이 코드를 짜주나요?"** → **아니요.**
> 코드는 여전히 Claude / Copilot 같은 AI가 작성합니다.
> Spec Kit은 그 AI에게 **좋은 지시서를 쥐여주는 역할**만 합니다.

### 실질적 이점

1. **AI 헛발질 방지** — 요구사항을 Given/When/Then 수용 기준까지 강제로 작성
2. **컨텍스트 소실 방지** — 대화창을 닫아도 `spec.md` 가 파일로 남음
3. **"다 됐습니다" 거짓말 차단** — `converge` 가 스펙 대비 실제 코드를 대조
4. **비개발자와의 공통 언어** — `spec.md` 는 기획자도 읽을 수 있음
5. **AI 비용 절감** — 재작업 횟수 감소

---

## 5. 설치 및 사용법

### 사전 준비

- Python 3.11 이상
- [uv](https://github.github.io/spec-kit/install/uv.html)
- 지원되는 AI 코딩 에이전트 1개 이상
- Linux / macOS / Windows

### 설치

```bash
# 1) uv 설치 (없는 경우)
curl -LsSf https://astral.sh/uv/install.sh | sh                 # macOS / Linux
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"      # Windows

# 2) Spec Kit 설치
uv tool install specify-cli

# 3) 새 프로젝트 생성
specify init my-project --integration claude
cd my-project

# 4) 환경 점검
specify check
```

- `--integration` 값: `claude`, `copilot`, `codex`, `cursor-agent`, `gemini` 등 41종 중 선택
  (미지정 시 기본값 `copilot`, 환경변수 `SPECKIT_INTEGRATION_DEFAULT` 로 변경 가능)
- **기존 프로젝트에 적용**: 해당 폴더에서 `specify init --here`
- 업그레이드: `specify self upgrade`

### AI 채팅창에서의 실행

```text
/speckit-constitution 코드 품질, 테스트, 유지보수성 중심의 원칙을 만들어줘
/speckit-specify 날짜별 앨범과 타일 미리보기가 있는 사진 정리 앱
/speckit-plan Vite + 바닐라 JS. 이미지는 로컬에 두고 메타데이터는 SQLite에 저장
/speckit-tasks
/speckit-implement
/speckit-converge
```

> 에이전트마다 호출 문법이 다를 수 있습니다
> (예: 스킬 모드 에이전트는 `/skill:speckit-specify` 또는 `$speckit-specify`).

### 주요 CLI 명령 전체

```bash
specify init / check / version
specify self        check | upgrade
specify extension   add | remove | list | search | info | update | enable | disable | set-priority
specify extension catalog  list | add | remove
specify preset      add | remove | list | search | resolve | info | enable | disable | set-priority
specify workflow    run | resume | status | list | add | update | remove | enable | disable | search | info
specify integration install | uninstall | switch | upgrade | list | status | use | search | info
specify artifact    list | info | lookup
specify bundle      (역할별 묶음 설치)
specify event run
```

---

## 6. 플러그인 / 스킬 / MCP 구분

**결론: 셋 중 어느 것도 아니며, 정확히는 "CLI 도구 + 에이전트별 스킬 생성기"입니다.**

| 구분 | 해당 여부 | 설명 |
| --- | --- | --- |
| MCP 서버 | ❌ | MCP 프로토콜 구현이 없습니다. 서버를 띄우지 않습니다 |
| 플러그인 | △ | Claude Code 플러그인 규격은 아닙니다 |
| 스킬 | ⭕ | Claude용으로 설치하면 `.claude/skills/<이름>/SKILL.md` 를 생성합니다 |
| CLI 도구 | ⭕⭕ | 본체는 Python Typer 기반 CLI입니다 |

소스에서 확인된 Claude 연동 설정 (`src/specify_cli/integrations/claude/__init__.py`):

```python
class ClaudeIntegration(SkillsIntegration):
    key = "claude"
    config = {
        "name": "Claude Code",
        "folder": ".claude/",
        "commands_subdir": "skills",
        "requires_cli": True,
    }
    registrar_config = {
        "dir": ".claude/skills",
        "format": "markdown",
        "args": "$ARGUMENTS",
        "extension": "/SKILL.md",
    }
```

즉 **에이전트별 네이티브 포맷으로 스킬·명령 파일을 찍어내는 공장**입니다.

| 에이전트 | 생성 위치 | 포맷 |
| --- | --- | --- |
| Claude Code | `.claude/skills/` | markdown (`SKILL.md`) |
| GitHub Copilot | `.github/` | markdown |
| Cursor | `.cursor/` | markdown |
| Gemini | `.gemini/` | toml / markdown |
| 기타 | 각 에이전트 규약 | markdown / toml / yaml |

---

## 7. API 토큰 필요 여부

**기본적으로 필요하지 않습니다.**

공식 문서(`docs/reference/authentication.md`) 원문:
> "Specify CLI uses **opt-in authentication** … No credentials are sent unless you explicitly configure them."

### 토큰이 필요한 경우 (3가지뿐)

1. 비공개 / 사내 카탈로그에서 확장·프리셋을 설치할 때
2. GitHub Enterprise Server(GHES) 를 사용할 때
3. GitHub API rate limit 을 회피해야 할 때

### 설정 방법

`~/.specify/auth.json` (권한은 `chmod 600` 권장)

```json
{
  "providers": [
    {
      "hosts": ["github.com", "api.github.com", "raw.githubusercontent.com", "codeload.github.com"],
      "provider": "github",
      "auth": "bearer",
      "token_env": "GH_TOKEN"
    }
  ]
}
```

지원 프로바이더: `github`(bearer), `azure-devops`(bearer / basic-pat / azure-cli / azure-ad).
`token` 인라인보다 `token_env` 환경변수 참조가 권장됩니다.

### 주의 — 비용은 따로입니다

- Spec Kit 자체는 무료이지만, **Claude / Copilot 등 AI 에이전트 구독료는 별도**입니다.
- Spec Kit은 문서를 많이 생성·재독하므로 **토큰 소모량 자체는 늘어납니다.**
  다만 재작업이 줄어 총비용은 일반적으로 유리합니다.

---

## 8. AI 에이전트 구축에 주는 도움

**직접적인 에이전트 프레임워크는 아니지만, 설계 레퍼런스로서의 가치가 매우 큽니다.**

### 바로 가져다 쓸 수 있는 자산

| 자산 | 위치 | 활용 포인트 |
| --- | --- | --- |
| 프롬프트 설계 패턴 | `templates/commands/*.md` | MUST/SHOULD 어투, 단계 분리, 선행 체크, 출력 포맷 고정 |
| 워크플로 엔진 | `src/specify_cli/workflows/` | `gate`(사람 승인) / `if-then` / `while` / `do-while` 스텝 타입 |
| 확장 훅 시스템 | `extensions/`, `.specify/extensions.yml` | `before_specify`, `before_converge` 등 수명주기 훅 설계 |
| 41종 통합 모듈 | `src/specify_cli/integrations/` | 각 에이전트가 요구하는 디렉터리·포맷 레퍼런스 (단독으로도 가치 있음) |
| 영속 규칙 주입 | `.specify/memory/constitution.md` | 에이전트에 장기 규칙을 주는 방식 |
| 산출물 수렴 루프 | `templates/commands/converge.md` | "완료 주장"을 실제 코드와 대조해 검증하는 패턴 |

### 한계

- 멀티에이전트 **병렬** 실행 없음
- 메모리 / RAG / 벡터 검색 없음
- 툴 호출 루프(tool-calling loop) 없음

→ LangGraph · CrewAI · AutoGen 의 **대체재가 아니라, 그 위에 얹는 방법론**으로 보는 것이 정확합니다.

---

## 9. React / PHP 재구현 가능성

질문이 두 갈래이며 **둘 다 "가능"** 입니다.

### (A) Spec Kit으로 React / PHP 앱을 만드는 것

당연히 됩니다. Spec Kit은 **언어·프레임워크 중립**입니다.

```text
/speckit-plan React 18 + TypeScript + Vite, 상태관리는 Zustand, 테스트는 Vitest
/speckit-plan Laravel 11 + MySQL, 인증은 Sanctum, 테스트는 Pest
```

### (B) Spec Kit 자체를 React / PHP로 재구현

가능하며 난이도도 높지 않습니다. 핵심 자산이 **마크다운 템플릿**이고,
Python 코드의 대부분은 "파일 복사 + YAML 파싱 + 포맷 변환"이기 때문입니다.

| 구현 형태 | 난이도 | 추천 포지션 |
| --- | --- | --- |
| React 웹앱 | ★★☆☆☆ | spec.md 작성 에디터 + 미리보기 + 진척 대시보드 ← **가장 유망** |
| Node/TypeScript CLI | ★★☆☆☆ | `npx create-spec-kit` — Node 생태계 대상 |
| PHP (Laravel) | ★★★☆☆ | 팀 공유 스펙 저장소 + 승인 워크플로 |
| VS Code 확장 | ★★★☆☆ | 사이드바에서 단계 진행 + 스펙↔코드 추적 |

MIT 라이선스이므로 **상업적 재구현·재배포가 모두 합법**입니다 (저작권 고지 유지 조건).

> **현실적 권고**: CLI를 통째로 재구현하기보다,
> **CLI는 그대로 쓰면서 React로 "스펙 뷰어 / 에디터 / 대시보드" 레이어를 얹는 쪽**이
> 투입 대비 효과가 가장 큽니다. 현재 Spec Kit에는 **GUI가 전혀 없습니다.**

---

## 10. 유튜브 강의 제작 가능성

**매우 적합합니다.**

### 근거

- GitHub 공식 프로젝트 + 높은 인지도 → 검색 유입 유리
- "AI 코딩"은 현재 최상위 관심 키워드
- **한국어 콘텐츠가 사실상 없음** (저장소에 영어·중국어 README만 존재) → 선점 가능
- MIT 라이선스 → 화면 녹화·코드 인용 자유
- 결과물이 시각적 (파일이 생기고 체크리스트가 채워지는 과정)

### 추천 커리큘럼 (10부작)

| 화 | 제목 | 길이 |
| --- | --- | --- |
| 0 | AI한테 코딩 시켰다가 망한 이야기 (훅 영상) | 5분 |
| 1 | Spec Kit 설치부터 첫 프로젝트까지 | 12분 |
| 2 | constitution — AI에게 원칙 심기 | 10분 |
| 3 | specify — 기획서 작성 (가장 중요) | 15분 |
| 4 | plan & tasks — 설계와 작업 분해 | 15분 |
| 5 | implement & converge 루프 | 15분 |
| 6 | 실전: 투두앱 1시간 완주 | 40분 |
| 7 | bug 확장으로 버그 잡기 | 12분 |
| 8 | assess 확장으로 아이디어 검증 | 12분 |
| 9 | 41개 에이전트 갈아타기 / 팀 도입 전략 | 15분 |

### 성공 포인트

- 설치법만 읊는 영상은 이미 영어권에 존재합니다. **반드시 실제 결과물**을 보여주세요.
- **"Spec Kit 쓴 결과 vs 안 쓴 결과" 비교 포맷**이 조회수에 유리합니다.
- 실패 사례(모호한 스펙 → 엉뚱한 구현)를 먼저 보여주면 설득력이 크게 올라갑니다.

---

## 11. 수익화 아이디어 상세

### 티어 A — 즉시 시작, 투자 0원

#### A-1. 한국어 콘텐츠 선점 ⭐ 최우선 추천

현재 한국어 자료가 사실상 없습니다.

- 유튜브 10부작 + 블로그/브런치 연재
- **수익 모델**: 애드센스 → 강의 / 컨설팅 유입 퍼널
- **예상**: 3~6개월 후 월 30~100만원 (애드센스), 퍼널 효과가 본체
- **투입**: 0원, 주 5~10시간

#### A-2. 한국어 번역 + 템플릿 팩

- `README.ko.md` 번역 PR → GitHub 프로필 권위 (무료, 평판 자산)
- **유료**: 한국 실무용 spec/plan 템플릿 팩
  (SI 제안서형 / 스타트업 MVP형 / 공공사업 RFP형)
- 판매처: 크몽, Gumroad, 노션 템플릿 마켓
- **가격**: 3~9만원 · **예상**: 월 50~200만원

#### A-3. 유료 전자책 / PDF 가이드

- 「AI와 일하는 법: 스펙 주도 개발 실전」 90~150페이지
- **가격**: 2~4만원 (크몽 · 탈잉 · 부크크)
- **예상**: 월 100~300만원 (유튜브 연계 시)

### 티어 B — 1~3개월 개발 투입

#### B-1. 🏆 Spec Kit 웹 GUI (SaaS) — 가장 큰 기회

**근거**: 현재 Spec Kit에는 **GUI가 전혀 없습니다.** CLI + 마크다운뿐이라 비개발자 접근이 불가능합니다.

React로 구현할 기능:

- `spec.md` 폼 기반 작성기 (User Story / 우선순위 / Given-When-Then 입력 UI)
- 진척도 대시보드 (`tasks.md` 체크박스 → 칸반 / 간트 시각화)
- 팀 리뷰 · 코멘트 · 승인 (workflow 의 `gate` 를 웹에서 처리)
- GitHub 저장소 연동 (`specs/` 폴더 양방향 싱크)
- AI 에이전트 전환 비교 뷰

| 플랜 | 가격 | 대상 |
| --- | --- | --- |
| Free | 0원 | 개인, 프로젝트 1개 |
| Pro | $12/월 | 개인, 무제한 |
| Team | $29/인·월 | 협업 · 승인 워크플로 |
| Enterprise | 별도 견적 | 온프레미스, SSO, 감사로그 |

- **예상**: 1년차 MRR $3,000~10,000
- **리스크**: GitHub가 직접 GUI를 출시할 가능성 (중간 — 다만 GitHub은 CLI 지향이라 단기 위험은 낮음)

#### B-2. 업종 특화 Preset / Extension 유료 판매

Spec Kit은 확장·프리셋 **카탈로그 구조가 이미 공개되어 있습니다** (`catalog.community.json`).

- 핀테크 (전자금융감독규정 / ISMS 체크리스트 자동 삽입)
- 의료 (의료기기 SW 허가 문서)
- 커머스 / 공공 (전자정부 표준 산출물)
- **가격**: 10~50만원/팩 또는 구독
- **차별점**: 규제 대응은 복붙으로 만들 수 없음 → 진입장벽 확보

#### B-3. VS Code / JetBrains 확장

- 사이드바에서 Spec Kit 단계 진행 + 스펙↔코드 추적
- **Free + Pro($5/월)**, 마켓플레이스 유입은 무료
- **예상**: 월 100~500만원

### 티어 C — 고단가, 즉시 현금화

#### C-1. 💰 기업 교육 / 도입 컨설팅 — 투입 대비 수익 최고

"AI 코딩 도구를 도입했는데 효과가 없다"는 기업이 많습니다. Spec Kit은 그에 대한 처방입니다.

| 상품 | 가격 | 기간 |
| --- | --- | --- |
| 반나절 워크숍 (4시간) | 150~300만원 | 1일 |
| 2일 집중과정 | 400~800만원 | 2일 |
| 도입 컨설팅 (팀 1개) | 1,000~3,000만원 | 1~2개월 |
| 사내 템플릿 구축 | 2,000~5,000만원 | 2~3개월 |

- **진입 경로**: 유튜브 / 블로그 권위 → 인바운드 문의
- **예상**: 월 1~2건으로 월 300~800만원
- **장점**: 선투자 0, 즉시 현금화 · **단점**: 시간을 파는 구조라 확장성 낮음

#### C-2. SI / 외주 개발 생산성 무기화

- 제안 단계에서 `spec.md` 를 뽑아 **요구사항 명세서로 납품**
- 효과: 제안 품질↑ 수주율↑ / 요구사항 변경 분쟁 시 증빙자료 / 개발 공수 20~40% 절감
- **수익 구조**: 동일 인력으로 더 많은 프로젝트 소화 → 마진 개선

#### C-3. 유료 커뮤니티 / 멤버십

- 월 1~3만원, 템플릿 공유 + 코드리뷰 + Q&A + 월간 라이브
- **예상**: 100명 × 2만원 = 월 200만원 (유튜브 1만 구독 이후 현실적)

---

## 12. 실행 로드맵 및 리스크

### 추천 실행 순서

```text
1개월차    유튜브 0~3화 + 블로그 연재            (투자 0원)
2개월차    템플릿 팩 + 전자책 출시                (첫 수익)
3개월차    기업 워크숍 영업 시작                  (고단가 현금)
4~6개월차  React 웹 GUI MVP 개발 + 베타 운영      (확장성 확보)
7개월차~   SaaS 유료화 + 업종 특화 Extension      (자산화)
```

### 조합 추천

- **가장 현실적**: A-1(유튜브) + C-1(기업교육)
  → 투자 0원, 6개월 내 월 300~500만원 가능, 두 축이 서로 시너지
- **가장 큰 업사이드**: B-1(웹 GUI SaaS)
  → GUI 공백이 실재하고, MIT 라이선스로 합법이며, React로 충분히 구현 가능

### 공통 리스크

| 리스크 | 수준 | 대응 |
| --- | --- | --- |
| GitHub가 공식 GUI / 한국어 지원 출시 | 중 | 업종 특화 · 한국 시장 특화로 차별화 |
| AI 모델 발전으로 "스펙 없이도 잘함" | 중 | 도구가 아닌 **요구사항 정의 역량** 자체를 상품화 |
| 오픈소스 무료 대비 유료화 저항 | 중 | 무료는 도구, 유료는 **시간 절약 · 규제 대응 · 협업** |
| 커리큘럼 선점 경쟁 | 하 | 속도 우선, 실전 사례 중심으로 차별화 |

> **핵심**: 특정 도구(Spec Kit)에 종속되지 말고
> **"AI에게 일을 제대로 시키는 방법론"** 자체에 무게를 두면 도구가 바뀌어도 자산이 남습니다.

---

## 참고 링크

- 이 저장소: <https://github.com/bmshin94/spec-kit>
- 업스트림 원본: <https://github.com/github/spec-kit>
- 공식 문서: <https://github.github.io/spec-kit/>
- 빠른 시작: <https://github.github.io/spec-kit/quickstart.html>
- 설치 가이드: <https://github.github.io/spec-kit/installation.html>
- 통합(에이전트) 레퍼런스: <https://github.github.io/spec-kit/reference/integrations.html>
- SDD 철학: <https://github.github.io/spec-kit/concepts/sdd.html>
- 전체 방법론 문서: [spec-driven.md](./spec-driven.md)
- 인증 레퍼런스: <https://github.github.io/spec-kit/reference/authentication.html>
- 커뮤니티 확장·프리셋: <https://github.github.io/spec-kit/community/overview.html>
- PyPI 패키지: <https://pypi.org/project/specify-cli/>
- 라이선스(MIT): [LICENSE](./LICENSE)
