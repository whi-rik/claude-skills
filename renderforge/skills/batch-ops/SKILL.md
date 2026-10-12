---
name: batch-ops
description: "RenderForge 배치 운영 스킬. VST 활성화/비활성화, 무음 재배치, unmatched 재매칭, 정규화 트리거, RMS 백필, 워커 리부트 등 운영 작업. 'VST 활성화', '재배치', '큐에 넣어', '리부트', '트리거', 'DB 업데이트' 등 요청 시 사용."
---

# batch-ops

RenderForge 렌더링 운영 작업을 수행한다.

## VST 활성화/비활성화

```python
# 1. 무음 기록 삭제
TB_350.objects.filter(vst_id__in=vst_ids).delete()
# 2. 비활성화 이력 삭제
TB_370.objects.filter(vst_id__in=vst_ids).delete()
# 3. 활성화
TB_210.objects.filter(vst_seq__in=vst_ids).update(is_active=True)
```

## unmatched 재매칭

```python
from apps.core.services.vst_matcher import match_vst

for job in qs.iterator(chunk_size=500):
    vst = match_vst(job.track)
    if not vst:
        continue  # 또는 같은 VST 유지
    job.vst = vst
    job.queue_type = vst.queue_type
    job.status = 'queued'
    job.meta_done = False
    updates.append(job)

TB_330.objects.bulk_update(updates, ['vst','queue_type','status','meta_done'], batch_size=500)
```

## 무음 재배치

API 호출 (Celery 태스크 위임):
```python
# 서버에서
curl -X POST http://localhost:8000/api/autobatch/retry-all-silent \
  -H "Content-Type: application/json" -d '{"limit": 5000}'
```

## 정규화 트리거

```bash
docker compose exec celery_worker uv run python scripts/trigger_normalize.py
```

## RMS 백필

```bash
# Celery 태스크 enqueue
docker compose exec celery_worker uv run python scripts/trigger_rms.py
# 또는 직접 워커
docker compose exec celery_worker uv run python rms_worker.py --batch-size 50 --loop
# Windows VM에서도 가능 (os.name 분기)
python rms_worker.py --batch-size 50 --loop
```

## 워커 리부트

```bash
# 전체
ssh sihan@192.168.100.150 "cd ~/workspace/RenderForge/deploy/ansible && ansible-playbook playbook-reboot.yml -i inventory.ini"
# 특정 워커만
ansible-playbook playbook-reboot.yml -i inventory.ini --limit 'RENDER-WIN-06,RENDER-WIN-07'
```

## 주의사항

- 로컬에서는 Redis 미연결로 Celery 태스크 enqueue 불가 → 서버에서 실행
- web 컨테이너는 NAS/gigamidi 볼륨이 read-only → 파일 쓰기 작업은 celery_worker에서
- `deactivate_silent_vsts` 비활성화됨 (수동 관리)
