---
name: notify-users
description: "Slack DM 브로드캐스트, 서버 점검 공지, 완료 안내, 사용자 알림 발송. 'DM 보내', '공지 보내', 'Slack으로 알려', '전체 사용자에게' 등 Slack 알림 관련 요청 시 반드시 사용."
---

# Slack 사용자 알림 발송

## 실행 방법
SSH로 서버 접속 → Django shell에서 Slack API 호출.

## DM 브로드캐스트 템플릿

```python
import http.client, json, os

token = os.environ.get("SLACK_DM_APP_TOKEN") or "${SLACK_DM_APP_TOKEN}"

message = """{메시지 내용}"""

emails = [
    "sihan@whirik.com", "mimiu@whirik.com", "jipark@whirik.com",
    "shpark@whirik.com", "sunjolee@whirik.com", "jwchoi@whirik.com",
    "hrjeoung@whirik.com", "shkim@whirik.com", "dhshin@whirik.com",
    "hbkim@whirik.com",
]

success = 0
for email in emails:
    conn = http.client.HTTPSConnection("slack.com")
    conn.request("GET", f"/api/users.lookupByEmail?email={email}", headers={
        "Authorization": f"Bearer {token}",
    })
    resp = conn.getresponse()
    body = json.loads(resp.read().decode())
    conn.close()

    if not body.get("ok"):
        print(f"{email}: 조회 실패")
        continue

    slack_id = body["user"]["id"]

    conn2 = http.client.HTTPSConnection("slack.com")
    payload = json.dumps({"channel": slack_id, "text": message})
    conn2.request("POST", "/api/chat.postMessage", body=payload, headers={
        "Authorization": f"Bearer {token}",
        "Content-Type": "application/json",
    })
    resp2 = conn2.getresponse()
    body2 = json.loads(resp2.read().decode())
    conn2.close()

    if body2.get("ok"):
        success += 1
        print(f"{email}: 전송 성공")
    else:
        print(f"{email}: 실패 - {body2.get('error')}")

print(f"\n총 {success}/{len(emails)}명 전송 완료")
```

## 메시지 톤 가이드
- 한국어, 존댓말
- 이모지 적절히 사용 (🙏 ✅ 🙇 등)
- 점검 시작: 일시, 작업내용, 소요시간 명시
- 점검 연장: 사과 + 추가 소요시간
- 점검 완료: 감사 인사 + 정상 이용 가능 안내

## 주의사항
- SLACK_DM_APP_TOKEN 사용 (users:read.email scope 있음)
- SLACK_BOT_TOKEN은 채널 메시지/파일 업로드용 (scope 다름)
- timeout 60000ms 설정 (10명 순차 발송)
