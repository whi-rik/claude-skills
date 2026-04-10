---
name: release-flow
description: "백오피스 릴리즈 플로우 실행. '릴리즈해', '릴리즈까지', '태그 고정', '서브모듈 업데이트', 'push해서 릴리즈', 'deploy→main 머지' 등 릴리즈/배포 관련 요청 시 반드시 사용. 백엔드(backoffice-server)와 프론트(Backoffice-Frontend) 모두 지원."
---

# 릴리즈 플로우

## 백엔드 릴리즈 (backoffice-server)

### 1단계: 커밋 + Push (deploy)
```bash
cd /Users/sihan/projects/Backoffice/whbo-deploy/backoffice-server
git checkout deploy
git add {변경파일들}
git commit -m "{커밋메시지}

Co-Authored-By: Claude Opus 4.6 (1M context) <noreply@anthropic.com>"
git push origin deploy
```

### 2단계: deploy → develop 머지
```bash
git checkout develop && git pull origin develop
git merge deploy --no-edit
git push origin develop
```
충돌 시 deploy 쪽을 채택하여 해결.

### 3단계: develop → main PR + 머지
```bash
gh pr create --repo whi-rik/backoffice-server --base main --head develop \
  --title "develop → main v{버전}" --body "{변경사항}"
gh pr merge {PR번호} --repo whi-rik/backoffice-server --merge --delete-branch=false
```

### 4단계: 릴리즈 생성
```bash
gh release create v{버전} --repo whi-rik/backoffice-server --target main \
  --title "v{버전}" --notes "{릴리즈 노트}"
```

### 5단계: 서브모듈 태그 고정
```bash
git fetch origin --tags && git checkout v{버전}
cd /Users/sihan/projects/Backoffice/whbo-deploy
git add backoffice-server
git commit -m "update: backoffice-server v{버전} 태그 고정

Co-Authored-By: Claude Opus 4.6 (1M context) <noreply@anthropic.com>"
git push origin deploy
```

## 프론트 릴리즈 (Backoffice-Frontend)

deploy → main PR → 머지 → 릴리즈 → 서브모듈 태그 고정.
백엔드와 동일한 패턴, 레포만 다름 (`whi-rik/Backoffice-Frontend`).

## 배포 안내 출력

릴리즈 완료 후 서버 배포 명령어를 제공:
```bash
cd ~/workspace/backoffice/stage/whbo-deploy
git pull origin deploy
cd backoffice-server
find . -name '*.pyc' -delete
find . -name '__pycache__' -type d -exec rm -rf {} + 2>/dev/null
git fetch origin
git checkout v{버전}
cd ..
docker compose down backend
docker rmi whbo-deploy-backend
docker compose up -d --build backend
docker compose exec backend uv run python manage.py migrate  # 마이그레이션 있을 때
```

## 주의사항
- detached HEAD에서 커밋하면 deploy 브랜치로 cherry-pick 필요
- 서버 코드 직접 수정 금지 — push 후 사용자가 pull
- Docker 캐시 문제 시 `docker rmi` 후 `--build`
