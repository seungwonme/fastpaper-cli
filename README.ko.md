[English](README.md) | **한국어** | [中文](README.zh-CN.md)

# Skill이 포함된 fastpaper CLI

AI 에이전트(Claude Code, Codex, Opencode 등)가 학술 논문과 과학 문헌을 검색하고 다운로드하고 읽을 수 있게 해주는 CLI 도구입니다. 에이전트가 분야에 맞는 출처를 선택하고 명령을 구성하도록 안내하는 [SKILL](skills/fastpaper/SKILL.md)도 함께 제공합니다.

명령 하나로 출처 하나를 설정 없이 사용합니다. 여러 출처를 병렬로 검색할 때는 에이전트가 여러 프로세스를 실행합니다.

## 설치

### CLI

**Homebrew (macOS / Linux)**

```sh
brew install zhangyee/tap/fastpaper
```

**Shell 스크립트 (macOS / Linux)**

```sh
curl --proto '=https' --tlsv1.2 -LsSf https://github.com/zhangyee/fastpaper-cli/releases/latest/download/fastpaper-cli-installer.sh | sh
```

**PowerShell (Windows)**

```powershell
powershell -ExecutionPolicy Bypass -c "irm https://github.com/zhangyee/fastpaper-cli/releases/latest/download/fastpaper-cli-installer.ps1 | iex"
```

**Cargo**

```sh
cargo install fastpaper-cli
```

### Skill

