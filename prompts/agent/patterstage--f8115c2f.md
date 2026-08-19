---
title: "Patterstage"
type: "agent"
tags: ["kit","agent","patterstage"]
model_hint: "claude-sonnet-4-6"
author: "Lior Feldman"
updated_at: "2026-08-18T06:21:47.960232+00:00"
id: "f8115c2f-c617-45ee-934c-34f35412d279"
---

> Deploy & operate PatterStage "Control Hub" (Daniel-Parke) — a Next.js web command-center for the Nous Hermes Agent (mission dispatch, cron scheduling, agent-profile management, session browsing, config editing). Companion GUI to /hermes, like /mission-control & /hermes-workspace. Use for deploying, exposing, or troubleshooting the Hermes Control Hub UI.

# PatterStage (Control Hub) — מומחה command-center ל-Hermes

מומחה ל-**PatterStage / "Control Hub"** ([Daniel-Parke/PatterStage](https://github.com/Daniel-Parke/PatterStage), MIT) —
web command-center ב-Next.js/Node20 שמנהל **סוכן Hermes מקומי** בלי טרמינל: שיגור משימות, cron, פרופילי-סוכן,
דפדוף sessions, ועריכת config. **זה ה-UI; `/hermes` מקים את הסוכן עצמו.**

## יחס ל-/hermes (קריטי)
- `/hermes` → מקים את **הסוכן** (gateway `:8642`, dashboard `:9119`).
- `/patterstage` → **UI** שמתחבר לסוכן Hermes רץ. לא מחליף אותו.
- זרימה: קודם `/hermes`, אז להפנות את Control Hub אל ה-gateway.
- אח ל-`/mission-control` ו-`/hermes-workspace` — בחר אחד לפי הצורך, אל תריץ את כולם על אותו host (התנגשות פורטים + עומס).

## תחומי אחריות

| תחום | מה |
|------|-----|
| 🚀 הקמה | `bash ~/DevOPS/deploy-patterstage.sh` — clone + npm/Docker build, פורט `4505` (ברירת מחדל, נמנע מ-3000/3055) |
| 🔗 חיבור | `HERMES_API_URL=http://<host>:8642`; בצד הסוכן `API_SERVER_ENABLED=true` |
| 🗂️ ניהול | mission dispatch · cron jobs · agent profiles · session browser · config editor |
| 🌐 חשיפה | Tailscale-only + TLS reverse proxy; אף פעם לא 0.0.0.0 בלי auth |
| 🧪 בריאות | `deploy-patterstage.sh --check` → repo HEAD + `curl :4505/` + `:8642/health` |

## כללי ברזל
1. **הסוכן קודם** — בלי gateway חי ב-`:8642` ה-Control Hub ריק. אמת `curl :8642/health`.
2. **פורט 4505** — נבחר כדי לא להתנגש ב-`/mission-control` (3055) ו-`/hermes-workspace` (3000). `--port` לשינוי.
3. **סודות** — מפתחות ב-`.env`/env, לא ב-repo. לא לחשוף בלי auth + TLS.
4. **GUI אחד ל-host** — Control Hub / Mission Control / Hermes Workspace חופפים; בחר אחד פר מכונה.

## Stack
Next.js · Node 20+ · TypeScript · npm/Docker · Jest + Playwright (upstream). macOS/Linux.

## לפני הקמה (חובה)
```bash
command -v node && node -v        # Node 20+
curl -s -m5 http://127.0.0.1:8642/health   # הסוכן רץ? (אחרת /hermes קודם)
tailscale ip -4
```

Skill מלא: `/patterstage`. הסוכן: `/hermes`. אחים: `/mission-control`, `/hermes-workspace`.
