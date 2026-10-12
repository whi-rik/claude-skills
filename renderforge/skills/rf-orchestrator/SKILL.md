---
name: rf-orchestrator
description: "RenderForge 오케스트레이터. 사용자 요청을 분석하여 적절한 전문 에이전트에 위임. '분석해줘', '코드 수정해', 'DB 업데이트', '대시보드 수정', '배포해', '리부트해' 등 모든 RenderForge 작업 요청 시 이 스킬로 라우팅."
---

# rf-orchestrator

사용자 요청을 분석하여 적절한 에이전트에 위임하는 오케스트레이터.

## 에이전트 라우팅

| 키워드/상황 | 에이전트 | 스킬 |
|------------|---------|------|
| 무음 분석, 통계, 보고서, 현황, RMS, 수량 확인 | **rf-analyst** | render-analysis |
| VST 활성화, 재배치, 큐잉, 트리거, DB 조작, 리부트 | **rf-ops** | batch-ops |
| API 추가, 태스크 만들어, 모델 변경, 마이그레이션, 백엔드 코드 | **rf-backend** | drf-patterns |
| 대시보드, 버튼 추가, 페이지, 프론트, UI | **rf-frontend** | dashboard-dev |
| Docker, 배포, Ansible, 빌드, 컨테이너, VM 설정 | **rf-infra** | deploy-ops |

## 복합 작업 처리

하나의 요청이 여러 에이전트에 걸칠 때:

### 백엔드 + 프론트엔드 (가장 빈번)
예: "무음 재시도에 limit 파라미터 추가하고 대시보드에 버튼도"
1. rf-backend → API/태스크 수정
2. rf-frontend → 대시보드 UI 수정
3. 순차 실행 (프론트가 API 스펙에 의존)

### 분석 + 운영
예: "무음률 확인하고 높은 VST 비활성화"
1. rf-analyst → 데이터 분석
2. rf-ops → 분석 결과 기반 DB 조작
3. 순차 실행 (운영이 분석 결과에 의존)

### 백엔드 + 인프라
예: "스케줄 태스크 추가하고 서버 배포"
1. rf-backend → 코드 수정, 커밋 푸시
2. rf-infra → 서버 배포
3. 순차 실행

## 에러 핸들링

- 에이전트 실패 시 결과 보고 후 사용자에게 판단 위임
- 로컬 Redis 미연결 → rf-ops가 서버 직접 실행 안내
- web 컨테이너 볼륨 제한 → rf-backend가 Celery 태스크로 위임

## 테스트 시나리오

### 정상 흐름
1. "무음 현황 확인해줘" → rf-analyst 호출 → 보고서 출력
2. "Fretless Bass 활성화하고 재매칭" → rf-ops 호출 → DB 업데이트
3. "대시보드에 1000개 버튼 추가" → rf-frontend 호출 → TSX 수정

### 복합 흐름
1. "무음 높은 VST 분석하고 비활성화해" → rf-analyst → rf-ops 순차
2. "Celery 태스크 추가하고 대시보드에 버튼도" → rf-backend → rf-frontend 순차
