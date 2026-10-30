---
name: setup
description: jobflow 를 처음 쓸 때 작업 폴더, 노션 취업 트래커, 설정 파일(config.yaml), 마스터 프로필, 수집 기준을 한 번에 준비한다. 다시 실행하면 빠진 것만 채운다. "jobflow 처음 설정해 줘", "트래커 만들어 줘", "setup 다시 돌려 줘" 같은 요청에 사용.
---

# /jobflow:setup

처음 한 번 돌리는 준비 스킬이다. 멱등이다. 이미 있는 것은 건너뛰고 빠진 것만 만든다. 각 단계 시작 전에 이미 있는지 먼저 확인하고, 있으면 "이미 있음, 건너뜀" 으로 알린다.

작업 폴더는 `${user_config.workspace_dir}` 이다. 값에 `~` 가 있으면 홈 디렉터리로 확장해 쓴다. 이하 `<workspace>` 로 부른다.

안전 원칙은 `${CLAUDE_PLUGIN_ROOT}/reference/safety.md` 를 따른다. 특히 개인정보는 작업 폴더와 사용자 노션 밖으로 보내지 않고, 값을 지어내지 않는다.

## 1. 작업 폴더

1. `<workspace>` 가 비어 있으면(설정 안 됨) 기본값 `~/jobflow-data` 를 제안하고 사용자에게 확인받는다
2. 경로가 `Desktop`, `Documents`, `Downloads` 아래면 경고한다: "macOS 에서는 이 위치의 파일을 Claude 가 못 읽는 권한 문제가 생길 수 있습니다. 홈 바로 아래를 권장합니다." 사용자가 그래도 쓰겠다고 하면 진행한다
3. 폴더와 하위 폴더를 만든다(이미 있으면 그대로 둔다): `<workspace>/`, `profile/`, `profile/experiences/`, `runs/`, `snapshots/`
4. 개인정보 폴더이므로 `git` 저장소로 만들거나 클라우드 동기화 폴더에 두지 말라고 한 줄 안내한다

## 2. 커넥터 점검

`${CLAUDE_PLUGIN_ROOT}/reference/connectors.md` 의 확인법대로 점검하고 결과를 한 번에 알린다.

- 노션 커넥터: 없으면 안내만 하고 여기서 멈춘다(노션이 없으면 트래커를 만들 수 없다). 1번에서 만든 폴더는 그대로 두고, 커넥터를 켠 뒤 다시 실행하라고 말한다
- Gmail 커넥터: 없어도 진행한다. 마감 알림을 메일로 못 보내고 세션 안에서만 보고한다고 알린다
- Playwright 서버: 없어도 진행한다. 지원서 자동입력과 웹 공고 열람이 안 된다고 알린다

## 3. 노션 트래커

구조의 정본은 `${CLAUDE_PLUGIN_ROOT}/reference/notion-schema.md` 이다. 속성 이름과 선택지를 그대로 쓴다.

1. `<workspace>/config.yaml` 이 이미 있고 `notion.jobs_ds` 가 채워져 있으면, 그 데이터 소스를 조회해 실제로 접근되는지 확인한다. 접근되면 3번 전체를 건너뛴다
2. config 에 ID 가 없더라도 노션에 `📡 취업 준비 트래커` 가 이미 있는지 검색한다. 있으면 새로 만들지 말고 "기존 트래커에 연결만 할까요, 새로 만들까요?" 를 묻는다. 연결을 고르면 페이지를 조회해 채용 일정 DB 와 취업 스크랩 DB 의 데이터 소스 ID 를 찾아 속성이 스키마와 맞는지 대조하고, 빠진 속성만 추가한다
3. 새로 만들 때: 사용자에게 "어느 페이지 아래에 만들까요? 정하지 않으면 개인 페이지로 만듭니다." 를 묻는다. 부모 페이지는 검색해서 후보를 보여 주거나 사용자가 링크를 주게 한다
4. 페이지 `📡 취업 준비 트래커` 를 만든다
5. 그 안에 채용 일정 DB 와 취업 스크랩 DB 를 인라인으로 만든다. DDL 은 notion-schema.md 의 예시를 쓴다
6. 채용 일정 DB 에 뷰 4개를 만든다: `달력`, `🎯 지원 대상`, `진행 중`, `지원 결과`. 필터가 막히면 사용자에게 노션 화면에서 걸도록 안내한다
7. 생성한 DB 를 다시 조회해 속성과 선택지가 스키마와 같은지 확인한다. 다르면 고친다
8. 페이지 ID 와 두 데이터 소스 ID(`collection://...`)를 기억해 4번 단계에서 config.yaml 에 기록한다

