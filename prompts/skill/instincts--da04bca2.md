---
title: "instincts"
type: "skill"
tags: ["kit","skill","instinct","instincts","אינסטינקט","כלל מותנה","תזכור שכש","learned rule"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T01:08:05.47331+00:00"
id: "da04bca2-2a5a-4ba8-8506-ad81da296008"
---

> אינסטינקטים — כללים מותנים ("כשקורה X, עשה Y") שנטענים אוטומטית רק כשהם רלוונטיים להקשר הנוכחי, במקום לשבת כמשקולת ב-CLAUDE.md. חנות אחת (`~/.claude/instincts.jsonl`), הזרקה ב-SessionStart דרך kit-hello, דירוג לפי ביטחון + התאמה לפרויקט + התאמה לסטאק. הצד השלישי של הזיכרון לצד MEMORY.md (ידע) ו-PROJECT.md (מצב). Triggers - "instinct", "instincts", "אינסטינקט", "כלל מותנה", "תזכור שכש", "learned rule", "conditional rule", "instincts.py", "למד את זה".

# Instincts — כללים שנטענים רק כשהם רלוונטיים

## הבעיה שזה פותר

לקיט היו שני יעדי זיכרון, ושניהם "תמיד":
- `CLAUDE.md` — כללי ברזל, **תמיד** בהקשר. מקום יקר. רק מה שנכון לכל משימה.
- `MEMORY.md` — ידע, נקרא כשפותחים אותו. עשיר, אבל אף אחד לא קורא 400 שורות לפני כל טאסק.

מה שנפל בין הכיסאות: **כלל מותנה**. "בפריסת openwa — המשתמש בשרת הוא `ubuntu`, לא `lior`" הוא
רעש מיותר ב-99% מהסשנים, וקריטי באחד. ב-`CLAUDE.md` הוא מנפח; ב-`MEMORY.md` הוא לא יגיע בזמן.
אינסטינקט הוא בדיוק זה: **טריגר + פעולה + ביטחון**, שנטען אוטומטית רק כשההקשר מתאים.

הרעיון מקורו ב-ECC (`affaan-m/ECC`); **המימוש כאן הוא של הקיט** — stdlib Python, קובץ אחד,
בלי תלות ובלי vendor.

## שימוש

```bash
~/DevOPS/instincts.py add "<מתי>" "<מה לעשות>" --domain ops --confidence 0.9
~/DevOPS/instincts.py add "כשמוסיפים קומפוננטה" "flex-row-reverse ולא flex" --stack nextjs
~/DevOPS/instincts.py add "בפרויקט הזה" "npm run check לפני commit" --project   # bare flag = cwd
~/DevOPS/instincts.py list          # מדורג להקשר של התיקייה הנוכחית
~/DevOPS/instincts.py list --all    # כולל מה שמתחת לסף
~/DevOPS/instincts.py hit <id>      # האינסטינקט באמת עבד → ביטחון עולה
~/DevOPS/instincts.py rm <id>
~/DevOPS/instincts.py selftest      # הדירוג הוא המוצר — זו הבדיקה שלו
```

## מתי לרשום אינסטינקט (ומתי לא)

| הידע | היעד |
|------|------|
| "כשקורה X — עשה/היזהר מ-Y" | **אינסטינקט** |
| "השרת X מריץ Y, ההחלטה הייתה Z ולמה" | `MEMORY.md` (כלל #6) |
| "מה הושלם / מה חסום" | `PROJECT.md` |
| כלל שנכון **תמיד**, לכל משימה | `CLAUDE.md` — אבל זו החלטה של הבעלים, לא שלך |

טעות נפוצה: לרשום ידע כללי כאינסטינקט. אינסטינקט בלי "כש..." אמיתי הוא סתם שורה שתוזרק
לחינם בכל סשן.

## איך זה נטען

`kit-hello.sh` (hook של SessionStart, אותו אחד של `/whats-new` ו-`/scale`) קורא ל-`inject`
בתוך אותו תהליך Python — בלי הרצת subprocess נוספת — ומזריק `<instincts>` עם עד 5 שורות.

**הדירוג הוא המוצר:** `confidence` + 0.25 אם האינסטינקט מקושר לפרויקט הנוכחי + 0.2 אם ה-stack
שלו זוהה בתיקייה (`next.config.*`, `pyproject.toml`, `Dockerfile`, `go.mod`…). כך כלל 0.7
**על הפרויקט הזה** עוקף כלל גלובלי 0.9 שלא קשור. אינסטינקט משויך-פרויקט **לא מוזרק בכלל**
מחוץ לפרויקט שלו — הוא לא "פחות חשוב" שם, הוא פשוט שגוי שם.

כוונון: `INSTINCTS_MIN_CONFIDENCE` (0.6), `INSTINCTS_MAX_INJECT` (5), `INSTINCTS_FILE`.

## החנות

`~/.claude/instincts.jsonl` — אובייקט JSON בשורה. JSONL ולא YAML בכוונה: הוספה = שורה אחת,
אין פרסר לכתוב, ושורה פגומה עולה אינסטינקט אחד ולא את כל הקובץ.

```json
{"id":"בפריסת-openwa","trigger":"בפריסת openwa","action":"המשתמש בשרת הוא ubuntu","domain":"ops","confidence":0.95,"project":null,"stack":[],"hits":0,"created":"2026-08-17"}
```

⚠️ החנות היא **פר-משתמש ומחוץ לגיט** — היא לא עוברת בין שרתים ולא שורדת מוות של שרת.
ידע שחייב לשרוד ולהיות משותף הולך ל-`MEMORY.md` שבריפו (כלל #6). אינסטינקט = תוספת מקומית
וזולה, לא תחליף.

## אזהרה (כלל #9)

`add` שכותב שורה לקובץ אינו הוכחה שהמודל קיבל אותה. האימות היחיד הוא בצד הצרכן:

```bash
KIT_HELLO_STATE=/tmp/x bash ~/DevOPS/kit-hello.sh | python3 -m json.tool | grep -A5 instincts
```
