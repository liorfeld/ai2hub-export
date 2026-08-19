---
title: "agentshield"
type: "skill"
tags: ["kit","skill","agentshield","agent shield","סריקת אבטחה","scan claude config","security scan","audit config"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T05:52:27.469296+00:00"
id: "f3cd718e-3e72-41fb-8a32-062f26adc478"
---

> AgentShield (affaan-m/agentshield, MIT) — סורק אבטחה סטטי ואופליין לתצורת סוכנים - סודות מוטמעים, כללי הרשאה רחבים מדי, הזרקה דרך hooks, שרתי MCP מסוכנים ווקטורי prompt-injection בקבצי agents. משלים את /skill-security-auditor (שבודק skill אחד שכבר חושדים בו) בכך שהוא סורק את **כל** התצורה דטרמיניסטית. מותקן מוצמד + מאומת sha512 מאחורי עטיפת default-deny שחוסמת את מצבי ה-LLM, ה-runtime וה-webhook. Triggers - "agentshield", "agent shield", "סריקת אבטחה", "scan claude config", "security scan", "audit config", "prompt injection scan", "hook injection", "האם הקונפיג בטוח".

# AgentShield — סריקת אבטחה לתצורת הסוכן

## מה זה נותן

`/skill-security-auditor` בודק **skill אחד** שכבר יש סיבה לחשוד בו — סקירה איכותית, לפי שיקול דעת.
AgentShield סורק את **כל** ה-`.claude/`: כל קובץ, כל ריצה, אותה תוצאה. חמישה צירים עם ציון:

| ציר | מה נתפס |
|-----|---------|
| Secrets | מפתחות מוטמעים (`sk-ant-…`, טוקנים, `.env` שדלף ל-CLAUDE.md) |
| Permissions | `Bash(*)`, allow רחב מדי, **אין deny list** |
| Hooks | פקודות מסוכנות ב-hooks, הזרקה, exfiltration |
| MCP | שרתים לא מוכרים, פקודות הרצה חשודות, טוקנים בקונפיג |
| Agents | prompt injection ו-"system prompt extraction" בקבצי agents |

```bash
bash ~/DevOPS/deploy-agentshield.sh          # התקנה (deploy-on-demand)
agentshield scan                             # סורק ~/.claude
agentshield scan --path ~/DevOPS -f json     # פלט מכונה
bash ~/DevOPS/deploy-agentshield.sh --check  # מצב + האם המגן עדיין אכוף
```

## מה חסום, ולמה (העטיפה, לא הבטחה)

`~/.local/bin/agentshield` הוא **default-deny**. מותר `scan` ו-`init` בלבד:

| חסום | הסיבה |
|------|-------|
| `--opus` · `--deep` · `--injection` · `--stream` | מעלים את ה-CLAUDE.md, ה-hooks וקונפיג ה-MCP שלך ל-API של Anthropic ל"ניתוח עמוק". זו התצורה שלך יוצאת מהקופסה — ועולה טוקנים. סורק שסומכים עליו עם סודות חייב להיות אופליין. |
| `--sandbox` | **מריץ** את ה-hooks שהוא סורק כדי לצפות בהתנהגות. |
| `miniclaw` | runtime שלם של סוכן בארגז חול. באנו לסורק. |
| `runtime` | hook של PreToolUse ששולט בקריאות הכלים **שלנו**. |
| `watch` | דמון שמפרסם ממצאים ל-webhook. |
| `--fix` | משכתב `settings.json` ו-`CLAUDE.md`. מותר רק עם `AGENTSHIELD_ALLOW_FIX=1` מפורש. |

התקנה: מוצמד `1.4.0` + אימות sha512 + `--ignore-scripts` לתוך prefix מבודד (`~/.local/lib/agentshield`).
60 חבילות טרנזיטיביות — אף אחת לא רצה בזמן התקנה.

## לקרוא את הדוח נכון

הריצה הראשונה על מארח קיט מחזירה **D (49/100)** — ורוב זה לא באג ולא סיכון:

- **~60 מתוך 74 ה-high** הם `Agent has no tools restriction`. כל סוכני הקיט הם `All tools`
  **בכוונה**. זו החלטת ארכיטקטורה, לא ממצא. אל תתקן בהיסח הדעת.
- `System prompt extraction attempt detected` ב-`CLAUDE.md` — נובע מהאזכור של קורפוס
  ה-system-prompts ל-red-team. false positive.
- **מה כן לקרוא:** ה-CRITICAL (כללי `allow` רחבים ל-`sudo`/`docker exec`), `No deny list
  configured`, וכל ממצא Secrets. אלה אמיתיים.

הסדר הנכון: CRITICAL → Secrets → Hooks → השאר. ציון נמוך שנובע מ-`All tools` הוא רעש קבוע;
אם רוצים מספר משמעותי לאורך זמן — להשוות בין ריצות, לא לרדוף אחרי הציון המוחלט.

## איפה זה יושב בשרשרת

`/skillsmith` (מוצא skill) → **`/agentshield`** (סורק את התצורה שאליה הוא ייכנס) →
`/skill-security-auditor` (בודק את ה-skill עצמו לעומק) → `/ponytail-review` → `DevOPS/skills/` → `kit-push`.

⚠️ שני חלקים מ-ECC נבחרו מתוך ערכה מתחרה (68 agents / 285 skills / hooks משלה). **ECC עצמו לא
מותקן** — הוא היה מתנגש עם `/master`, עם ה-hooks ועם כלל #8. נלקחו הרעיון (`/instincts`) והכלי
(AgentShield), לא הריפו.
