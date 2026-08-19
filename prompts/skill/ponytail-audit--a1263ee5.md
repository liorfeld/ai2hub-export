---
title: "ponytail-audit"
type: "skill"
tags: ["kit","skill","ponytail audit","ponytail-audit","whole repo over-engineering","remove dependencies","סריקת ניפוח","ניקוי ריפו"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T06:08:22.436796+00:00"
id: "a1263ee5-bc1d-4cc6-ba15-fce3e417dedc"
---

> Ponytail Audit — scan the WHOLE repository (not just a diff) for over-engineering, ranked biggest cut first, ending with the net lines and dependencies removable. Triggers - "ponytail audit", "ponytail-audit", "audit repo for bloat", "whole repo over-engineering", "what can we delete repo-wide", "remove dependencies", "סריקת ניפוח", "ניקוי ריפו".

# Ponytail Audit — ביקורת over-engineering על כל הריפו

חלק מ-plugin ה-ponytail המווונדר (`/ponytail-audit`). סורק את **כל עץ הקבצים** (לא diff) ל-**over-engineering בלבד, לא נכונות**.

## פלט (פורמט קבוע)
שורה אחת לכל ממצא, **מדורג מהחיתוך הגדול ביותר**:
```
<tag> <מה לחתוך>. <תחליף>. [path]
```
מסתיים ב-**net lines + dependencies removable**. אם אין מה לחתוך: **"Lean already. Ship."**

## תגיות
| tag | משמעות |
|------|---------|
| `delete` | קוד מת / פיצ'ר ספקולטיבי |
| `stdlib` | המצאה מחדש של ספריית התקן |
| `native` | תלות שעושה מה שהפלטפורמה כבר עושה |
| `yagni` | אבסטרקציה עם מימוש יחיד |
| `shrink` | אותה לוגיקה, פחות שורות |

## מתי
- ניקוי תקופתי / לפני release.
- triage של קוד שירשת.
- אחרי תקופת פיצ'רים אגרסיבית, לראות מה הצטבר.

## משלים
- `/dependency-auditor` — CVE/רישוי/תלויות מיושנות (הצד האבטחתי של "להסיר תלויות").
- `/ponytail-review` — אותו דבר אבל על ה-diff בלבד.
- `/ponytail-debt` — קצירת הערות `ponytail:` ל-ledger.

## שימוש
```
/ponytail-audit           # כל הריפו
```
**לא משנה קוד** — מחזיר רשימה מדורגת; אתה מחליט מה למחוק.
