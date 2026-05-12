# bot-health-monitor

5분마다 두 봇 `/health` 를 폴링해 다운 시 알림을 발송하는 외부 모니터.

## 모니터 대상

| 봇 | 인프라 | URL |
|---|---|---|
| reversal-alert-bot | Cloudflare Workers | https://reversal-alert-bot.j-810.workers.dev/health |
| superbuysell-bot   | Deno Deploy        | https://superbuysell-deno.gallas111.deno.net/health |

각 워커의 `/health` 가 200 이 아니면 (503 / 타임아웃 / 무응답) 워크플로우 step 이 fail 처리됨.

## 알림 채널

### 1. GitHub 자동 알림 (기본 — 별도 설정 0)
워크플로우 실패 시 GitHub 이 자동으로 발송:
- 이메일 (계정 이메일)
- GitHub 모바일 앱 푸시 (앱 설치 시)

### 2. Telegram 알림 (옵션 — 별도 알림용 봇 필요)

⚠️ **모니터 대상 봇과 다른 봇** 으로 secret 등록 (대상 봇이 죽으면 그 봇으로 알림도 못 옴).

```bash
# 1. BotFather 에서 알림용 새 봇 생성 → 토큰 받음
# 2. 알림 받을 chat_id 확인 (대상 봇과 동일하게 본인 DM 가능)
# 3. 이 리포 Settings → Secrets and variables → Actions 에 등록:
#    ALERT_TG_BOT_TOKEN = <새 봇 토큰>
#    ALERT_TG_CHAT_ID   = <chat_id>
```

또는 gh CLI 로:
```bash
gh secret set ALERT_TG_BOT_TOKEN --body "<token>"
gh secret set ALERT_TG_CHAT_ID --body "<chat_id>"
```

secret 미설정 시 텔레그램 step 은 skip (워크플로우는 fail 그대로).

## 수동 실행

```bash
gh workflow run health-check.yml
```

## 왜 PUBLIC 리포인가
GitHub Free 의 PRIVATE 리포 GH Actions 한도는 월 2000분.
5분 간격 × 2 jobs × 1분(빌링 단위) = 월 ~17,280분 → 초과.
PUBLIC 리포는 무제한 무료. 이 리포에 민감 정보 0 (워커 도메인은 이미 공개).
