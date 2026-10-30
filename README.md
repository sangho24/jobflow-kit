# jobflow-kit

취업 준비를 도와주는 Claude Code 플러그인 모음입니다. 공고를 노션에 정리하고, 내 자격에 맞는지 판정하고, 마감이 다가오면 알려 주고, 경험을 정리하고, 지원서 폼을 대신 채워 임시저장까지 해 줍니다. 제출 버튼은 항상 내가 직접 누릅니다.

## 이 플러그인이 하는 일

- 공고 정리: 공고 스크린샷, URL, 기업명과 직무를 주면 마감일, 원본 지원 URL, 지원 시스템(ATS)을 확인해 노션 트래커에 등록합니다
- 자격 판정: 경력, 학위, 졸업, 전공, 어학, 해외·비자 6가지 축을 공고 원문 문장과 대조해 지원 가능 여부를 가립니다
- 마감 알림: 마감 7일, 3일, 1일 전과 당일 건을 모아 알려 주고, 원하면 Gmail 로 보냅니다
- 직무 적합도: 공고의 직무 요건을 읽고 내 프로필과 맞춰 어느 직무에 지원할지 순위를 매깁니다
- 경험 정리: 질문을 받으며 경험을 카드로 정리해 자소서의 원재료로 쌓습니다
- 지원서 자동입력: 내 프로필로 지원서 폼을 채우고 임시저장한 뒤, 사람이 정해야 할 항목을 노션에 남깁니다

## 미리 필요한 것

- Claude Code 와 유료 플랜
- claude.ai 에서 노션 커넥터 켜기 (설정 > 커넥터 > Notion 연결). 트래커가 노션에 만들어지므로 필수입니다
- 마감 알림을 메일로 받으려면 Gmail 커넥터도 켭니다 (선택)
- Node.js (`npx` 가 동작해야 합니다). 지원서 자동입력에 쓰는 Playwright 서버가 npx 로 실행됩니다

## 설치

터미널에서 순서대로 실행합니다.

```bash
claude plugin marketplace add sangho24/jobflow-kit
claude plugin install jobflow@jobflow-kit
```

Claude Code 세션 안에서 설치하려면 다음처럼 합니다.

```
/plugin marketplace add sangho24/jobflow-kit
/plugin install jobflow@jobflow-kit
```

설치 중 git clone 단계에서 SSH 오류가 나면 환경변수 `CLAUDE_CODE_PLUGIN_PREFER_HTTPS=1` 을 지정해 HTTPS 로 받게 합니다.

```bash
export CLAUDE_CODE_PLUGIN_PREFER_HTTPS=1
```

설치할 때 작업 폴더(기본 `~/jobflow-data`)와 알림 받을 메일 주소(선택)를 물어봅니다. 작업 폴더는 홈 바로 아래를 권장합니다. macOS 에서 Desktop, Documents, Downloads 아래에 두면 권한 문제로 파일을 못 읽는 일이 생깁니다.

## 처음 할 일

세션에서 한 번만 실행합니다.

```
/jobflow:setup
```

작업 폴더를 만들고, 노션에 `📡 취업 준비 트래커` 페이지와 DB 를 만들고, 내 자격 정보와 관심 직무를 대화로 정해 설정 파일에 저장합니다. 개인정보를 전부 넣을 필요는 없습니다. 지원서 자동입력을 쓰기 전까지만 채우면 됩니다. 다시 실행해도 이미 있는 것은 건너뛰고 빠진 것만 채웁니다.

## 스킬 목록

| 스킬 | 하는 일 | 예시 요청 |
|---|---|---|
| `/jobflow:setup` | 작업 폴더, 노션 트래커, 프로필 첫 설정 | "jobflow 처음 설정해 줘" |
| `/jobflow:intake` | 공고를 자격 판정 후 트래커에 등록 | "이 공고 넣어 줘", 공고 스크린샷 첨부 |
| `/jobflow:fit` | 공고 직무 요건과 내 프로필 대조, 순위 | "어느 직무 쓸까", "JD 분석해 줘" |
| `/jobflow:experience` | 경험을 캐물어 경험 카드로 정리 | "경험 정리하자", "이 프로젝트 정리해 줘" |
| `/jobflow:remind` | 마감 임박 건 점검과 메일 알림 | "마감 확인", "뭐 급한 거 있어?" |
| `/jobflow:apply` | 지원서 폼 자동입력과 임시저장 | "지원서 채워 줘" |

## 업데이트

```bash
claude plugin marketplace update jobflow-kit
claude plugin update jobflow@jobflow-kit
```

그다음 열려 있는 세션에서 `/reload-plugins` 를 실행하면 바로 반영됩니다.

## 개인정보 원칙

- 내 데이터는 내 컴퓨터의 작업 폴더와 내 노션에만 쌓입니다. 플러그인 제작자에게도, 다른 곳에도 보내지 않습니다
- 지원서 제출은 항상 내가 합니다. 자동입력은 임시저장까지만 합니다
- 비밀번호는 파일, 노션, 메일에 적지 않습니다. macOS 키체인에만 둡니다
- 본인인증, 캡차, 2단계 인증은 우회하지 않고 내게 넘깁니다
- 자세한 원칙은 `plugins/jobflow/reference/safety.md` 에 있습니다

## 문제 해결

**설치할 때 저장소를 못 찾는다고 나옵니다.**
명령의 저장소 이름(`sangho24/jobflow-kit`)에 오타가 없는지, 인터넷에 연결돼 있는지 확인합니다. `git --version` 으로 git 이 설치돼 있는지도 봅니다.

**git clone 단계에서 SSH 오류가 납니다.**
`CLAUDE_CODE_PLUGIN_PREFER_HTTPS=1` 을 지정하고 다시 설치합니다.

**노션 도구가 안 보인다고 합니다.**
claude.ai 설정 > 커넥터에서 Notion 이 켜져 있는지 확인하고, 새 세션을 엽니다. 연결 후에도 안 보이면 `/mcp` 로 연결 상태를 봅니다.

**Playwright 가 실행되지 않습니다.**
`node -v` 와 `npx -v` 가 동작하는지 확인합니다. 브라우저 설치가 필요하다는 안내가 나오면 안내대로 설치한 뒤 세션을 다시 엽니다.

**작업 폴더의 파일을 읽지 못합니다.**
macOS 에서 Desktop, Documents, Downloads 아래에 두면 권한 문제가 납니다. 홈 바로 아래(`~/jobflow-data`)로 옮기고 설정에서 경로를 바꿉니다.

**업데이트했는데 그대로입니다.**
`claude plugin marketplace update jobflow-kit` 를 먼저 실행해야 새 버전 정보를 받습니다. 그다음 `claude plugin update jobflow@jobflow-kit` 와 `/reload-plugins` 를 실행합니다.

## 라이선스

MIT. 자유롭게 쓰고 고쳐 쓸 수 있습니다. 자세한 내용은 `LICENSE` 를 봅니다.