## 4. config.yaml

1. `<workspace>/config.yaml` 이 없으면 `${CLAUDE_PLUGIN_ROOT}/templates/config.example.yaml` 을 복사한다. 있으면 덮어쓰지 않고 빈 키만 채운다
2. 3번에서 얻은 `notion.root_page_id`, `notion.jobs_ds`, `notion.scrap_ds` 를 기록한다
3. 키의 의미는 `${CLAUDE_PLUGIN_ROOT}/reference/config-schema.md` 를 따른다. 경로는 확장한 절대경로로 바꿔 쓰지 않고 `~` 를 그대로 둔다

## 5. 프로필 시작

1. `<workspace>/profile/master.yaml` 이 없으면 `${CLAUDE_PLUGIN_ROOT}/templates/master.example.yaml` 을 복사한다. 있으면 건드리지 않는다
2. 자격 판정에 쓰는 6축을 한 축씩 대화로 묻는다. 한 번에 한 가지만 묻고, 모르면 비워 둔다. 값을 추측해 채우지 않는다
   - 경력: 신입인지, 경력이면 몇 년인지
   - 학위: 학사, 석사 등 최종 학위
   - 졸업: 졸업 / 졸업예정 / 재학, 그리고 시점(YYYY-MM)
   - 전공: 전공, 복수전공, 부전공
   - 어학: 가진 시험 이름, 점수나 등급, 취득일 (없으면 없다고 기록)
   - 해외·비자: 해외 근무지 공고도 볼지, 해외 취업 시 비자 후원이 필요한지
3. 답을 `config.yaml` 의 `eligibility` 와 `profile/master.yaml` 의 해당 필드(`education.university[0]`, `languages`, `preferences.overseas_ok` 등)에 같이 기록한다. 두 곳의 값이 어긋나지 않게 한다
4. 이름, 연락처, 주소, 주민번호 같은 개인정보 전체를 지금 입력하라고 강요하지 않는다. "나머지는 지원서 자동입력(`/jobflow:apply`)을 쓰기 전까지 `profile/master.yaml` 을 직접 열어 채우거나, 그때 Claude 가 물어보면 됩니다." 라고 안내한다. 사용자가 원할 때만 더 받는다
5. 마스터 프로필의 민감 정보는 이 컴퓨터 밖으로 보내지 않고, 노션에도 적지 않는다는 점을 한 줄로 알린다

## 6. 수집 기준

1. `<workspace>/collect-target.md` 가 없으면 `${CLAUDE_PLUGIN_ROOT}/templates/collect-target.md` 를 복사한다. 있으면 건드리지 않는다
2. 사용자와 1순위 직무 몇 개(3개 안팎)만 함께 정한다. "어떤 직무 공고를 가장 먼저 보고 싶으세요?" 처럼 묻고, 답을 문서의 `1순위 직무` 와 `config.yaml` 의 `targets.tier1_roles` 에 기록한다. 직무 방향 한 문단(`왜`)도 사용자 말 그대로 적는다
3. 2순위, 제외 조건, 연봉 기준선 등은 지금 정하지 않아도 된다고 안내한다. 나중에 이 파일을 직접 고치거나 다시 말하면 된다

## 7. 마무리

완료한 것과 건너뛴 것을 짧게 보고한다. 이어서 다음 단계를 안내한다.

- `/jobflow:intake`: 공고 스크린샷이나 URL 을 주면 자격을 판정해 트래커에 등록한다
- `/jobflow:remind`: 마감이 다가오는 건을 점검한다 (메일로 받으려면 플러그인 설정의 알림 주소와 Gmail 커넥터가 필요하다)
- `/jobflow:experience`: 경험을 카드로 정리한다
- `/jobflow:apply`: 지원서 자동입력. 쓰기 전에 `profile/master.yaml` 을 채워야 한다

## 금지

- 개인정보를 지어내서 채우지 않는다. 모르면 비워 둔다
- 이미 있는 노션 트래커나 설정 파일을 덮어쓰지 않는다. 확인 없이 새로 만들지 않는다
- 작업 폴더 밖에 개인 데이터를 쓰지 않는다
- 사용자 승인 없이 브라우저나 패키지를 설치하지 않는다

## 참고 문서

- `${CLAUDE_PLUGIN_ROOT}/reference/safety.md`
- `${CLAUDE_PLUGIN_ROOT}/reference/connectors.md`
- `${CLAUDE_PLUGIN_ROOT}/reference/notion-schema.md`
- `${CLAUDE_PLUGIN_ROOT}/reference/config-schema.md`
