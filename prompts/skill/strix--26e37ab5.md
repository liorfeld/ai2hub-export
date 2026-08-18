---
title: "strix"
type: "skill"
tags: ["kit","skill","strix","pentest agent","autonomous pentest","ai security testing","exploit poc","owasp scan"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T01:08:05.47331+00:00"
id: "26e37ab5-9415-45a6-ac61-5be02887767e"
---

> Deploy & operate Strix (usestrix/strix, Apache-2.0) — autonomous AI penetration-testing agents that run inside a Docker Kali sandbox (full offensive toolkit + passwordless sudo) to do recon, exploitation with working PoCs, validation, and remediation across the OWASP Top 10. Triggers - "strix", "pentest agent", "autonomous pentest", "ai security testing", "exploit poc", "owasp scan", "vulnerability agent", "בדיקת חדירות", "סוכן תקיפה".

# Strix — סוכני חדירה אוטונומיים

[usestrix/strix](https://github.com/usestrix/strix) (Apache-2.0) — פלטפורמת pentest רב-סוכנית: recon → exploitation עם PoC עובד → validation → המלצות תיקון, על OWASP Top 10. הסוכנים רצים **בתוך Docker sandbox** מבוסס `kalilinux/kali-rolling` עם ערכת התקפה מלאה (nmap, nuclei, ffuf, sqlmap, ZAP, wapiti, subfinder, trufflehog, semgrep…) ומשתמש `pentester` עם **sudo ללא סיסמה**.

> **מול הקיים:** `/pentest` + `/skill-security-auditor` + קורפוס `system-prompts-leaks` = עיון/בדיקה. **strix = ריצת exploitation אקטיבית.** משלים, לא מחליף.

## ⚠️ הרשאה — כלל-על
- **יעדים מורשים בלבד.** אשר scope בכל ריצה. לעולם לא נגד מערכת צד-ג'/production בלי הרשאה בכתב.
- Docker daemon = הרשאת host. ה-sandbox מחזיק Kali חי + sudo.
- Target+source זולגים ל-LLM אלא אם משתמשים במודל מקומי (`LLM_API_BASE`).

## התקנה ותפעול
```bash
bash ~/DevOPS/deploy-strix.sh          # venv מבודד, strix-agent==1.2.0 (Python≥3.12), Docker חובה
bash ~/DevOPS/deploy-strix.sh --check  # python/venv/CLI/docker/image/key
export STRIX_LLM='anthropic/claude-sonnet-4-6' LLM_API_KEY=…   # או gpt-5.6 דרך LLM_API_BASE
strix --target <authorized-url>        # TUI; `strix -n` headless ל-CI; `strix view` דשבורד localhost
```
תוצאות ב-`strix_runs/<name>/` (JSON/PDF), בלי העלאה לענן כברירת מחדל.

## Scale (כלל #8)
`STRIX_LLM` מנותב למודל מורשה: Claude Sonnet 4.6 או `gpt-5.6-codex` (דרך `LLM_API_BASE`); מודל מקומי (Ollama/LMStudio) לריצה ללא egress. ה-CLI/פרונט = 🔵; המימוש הפנימי של strix מונע LLM לבחירתך.

⚠️ ה-CLI open-source (Apache-2.0); קיימים ענן מנוהל חינמי ו-Enterprise בתשלום.
