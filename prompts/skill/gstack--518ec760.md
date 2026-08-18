---
title: "gstack"
type: "skill"
tags: ["kit","skill","gstack","garry tan","virtual engineering team","autoplan","sprint workflow skills","צוות הנדסה וירטואלי"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T01:08:05.47331+00:00"
id: "518ec760-be95-4877-a43e-8bb6ad710ec8"
---

> Deploy & operate gstack (garrytan/gstack, MIT) — Garry Tan's Claude Code "virtual engineering team" skill pack (~23 opinionated slash commands acting as CEO / Designer / Eng-Manager / Release-Manager / Doc-Engineer / QA) that scripts a Think→Plan→Build→Review→Test→Ship→Reflect sprint. Triggers - "gstack", "garry tan", "virtual engineering team", "autoplan", "sprint workflow skills", "eng team skill pack", "צוות הנדסה וירטואלי".

# gstack — צוות ההנדסה של Garry Tan

[garrytan/gstack](https://github.com/garrytan/gstack) (MIT) — חבילת ~23 skills דעתניים ל-Claude Code שהופכים אותו לזרימת ספרינט: Think → Plan → Build → Review → Test → Ship → Reflect. פקודות: `/autoplan`, `/spec`, `/review`, `/codex`, `/qa`, `/cso` (OWASP+STRIDE), `/ship`, `/canary`, `/browse`, `/pair-agent`. Runtime: **Bun**; מגיע עם דפדפן headless (Playwright/Puppeteer) + `browse`/`make-pdf` binaries.

> **חופף לקיט:** `/master` + `/ruflo` + `/babysitter` כבר מכסים אורקסטרציה. gstack הוא **אלטרנטיבה opt-in** למי שרוצה דווקא את זרימת 23-הפקודות של Garry Tan. לא fleet-wide.

## התקנה (opt-in per host)
```bash
bash ~/DevOPS/deploy-gstack.sh          # git clone pinned → ~/.claude/skills/gstack + ./setup (Bun≥1.0)
bash ~/DevOPS/deploy-gstack.sh --check  # bun/install/ref/ngrok
```
בלי Bun → דילוג נקי (לא משאירים skill pack שלא רץ). מוצמד ל-commit SHA (אין tags upstream) — override ב-`GSTACK_REF`.

## אבטחה
1. **סקריפטים ניתנים-להרצה** נכנסים ל-`~/.claude`; team mode מכניס `.claude/` ל-repo → הסוכן של כל שותף מריץ אותם. לבדוק לפני אמון.
2. **ngrok מצורף** = סיכון tunnel ציבורי. ה-deploy לא פותח שום tunnel; להפעיל רק במכוון.

## Scale (כלל #8)
gstack מפעיל את Claude Code של ההוסט → יורש את שכבת ה-session (🔵 תכנון). `/codex` שבו הוא handoff ל-🟢. אין ניתוב עצמאי.
