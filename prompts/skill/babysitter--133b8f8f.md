---
title: "babysitter"
type: "skill"
tags: ["kit","skill","babysitter","babysit","deterministic orchestration","process as code","long running task","resume run"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T05:53:24.55937+00:00"
id: "133b8f8f-a535-4fac-8e97-48774f6e2cd4"
---

> Babysitter (a5c-ai/babysitter) — deterministic orchestration layer over AI coding agents. Triggers - "babysitter", "babysit", "deterministic orchestration", "process as code", "long running task", "resume run", "event journal", "breakpoint approval", "quality gate", "~/.a5c", "משימה ארוכה", "אורקסטרציה דטרמיניסטית", "המשך ריצה".

# Babysitter — אורקסטרציה דטרמיניסטית לסוכני קוד

Deploy ותפעול של [a5c-ai/babysitter](https://github.com/a5c-ai/babysitter) (MIT) — שכבת אכיפה מעל סוכני קוד: ה-workflow מוגדר **כקוד** (`process(inputs, ctx)` ב-JS) והסוכן יכול לעשות **רק** מה שה-process מתיר. מותקן בקיט כ-CLI (`@a5c-ai/babysitter`, npm 6.x) + **plugin חי** `babysitter@a5c.ai` + plugin ל-Codex.

> **מיקום במערך:** babysitter = אורקסטרציה **דטרמיניסטית ו-resumable** — ה-process מגביל את הסוכן, journal הוא מקור האמת, breakpoints = אישורי אדם. לעומתו: `/ruflo` = swarm חי ו-dual-mode אינטראקטיבי; `/gsd` + `/ralph` = מתודולוגיות לולאת-פרומפט; `/ponytail` = מינימליזם בזמן כתיבה. כלל אצבע: משימה של שעות / עם שערי אישור / שחייבת לשרוד קריסת context → **babysitter**; פרץ multi-agent אינטראקטיבי → **ruflo**.

## המושגים

| מושג | מה |
|------|-----|
| **Process** | פונקציית JS `process(inputs, ctx)` — ה-workflow עצמו. הקוד הוא הסמכות, לא הסוכן. |
| **Task** | `ctx.task(...)` — יחידת עבודה. אחרי כל task יש **עצירה חובה**; ה-process מחליט מה הבא. |
| **Breakpoint** | `ctx.breakpoint({question})` — שער אישור אדם. נאכף, לא אופציונלי. |
| **Quality gate** | בדיקה מוגדרת בקוד (score < 80 → refine) שחוסמת התקדמות. |
| **Journal** | יומן אירועים בלתי-ניתן-לשינוי ב-`~/.a5c/runs/<run-id>/` — replay ו-resume דטרמיניסטיים. |

## פקודות (מה-plugin — `/babysitter:*`)

| מצב | פקודה | מתי |
|-----|-------|-----|
| Call | `/babysitter:call <task>` | ריצה אינטראקטיבית — עוצר ב-breakpoints |
| Plan | `/babysitter:plan <task>` | תכנון בלבד → process |
| Yolo | `/babysitter:yolo <task>` | אוטונומי, בלי breakpoints |
| Forever | `/babysitter:forever <task>` | ריצה מתמשכת |
| Resume | `/babysitter:resume` | המשך ריצה מה-journal (אחרי קריסה/הפסקה) |

Ops: `/babysitter:doctor` (אבחון) · `/babysitter:observe` (מעקב ריצה) · `/babysitter:retrospect` · `/babysitter:cleanup` · `/babysitter:blueprints` (תבניות process).

**פעם ראשונה:** `/babysitter:user-install` (פעם למשתמש) → `/babysitter:project-install` (פעם לריפו) → `/babysitter:doctor`.

> `/babysitter` (הסקיל הזה, של הקיט) חי **לצד** `/babysitter:*` (פקודות ה-plugin) — אין התנגשות.

## Stack

- **Node ≥ 20** (22 LTS מומלץ) + **jq** + git. בלי daemon, בלי container, בלי ports, בלי API keys משלו (יורש את ה-auth של ה-harness).
- npm: `@a5c-ai/babysitter` (metapackage, bin `babysitter`) — **תמיד מ-npm @latest (6.x)**; ה-git tags תקועים על v0.0.188.
- Claude plugin: marketplace `a5c-ai/babysitter-claude` (שם: `a5c.ai`) → `babysitter@a5c.ai`. Hooks (כולל Stop-hook להמשכיות per-turn) נטענים **בסשן הבא**.
- Codex plugin: marketplace `a5c-ai/babysitter` (המונוריפו) → `babysitter@babysitter`.
- MCP server (`babysitter-mcp-server`) קיים אך **לא מחווט** בקיט; לא מאמצים את `.mcp.json` של הריפו (atlas-staging).

## Deploy

```bash
bash ~/DevOPS/deploy-babysitter.sh              # התקנה/תיקון (kit-update מריץ את ה-CORE אוטומטית בכל שרת)
bash ~/DevOPS/deploy-babysitter.sh --check      # סטטוס: node, jq, CLI, plugins, journal
bash ~/DevOPS/deploy-babysitter.sh --no-codex   # בלי ה-Codex plugin
bash ~/DevOPS/deploy-babysitter.sh --with-adapters --with-genty   # extras: adapters-cli (כל harness מהשל), genty (headless/CI)
```

## Verify

```bash
babysitter --version                          # 6.x
claude plugin list | grep -i babysitter       # babysitter@a5c.ai, enabled
codex plugin list | grep -i babysitter        # babysitter@babysitter, installed
node -v && jq --version                       # >=20, jq קיים
ls ~/.a5c/runs                                # journal home
bash ~/DevOPS/setup-babysitter.sh --check     # דוח מלא
# ובתוך סשן Claude (אחרי restart): /babysitter:doctor
```

## פתרון תקלות

| בעיה | פתרון |
|------|-------|
| `babysitter: command not found` | kit-update עוד לא רץ, או Node < 20 → שדרג ל-22 LTS ואז `npm i -g @a5c-ai/babysitter@latest` (או `bash ~/DevOPS/deploy-babysitter.sh`) |
| `MODULE_NOT_FOUND` מה-CLI | shim שבור — `npm rm -g @a5c-ai/babysitter @a5c-ai/babysitter-sdk && npm i -g @a5c-ai/babysitter@latest` ואז `babysitter --version` |
| `/babysitter:*` לא מופיעות / hooks לא פועלים | ה-plugin הותקן באמצע סשן — **restart לסשן**, ואז `claude plugin list` + `/babysitter:doctor` |
| `jq: command not found` | `sudo apt-get install -y jq` (ה-setup מנסה לבד עם sudo -n) |
| marketplace add נכשל | אין גישה ל-GitHub מהשרת — kit-update ינסה שוב בריצה הלילית |
| ריצה נתקעה / סשן קרס | `/babysitter:resume` (ה-journal ב-`~/.a5c/runs` שורד הכל) · מעקב: `/babysitter:observe` |
| שרתי Node-18 (cogo, n8n.nesyas, n8n.talikasis) | ה-setup מדלג שם **בכוונה** (SKIP רועש). שדרוג Node 22 LTS יפעיל אוטומטית ב-kit-update הבא |
| גרסה מוזרה (0.0.x) | הותקן מ-git tag — להסיר ולהתקין רק מ-npm `@latest` (6.x) |

## Related Skills

`/ruflo` (swarm חי) · `/codex` (המסלול הירוק — babysitter מחווט גם אליו) · `/gsd`, `/ralph` (לולאות פרומפט) · `/ponytail` (מינימליזם) · `/engineering-pro` (איכות הנדסית).
