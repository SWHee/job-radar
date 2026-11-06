<div align="center">

# Job Radar

**흩어진 채용 공고를 모아, 나에게 맞는 공고만 Notion에.**

기업 채용 페이지를 검색하고 직무와 경력 조건에 맞춰 정리하는<br>
Claude Code 플러그인입니다.

[![Claude Code](https://img.shields.io/badge/Claude_Code-Plugin-D97757?style=for-the-badge)](https://claude.com/claude-code)
[![Python](https://img.shields.io/badge/Python-3.11%2B-3776AB?style=for-the-badge&logo=python&logoColor=white)](pyproject.toml)
[![MCP](https://img.shields.io/badge/MCP-Server-2D6CDF?style=for-the-badge)](https://modelcontextprotocol.io/)
[![License](https://img.shields.io/badge/License-MIT-34A853?style=for-the-badge)](LICENSE)

[빠른 시작](#-빠른-시작) · [작동 방식](#-작동-방식) · [수집 출처](#-수집-출처) · [회사 추가](#-회사-추가) · [문제 해결](#-문제-해결)

</div>

---

## 왜 Job Radar인가요?

관심 있는 기업의 채용 페이지를 하나씩 확인하고, 직무와 연차가 맞는지 살핀 뒤, 공고 링크를 다시 정리하는 일은 시간이 걸립니다. Job Radar는 이 과정을 **Claude Code 스킬 + MCP 서버**로 연결합니다.

| | Job Radar가 하는 일 |
| --- | --- |
| 🔎 **모으기** | 등록된 기업의 채용 페이지에서 진행 중인 공고를 수집합니다. |
| 🎯 **걸러내기** | 직무 키워드와 경력 조건으로 후보를 고르고, Claude가 직무와 자격 요건을 다시 살핍니다. |
| 🗂️ **정리하기** | 공식 지원 링크를 기준으로 중복을 확인하고 새 공고를 Notion DB에 기록합니다. |
| 🧹 **관리하기** | 실행할 때 Notion DB에서 마감일이 지난 공고를 삭제하고 결과를 보고합니다. |

> [!IMPORTANT]
> 실행 시 연결한 Notion DB에서 **마감일이 오늘 이전인 공고를 삭제**합니다. 기존 DB를 연결하기 전에 정리 대상과 접근 권한을 확인하세요. 마감일이 없는 공고는 유지합니다.

## ⚡ 빠른 시작

### 1. 준비물

- [Claude Code](https://claude.com/claude-code)
- [uv](https://docs.astral.sh/uv/)와 Python 3.11 이상
- Notion 계정과 공고를 저장할 페이지에 대한 접근 권한

### 2. 플러그인 설치

Claude Code에서 실행하세요.

```text
/plugin marketplace add SWHee/job-radar
/plugin install job-radar@job-radar
```

설치 후 Claude Code를 다시 시작하세요. 플러그인에 스킬과 MCP 서버가 함께 포함되어 있습니다.

### 3. Notion 연결

claude.ai 설정의 커넥터에서 Notion을 연결하고, Claude Code에도 같은 계정으로 로그인하세요. 또는 터미널에서 Notion MCP를 등록한 뒤 Claude Code의 `/mcp`에서 인증할 수 있습니다.

```bash
claude mcp add --transport http notion https://mcp.notion.com/mcp
```

### 4. 공고 모으기

Claude Code에서 실행하세요.

```text
/job-radar:job-radar
```

또는 **“채용 공고 업데이트해줘”**라고 요청할 수 있습니다. 처음 실행하면 원하는 직무, 경력, 기업 분류, 제외 조건과 Notion DB를 물어봅니다. 답변을 바탕으로 검색 키워드를 제안하고, 확인 후 첫 수집을 시작합니다.

개인 설정은 `~/.job-radar/profile.yaml`에 저장됩니다. 조건을 바꾸려면 **“job-radar 설정 바꿔줘”**라고 요청하세요.

<details>
<summary><b>플러그인 없이 직접 설치하기</b></summary>

```bash
git clone https://github.com/SWHee/job-radar.git
cd job-radar
uv sync
claude mcp add job-radar -s user -- uv run --directory "$(pwd)" server.py
mkdir -p ~/.claude/skills
ln -s "$(pwd)/skills/job-radar" ~/.claude/skills/job-radar
```

직접 설치한 스킬은 `/job-radar`로 실행합니다. Notion MCP 연결은 별도로 필요합니다. 이미 같은 이름의 스킬이나 MCP가 등록되어 있다면 기존 설정을 먼저 확인하세요.

</details>

## 🧭 작동 방식

```text
내 직무 · 경력 · 기업 분류
             │
             ▼
  기업 채용 페이지에서 공고 수집
             │
             ▼
   키워드 · 경력 조건으로 1차 필터
             │
             ▼
   Claude가 직무 · 자격 요건 확인
             │
             ▼
    중복 확인 후 Notion DB에 저장
```

실제로는 수집 전에 마감 공고를 정리하며, 끝나면 추가·삭제 건수와 수집 실패 또는 확인하지 못한 출처를 보고합니다. 경력 정보가 불명확한 공고는 본문 확인을 시도합니다. Claude의 판단과 채용 사이트 상태에 따라 결과가 달라질 수 있으므로 **지원 전 원문에서 조건과 마감일을 확인하세요.**

## 🌐 수집 출처

수집 대상은 [`companies.yaml`](companies.yaml)에서 관리합니다. 이 목록에는 기업별 채용 페이지와 공공기관 채용 출처가 함께 포함됩니다.

| 방식 | 지원 출처 |
| --- | --- |
| 전용 수집기 | 삼성, CJ, 네이버, 카카오, 토스, 잡알리오 |
| 채용 플랫폼 기반 | Greenhouse, 그리팅, 마이다스 (`recruiter.co.kr`) |
| 페이지 확인 시도 | `custom`으로 등록한 기업 채용 페이지를 Claude가 WebFetch로 확인 |

마이다스, 삼성, 그리팅, Greenhouse 계열은 `get_job_detail`로 본문 확인을 시도합니다. 지원하지 않는 출처는 WebFetch로 확인합니다. 사이트의 변경·접속 제한이나 이미지로 된 본문 때문에 수집 또는 확인에 실패할 수 있습니다.

**직접 확인이 필요한 예:** SK, LG, 현대자동차그룹 채용 페이지는 현재 코드 수집 대상이 아닙니다. 사람인·원티드·잡코리아 등 채용 플랫폼 전체를 검색하는 기능도 현재 포함되어 있지 않습니다.

## 🏢 회사 추가

[`companies.yaml`](companies.yaml)의 `companies` 목록에 항목을 추가하세요. 실제 회사의 채용 페이지 주소와 사용하는 플랫폼을 확인한 뒤 입력합니다.

```yaml
# 예시: 값을 실제 회사 정보로 바꿔 주세요.
- {name: 회사명, category: 중견기업, source: greeting, id: example.career.greetinghr.com}
```

| `source` | 필요한 값 |
| --- | --- |
| `greeting` | `id`: 그리팅 채용 페이지 호스트 |
| `greenhouse` | `id`: Greenhouse 보드 이름 |
| `recruiter` | `id`: 마이다스 채용 페이지 호스트 |
| `custom` | `url`: 직접 확인을 시도할 채용 페이지 주소 |

`category`는 `대기업`, `대기업 계열사`, `중견기업`, `강소·유망`, `공기업`, `외국계` 중에서 지정합니다. 기존 항목과 주석을 참고해 형식을 맞춰 주세요.

## 🧩 프로젝트 구성

```text
job-radar/
├── .claude-plugin/          플러그인 및 마켓플레이스 설정
├── .mcp.json                MCP 서버 실행 설정
├── server.py                채용 공고 수집 및 검색 도구
├── companies.yaml           수집 대상과 출처
├── pyproject.toml           Python 버전과 의존성
└── skills/job-radar/
    └── SKILL.md             설정·필터링·Notion 저장 워크플로
```

MCP 서버는 `search_jobs`, `get_job_detail`, `fetch_jobs`, `list_companies` 도구를 제공합니다. 검색 시 제목과 직무 태그를 먼저 필터링하고, 상세 판단은 스킬에서 수행합니다.

## 🛠️ 문제 해결

<details>
<summary><b>명령어가 보이지 않아요</b></summary>

플러그인 설치 후 Claude Code를 다시 시작했는지 확인하세요. 플러그인 설치 시에는 `/job-radar:job-radar`, 직접 설치 시에는 `/job-radar`를 사용합니다.

</details>

<details>
<summary><b>Notion에 저장되지 않아요</b></summary>

Notion 커넥터 또는 MCP 연결을 확인하고 Claude Code의 `/mcp`에서 인증 상태를 확인하세요. Notion에서 대상 페이지와 DB에 접근할 권한도 필요합니다.

</details>

<details>
<summary><b>일부 회사의 공고가 없어요</b></summary>

채용 페이지 구조가 바뀌었거나 접근이 차단되었을 수 있습니다. 실행 후 보고되는 수집 실패 목록과 확인하지 못한 페이지를 살펴보고 해당 회사 채용 사이트에서 직접 확인하세요.

</details>

## 🤝 기여

잘못된 채용 페이지, 누락된 기업이나 개선할 점을 발견했다면 [이슈](https://github.com/SWHee/job-radar/issues)를 열거나 Pull Request를 보내 주세요. 새 출처를 추가할 때는 해당 사이트의 이용 조건과 접근 정책을 확인해 주세요.

이 저장소는 [KIMDONGWOOK12/job-radar](https://github.com/KIMDONGWOOK12/job-radar)의 포크입니다.

## 라이선스

[MIT License](LICENSE) · [@KIMDONGWOOK12](https://github.com/KIMDONGWOOK12) · [@SWHee](https://github.com/SWHee)
