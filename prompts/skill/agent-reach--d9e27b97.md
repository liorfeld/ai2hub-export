---
title: "agent-reach"
type: "skill"
tags: ["kit","skill","agent-reach","agent reach","read twitter","read reddit","youtube transcript","monitor rss"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T05:51:16.942997+00:00"
id: "d9e27b97-0de6-40be-a321-b46649341cb6"
---

> Give agents read+search access to the wider internet through one CLI (Panniantong/Agent-Reach, MIT) — Twitter/X, Reddit, YouTube, GitHub, Bilibili, XiaoHongShu, RSS, and general web, using Exa (free, keyless) for web search/read. Triggers - "agent-reach", "agent reach", "read twitter", "read reddit", "youtube transcript", "monitor rss", "social reading", "web research agent", "exa search", "קריאת רשתות", "מחקר רשת".

# Agent-Reach — עיניים לאינטרנט לסוכנים

[Panniantong/Agent-Reach](https://github.com/Panniantong/Agent-Reach) (MIT) — שכבת יכולת שנותנת לסוכן **קריאה+חיפוש** באינטרנט הרחב דרך CLI אחד: Twitter/X, Reddit, YouTube, GitHub, Bilibili, XiaoHongShu, RSS, ו-web כללי. משתמש ב-**Exa (חינם, בלי מפתח)** לחיפוש/קריאת web.

> **ממלא פער אמיתי:** שום דבר בקיט לא קורא social/web — Pexels = מדיה, OpenWA = WhatsApp יוצא. זו יכולת קריאה נכנסת חדשה.

## התקנה — deploy-on-demand (לא fleet-wide!)

מותקן ידנית על **נוד תוכן/מחקר נבחר**, לא על כל הצי. **לא** דרך המתקין המרוחק שלהם (סקריפט לא-נעוץ ש"מתקין ומקנפג" ומשנה מערכת) — אלא מ-**tag נעוץ (v1.5.0)** לתוך **venv מבודד**:

```bash
bash ~/DevOPS/deploy-agent-reach.sh            # venv + CLI (keyless-ready)
bash ~/DevOPS/deploy-agent-reach.sh --check    # python>=3.10, venv, CLI, doctor
```

## מדיניות אבטחה (חובה!)

1. **Keyless כברירת מחדל** — Exa (web) + RSS + GitHub (דרך `gh`) עובדים **בלי שום login**. זו נקודת ההתחלה והמצב המועדף.
2. **Cookies/logins של פלטפורמות = סיכון חסימת חשבון (封号风险).** אם בכל זאת — **חשבונות זריקים בלבד**, אף פעם לא ראשי.
3. `~/.agent-reach/config.yaml` (chmod 600) — **מחוץ ל-git**, אף פעם לא נכנס ל-commit.
4. חיווט מערכת אופציונלי (`agent-reach install`) מתקין כלים (gh/mcporter) ומשנה מערכת — **תמיד preview קודם**: `agent-reach install --safe --dry-run` לפני `--env=auto`.

## שימוש טיפוסי
```bash
agent-reach doctor                 # מה מותקן/מחווט
agent-reach <platform> <query>     # קריאה/חיפוש (ראה agent-reach --help לתת-פקודות)
```

Slash: `/agent-reach`.
