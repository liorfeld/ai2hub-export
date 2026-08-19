---
title: "kit-verify"
type: "skill"
tags: ["kit","skill","kit-verify","kit verify","verify kit","בדוק שהכל עובד","health check","fleet health"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T06:04:34.884047+00:00"
id: "b9ff4033-3371-4ea1-9f85-2f4f456fa8f0"
---

> The kit's consumer-side health gate — asserts what the kit actually DOES, never what it wrote. Triggers - "kit-verify", "kit verify", "verify kit", "האם זה באמת עובד", "בדוק שהכל עובד", "health check", "fleet health", "silent failure", "כשל שקט", "did it actually load", "is it really installed".

# kit-verify — לאמת תוצאות, לא כתיבות

## למה זה קיים

שלוש יכולות בקיט נבנו, **עברו את הבדיקה שלהן**, והיו מתות חודשים או שנים:

| מה | הבדיקה שהייתה | המציאות |
|---|---|---|
| הוק `memory-counter.sh` | "ההוק רשום ב-settings.json" ✅ | ב-`PostToolUse`/`Stop` הפלט הולך ל-debug log ולא למודל. רץ שנה |
| `MEMORY.md` | "ההנחיה כתובה ב-CLAUDE.md" ✅ | אף שורת קוד לא יצרה את הקובץ. 97 מ-156 פרויקטים בלי |
| `pg-memory` | `grep` על הקובץ שהרגע נכתב ✅ | `mcpServers` ב-`settings.json` הוא מפתח ש-Claude Code לא קורא |

הצורה משותפת: **הבדיקה קראה חזרה את הקובץ שהסקריפט הרגע כתב.** אימות כזה מאשר את
עצמו ולא יכול להיכשל מהסיבה שחשובה. זה כלל ברזל #9 — *כתיבה אינה הפעלה*.

לכן כל בדיקה כאן שואלת את **הצרכן**: מה באמת עלה, מה באמת נקרא, מה באמת נשמר.

## שימוש

```bash
kit-verify                # ריצה מלאה, פלט קריא
kit-verify --daily        # מלאה רק אם עברו >20 שעות (מה ש-kit-update מריץ)
kit-verify --json         # פלט מכונה בלבד
kit-verify --no-api       # דילוג על שתי הבדיקות שעולות בקשת API
kit-verify --self-check   # אימות שה-harness עצמו שלם
```

יציאה `0` = אין FAIL. יציאה `1` = לפחות FAIL אחד. `WARN` לעולם לא מפיל.
פלט מכונה: `~/DevOPS/logs/verify.json`. חותמת הריצה היומית: `logs/.verify-last-full`.

## שמונה הבדיקות

| בדיקה | שואלת את | תופסת |
|---|---|---|
| `mcp_loaded` | `claude --debug` → `MCP server "X": Starting connection` | מפתח קונפיג מת, שרת שזחל חזרה ל-user scope |
| `mcp_tool_count` | `Dynamic tool loading: 0/N` | חזרה ל-142 כלים בלי ששמים לב (FAIL מעל 80, WARN מעל 45) |
| `mcp_connected` | `claude mcp list` → `✔` | רשום אבל לא מתחבר — שרת על כתובת חסומה (כמו pg-memory בזמנו) |
| `mcp_scope` | user scope ב-`~/.claude.json` + טבלאות ה-MCP של Codex מול קטלוג `mcp-on` | user scope ≠ `{codebase-memory}` בלבד · שרת Codex דלוק בלי מתג בקטלוג · שרידי pg-memory (הוצא משימוש ב-1.35.0) |
| `project_docs` | ספירת פרויקטים עם CLAUDE.md מול MEMORY/PROJECT/DESIGN | כלל שמפנה לקובץ שלא קיים |
| `hooks` | שגיאות ריצה בלוג + סוג האירוע | הוק רשום שנכשל בשקט, או שהפלט שלו לא מגיע למודל |
| `rules` | שתי רשימות הכללים + טווח ההפניות | דריפט מספור, הפניה ל-כלל שלא קיים |
| `warnings` | `[WARN]` בלוג הסשן | דגרדציה ש-Claude Code מדווח עליה ואיש לא קורא |

## הפעלה אוטומטית

`kit-update` מריץ `kit-verify --daily` **אחרון**, אחרי כל מה שהוא מתקין. kit-update
רץ פעמיים ביום, ולכן השער של 20 שעות גורם לבקשת ה-API היחידה להיות משולמת **פעם ביום**.
כשל ב-kit-verify **לא מפיל** את kit-update.

## מה זה לא עושה

**לא מתקן.** תיקון אוטומטי הוא בדיוק הדבר שיכול להיכשל בשקט וליצור את הכשל הבא.
הכלי מודד ומדווח; אדם מחליט.

## מלכודות שכבר נתפסו בו

- **`.+?` ב-ERE אינו עצל.** הגרסה הראשונה של `rules` השתמשה ב-`\*\*.+?\*\*` והתאימה עד
  ה-`**` האחרון בשורה — כולל טקסט ההסבר, שכן שונה בין הקבצים. התוצאה: FAIL על רשימות
  זהות. `[^*]+` הוא הנכון. **בדיקה שצועקת זאב מושתקת, ובדיקה מושתקת אינה קיימת.**
- **`psql` לא מותקן על רוב הצי**, ו-central מריץ את ה-DB במכולה. בלי ה-fallback ל-docker
  הבדיקה עשתה SKIP דווקא על המארח היחיד שיכול לענות.
- הבדיקה חייבת לדרוש **לוג חדש**. שימוש חוזר בלוג הקודם היה מדווח מצב ישן כאילו הוא נוכחי.

## קשור

כלל ברזל #9 · `/no-mistakes` (שער pre-push, לפני מיזוג) · `/distribute-skill` (השלב
האחרון שם — "אימות ב-ssh" — הוא בדיוק מה ש-kit-verify עושה אוטומטית).
