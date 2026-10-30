# jobflow 플러그인 개발 규약

> 여러 작성자가 나눠 만들 때 서로 어긋나지 않도록 정한 공통 약속이다. 모든 스킬과 참고 문서는 이 규약을 따른다.
> 원본은 한 사람의 실사용 저장소이고, 이 플러그인은 그 도구만 데이터 없이 옮긴 것이다.

## 0. 확정된 정책 결정 (2026-10-30, 검증 후)

- **이메일 인증코드**: 기본은 멈추고 사용자에게 받는다. 사용자가 그 세션에서 "메일에서 찾아서 넣어줘"처럼 명시적으로 요청한 경우에만 Gmail 커넥터로 그 코드 하나를 읽어 입력할 수 있다. 휴대폰·PASS 본인인증, 캡차, 2단계 인증 앱은 예외 없이 사람 몫이다
- **자격 미달 확정 행의 마감일**: "마감일은 비워 두지 않는다"의 유일한 예외다. 비고 첫 줄이 `❌ 자격 미달 확정` 인 행은 달력에서 빼기 위해 마감일을 비운다. remind 의 "마감일 확인 필요" 경고는 이 행과 `마감 유형 = 상시` 행을 제외한다
- **`remind.stages` 의미**: 구간 경계다. 기본 `[7, 3, 1, 0]` 이면 남은 일수 0 = 🚨 오늘, 1 이하 = 🚨, 3 이하 = ⚠️, 7 이하 = 알림, 그보다 멀면 목록에만. 정확히 그날에만 울리는 방식이 아니다(수동 실행이라 하루를 건너뛸 수 있다)
- **급여·제외 필터**: `filters.salary_floor` 는 공고에 급여가 적혀 있을 때만 intake·fit 이 비교하고, 적혀 있지 않으면 "미명시"로 둔다(추측 금지). `targets.exclude` 와 `targets.tier2_roles` 는 intake 의 우선순위 판정에 쓴다
- **비밀번호를 명령줄 인자로 넘기지 않는다**: 키체인 저장은 사용자가 터미널에서 직접 대화형으로 입력하게 안내한다
- **민감 필드 마스킹**: 노션·보고서에는 주민번호·주소·전화 같은 민감 값 대신 `(입력함)` 만 적는다

## 1. 대원칙

- **데이터를 담지 않는다.** 이름, 연락처, 주소, 생년월일, 학교명, 학번, 자격번호, 회사명이 들어간 개인 이력, 노션 페이지·DB ID, 이메일 주소, 로컬 절대경로를 플러그인 파일에 쓰지 않는다. 예시가 필요하면 `<홍길동>`, `<example@email.com>` 같은 자리표시자를 쓴다
- **강의·유료 자료에서 온 내용을 담지 않는다.** 특정 강의의 양식, 목록 순서, 슬라이드 표현, 인용문을 옮기지 않는다. STAR 같은 일반 지식은 자체 표현으로 쓴다
- **원본 사고 사례는 익명화한다.** 회사명은 `A사(대형 이커머스)`처럼 바꾸고, 공고 원문 문장은 요지만 남긴다
- **한국어로 쓴다.** 긴 하이픈(U+2014, em dash)은 쓰지 않는다. 짧은 하이픈 `-`, 쉼표, 콜론, 괄호를 쓴다
- **안전 원칙은 그대로 살린다.** `reference/safety.md` 가 정본이다

## 2. 저장소 구조

