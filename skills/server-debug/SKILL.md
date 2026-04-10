---
name: server-debug
description: "서버 로그 분석, DB 조작, 권한 관리, 데이터 확인/삭제. request_id로 로그 검색, 에러 원인 파악, Django shell로 DB 조회/수정/삭제, 사용자 권한 확인/부여, 케이스 하위 데이터 정리 등. 서버 관련 읽기/조회 작업 시 반드시 사용."
---

# 서버 디버그 & DB 관리

## SSH 접속
```bash
ssh -i ~/.ssh/id_renderforge whirik@192.168.100.15
```

## Docker compose 경로
```
~/workspace/backoffice/stage/whbo-deploy/docker-compose.yml
```

## 로그 분석 패턴

### request_id로 검색
```bash
docker compose -f {경로} logs backend 2>&1 | grep '{request_id}'
```

### 최근 에러만
```bash
docker compose -f {경로} logs backend --since {N}m 2>&1 | grep -E 'status_code..(400|403|500)'
```

### Traceback 확인
```bash
docker compose -f {경로} logs backend 2>&1 | grep -A 25 '{시간대}.*Unhandled exception'
```

### 특정 사용자 로그
```bash
docker compose -f {경로} logs backend 2>&1 | grep '"account_id":"{user_seq}"'
```

## Django Shell 실행
```bash
docker compose -f {경로} exec -T backend uv run python manage.py shell -c "..."
```

## DB 조작 패턴

### 사용자/권한 조회
```python
from apps.core.models import TB_110, TB_131
from apps.permissions.models import TB_190

for u in TB_110.objects.all():
    perms = TB_190.objects.filter(subject_id=u.user_ulid)
    teams = TB_131.objects.filter(user=u).select_related('team')
```

### 권한 부여
```python
from apps.permissions.services.writer import upsert_tuple
from apps.permissions.constants import ObjectType

upsert_tuple(
    object_type=ObjectType.ORG, object_id='whbo',
    relation='c_level',  # 또는 'team_leader'
    subject_type=ObjectType.USER, subject_id='{user_ulid}',
    created_by='system',
)
```

### 케이스 + 하위 데이터 삭제
```python
from apps.core.models.production import TB_310, TB_320, TB_330, TB_331, TB_334, TB_332, TB_333, TB_340, TB_341, TB_342, TB_390
from apps.permissions.models import TB_190

case = TB_310.objects.get(case_ulid='{ulid}')
units = TB_320.objects.filter(case_seq=case)
for u in units:
    tasks = TB_330.objects.filter(unit_seq=u)
    for t in tasks:
        subs = TB_331.objects.filter(task_seq=t)
        for s in subs:
            TB_390.objects.filter(task_sub_seq=s).delete()
            TB_334.objects.filter(task_sub_seq=s).delete()
            TB_332.objects.filter(task_sub_seq=s).delete()
            TB_333.objects.filter(task_sub_seq=s).delete()
        subs.delete()
        TB_190.objects.filter(object_type='task', object_id=t.task_ulid).delete()
    tasks.delete()
    TB_190.objects.filter(object_type='unit', object_id=u.unit_ulid).delete()
TB_342.objects.filter(meeting_seq__case_seq=case).delete()
TB_341.objects.filter(meeting_seq__case_seq=case).delete()
TB_340.objects.filter(case_seq=case).delete()
units.delete()
TB_190.objects.filter(object_type='case', object_id=case.case_ulid).delete()
case.delete()
```

## 주의사항
- 서버 코드 직접 수정 금지 — 읽기만
- Redis 권한 캐시 문제 시: `perm_obj:*` 키 삭제
- timeout 항상 설정 (기본 15000ms, DB 작업 30000ms)
