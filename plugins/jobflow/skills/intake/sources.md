# 원본 공고 찾기와 수집 경로

`/jobflow:intake` 3단계의 상세다. 집계 사이트 링크에서 원본 ATS 를 복원하는 방법과, 기업이 공개한 ATS 엔드포인트를 읽는 패턴을 적는다.

## 1. 집계 사이트에서 원본 URL 복원

일부 집계 사이트는 WebFetch 같은 단순 요청에는 403 을 주지만, Playwright(실제 브라우저)로는 정상적으로 열린다. 이 경우 본문 하단 "지원하러 가기" 링크에 원본 ATS URL 이 들어 있다. 아래 스크립트를 `browser_evaluate` 로 실행하면 원본 URL 과 마감일을 한 번에 건진다.

```js
() => { const a=document.querySelector('article,.entry-content,main')||document.body;
  return {text:a.innerText.slice(0,3000),
          links:[...new Set([...a.querySelectorAll('a')].map(x=>x.href)
                 .filter(h=>h && !h.includes(location.hostname)))]}; }
```

주의:

- 사람이 브라우저로 여는 것과 같은 정상 접근만 쓴다. 봇 차단을 우회하려고 User-Agent 를 위조하거나 로그인 제한을 넘지 않는다. 막히면 사용자에게 스크린샷을 요청한다
- 집계글의 제목과 요약은 판정 근거가 아니다. 복원한 원본 ATS 를 열어서 판정한다
- 기업 채용 홈 메인이 백지면 채용 목록 경로가 다른 주소일 수 있다. 검색으로 채용 목록 URL 을 찾는다
- 검색으로 찾은 공고는 이미 마감된 것이 많다(캐시 지연). 원본을 열어 접수 기간을 재확인한다
- 샌드박스에서 국내 채용 사이트에 curl 이 타임아웃 나는 것은 사이트가 죽어서가 아닐 수 있다. Playwright 로 다시 확인한 뒤 판단한다

## 2. 새 도메인에 요청하기 전

robots.txt 전문을 받아 `ClaudeBot` 과 `User-agent: *` 규칙을 확인한다. 일부만 읽지 않는다(차단 선언이 관리형 블록 안쪽에 있는 경우가 있다). 차단 대상이거나 접근이 허용되지 않은 경로면 수집하지 않고 사용자에게 직접 열어 달라고 요청한다. 이 규칙은 자동 수집(intake)에 해당한다. 사용자 본인의 지원서를 대신 작성하는 일(apply)은 별개다.

## 3. ATS 공개 엔드포인트 패턴

기업이 직접 쓰는 ATS 는 공개 엔드포인트가 있는 경우가 있다. 응답 본문이 곧 공고 원문이라 그 자리에서 판정이 끝난다.

| ATS | 엔드포인트 | 비고 |
|---|---|---|
| NineHire | `{도메인}/sitemap.xml` 에서 `/job_posting/{id}` 를 찾아 상세 페이지의 `__NEXT_DATA__` 를 읽는다 | 목록 페이지는 클라이언트 렌더링이라 curl 로는 빈 페이지다 |
| Greenhouse | `boards-api.greenhouse.io/v1/boards/{토큰}/jobs?content=true` | `gh_jid` 가 주소에 보이면 이 방식 |
| Lever | `api.lever.co/v0/postings/{토큰}?mode=json` | |
| Ashby | `api.ashbyhq.com/posting-api/job-board/{토큰}` | |
| Workable | `apply.workable.com/api/v1/widget/accounts/{토큰}` | |
| Workday | POST `/wday/cxs/{회사}/{사이트}/jobs`, body `{"searchText":"Seoul"}` | |

## 4. 읽을 때의 주의

- 본문에 연차 숫자가 없어도 ATS 구조화 필드(경력 유형, 최소/최대 연차)에 있을 수 있다. 필드와 본문을 둘 다 인용하고 더 엄격한 쪽으로 판정한다
- 자격요건과 우대사항을 구분해서 읽는다. "N년 이상"이 우대사항에 있는 경우가 있다
- 같은 리소스를 두 번 조회해 값이 바뀌면 마스킹이 아니라 요청 제한(스로틀)일 수 있다. 상세 페이지는 한 세션에 소량만, 저속으로 연다. 목록에서 먼저 후보를 좁힌 뒤 상세를 연다. 가려진 것처럼 보이는 응답은 시간을 두고 재조회해 대조한다
- sitemap 의 `lastmod` 는 개별 공고 수정 시각이 아닐 수 있다. 신규 판별은 공고 ID 집합을 저장해 두고 차집합으로 한다
- 직무명과 태그는 근거가 아니다. 주요 업무와 자격요건 본문을 읽는다
