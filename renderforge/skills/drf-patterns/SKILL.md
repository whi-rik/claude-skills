---
name: drf-patterns
description: "RenderForge Django/Celery 백엔드 개발 스킬. API 뷰 추가, Celery 태스크 작성, 서비스 로직 구현, 마이그레이션 생성. Django 코드 수정, 'API 추가', '태스크 만들어', '모델 변경', '마이그레이션' 등 요청 시 사용."
---

# drf-patterns

RenderForge Django REST Framework + Celery 개발 패턴.

## 응답 엔벨로프

모든 API 응답은 표준 엔벨로프 사용:
```python
from apps.common.utils.decorators.responseForm.response import ok, fail
return ok(code=RenderCode.XXX, data=result, request=request)
```

## API 뷰 패턴

```python
class MyView(APIView):
    @api_response(
        responses={200: {"serializer": MySerializer, "msg": "설명", "code": RenderCode.XXX, "result": True, "name": "MyResponse"}},
        operation_id="IF_RF_XX_001",
        description="설명",
        tags="domain",
    )
    def post(self, request):
        ...
```

## Celery 태스크 패턴

```python
# apps/core/tasks/xxx.py
@shared_task(name="renderforge.task_name")
def task_name() -> dict:
    ...
    return {"ok": True, ...}
```

Beat 스케줄 등록: `config/settings/base.py` CELERY_BEAT_SCHEDULE

## 서비스 ↔ 태스크 분리

- `apps/core/services/` — 비즈니스 로직 (동기)
- `apps/core/tasks/` — Celery 태스크 (`.delay()` 위임)
- 뷰에서 파일 I/O 필요 시 → Celery 태스크로 위임 (web 컨테이너 볼륨 제한)

## 파일 구조

```
apps/core/
├── models/models.py    — TB_200~TB_400
├── views/              — 도메인별 APIView
├── serializers/        — 도메인별 Serializer
├── tasks/              — Celery 태스크
├── services/           — 비즈니스 로직
└── admin_site.py
```

## 규칙

- Python 3.14+, uv 사용
- 린트: `uv run ruff check .` / `uv run ruff format .`
- 커밋 시 Co-Authored-By 포함
- 한국어 주석/admin 레이블 정상
