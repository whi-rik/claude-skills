---
name: bug-fix-flow
description: "버그 수정 전체 사이클. request_id나 에러 로그가 주어지면 서버 로그 분석 → 원인 파악 → 코드 수정 → 커밋/push → 릴리즈 → 배포 안내까지 end-to-end 실행. '에러 확인해', '이거 수정해', '로그 확인하고 고쳐' 등의 요청 시 사용."
---

# 버그 수정 전체 사이클

## 워크플로우

### Phase 1: 원인 분석
1. request_id 또는 에러 메시지로 서버 로그 검색
2. traceback 확인하여 원인 코드 위치 파악
3. 에러 유형 분류: DB/권한/코드로직/외부연동

### Phase 2: 코드 수정
1. 원인 파일 읽기
2. 최소 변경으로 수정
3. 관련 파일 영향도 확인 (serializer, service, view)

### Phase 3: 릴리즈
1. deploy 브랜치에서 커밋 + push
2. develop 머지 → main PR → 머지 → 릴리즈
3. 서브모듈 태그 고정

### Phase 4: 배포 안내
서버에서 실행할 명령어를 사용자에게 제공.

## 에러 유형별 체크리스트

### 권한 에러 (403)
- permission_map.py에 해당 키 있는지 확인
- org#team_leader 포함 여부 (한줄짜리도!)
- can_any 캐시 시드 tuple 형식 확인
- 사용자의 TB_190 레코드 확인 (object_id="whbo" 맞는지)

### DB 에러 (500 + column not found)
- 마이그레이션 누락 확인
- 서버에서 migrate 실행 안내

### FTP 에러 (AUTH TLS / SIZE)
- sendcmd로 USER/PASS 직접 전송 패턴
- TYPE I 바이너리 모드 전환 확인
- FTP_USE_TLS 기본값 False 확인

### NoneType 에러
- FK 관계에서 null 가능한 필드 체크
- getattr fallback 패턴 적용

### ValueError (tuple unpacking)
- can_any 캐시 시드 3-tuple vs 4-tuple 확인
- Redis perm_obj:* 캐시 삭제 필요 여부

## 주의사항
- 서버 코드 직접 수정 금지
- Docker 캐시 문제: pyc 삭제 + docker rmi + --build
- .pyc 캐시가 새 코드를 가리는 경우 주의
