---
name: deploy-ops
description: "RenderForge 인프라/배포 스킬. Docker 빌드, Ansible 배포, 워커 관리, NAS 설정. 'docker', '배포', 'ansible', '빌드', '컨테이너', 'VM' 등 요청 시 사용."
---

# deploy-ops

RenderForge 인프라 및 배포 관리.

## Docker

```bash
# 특정 서비스만 빌드/실행
docker compose up -d --build celery_worker celery_beat web
# 로그 확인
docker compose logs celery_worker --tail=50
# 캐시 정리
docker system prune -f && docker builder prune -f
```

서비스: web, celery_worker, celery_beat, redis, flower

볼륨 주의:
- web: NAS **ro**, gigamidi 없음
- celery_worker: NAS **rw**, gigamidi **rw**

## Ansible

```bash
cd deploy/ansible/
# 전체 리부트
ansible-playbook playbook-reboot.yml -i inventory.ini
# 특정 워커만
ansible-playbook playbook-reboot.yml -i inventory.ini --limit 'RENDER-WIN-06,RENDER-WIN-07'
# 워커 업데이트 배포
ansible-playbook playbook-update.yml -i inventory.ini
# config 재배포
ansible-playbook playbook-restart.yml -i inventory.ini
```

## 워커 빌드

WIN-01에서:
```powershell
powershell -ExecutionPolicy Bypass -File scripts\build.ps1
```
빌드 결과: `\\192.168.100.203\Public\renderforge\build\dist\renderforge`

## 인프라 구성

- API 서버: 192.168.100.150
- DB: 192.168.100.15 (PostgreSQL)
- Windows VM: WIN-01~12 (192.168.100.151~162)
- NAS: 201~203 (SMB)
- 큐 할당: WIN-01~05 Kontakt, WIN-06~10 Spectrasonics

## NAS 인증

- `nas-auth.bat` 반드시 ASCII 인코딩
- `net use \\host\share /user:xxx pwd`로 매핑
- NSSM: `start-worker.bat` → `nas-auth.bat` → `renderforge.exe render`