```
jobflow-kit/                         # 마켓플레이스 저장소 (비공개)
├── .claude-plugin/marketplace.json  # name: jobflow-kit
├── README.md                        # 친구용 설치·사용 안내
├── docs/CONVENTIONS.md              # 이 문서 (개발용)
└── plugins/jobflow/
    ├── .claude-plugin/plugin.json   # name: jobflow
    ├── .mcp.json                    # Playwright MCP 묶음
    ├── README.md                    # 플러그인 기능 요약
    ├── reference/                   # 여러 스킬이 함께 보는 정본 문서
    │   ├── safety.md                # 안전 원칙 정본
    │   ├── config-schema.md         # 작업 폴더 config.yaml 형식
    │   ├── notion-schema.md         # 노션 트래커 구조 (setup 이 이대로 만든다)
    │   ├── intake-gate.md           # 자격 판정 게이트
    │   └── connectors.md            # 노션·Gmail·Playwright 연결 확인법
    ├── templates/
    │   ├── master.example.yaml      # 마스터 프로필 빈 양식
    │   ├── config.example.yaml      # 작업 폴더 설정 빈 양식
    │   └── collect-target.md        # 수집 대상 기준 빈 양식
    └── skills/
        ├── setup/SKILL.md           # /jobflow:setup
        ├── intake/SKILL.md          # /jobflow:intake
        ├── fit/SKILL.md             # /jobflow:fit
        ├── experience/SKILL.md      # /jobflow:experience (+ method.md)
        ├── remind/SKILL.md          # /jobflow:remind
        └── apply/SKILL.md           # /jobflow:apply (+ adapters/, field-registry.yaml, 참고 문서)
```

다른 파일 참조는 `${CLAUDE_PLUGIN_ROOT}/reference/safety.md` 처럼 플러그인 루트 기준으로 쓴다. 같은 스킬 폴더 안 파일은 `${CLAUDE_SKILL_DIR}/...` 또는 상대 링크로 쓴다.

## 3. 사용자 설정 (plugin.json userConfig)

설치할 때 입력받는 값은 두 개뿐이다.

| 키 | 타입 | 기본값 | 용도 |
|---|---|---|---|
| `workspace_dir` | directory | `~/jobflow-data` | 개인 데이터가 쌓이는 작업 폴더. 스킬 본문에서 `${user_config.workspace_dir}` 로 참조 |
| `notify_email` | string | 없음 (선택) | 마감 알림을 받을 Gmail 주소. 비어 있으면 세션 안에서만 보고 |

macOS 에서 작업 폴더를 Desktop·Documents·Downloads 아래에 두면 권한(TCC) 문제로 파일을 못 읽는 일이 생긴다. 기본값을 홈 바로 아래로 둔 이유다.

## 4. 작업 폴더 구조 (사용자 컴퓨터, 플러그인 밖)

```
<workspace_dir>/
├── config.yaml              # setup 이 만든다. 노션 ID, 판정값, 수집 기준 등
├── collect-target.md        # setup 이 템플릿을 복사해 사용자와 함께 채운다
├── profile/
│   ├── master.yaml          # 마스터 프로필 (PII. 이 컴퓨터 밖으로 보내지 않는다)
│   └── experiences/*.md     # 경험 카드
├── runs/<기업명>/           # 지원서 자동입력 산출물 (fields.json, 스크린샷)
└── snapshots/               # 공고 스크린샷 원본
```

모든 스킬은 시작할 때 `<workspace_dir>/config.yaml` 이 있는지 보고, 없으면 `/jobflow:setup` 을 먼저 돌리라고 안내하고 멈춘다.

## 5. config.yaml 키 (정본은 reference/config-schema.md)

```yaml
version: 1
notion:
  root_page_id: ""        # 📡 취업 준비 트래커 페이지
  jobs_ds: ""             # 채용 일정 데이터 소스 (collection://...)
  scrap_ds: ""            # 취업 스크랩 데이터 소스
eligibility:              # intake 6축 판정에 쓰는 내 값
  career: "신입"          # 신입 / 경력 N년
  degree: "학사"
  graduation: { status: "졸업", date: "YYYY-MM" }   # 졸업 / 졸업예정 / 재학
  majors: []
  languages: []           # [{test: "TOEIC", score: "900", date: "YYYY-MM"}]
  overseas_ok: true       # 해외 근무지 공고도 볼지
  needs_visa_sponsorship: false
filters:
  salary_floor: null      # 만원 단위. null 이면 급여 필터 끄기
targets:
  tier1_roles: []         # 1순위 직무 키워드
  tier2_roles: []
  exclude: []             # 제외할 회사 유형·직무
  max_years_required: 2   # 원문 연차 요건이 이보다 크면 지원 대상 아님(추적만)
  priority_map: { tier1: "높음", tier2: "중간", other: "낮음" }
remind:
  stages: [7, 3, 1, 0]
intake:
  default_target_days: 14 # 마감일 미확인 공고의 잠정 목표일
credentials:
  backend: "keychain"     # keychain(macOS) / ask(매번 묻기)
  prefix: "jobflow"
```

