---
name: jira-resolve
description: "Jira 이슈 해결 사이클. 'WRBO-N 처리해', 'Jira 이슈 해결', '이슈 읽고 수정해' 등 Jira 이슈 기반 작업 시 반드시 사용. 이슈 읽기 → 코드 분석/수정 → 커밋 → Jira에 해결 댓글 작성 + 유사 이슈 검토 + 상태 변경까지 end-to-end 실행."
---

# Jira 이슈 해결 사이클

## 워크플로우

### Phase 1: 이슈 읽기
Jira API로 이슈 상세 조회:
```bash
curl -s -u "{JIRA_EMAIL}:{JIRA_API_TOKEN}" \
  "https://whirik.atlassian.net/rest/api/3/issue/{이슈키}?fields=summary,description,comment,attachment,status,priority"
```

추출할 정보:
- 제목 (summary)
- 설명 (description) — 페이지 URL, CSS 셀렉터, 좌표, 보고자
- 첨부파일 (attachment) — 스크린샷 파일명
- 기존 댓글 (comment) — 이미 분석된 내용 확인
- 상태/우선도

### Phase 1.5: 이슈 분류 + 담당자 자동 할당

이슈 내용(제목, 설명, 페이지 URL)을 분석하여 프론트/백엔드를 판별하고 담당자를 할당한다.

**분류 기준:**

| 키워드/패턴 | 분류 | 담당자 |
|------------|------|--------|
| UI, 화면, 버튼, 표시, 레이아웃, 컬럼, 필터, 소팅 | 프론트 | 조창현 |
| 페이지 URL만 있고 서버 에러 없음 | 프론트 | 조창현 |
| API 에러, 500, 403, DB, 권한, 서버 | 백엔드 | 손계원 |
| request_id, 로그 에러 | 백엔드 | 손계원 |
| 프론트+백엔드 모두 필요 | 백엔드 | 손계원 (프론트 작업은 별도 요청) |

**Jira 담당자 ID:**
- 손계원 (백엔드): `712020:4408e3ac-e473-442c-8c29-e5e2c8b6e281`
- 조창현 (프론트): `712020:da49cb99-bfb1-42e6-aabe-ed400d083d32`

**할당 API:**
```bash
curl -s -X PUT -u "{email}:{token}" -H "Content-Type: application/json" \
  -d '{"accountId":"{담당자_accountId}"}' \
  "https://whirik.atlassian.net/rest/api/3/issue/{이슈키}/assignee"
```

### Phase 2: 상태 → "진행 중" 변경
```bash
curl -s -X POST -u "{email}:{token}" -H "Content-Type: application/json" \
  -d '{"transition":{"id":"21"}}' \
  "https://whirik.atlassian.net/rest/api/3/issue/{이슈키}/transitions"
```

전이 ID:
- 11: 해야 할 일
- 21: 진행 중
- 2: 종료
- 31: 완료

### Phase 3: 원인 분석 + 코드 수정
1. 설명의 페이지 URL/CSS 셀렉터로 관련 코드 위치 파악
2. 서버 로그 확인 (에러 이슈인 경우)
3. 코드 수정 (backend-dev / frontend-dev)
4. 커밋 + push

### Phase 4: 해결 댓글 작성
수정 완료 후 Jira에 댓글을 남긴다. 댓글 구조:

```
[Claude] 해결 완료

커밋: {커밋 메시지 제목}
브랜치: deploy
릴리즈: v{버전} (해당 시)

수정 내용:
- {변경 파일 1}: {변경 설명}
- {변경 파일 2}: {변경 설명}

해결 방법:
{간단한 해결 방법 설명 2~3줄}
```

API 호출:
```bash
curl -s -X POST -u "{email}:{token}" -H "Content-Type: application/json" \
  -d '{
    "body": {
      "type": "doc", "version": 1,
      "content": [
        {"type": "heading", "attrs": {"level": 3},
         "content": [{"type": "text", "text": "[Claude] 해결 완료"}]},
        {"type": "paragraph",
         "content": [{"type": "text", "text": "{해결 내용}"}]}
      ]
    }
  }' \
  "https://whirik.atlassian.net/rest/api/3/issue/{이슈키}/comment"
```

### Phase 5: 유사 이슈 검토
같은 프로젝트에서 유사한 이슈가 있는지 JQL로 검색:
```bash
curl -s -u "{email}:{token}" \
  "https://whirik.atlassian.net/rest/api/3/search/jql?jql=project=WRBO+AND+text~\"{키워드}\"+ORDER+BY+created+DESC&maxResults=5&fields=summary,status"
```

유사 이슈가 있으면 추가 댓글:
```
[Claude] 유사 이슈 검토

관련 가능성 있는 이슈:
- WRBO-N: {제목} (상태: {상태}) — {관련성 설명}
- WRBO-M: {제목} (상태: {상태}) — {관련성 설명}

동일 원인인 경우 함께 해결되었을 수 있습니다.
```

### Phase 6: 상태 → "완료" 변경
```bash
curl -s -X POST -u "{email}:{token}" -H "Content-Type: application/json" \
  -d '{"transition":{"id":"31"}}' \
  "https://whirik.atlassian.net/rest/api/3/issue/{이슈키}/transitions"
```

## Jira 인증 정보
- URL: https://whirik.atlassian.net
- Email: sihan@whirik.com
- API Token: 환경변수 또는 .env에서 JIRA_API_TOKEN
- 프로젝트 키: WRBO

## 주의사항
- 댓글은 ADF(Atlassian Document Format) 형식으로 작성
- 이슈 상태 변경 전 현재 상태 확인 (이미 완료면 스킵)
- 코드 수정 없이 분석만 하는 경우 댓글만 남기고 상태는 변경하지 않음
- 유사 이슈 검색 키워드는 제목에서 핵심 명사 2~3개 추출
