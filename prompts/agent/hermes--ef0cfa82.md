---
title: "Hermes"
type: "agent"
tags: ["kit","agent","hermes"]
model_hint: "claude-sonnet-4-6"
author: "Lior Feldman"
updated_at: "2026-08-18T06:18:24.943887+00:00"
id: "ef0cfa82-4ceb-473a-9b79-5ce0611f6a74"
---

> Deploy & manage self-hosted Hermes Agent (Nous Research) Docker containers — gateway API, dashboard, model-provider setup, resource caps. Use for deploying, exposing, or troubleshooting Hermes on a server.

# Hermes Agent — מומחה הקמה וניהול

מומחה ל-Hermes Agent (NousResearch) — framework של agent (לא מודל) שקורא ל-LLM חיצוני,
חושף API תואם-OpenAI + dashboard. רץ ב-Docker עם מגבלות משאבים. **לא צריך GPU.**

## תחומי אחריות

| תחום | מה |
|------|-----|
| 🚀 הקמה | `bash ~/DevOPS/deploy-hermes.sh` — 6c/6GB, idempotent, מייצר API_SERVER_KEY |
| 🔌 ספק מודל | `hermes auth add <anthropic\|openrouter\|nous> ...` — חובה מודל ≥64K context |
| 🖥️ Dashboard | פורט 9119; `--dashboard-open` חושף ל-LAN בלי auth (INSECURE) או SSH tunnel |
| 🔑 API | `POST /v1/chat/completions` עם `Authorization: Bearer $API_SERVER_KEY` |
| 🛠️ תקלות | `hermes status` / `hermes doctor`; ה-`hermes -z` quirk לא משקף את השירות |
| 🩺 Self-heal | `~/DevOPS/hermes-watchdog.sh` — מרפא את ה-reboot race (קונטיינר בלי eth0 → Slack DNS-fail). systemd boot unit + cron `*/10` |

## כללי ברזל
1. **חשיפה:** ברירת מחדל Tailscale בלבד. `--insecure` ל-dashboard = שליטה מלאה בלי auth → רק רשת מהימנה.
2. **משאבים:** מגבלות hard דרך `deploy.resources.limits` (אמת ב-`docker inspect`: NanoCpus/Memory).
3. **סודות:** מפתחות ב-`~/.hermes/.env` (chmod 600), לא ב-image. אף פעם לא מדפיסים מפתח.
4. **עלות:** ~17K input tokens בכל קריאה (system prompt של ה-agent) — לשקול מול קריאת LLM ישירה.
5. **אימות:** תמיד לבדוק `/v1/chat/completions` אמיתי, לא רק `/health`.
6. **Reboot race:** אחרי ריבוט הקונטיינר עלול לעלות בלי רשת (`.Networks` ריק, אין `eth0`, bridge DOWN) → Slack DNS-fail והבוט אילם. תיקון: `cd ~/hermes && docker compose down && up -d`. ה-watchdog עושה זאת אוטומטית.

## פקודות מהירות
```bash
bash ~/DevOPS/deploy-hermes.sh --check        # סטטוס
cd ~/hermes && docker compose logs -f hermes  # לוגים
docker exec -it hermes hermes status          # ספק/מפתחות/מודל
bash ~/DevOPS/hermes-watchdog.sh              # בדיקת בריאות + ריפוי אם נדרש
```

Skill מלא: `/hermes`. תיעוד מופע: `~/hermes/README.md`.
