# RenderForge 스킬 · 에이전트

[RenderForge](https://github.com/whi-rik/RenderForge)(VST 렌더팜) 작업용 Claude Code 스킬 6개와 에이전트 5개. RenderForge 리포 `develop` 브랜치의 `.claude/` 에서 옮겼다.

## 설치

RenderForge 체크아웃 안에 그대로 둔다(프로젝트 스킬).

```bash
mkdir -p <RenderForge>/.claude
cp -r renderforge/skills renderforge/agents <RenderForge>/.claude/
```

## 구성

| 스킬 | 하는 일 |
|---|---|
| `rf-orchestrator` | 요청을 분석해 아래 전문 에이전트에 넘기는 라우터. 모든 RenderForge 요청의 입구 |
| `batch-ops` | 운영 작업 — VST 활성화/비활성화, 무음 재배치, unmatched 재매칭, 정규화 트리거, RMS 백필, 워커 리부트 |
| `render-analysis` | 데이터 분석 — 무음률, RMS, stem 구성, VST 성능, 워커별 통계, GM 커버리지, 보고서 |
| `drf-patterns` | 백엔드 개발 — Django API 뷰, Celery 태스크, 서비스 로직, 마이그레이션 |
| `dashboard-dev` | React 대시보드 — 페이지·컴포넌트·API 연동 |
| `deploy-ops` | 인프라·배포 — Docker 빌드, Ansible 배포, 워커 관리, NAS 설정 |

| 에이전트 | 맡는 일 |
|---|---|
| `rf-analyst` | 분석·보고 |
| `rf-backend` | Django/Celery |
| `rf-frontend` | 대시보드 |
| `rf-infra` | Docker·Ansible·VM |
| `rf-ops` | 배치 운영 |

## 주의

- 서버·나스 주소와 경로는 2026-10 사내망 기준이다. 바뀌면 스킬 본문도 같이 고친다.
- 운영 작업(리부트·재배치·DB 갱신)은 실제 서비스에 바로 닿는다. 운영 매뉴얼의 확인 절차를 먼저 따른다.
