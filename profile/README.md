# Artel

**https://artel.kr**

Artel은 AI Agent가 실제 플레이어처럼 게임을 플레이하며 QA를 수행하는 프로그램입니다.

AI SW 마에스트로 프로젝트로 시작한 Artel은 게임의 주요 기능과 플레이 흐름을 자동으로 확인하고, 플레이 중 발생하는 오류와 이상 현상을 찾아내는 것을 목표로 합니다.

AI Agent는 게임 안에서 정해진 시나리오에 따라 행동하거나 자유롭게 플레이하면서 화면과 게임 상태를 관찰합니다. 문제가 발견되면 해당 상황을 다시 재현할 수 있도록 과정과 결과를 기록하고, QA에 활용할 수 있는 형태로 정리합니다.

## 주요 기능

- AI Agent 기반 게임 플레이
- 게임 시나리오 및 기능 테스트
- 오류와 이상 현상 탐지
- 문제 상황 재현 과정 기록
- 테스트 결과 및 QA 리포트 생성

Artel은 반복적인 게임 테스트를 자동화하고, 개발자가 게임의 문제를 더 빠르게 발견하고 개선할 수 있도록 돕는 AI 기반 게임 QA 도구입니다.

## 저장소

| 저장소 | 무엇인가 | 배포되는 image |
| --- | --- | --- |
| [artel-orchestration-server](https://github.com/project-artel/artel-orchestration-server) | Kotlin · Spring WebFlux. 프로젝트, QA 런, content map, 인증 | `ghcr.io/project-artel/orchestration` |
| [artel-agent-server](https://github.com/project-artel/artel-agent-server) | Python · FastAPI. 게임을 플레이하는 QA agent | `ghcr.io/project-artel/agent` |
| [artel-home](https://github.com/project-artel/artel-home) | React. Replay Studio, 로그인한 사용자가 쓰는 화면 | `ghcr.io/project-artel/console` |
| [admin-page](https://github.com/project-artel/admin-page) | React. 개발자 등급이 보는 전체 프로젝트 통계 | `ghcr.io/project-artel/admin` |
| [artel-sdk](https://github.com/project-artel/artel-sdk) | Unity SDK. 게임 빌드에 붙는다 | — |
| [artel-cli](https://github.com/project-artel/artel-cli) | 터미널에서 QA 런을 돌린다 | — |

## 전체 stack 띄우기

이 저장소의 `docker-compose.yml`이 위의 서비스 넷을 Postgres, Redis와 함께 띄웁니다. 명령 하나와 `.env` 하나면 됩니다.

### 시작하기 전에

Docker와 Compose v2만 있으면 됩니다. image는 public이라 `docker login`도, GitHub 계정도 필요 없습니다.

### 절차

```bash
git clone https://github.com/project-artel/.github.git artel-stack
cd artel-stack
cp .env.example .env
```

`.env`가 필수라고 표시한 값 셋을 채운 뒤 띄웁니다.

```bash
docker compose up -d
```

처음 한 번은 image를 받고 Postgres가 스키마를 만드느라 1분쯤 걸립니다. `docker compose logs -f orchestration`을 보면 Flyway가 마이그레이션을 적용하고 마지막에 `Started ArtelOrchestrationApplicationKt`가 찍힙니다.

### 떠 있는 것

| 주소 | 서비스 |
| --- | --- |
| http://localhost:5173 | Console — Replay Studio |
| http://localhost:5174 | Admin |
| http://localhost:8080 | Orchestration API. Swagger는 `/swagger-ui.html` |
| http://localhost:8000 | Agent API. 문서는 `/docs` |

orchestration은 agent가 부르는 무인증 내부 API를 8081에서 따로 서빙합니다. 그 포트는 호스트에 게시되지 않으며, 그것이 내부 API를 network에서 떼어 놓는 유일한 근거입니다. `ports`에 추가하지 마세요.

### 환경변수

`.env.example`이 하나하나 설명합니다. 짧게 줄이면 이렇습니다.

| 변수 | 없으면 |
| --- | --- |
| `ARTEL_JWT_SECRET` | orchestration이 기동에 실패합니다. 32바이트 이상이어야 합니다 |
| `GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET` | orchestration이 기동에 실패합니다. 로그인하려면 실제 OAuth app이 필요하지만, 띄우는 데는 필요 없습니다 |
| `LLM_API_KEY` | 전부 뜨고 model 호출만 실패합니다. QA 런, 시나리오 저작, 지식 추출에 필요합니다 |

나머지 변수는 전부 쓸 만한 기본값이 있습니다. 호스트 포트, image 태그, database 이름과 접속 정보를 옮겨야 하면 모두 `.env`에 있습니다.

### 로그인

로그인은 GitHub OAuth를 거치므로 위 주소로 등록한 OAuth app이 필요합니다. 입력할 URL 두 개는 `.env.example`에 적혀 있습니다. 없어도 stack은 그대로 돌고 API도 `curl`에 답합니다. 브라우저 세션만 못 씁니다.

### 이것으로 안 되는 것

QA 런에는 Unity SDK가 붙은 게임이 필요하고, 그것은 어떤 container도 대신할 수 없습니다. 기획서 업로드와 화면 capture 저장에는 S3 자격증명이 필요합니다. 비워 두면 그 기능만 못 씁니다.

### 내리기

```bash
docker compose down
```

database는 named volume에 남습니다. `docker compose down -v`는 그것까지 지우고, 다음에 띄울 때 스키마를 처음부터 다시 만듭니다.
