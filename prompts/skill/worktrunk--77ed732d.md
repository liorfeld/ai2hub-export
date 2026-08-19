---
title: "worktrunk"
type: "skill"
tags: ["kit","skill","worktrunk","wt switch","wt list","wt merge","git worktree","worktree"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T04:33:13.887228+00:00"
id: "77ed732d-d8b2-418f-82a2-10ecdfeeaa18"
---

> Git-worktree lifecycle manager (max-sixty/worktrunk, MIT OR Apache-2.0) — the `wt` CLI addresses worktrees BY BRANCH instead of by path: `wt switch -c feat` creates+enters, `wt list` shows every branch's real state (dirty/ahead-behind/CI/PR), `wt merge` squashes→rebases→fast-forwards→removes in one command. Triggers - "worktrunk", "wt switch", "wt list", "wt merge", "git worktree", "worktree", "parallel agents", "branch per agent", "isolated worktree", "עץ עבודה", "וורקטרי", "סוכנים במקביל", "ענף מבודד".

# worktrunk — ניהול עצי-עבודה של git לפי ענף

[max-sixty/worktrunk](https://github.com/max-sixty/worktrunk) (MIT OR Apache-2.0) — כלי `wt` שמחליף
את `git worktree` המסורבל. במקום לנהל **נתיבים**, אתה עובד מול **ענפים**: הכלי יוצר את התיקייה,
נכנס אליה, ומוחק אותה כשסיימת. בינארי **Rust סטטי יחיד** — בלי Node, בלי Python, **בלי שום תלות ריצה**.
מותקן fleet-wide ע"י kit-update עם **גרסה נעוצה + SHA-256 מקודד** (אי-התאמה = לא מותקן).

> **מול הקיים:** `/no-mistakes` מריץ ביקורת בתוך worktree זמני ו-`/clone-website` מפצל לכמה worktrees —
> אבל אף אחד מהם לא **מנהל את מחזור החיים** שלהם, והם נוטים להישאר מאחור. worktrunk הוא בדיוק
> השכבה החסרה. אין לו קשר ל-`/codebase-memory` (גרף קוד) או ל-`/babysitter` (יומן ריצה).

## למה זה חשוב לסוכנים

הבעיה: שני סוכנים שעובדים על אותו ריפו **דורסים אחד את השני**. הפתרון: worktree נפרד לכל אחד.

```bash
wt switch -x claude -c feature-a -- 'הוסף אימות משתמשים'
wt switch -x claude -c feature-b -- 'תקן את באג העימוד'
```
כל סוכן מקבל תיקייה משלו, ענף משלו, ואפילו **פורט דטרמיניסטי** משלו
(`{{ branch | hash_port }}` → 10000–19999) כך ששרתי הפיתוח לא מתנגשים.

## הפקודות

| פקודה | מה עושה |
|---|---|
| `wt switch [ענף]` | נכנס ל-worktree; `-c` יוצר חדש · בלי ארגומנט → בורר אינטראקטיבי |
| `wt list` | טבלת מצב: dirty · ahead/behind · CI · PR · נתיב · גיל |
| `wt merge [יעד]` | ממזג את **הענף הנוכחי** ליעד: commit → squash → rebase → ff → מחיקת ה-worktree |
| `wt remove [ענפים]` | מוחק worktree, ומוחק את הענף אם כבר מוזג (6 בדיקות מיזוג לפני מחיקה) |
| `wt step commit` | commit עם הודעה שנוצרת ב-LLM (`--dry-run` מציג בלי לבצע) |
| `wt step copy-ignored` | מעתיק `node_modules/`/`target/` בין worktrees — חוסך בנייה מחדש |
| `wt step prune` | מחיקה מרוכזת של worktrees שכבר מוזגו |
| `wt hook show` | 10 סוגי hooks (pre/post × switch/start/commit/merge/remove) |

קיצורי ענף: `^` ברירת מחדל · `@` נוכחי · `-` הקודם · `pr:{N}` / `mr:{N}` (או URL מלא).
`git wt …` עובד גם — מותקן `git-wt` לצד `wt`.

## הפעלה (opt-in — לא נכתב לך אוטומטית)

הסקריפט **לא נוגע** בקבצי המשתמש שלך. שתי הפעלות אופציונליות:

```bash
wt config shell install          # מוסיף hook ל-shell כדי ש-`wt switch` באמת יחליף תיקייה
```
בלי זה `wt switch` עדיין עובד — הוא פשוט מריץ תת-תהליך במקום להחליף לך את ה-cwd.

הודעות commit ב-LLM — ב-`~/.config/worktrunk/config.toml`:
```toml
[commit.generation]
command = "MAX_THINKING_TOKENS=0 claude -p --no-session-persistence --model=haiku --tools='' --setting-sources='user' --system-prompt=''"
```

## Scale (כלל #8)

הודעת commit היא משימה **מכנית** → הזול והמהיר ביותר, **לעולם לא Opus**.
🟢 ברירת המחדל: `claude -p --model=haiku` (keyless, מנוי מקומי).
חלופה בטראק ה-🟢 אם `codex` מחובר: `codex exec -c model_reasoning_effort='low' … | jq -sr …` (המודל המוצמד של הצי; דורש `jq`).
worktrunk עצמו **לא מחזיק מפתחות** — הוא רק מזרים prompt ל-stdin של פקודה שאתה בוחר וקורא את ה-stdout.

## אבטחה

- גרסה **נעוצה** (v0.69.2) + **SHA-256 מקודד לכל ארכיטקטורה**; אי-התאמה → ABORT בלי התקנה.
- **לא מריצים** את `worktrunk-installer.sh` של המקור — הוא עורך `~/.profile` ו-`~/.bashrc`.
- אין רשת בזמן ריצה, אין מפתחות, אין פורטים, אין טלמטריה. `gh`/`glab` הם אופציונליים (עמודת CI/PR בלבד).
- ⚠️ `wt remove` מוחק worktree **ומוחק את הענף אם מוזג**. `--reap` (ניסיוני) **הורג תהליכים**
  שה-cwd שלהם תחת ה-worktree — אל תריץ אותו על מכונה שמריצה שירותים משם.
- ⚠️ הפרויקט בקצב שחרור גבוה (3 גרסאות ב-26 שעות). הנעיצה מכוונת — לא לעדכן בלי לרענן את ה-SHA.

## ניהול

```bash
bash ~/DevOPS/setup-worktrunk.sh --check       # מצב: גרסה, נתיב, shell, LLM
bash ~/DevOPS/setup-worktrunk.sh               # התקנה/רענון (אידמפוטנטי)
bash ~/DevOPS/setup-worktrunk.sh --uninstall   # הסרה (הקונפיג נשאר)
```
כדי לנעוץ גרסה אחרת: `WORKTRUNK_VERSION=x.y.z` — **ורק אחרי** עדכון ה-SHA בסקריפט.

Slash: `/worktrunk`.
