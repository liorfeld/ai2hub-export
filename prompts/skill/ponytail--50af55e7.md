---
title: "ponytail"
type: "skill"
tags: ["kit","skill","ponytail","be lazy","lazy mode","simplest solution","minimal solution","yagni"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T06:08:52.264397+00:00"
id: "50af55e7-4bf9-4f29-b742-7c51330c5daf"
---

> Ponytail — "lazy senior dev" generation-time minimalism (DietrichGebert/ponytail, vendored & installed as a live Claude Code plugin). The best code is the code you never wrote. Triggers - "ponytail", "be lazy", "lazy mode", "simplest solution", "minimal solution", "yagni", "do less", "shortest path", "over-engineering", "too much code", "מינימליזם", "פחות קוד".

# Ponytail — מינימליזם בזמן כתיבה (Lazy Senior Dev)

מבוסס [DietrichGebert/ponytail](https://github.com/DietrichGebert) — **מווונדר בקיט** ב-`vendor/ponytail/`
ומותקן כ-**plugin חי של Claude Code** (`ponytail@ponytail`, v4.7.0) בכל הצי דרך `setup-ponytail.sh`.
הפילוסופיה: **"הקוד הכי טוב הוא קוד שלא כתבת"**. בבנצ'מרק הרשמי — ~80-94% פחות שורות, 47-77% פחות עלות, פי 3-6 מהר יותר, ב-100% בטיחות.

> **איפה זה יושב במערך:** `ponytail` = מינימליזם **בזמן הכתיבה** (reflex לפני שכותבים קוד). זה משלים את
> `/code-reviewer`, `/simplify` ו-`/engineering-pro` שהם **בזמן הביקורת**. ponytail מונע את הבלון; code-review מנקה אותו.

## ⚙️ Plugin חי — לא רק מסמך
ה-skill הזה הוא נקודת הכניסה המתועדת; ה-**plugin המותקן** מספק את ההתנהגות החיה:
- **6 פקודות native**: `/ponytail [lite|full|ultra|off]`, `/ponytail-review`, `/ponytail-audit`, `/ponytail-debt`, `/ponytail-gain`, `/ponytail-help`.
- **2 hooks (harness-only, אפס עלות טוקנים)**: `SessionStart` + `UserPromptSubmit` — מפעילים את המצב אוטומטית בכל סשן. ה-hooks עושים no-op בחן אם `node` חסר.
- **6 skills** ב-plugin (ponytail + 5 תתי).

## הסולם (The Ladder) — עצור ברגע הראשון שמחזיק
1. **האם זה צריך להתקיים בכלל?** צורך ספקולטיבי = דלג, אמור זאת בשורה. (YAGNI)
2. **ה-stdlib עושה את זה?** השתמש בו.
3. **פיצ'ר native של הפלטפורמה מכסה?** `<input type="date">` לפני ספריית picker, CSS לפני JS, constraint ב-DB לפני קוד אפליקציה.
4. **תלות שכבר מותקנת פותרת?** השתמש. לעולם אל תוסיף חדשה למה שכמה שורות עושות.
5. **אפשר בשורה אחת?** שורה אחת.
6. **רק אז:** הקוד המינימלי שעובד.

הסולם הוא **reflex, לא פרויקט מחקר**. שתי דרגות עובדות → קח את הגבוהה והמשך.

## מצבי עוצמה (Intensity)
| מצב | מה משתנה |
|------|-----------|
| **lite** | בנה מה שביקשו, אבל ציין את החלופה העצלה יותר בשורה אחת. המשתמש בוחר. |
| **full** | הסולם נאכף. stdlib ו-native קודם. ה-diff הקצר ביותר, ההסבר הקצר ביותר. **ברירת מחדל.** |
| **ultra** | YAGNI קיצוני. מחיקה לפני הוספה. שלח את ה-one-liner ותתגר על שאר הדרישה באותה נשימה. |
| **off** | כבוי. `/ponytail off` / "stop ponytail" / "normal mode". |

**החלפה חיה:** `/ponytail lite|full|ultra|off`. **ברירת מחדל fleet-wide:** `full` (נאמן למקור).
שינוי ברירת המחדל: env `PONYTAIL_DEFAULT_MODE=off|lite|full|ultra` או `~/.config/ponytail/config.json` עם `{"defaultMode":"lite"}`. סדר הכרעה: env → config → full.

## כללים
- **בלי אבסטרקציות שלא ביקשו**: בלי interface עם מימוש יחיד, בלי factory למוצר יחיד, בלי config לערך שלא משתנה.
- **מחיקה לפני הוספה.** משעמם לפני חכם (חכם = מה שמישהו מפענח ב-3 לפנות בוקר).
- **הכי מעט קבצים.** ה-diff העובד הקצר ביותר מנצח.
- בקשה מורכבת? שלח את הגרסה העצלה ותתגר עליה באותה תגובה: "עשיתי X; Y מכסה. צריך X מלא? אמור."
- סמן פישוטים מכוונים עם הערת `ponytail:` שמכריזה על התקרה ונתיב השדרוג: `# ponytail: global lock, per-account locks if throughput matters`. `/ponytail-debt` קוצר אותן ל-ledger.

## מתי **לא** להיות עצלן (גבולות בטיחות)
לעולם אל תפשט החוצה: **input validation** בגבולות אמון, **error handling** שמונע אובדן נתונים, **אבטחה**, **נגישות בסיסית**, וכל דבר שביקשו במפורש. המשתמש מתעקש על הגרסה המלאה → בנה אותה, בלי ויכוח.
לוגיקה לא טריוויאלית משאירה **בדיקה אחת רצה** (assert-based `demo()`/`__main__` או `test_*.py` קטן) — YAGNI חל גם על בדיקות.

## פקודות עזר
- `/ponytail-review` — ביקורת over-engineering על ה-diff הנוכחי → רשימת מחיקה. → `/ponytail-review` skill.
- `/ponytail-audit` — סריקת over-engineering של כל הריפו. → `/ponytail-audit` skill.
- `/ponytail-debt` — קצירת הערות `ponytail:` ל-ledger מעקב ("later" לא הופך ל"never").
- `/ponytail-gain` — לוח תוצאות מדוד מהבנצ'מרק (לא מספר per-repo — הגרסה הלא-בנויה מעולם לא נכתבה).
- `/ponytail-help` — כרטיס עזר מהיר.

## הקמה / תפעול (fleet)
```bash
bash ~/DevOPS/setup-ponytail.sh           # core idempotent (kit-update מריץ אוטומטית בכל שרת)
bash ~/DevOPS/setup-ponytail.sh --check   # מצב: node, claude CLI, marketplace, plugin, mode
bash ~/DevOPS/deploy-ponytail.sh --mode lite   # front-end verbose + קביעת ברירת מחדל
```
מותקן offline מה-marketplace המקומי המווונדר (`claude plugin marketplace add ~/DevOPS/vendor/ponytail` → `claude plugin install ponytail@ponytail`) — בלי תלות GitHub per-host. דורש את ה-`claude` CLI; `node` נחוץ רק להפעלה האוטומטית של ה-hooks.

**Stack:** Claude Code plugin (`.claude-plugin/`), 2 Node lifecycle hooks, 6 TOML commands, 6 skills. MIT.
