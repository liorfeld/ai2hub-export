---
title: "skillsmith"
type: "skill"
tags: ["kit","skill","skillsmith","skill discovery","find a skill","search skills","agent skills registry","חיפוש skill"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T06:12:28.956601+00:00"
id: "9e4b90e7-cfa8-4876-99be-b1a23fe41c6d"
---

> Skill DISCOVERY for the kit (smith-horn/skillsmith, Elastic-2.0) — semantic search over ~7,900 SKILL.md files crawled from GitHub, answering "who already wrote a skill for X" before we write one. Triggers - "skillsmith", "skill discovery", "find a skill", "is there a skill for", "search skills", "agent skills registry", "חיפוש skill", "יש כבר skill", "מצא skill".

# Skillsmith — גילוי skills, לקריאה בלבד

[smith-horn/skillsmith](https://github.com/smith-horn/skillsmith) (Elastic-2.0) סורק את GitHub אחרי קבצי `SKILL.md` ונותן עליהם חיפוש סמנטי + ציון. הוא סוגר פער אמיתי בקיט: `/skill-creator` כותב skill, `/trend-scout` מגלה **ריפוזיטוריז**, ואף אחד לא ענה על "מי כבר כתב skill ל-X".

> **הוא כלי גילוי בלבד. הוא לא מתקין כלום, ולא נכנס לצי.**

## התקנה — deploy-on-demand, מארח נבחר בלבד

```bash
bash ~/DevOPS/deploy-skillsmith.sh --check     # מצב בלבד, לא משנה כלום
bash ~/DevOPS/deploy-skillsmith.sh
bash ~/DevOPS/deploy-skillsmith.sh --uninstall
```

לא נקרא מ-`upgrade.sh`/kit-update. גרסה נעוצה (`@skillsmith/cli@0.8.3`) עם אימות sha512 לפני התקנה, ל-prefix מבודד `~/.local/share/skillsmith/`. **לעולם לא `npx -y`** — זו גרסה נעה של חבילה בת חצי שנה עם maintainer יחיד.

**Node:** החבילה דורשת `node>=22.22.0` (התיעוד באתר סותר ואומר "18+"). המרכזי רץ Node 20 וכל הטולצ'יין תלוי בו, ולכן הסקריפט מוריד Node 22 LTS **נעוץ ומאומת SHA-256 לתיקייה של skillsmith בלבד**. ה-node המערכתי לא זז.

## שלוש הסירובים שמגדירים את ההתקנה

**1. ה-MCP לא נרשם.** שרת ה-MCP חושף תמיד `install_skill` ו-`uninstall_skill` שכותבים ל-`~/.claude/skills/`, ואין upstream שום env var או קונפיג לכבות אותם. ה-CLI לבדו מכיל את כל פקודות הקריאה — 100% מערך הגילוי, אפס נתיב כתיבה. `deploy-skillsmith.sh --wire-mcp` מסרב במכוון ומסביר.

**2. `npx -y` נדחה** לטובת pin + אימות sha512, בדיוק כמו `/worktrunk`, `/simplex` ו-`/codebase-memory`.

**3. חיווט-עצמי חסום.** שלוש תת-פקודות משכתבות את הסביבה מאחורי הגב:

| פקודה | מה היא עושה |
|---|---|
| `skillsmith setup` | מתקין skill משלה בשם `/skillsmith` ל-`~/.claude/skills/` — התנגשות שם עם הקיט + drift |
| `skillsmith agent` | "SKILL.md, shims, hooks, **MCP registration** across detected harnesses" — יודעת לכתוב ל-`~/.claude/settings.json`, `~/.codex/config.toml`, `~/.hermes/config.yaml` ועוד |
| `skillsmith telemetry` | ניהול העדפות **והתקנת hook ל-Claude Code** |

העטיפה חוסמת את שלושתן ומייצאת `SKILLSMITH_TELEMETRY=0` ו-`SKILLSMITH_AGENT_HOOK_DISABLE=1`. (‏`posthog-node` וה-SDK המלא של OpenTelemetry הם תלויות **קשיחות** של ה-CLI.)

## מה מותר ומה חסום

העטיפה `~/.local/bin/skillsmith` היא **default-deny**: כל מה שלא ברשימה נחסם, כולל כל תת-פקודה שתתווסף upstream בעתיד — עד שמישהו יקרא אותה.

```
מותר:  search · info · list · recommend · diff · validate · sync · analyze
       whoami · diagnose · logs · config get
חסום:  install · remove · update · create · init · setup · agent · telemetry
       inventory · import · import-local · publish · pin · unpin · audit
       author · login · logout · config set
```

`sync` מותרת כי היא כותבת רק את מטמון הרג׳יסטרי ב-`~/.skillsmith/`, אף פעם לא ל-`~/.claude/skills/`.

## צינור ה-vetting — הדרך היחידה ש-skill מגיע לצי

```
skillsmith sync                      →  מושך את הרג׳יסטרי למטמון מקומי (חובה לפני חיפוש ראשון)
skillsmith search "<כוונה>" --safe-only --min-score 60
skillsmith info <author/name>        →  פרטים, trust tier, קישור לריפו
קריאת ה-SKILL.md הגולמי ב-GitHub    →  ידנית. תמיד. בעיניים שלך.
/skill-security-auditor              →  שער אבטחה
/ponytail-review                     →  שער ניפוח
vendoring ל-~/DevOPS/skills/NAME.md  →  שם UPPERCASE + frontmatter בסגנון הקיט
gen-catalog.sh                       →  אוטומטי (אל תערוך בין סמני AUTOGEN)
git commit && kit-push               →  ההפצה היחידה לצי
```

**אף פעם לא `skillsmith install`.** skill שמותקן ישירות הוא drift per-host: לא בקטלוג, לא ב-kit-push, לא בריפו — ומאז v1.23.0 הוא גם *שורד* את kit-update, כלומר נשאר בשקט לנצח בלי שאף אחד יסקור אותו.

## מה שחייבים לדעת לפני שמסתמכים על המספרים

1. **הציון הוא פופולריות, לא בטיחות.** הנוסחה של ה-CLI עצמו: `log₁₀(stars+1)×15 + log₁₀(forks+1)×10 + 25`. skill מסוכן עם 1,000 כוכבים יקבל ~86%. הציון עונה על "מי משתמש בזה", לא על "זה בטוח".
2. **הסריקה היא regex, לא sandbox.** AIDefence/SecurityScanner מחפשים דפוסי prompt-injection וגישה לקבצים רגישים בטקסט. skill *הוא* הוראות לסוכן — אפשר לנסח כוונה זדונית בשפה תמימה שלא נוגעת באף דפוס. חתימת רג׳יסטרי עדיין `planned` אצלם. `--safe-only` הוא סינון ראשוני, לא אישור.
3. **האינדוקס הוא crawl אוטומטי בלי opt-in.** כל `SKILL.md` בריפו ציבורי נכנס בין אם המחבר ביקש ובין אם לא. (הקיט פרטי — `Ronus922/DevOPS-V2` מחזיר 404 — אז ה-skills שלנו לא נסרקים.)
4. **רישיון Elastic-2.0** — source-available, לא קוד פתוח. שימוש פנימי ו-self-host מותרים; הצעה כשירות מנוהל ללקוחות **אסורה**. רלוונטי ל-satori-ai.
5. **מגבלת הקצב חוסמת בפועל, לא מאטה.** נמדד ב-2026-07-26 על המרכזי, בלי חשבון: `sync` מושך ~100 skills לעמוד ואז מקבל `Rate limit exceeded`; שלוש הרצות הצטברו ל-**292 שורות** במסד מתוך ~7,900. וגם עם מסד מלא זה לא מספיק — **`search` עצמה יוצאת לקריאת API בכל הרצה**, ולכן ברגע שהמכסה נגמרה כל חיפוש מחזיר `Rate limit exceeded` גם כשהתוצאות כבר במסד המקומי. **בלי מפתח הכלי לא שמיש.** הרובד הבא הוא $9.99/חודש (1,000 קריאות). את המפתח שמים ב-`~/.devops-secrets` בשם `SKILLSMITH_API_KEY` (העטיפה טוענת אותו לבד) — **לעולם לא** דרך `skillsmith login`, שכותב אותו לדיסק.
6. **באג upstream ב-0.8.3:** ה-CLI פותח את `~/.skillsmith/skills.db` בלי ליצור את התיקייה, וכל sync נופל ב-ENOENT בהתקנה טרייה. `deploy-skillsmith.sh` יוצר אותה מראש.
7. **embeddings מקומיים לא מותקנים** — `@huggingface/transformers` אינו תלות של החבילה, אז הדירוג נופל ל-mock. לא מתקינים אותו (‏~100MB של onnxruntime) עד שיוכח שהדירוג הקיים לא מספיק.

## פתרון תקלות

| תסמין | סיבה |
|---|---|
| `Rate limit exceeded` בחיפוש | הרובד החינמי נגמר. `search` קוראת ל-API גם כשהמסד מלא, אז מסד מקומי לא עוקף את זה. הוסף `SKILLSMITH_API_KEY` ל-`~/.devops-secrets`. |
| `ENOENT ~/.skillsmith/skills.db` | התיקייה חסרה — `mkdir -p ~/.skillsmith` (הסקריפט עושה זאת). |
| `embeddings: mock` | צפוי. ראה סעיף 7. |
| `'<cmd>' is blocked` | פקודת כתיבה. זה תקין — עבור דרך צינור ה-vetting. |
| `Using WASM SQLite driver` | צפוי ולא מזיק (אין better-sqlite3 מקומפל). |

Slash: `/skillsmith`. סוכן: `@skillsmith`.
