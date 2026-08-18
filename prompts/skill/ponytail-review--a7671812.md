---
title: "ponytail-review"
type: "skill"
tags: ["kit","skill","ponytail review","ponytail-review","review for over-engineering","delete-list","trim the diff","מה למחוק"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T01:08:05.47331+00:00"
id: "a7671812-0d26-41c9-b645-32de9daebc1d"
---

> Ponytail Review — scan the CURRENT diff for over-engineering only (not correctness) and hand back a delete-list. One line per finding with a tag (delete/stdlib/native/yagni/shrink) and the replacement, ending with the net lines removable. Part of the vendored ponytail plugin (/ponytail-review). Use before committing/PR to strip bloat from your changes; pair with /code-reviewer (correctness) and /simplify. Triggers - "ponytail review", "ponytail-review", "review for over-engineering", "what can I delete", "delete-list", "trim the diff", "מה למחוק", "לקצץ את ה-diff".

# Ponytail Review — ביקורת over-engineering על ה-diff

חלק מ-plugin ה-ponytail המווונדר (`/ponytail-review`). סורק **רק את השינויים הנוכחיים** ל-**over-engineering בלבד, לא נכונות**.

## פלט (פורמט קבוע)
שורה אחת לכל ממצא:
```
L<line>: <tag> <מה לחתוך>. <תחליף>.
```
מסתיים בשורת **net lines removable**. אם אין מה לחתוך: **"Lean already. Ship."**

## תגיות
| tag | משמעות |
|------|---------|
| `delete` | קוד מת / פיצ'ר ספקולטיבי |
| `stdlib` | המצאה מחדש של ספריית התקן |
| `native` | תלות שעושה מה שהפלטפורמה כבר עושה |
| `yagni` | אבסטרקציה עם מימוש יחיד |
| `shrink` | אותה לוגיקה, פחות שורות |

## איפה זה במערך
- `/ponytail-review` = **review-time** מינימליזם על ה-**diff**. 
- `/ponytail-audit` = אותו דבר על **כל הריפו**.
- `/code-reviewer` + `/simplify` = איכות/נכונות (משלימים, לא חופפים).
- `/ponytail` (full/ultra) = מניעה ב-**זמן הכתיבה** מלכתחילה.

## שימוש
```
/ponytail-review          # על ה-diff הנוכחי (git diff / staged)
```
הפעלה ידנית מהירה: ודא ש-`git diff` מציג שינויים, ואז קרא לפקודה. הביקורת **לא משנה קוד** — היא מחזירה רשימה; אתה מוחק.
