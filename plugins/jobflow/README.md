# jobflow 플러그인

취업 준비 흐름(공고 수집, 자격 판정, 마감 관리, 경험 정리, 지원서 입력)을 Claude Code 안에서 돕는 플러그인입니다. 설치와 사용법은 저장소 루트의 `README.md` 를 보세요.

## 스킬

- `/jobflow:setup`: 작업 폴더, 노션 트래커, 설정 파일, 프로필 첫 설정
- `/jobflow:intake`: 공고를 6축으로 자격 판정한 뒤 노션 트래커에 등록
- `/jobflow:fit`: 공고 직무 요건과 마스터 프로필을 대조해 지원 직무 순위 산정
- `/jobflow:experience`: 질문을 통해 경험 카드 작성
- `/jobflow:remind`: 마감 D-7, D-3, D-1, 당일 건 점검과 Gmail 알림
- `/jobflow:apply`: Playwright 로 지원서 폼 조사, 자동입력, 임시저장 (제출은 하지 않음)

## 구성

- `skills/`: 각 스킬 정의
- `reference/`: 여러 스킬이 함께 보는 정본 문서 (안전 원칙, 설정 형식, 노션 구조, 커넥터 확인법)
- `templates/`: 작업 폴더에 복사되는 빈 양식 (프로필, 설정, 수집 기준)
- `.mcp.json`: Playwright MCP 서버 묶음

## 필요한 연결

노션 커넥터(필수), Gmail 커넥터(선택), Node.js(`npx`). 자세한 확인법은 `reference/connectors.md`.

## 데이터

개인 데이터는 플러그인이 아니라 사용자 작업 폴더(`workspace_dir`)와 사용자 노션에만 저장됩니다.

## 설치

```
/plugin marketplace add sangho24/jobflow-kit
/plugin install jobflow@jobflow-kit
```

자세한 준비물과 설치 순서는 저장소 루트의 `README.md` 를 본다.