## 6. 노션 트래커 (정본은 reference/notion-schema.md)

setup 이 사용자 노션에 `📡 취업 준비 트래커` 페이지를 만들고 그 안에 인라인 DB 두 개를 만든다.

### 채용 일정 DB 속성

| 속성 | 타입 | 선택지 |
|---|---|---|
| 기업명 | title | |
| 직무 | text | |
| 모집 유형 | select | 체험형인턴 / 채용형인턴 / 공채 / 수시채용 / 경력채용 / 계약직 / 파트타임 |
| 진행 상태 | select | 지원 전 / 서류제출 / 필기전형 / 면접전형 / 서류합격 / 필기합격 / 면접합격 / 최종합격 / 서류탈락 / 필기탈락 / 면접탈락 / 공고마감 |
| 우선순위 | select | 높음 / 중간 / 낮음 (빈 값 = 원문 검증 전) |
| 자동화 상태 | select | 미시작 / 조사완료 / 입력완료 / 검토대기 / 차단됨 |
| 마감일 | date | 비워 두지 않는다 |
| 마감 유형 | select | 확정 / 상시 / 잠정 |
| 지원 페이지 | url | 원본 ATS URL |
| ATS | select | recruiter.co.kr / Greenhouse / Lever / Workday / Greeting / careerlink / 인크루트 / 자체구축 / 미확인 |
| 계정 상태 | select | 불필요 / 생성필요 / 보유 |
| 지원일 | date | |
| 비고 | text | 첫 줄에 결론 표식 (`🔴 미확인`, `❌ 자격 미달 확정`, `⚠️`, `📸`) |

뷰: `달력`(마감일 기준 캘린더), `🎯 지원 대상`(진행 상태 = 지원 전), `진행 중`(서류제출~면접합격), `지원 결과`(탈락·최종합격).

행 본문 구조: `## 1. 공고` / `## 2. 자기소개서` / `## 3. 준비 항목` / `## 4. 입력 내역` / `## 5. 결정 필요`. fit 은 맨 아래에 `## JD 분석 (YYYY-MM-DD)` 를 붙인다.

### 취업 스크랩 DB 속성

| 속성 | 타입 | 선택지 |
|---|---|---|
| 이름 | title | |
| 유형 | select | 공모전 / 해커톤 / 특강 / 합격자료 / 기타 |
| 링크 | url | |
| 마감 | date | |
| 메모 | text | |

## 7. 커넥터와 도구 (정본은 reference/connectors.md)

- 노션·Gmail 은 claude.ai 커넥터다. 플러그인이 깔아 줄 수 없다. 스킬은 도구 이름을 하드코딩하지 말고 "노션 커넥터의 검색·조회·페이지 생성 도구" 처럼 기능으로 부르고, 도구가 안 보이면 ToolSearch 로 `notion`, `gmail` 을 찾아보게 한다. 그래도 없으면 claude.ai 설정에서 커넥터를 켜라고 안내하고 멈춘다
- Playwright 는 플러그인 `.mcp.json` 에 묶는다. 사용자가 따로 쓰는 Playwright 서버가 있으면 그것을 써도 된다
- 노션 SQL 조회에서 날짜는 `"date:마감일:start"` 확장 컬럼으로만 접근된다. 페이지 쓰기에서 `is_datetime` 은 문자열이 아니라 숫자 0/1 이다

## 8. 스킬 공통 형식

```yaml
---
name: intake
description: <한 문단. 언제 쓰는지 사용자 표현 예시를 따옴표로 2~3개>
---
```

본문 순서: 시작 점검(config 존재, 필요한 커넥터) → 절차 → 금지 → 참고 문서 링크. SKILL.md 는 500줄 이하로, 긴 내용은 같은 폴더의 보조 문서로 뺀다.
