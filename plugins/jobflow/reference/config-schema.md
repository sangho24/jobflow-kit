# config.yaml 형식

작업 폴더(`${user_config.workspace_dir}`) 바로 아래의 `config.yaml` 은 `/jobflow:setup` 이 `templates/config.example.yaml` 을 복사해 만든다. 모든 스킬은 시작할 때 이 파일이 있는지 보고, 없으면 `/jobflow:setup` 을 먼저 돌리라고 안내한 뒤 멈춘다. 이 파일에는 노션 ID 와 판정 기준이 들어가므로 작업 폴더 밖으로 내보내지 않는다.

## 전체 모양

```yaml
version: 1
notion:
  root_page_id: ""
  jobs_ds: ""
  scrap_ds: ""
eligibility:
  career: "신입"
  degree: "학사"
  graduation: { status: "졸업", date: "YYYY-MM" }
  majors: []
  languages: []
  overseas_ok: true
  needs_visa_sponsorship: false
filters:
  salary_floor: null
targets:
  tier1_roles: []
  tier2_roles: []
  exclude: []
  max_years_required: 2
  priority_map: { tier1: "높음", tier2: "중간", other: "낮음" }
remind:
  stages: [7, 3, 1, 0]
intake:
  default_target_days: 14
credentials:
  backend: "keychain"
  prefix: "jobflow"
```

## 경로 표기

모든 스킬은 `workspace_dir` 값의 `~` 를 홈 경로로 확장해 쓴다. setup 은 `config.yaml` 에 확장된 절대경로를 기록하지 않는다. 사람마다 홈 경로가 다르므로 `~` 를 그대로 둔다.

## 키 설명

### version
스키마 버전. 지금은 `1`. 스킬이 형식이 바뀌었는지 판단하는 데 쓴다. 쓰는 스킬: 전부.

### notion
setup 이 트래커를 만들거나 연결할 때 기록한다. 쓰는 스킬: intake, fit, remind, apply, setup.

- `root_page_id`: `📡 취업 준비 트래커` 페이지 ID. 예: `"<페이지 ID 32자리>"`
- `jobs_ds`: 채용 일정 데이터 소스. `collection://...` 형태 전체를 적는다. 행 생성의 부모와 SQL 조회의 대상이 된다
- `scrap_ds`: 취업 스크랩 데이터 소스. 공모전, 특강 같은 보조 자료용

### eligibility
intake 의 6축 판정에 쓰는 내 값. 쓰는 스킬: intake, fit, setup(채움).

- `career`: `"신입"` 또는 `"경력 N년"`. 경력 칸에 적을 연수와 일치시킨다
- `degree`: `"학사"`, `"석사"` 등 최종 학위 기준
- `graduation`: `status` 는 `졸업` / `졸업예정` / `재학`, `date` 는 `YYYY-MM`. 예: `{ status: "졸업예정", date: "2027-02" }`
- `majors`: 전공 목록. 복수전공과 부전공도 넣는다. 예: `["<전공1>", "<전공2>"]`
- `languages`: 어학 성적 목록. 예: `[{test: "TOEIC", score: "900", date: "2026-03"}]`
- `overseas_ok`: 해외 근무지 공고도 볼지. `true` / `false`
- `needs_visa_sponsorship`: 해외 취업에 비자 후원이 필요한지. `true` 면 후원 불가 공고는 자격 미달로 본다

### filters
- `salary_floor`: 만원 단위 연봉 기준선. 예: `4000`. `null` 이면 급여 필터를 끈다. 공고에 급여가 적혀 있을 때만 비교하고, 적혀 있지 않으면 "미명시"로 둔다(추측 금지). 쓰는 스킬: intake, fit

### targets
공고의 우선순위와 지원 대상 여부를 정한다. `tier1_roles`, `tier2_roles`, `exclude`, `priority_map` 은 intake 가 우선순위를 정할 때 쓰고, `tier1_roles` 는 fit 도 참고한다. `max_years_required` 는 intake 의 연차 판정에 쓴다.

- `tier1_roles`: 1순위 직무 키워드. 예: `["<직무A>", "<직무B>"]`. 직무명은 자격의 근거가 아니라 모으는 기준일 뿐이다
- `tier2_roles`: 2순위 직무 키워드
- `exclude`: 제외할 회사 유형이나 직무. 예: `["<회사 유형>"]`
- `max_years_required`: 공고 원문의 연차 요건이 이 값보다 크면 지원 대상이 아니다. 추적만 한다
- `priority_map`: 분류에 따른 노션 `우선순위` 값. `tier1` 은 높음, `tier2` 는 중간, 그 외는 낮음이 기본. 자격 판정을 통과한 행에만 우선순위를 준다

### remind
- `stages`: 남은 일수의 구간 경계다. 기본 `[7, 3, 1, 0]` 이면 남은 일수 0 은 🚨 오늘, 1 이하는 🚨, 3 이하는 ⚠️, 7 이하는 알림, 그보다 멀면 목록에만 올린다. 정확히 그날에만 울리는 방식이 아니다(수동 실행이라 하루를 건너뛸 수 있다). 같은 알림을 하루에 두 번 보내지 않는다. 쓰는 스킬: remind

### intake
- `default_target_days`: 마감일을 확인하지 못한 공고에 오늘부터 잡는 잠정 목표일(일). 이때 마감 유형은 `잠정`. 마감일은 비워 두지 않는다. 쓰는 스킬: intake

### credentials
지원 사이트 계정 비밀번호를 어디서 가져오는지 정한다. 쓰는 스킬: apply.

- `backend`: `keychain`(macOS 키체인) 또는 `ask`(필요할 때마다 사용자에게 묻기). 어느 쪽이든 비밀번호를 파일, 노션, 로그, 메일에 쓰지 않는다
- `prefix`: 키체인 항목 이름 앞에 붙는 접두어. 예: `jobflow` 면 `jobflow-<사이트>` 형태로 찾는다
