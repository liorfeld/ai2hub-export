---
title: "no-mistakes"
type: "skill"
tags: ["kit","skill","no-mistakes","no mistakes","pre-push gate","push quality gate","clean pr gate","git proxy review"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T01:08:05.47331+00:00"
id: "3165789a-89eb-4ab3-bbdd-d3f426313b6d"
---

> Pre-push AI quality gate (kunchenguid/no-mistakes, MIT) — a local git proxy. Triggers - "no-mistakes", "no mistakes", "pre-push gate", "push quality gate", "clean PR gate", "git proxy review", "auto-fix before push", "שער לפני push".

# no-mistakes — שער איכות לפני push

[kunchenguid/no-mistakes](https://github.com/kunchenguid/no-mistakes) (MIT) — **git proxy** מקומי. דוחפים ענף ל-remote בשם `no-mistakes`; הוא פותח **worktree מבודד**, מריץ pipeline של AI (review, tests, docs, lint) שמנהל את ה-CLI שכבר על המכונה (claude/codex), מחיל fixes בטוחים, ורק אם הכל עבר — מעביר את הענף ל-remote האמיתי ופותח **PR נקי**. בינארי Go סטטי, בלי Node.

> **מול הקיים:** babysitter = אורקסטרציה דטרמיניסטית של משימות · code-reviewer = ביקורת בזמן review. no-mistakes = **שער ספציפי בזמן push** שמפיק PR נקי. משלים, לא מחליף.

## איך מפעילים (opt-in per-repo — לא אוטומטי!)

kit-update מתקין רק את **הבינארי** (נעוץ-גרסה + SHA-256). שום ריפו לא מחווט אוטומטית — ההפעלה יזומה, per-repo, לפיילוט:

```bash
bash ~/DevOPS/setup-no-mistakes.sh --check     # binary, version, agent CLI, git

# פיילוט על ריפו נבחר (מוסיפים remote בשם no-mistakes ודוחפים אליו):
cd /path/to/repo
no-mistakes init                 # מגדיר את ה-proxy remote לריפו הזה
git push no-mistakes my-branch   # → worktree מבודד → review+tests+lint → PR נקי אם עבר
```

(פקודות מדויקות: `no-mistakes --help` — ה-CLI מתעד את שמות התת-פקודות בגרסה המותקנת.)

## עקרונות

1. **opt-in בלבד** — לא נכפה על אף ריפו; מפעילים ידנית איפה שרוצים לפלט.
2. **מבודד** — הרצה ב-worktree חד-פעמי; לא נוגע ב-working tree שלך.
3. **מריץ AI + tests/lint שלך** בתוך ה-worktree = הרצת קוד. הרץ רק על ריפואים מהימנים; משתמש ב-credentials שכבר ב-git config.
4. דורש `git` עם worktree + agent CLI (claude/codex — קיים בצי).

## הקמה
```bash
bash ~/DevOPS/setup-no-mistakes.sh --check
bash ~/DevOPS/setup-no-mistakes.sh
```

Slash: `/no-mistakes`.

## שער ניגודיות (כלל ברזל #3)
הגייט חייב לחסום דark-on-dark. הוסף לשלב ה‑lint של הריפו (או ל‑pre-push) את הלינטר הסטטי, וה‑AI-review בודק זוגות טוקנים:
```bash
bash ~/DevOPS/contrast-lint.sh src   # exit 1 = רקע חשוף בלי text-* מזווג / hex bg / inline bg בלי color
```
פירוט + הרשת השנייה (runtime): `/qa` (`contrast-audit.mjs`). הסטנדרט המלא: `skills/DESIGN.md §0`.
