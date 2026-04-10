---
name: whbo-ops
description: "WHBO 백오피스 운영 오케스트레이터. 버그 수정, 기능 추가, 릴리즈, 서버 로그 분석, DB 조작, Slack 공지 등 모든 백오피스 운영 작업을 담당. request_id 조회, 에러 로그 확인, 코드 수정, 릴리즈 플로우, 서버 배포, 사용자 DM 발송 요청 시 반드시 이 스킬을 사용할 것."
---

# WHBO 백오피스 운영 오케스트레이터

## 요청 분류 및 라우팅

사용자 요청을 분석하여 적절한 에이전트로 라우팅한다.

### 분류표

| 키워드/패턴 | 에이전트 | 설명 |
|------------|----------|------|
| request_id, 로그 확인, 에러 파악 | server-ops | 서버 로그 분석 |
| DB 삭제, 권한 부여/제거, 데이터 확인 | server-ops | DB 조작 |
| 백엔드 수정, API 추가, 모델 변경 | backend-dev | 코드 수정 |
| 프론트 수정, 페이지 추가, UI 변경 | frontend-dev | 프론트 수정 |
| 릴리즈, 태그, 서브모듈, push | release-ops | 릴리즈 관리 |
| DM 보내, 공지, Slack | notifier | 사용자 알림 |
| WRBO-N 처리, Jira 이슈, 이슈 해결 | jira-resolve | Jira 이슈 해결 사이클 |
| 문서 동기화, sync-docs | doc-sync | 스킬 실행 |

### 복합 요청 패턴

**Jira 이슈 해결 풀 사이클:**
1. Jira 이슈 읽기 (제목, 설명, 첨부, 댓글)
2. 상태 → "진행 중"
3. 코드 분석/수정 (backend-dev / frontend-dev)
4. 커밋 + 릴리즈 (release-ops)
5. Jira에 해결 댓글 + 유사 이슈 검토 댓글
6. 상태 → "완료"

**버그 수정 풀 사이클:**
1. server-ops: 로그 분석 → 원인 파악
2. backend-dev / frontend-dev: 코드 수정
3. release-ops: 커밋 → push → 릴리즈 → 서브모듈 업데이트
4. 사용자에게 배포 안내

**기능 추가:**
1. backend-dev + frontend-dev: 병렬 구현
2. release-ops: 릴리즈
3. 배포 안내

**서버 점검:**
1. notifier: 점검 시작 공지 DM
2. server-ops: 서버 작업
3. notifier: 점검 완료 공지 DM

## 실행 방식

### 단일 에이전트 작업
서브 에이전트로 실행. 빠르고 효율적.

```
Agent(subagent_type="{에이전트}", model="opus", prompt="...")
```

### 복합 작업 (2개 이상 에이전트)
에이전트 팀으로 실행. 팀원 간 직접 통신.

```
TeamCreate(members=[...])
TaskCreate(tasks=[...])
```

## 데이터 전달

- 에이전트 간: 파일 기반 (`_workspace/` 디렉토리)
- 파일명: `{phase}_{agent}_{artifact}.{ext}`
- 최종 산출물만 프로젝트 경로에 출력

## 에러 핸들링

- 코드 수정 실패 → 에러 메시지 사용자에게 보고
- 릴리즈 실패 → 수동 가이드 제공
- SSH 접속 실패 → 재시도 1회 후 보고
- Slack DM 실패 → 실패 사용자 목록 보고

## 테스트 시나리오

### 정상 흐름: 버그 수정
```
사용자: "request_id abc123 에러 확인해"
→ server-ops 로그 분석
→ backend-dev 코드 수정
→ release-ops 릴리즈
→ 배포 안내
```

### 에러 흐름: SSH 접속 불가
```
사용자: "서버 로그 확인"
→ server-ops SSH 시도 → 실패
→ "서버 접속이 안 됩니다. 네트워크 확인 필요합니다." 보고
```