AI 에이전트가 fastpaper 사용법을 알 수 있도록 Skill을 설치합니다. 에이전트에 SKILL.md 파일을 설치하는 [Vercel Skills](https://github.com/vercel-labs/skills)를 사용합니다.

```sh
npx skills add zhangyee/fastpaper-cli --skill fastpaper
```

SKILL.md는 연구 분야에 맞는 출처 선택과 명령 구성을 안내합니다. 구조화된 출력이 필요하면 `--format json`을 사용하세요. 누락된 JSON 필드는 생략되지 않고 항상 `null`로 표시되므로 스키마가 안정적입니다.

## 빠른 시작

```sh
# arXiv 검색
fastpaper search arxiv "transformer attention mechanism"

# 필터를 적용해 검색
fastpaper search arxiv "large language model" --after 2024-01-01 --field cs.CL --limit 20

# DOI로 논문 조회(출처 자동 감지)
fastpaper get 10.1038/nature12373

# arXiv ID로 조회
fastpaper get 2301.08745

# 1-hop 인용 관계 탐색
fastpaper citations 2301.08745 --limit 20
fastpaper references 2301.08745 --limit 20

# PDF 다운로드
fastpaper download arxiv 2301.08745

# 전문 읽기
fastpaper read arxiv 2301.08745

# 특정 섹션 읽기
fastpaper read pmc PMC7318926 --section methods

# 로컬 PDF 읽기
fastpaper read local ./paper.pdf

# 스크립트와 AI 에이전트를 위한 JSON 출력
fastpaper search semantic "CRISPR gene editing" --format json

# 여러 출처를 병렬로 검색
fastpaper search arxiv "protein folding" --format json &
fastpaper search pubmed "protein folding" --format json &
fastpaper search semantic "protein folding" --format json &
wait
```

## 출처

18개 학술 출처를 명령마다 독립적으로 사용합니다.

| 출처 | 정식 명칭 | 검색 | 다운로드 | 읽기 | 분야 |
|------|-----------|:----:|:----------:|:----:|------|
| `arxiv` | arXiv | 지원 | 지원 | 지원 | 물리학, 수학, 컴퓨터 과학, 통계학, 전기공학, q-bio, q-fin, 경제학 |
| `biorxiv` | bioRxiv | 지원 | 지원 | 지원 | 생명과학 |
| `medrxiv` | medRxiv | 지원 | 지원 | 지원 | 의학 및 보건과학 |
| `pubmed` | PubMed | 지원 | | | 생의학 및 생명과학(메타데이터만 제공) |
| `pmc` | PubMed Central | 지원 | 지원 | 지원 | 생의학 및 생명과학(전문 제공) |
| `europepmc` | Europe PMC | 지원 | | | PMC를 포함하는 생명과학 통합 출처 |
| `scholar` | Google Scholar | 지원 | | | 전 분야(실험적 기능, 요청 제한 있음) |
| `xueshu` | Baidu Xueshu (百度学术) | 지원 | | | 전 분야, 중국어 문헌에 강함(실험적 비공식 API) |
| `semantic` | Semantic Scholar | 지원 | 지원 | 지원 | 전 분야, AI 기반 인용 그래프 |
| `crossref` | CrossRef | 지원 | | | 전 분야 DOI 메타데이터 |
| `openalex` | OpenAlex | 지원 | | | 2억 건 이상의 공개 메타데이터 색인 |
| `dblp` | DBLP | 지원 | | | 컴퓨터 과학 |
| `core` | CORE | 지원 | 지원 | 지원 | 오픈 액세스 통합 출처 |
| `openaire` | OpenAIRE | 지원 | | | 유럽 오픈 사이언스 |
| `doaj` | DOAJ | 지원 | 지원 | 지원 | 전 분야 오픈 액세스 학술지 |
| `unpaywall` | Unpaywall | 지원 | | | OA 링크 탐색기(`UNPAYWALL_EMAIL` 필요) |
| `zenodo` | Zenodo | 지원 | 지원 | 지원 | 전 분야(데이터셋, 소프트웨어, 논문) |
| `hal` | HAL | 지원 | 지원 | 지원 | 프랑스 국가 다학제 아카이브 |

## 명령

### `search` — 논문 검색

```
fastpaper search <SOURCE> <QUERY> [OPTIONS]

옵션:
  -n, --limit <N>        최대 결과 수 [기본값: 10]
      --offset <N>       앞에서 N개 결과 건너뛰기 [기본값: 0]
      --sort <FIELD>     정렬 기준: relevance, date, citations [기본값: relevance]
      --author <NAME>    저자 필터
      --after <DATE>     YYYY-MM-DD 이후 논문
      --before <DATE>    YYYY-MM-DD 이전 논문
      --year <YEAR>      특정 연도 논문
      --field <FIELD>    연구 분야 또는 카테고리(예: cs.AI)
      --open-access      오픈 액세스 논문만 표시
  -f, --format <FMT>     table, json, jsonl, csv, bibtex [기본값: table]
  -o, --output <PATH>    결과를 파일에 저장
```

### `get` — 식별자로 논문 조회

식별자 형식(DOI, arXiv ID, PMID, PMC ID, URL)을 감지해 출처를 자동 선택합니다.

```
fastpaper get <IDENTIFIER> [OPTIONS]

옵션:
      --resolve           사용 가능한 모든 OA 버전 탐색
      --with-citations    인용 수와 참고문헌 포함
      --with-abstract     초록 포함
```

### `citations` / `references` — 인용 관계 탐색

Semantic Scholar를 사용해 1-hop 인용 관계의 양쪽 논문을 조회합니다.

```
fastpaper citations <IDENTIFIER> [--limit 20] [--offset 0]
fastpaper references <IDENTIFIER> [--limit 20] [--offset 0]
```

DOI, arXiv, PMID, PMC, URL 및 `S2:` 식별자를 지원합니다.

### `download` — PDF 다운로드

```
fastpaper download <SOURCE> <IDENTIFIER> [OPTIONS]

옵션:
  -d, --dir <PATH>       다운로드 디렉터리 [기본값: ./papers]
      --filename <FMT>   템플릿: {id}, {title}, {authors}, {year}, {doi}
      --overwrite        기존 파일 덮어쓰기
      --source-files     arXiv 소스를 <ID>.tar.gz로 저장하고 압축은 해제하지 않음
```

### `read` — 논문 내용 읽기

```
fastpaper read <SOURCE> <IDENTIFIER> [OPTIONS]

옵션:
      --section <SEC>    abstract, introduction, methods, results,
                         discussion, conclusion, references, full [기본값: full]
      --metadata-only    메타데이터만 표시
      --raw              서식 없는 원문 텍스트
      --max-length <N>   출력을 N자로 자르기
  -o, --output <PATH>    파일로 저장
```

### `sources` — 출처와 기능 목록

```
fastpaper sources [--check] [--capabilities]
```

### `completions` — 셸 자동 완성

```
fastpaper completions fish > ~/.config/fish/completions/fastpaper.fish
fastpaper completions zsh > ~/.zfunc/_fastpaper
fastpaper completions bash >> ~/.bashrc
```

## 환경 변수

별도 표시가 없는 환경 변수는 선택 사항입니다. 18개 출처 중 17개는 설정 없이 사용할 수 있습니다.

| 변수 | 용도 |
|------|------|
| `FASTPAPER_DOWNLOAD_DIR` | 기본 다운로드 디렉터리(미설정 시 `./papers`) |
| `FASTPAPER_EMAIL` | CrossRef / OpenAlex polite pool 이메일 |
| `SEMANTIC_SCHOLAR_API_KEY` | Semantic Scholar 요청 한도 상향 |
| `CORE_API_KEY` | CORE 요청 한도 상향 |
| `NCBI_API_KEY` | PubMed / PMC 요청 한도 상향 |
| `UNPAYWALL_EMAIL` | Unpaywall 사용 시 **필수** |

## 종료 코드

| 코드 | 의미 |
|------|------|
| `0` | 성공 |
| `1` | 일반 오류(잘못된 인자, 파싱 실패) |
| `2` | 네트워크 오류(시간 초과, DNS 실패) |
| `3` | 출처 오류(API 오류, 요청 제한 초과) |
| `4` | 결과 없음 |
| `5` | 권한 오류(오픈 액세스 아님, 환경 변수 누락) |

## 기여

기여를 환영합니다. [CONTRIBUTING.md](CONTRIBUTING.md)와 [docs](docs/)의 개발자 문서를 참고하세요.

## 감사의 말

이 프로젝트는 여러 출처에서 학술 논문을 검색하고 다운로드하는 MCP 서버인 [paper-search-mcp](https://github.com/openags/paper-search-mcp)에서 영감을 받았습니다. 다중 출처 논문 도구의 가능성을 보여준 프로젝트 작성자들에게 감사드립니다.

## 라이선스

[GPL-3.0](LICENSE)
