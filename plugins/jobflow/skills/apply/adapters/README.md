# ATS 어댑터

URL 패턴으로 ATS 를 판별하고 폼 구조와 함정을 기술한다. 모든 파일은 1인 실측이라 사이트가 개편되면 달라질 수 있다. 각 파일 맨 위의 `verified:` 주석으로 확인 시점을 본다.

`status: unverified` 는 아직 실제로 열어 보지 않았다는 뜻이다. `/jobflow:apply` 의 조사 단계를 한 번 돌린 뒤 `verified` 로 바꾸고 `field_hints` 를 채운다.

`maps_to` 는 작업 폴더의 `profile/master.yaml` 키 경로다. 어학은 `languages[test=OPIc].level` 처럼 목록에서 시험명으로 항목을 고른다.

## 범용 ATS

| 어댑터 | 임시저장 | 계정 | 비고 |
|---|---|---|---|
| `recruiter_co_kr` | 있음 | 공고별 계정 + 이메일 인증 | 국내 대기업 다수. 학점 100점 환산 요구 |
| `greenhouse` | 없음 | 불필요 | 기입안만 보고 |
| `greeting` | 있음 | 계정 필요 | 커스텀 콤보박스 위젯 |
| `lever` | 없음 | 불필요 | 기입안만 보고 |
| `careerlink` | 있음 | 회사별 계정 | MUI 기반 SPA |
| `incruit` | 있음 | 상황별 | 실측 1건, 팝업이 폼 초기화 |

미등록 ATS(Workday, Oracle HCM, 자체 구축 등)는 조사 단계에서 새 어댑터를 만든다. 자체 구축은 재사용성이 낮아 개별 대응한다.

## 노션 `ATS` 선택지와 어댑터 대응

| 노션 `ATS` | 어댑터 |
|---|---|
| recruiter.co.kr | `recruiter_co_kr.yaml` |
| Greenhouse | `greenhouse.yaml` |
| Lever | `lever.yaml` |
| Greeting | `greeting.yaml` |
| careerlink | `careerlink.yaml` |
| 인크루트 | `incruit.yaml` |
| Workday | 어댑터 없음 (조사부터) |
| 자체구축 | 어댑터 없음 (조사부터). 회사 전용은 `examples/` 참고 |
| 미확인 | 어댑터 없음 (조사부터) |

## 회사 전용 예시 (`examples/`)

`toss.yaml`, `shinsegae.yaml`, `sk_careers.yaml`, `lg_careers.yaml` 은 특정 회사 사이트용 어댑터다. 새 어댑터를 만들 때 구조와 서술 수준을 참고하는 용도다.
