---
title: "scale"
type: "skill"
tags: ["kit","skill","scale","model scale","which model","model routing","grok","openrouter"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T01:08:05.47331+00:00"
id: "af7e8785-0f02-44f6-97f3-273a51e8883d"
---

> The kit's three-tier model-routing policy v2 ("the perfect scale") — each tier offers two models with a sticky per-user default in ~/.claude/scale-prefs. 🔵 Plan = Fable 5 (claude-fable-5, default) or Opus 5 (claude-opus-5) — planning, architecture, review, PRD breakdown (main session / plan-subagent); 🟢 Build = Codex (gpt-5.6-codex, pinned fleet-wide, default) or Grok 4.6 (grok-4.6, xAI Grok CLI) — everything else via om/codex exec or scale-env grok -p; 🟣 Complex = Opus 5 (default) or Fable 5 — subagent only, after a written plan. Announce the routing at task start (ask only on genuine doubt); auto-activates on a new topic or big task — Plan mode without being asked. OpenRouter (codex profiles `or`/`grok-or`) is the universal fallback for any downed channel. Never mid-session model switching. Triggers - "scale", "model scale", "which model", "model routing", "grok", "openrouter", "route to opus", "escalate model", "pin codex model", "perfect scale", "בחר מודל", "העדפת מודל", "איזה מודל", "ניתוב מודלים", "הסלמה לאופוס", "סקייל מודלים".

# Scale v2 — the perfect scale (ניתוב תלת-שכבתי, שתי אפשרויות לשכבה)

`/scale` הוא כלל ברזל #8 — Dual-Mode + **מדיניות תלת-שכבתית** שבה לכל שכבה שני מודלים
ו**ברירת מחדל דביקה**. הניתוב הוא **per-task**, לעולם לא החלפת מודל באמצע session:
ה-session הראשי נשאר על המודל שלו; שכבות אחרות מושגות ב-*handoff* (CLI) או *delegation* (subagent).

| שכבה | אפשרויות (ברירת מחדל ראשונה) | Owns | ערוץ |
|------|------------------------------|------|-------|
| 🔵 Plan | **Fable 5** `claude-fable-5` · Opus 5 `claude-opus-5` | תכנון, ארכיטקטורה, עיצוב, חישובים, review, פירוק PRD | Fable: ה-session הראשי / Plan mode · Opus 5: plan-subagent (Agent tool, `model: opus`) |
| 🟢 Build | **Codex** `gpt-5.6-codex` (מוצמד fleet-wide) · Grok 4.6 `grok-4.6` | **כל השאר** — מימוש, ריפקטורינג, boilerplate, אופטימיזציה | `om "<task>"` · `codex exec` · `/codex:rescue` — או `scale-env grok -p "<task>" -m grok-4.6` |
| 🟣 Complex | **Opus 5** `claude-opus-5` · Fable 5 | מורכבות קיצונית **בלבד** (קריטריונים למטה) | subagent בלבד: Agent tool `model: opus` · Fable: `model: fable` (ואם אין — inherit מה-session) |

## העדפות דביקות — `~/.claude/scale-prefs`

```bash
SCALE_PLAN=fable5        # fable5|opus5
SCALE_EXEC=codex         # codex|grok
SCALE_COMPLEX=opus5      # opus5|fable5
```

- קובץ חסר / מפתח חסר ⇒ ברירות המחדל שבטבלה. הקובץ פר-משתמש ושורד sessions.
- המשתמש הביע העדפה ("מהיום ביצוע על Grok", "תתכנן על אופוס") ⇒ **עדכן את הקובץ מיד** — זו הדביקות.
- hook ה-SessionStart של הקיט (`kit-hello.sh`) מזריק את שורת ההעדפות לקונטקסט בכל session — הטבלה תמיד לפניך בלי `cat`.

## פרוטוקול הכרזה — מכריזים, לא שואלים

בתחילת כל משימה משמעותית פלוט שורה אחת, מההעדפות:

> ⚖️ ניתוב: 🔵 Fable 5 · 🟢 Codex · 🟣 Opus 5

**AskUserQuestion רק בספק אמיתי**: משימה גבולית בין שכבות, ערוץ שידוע כמושבת (אין מפתח/login),
או שההעדפה מתנגשת עם מצב בפועל. כל השאר — להכריע, להכריז, להמשיך (ר' `/work-the-queue`).

## הפעלה אוטומטית (Auto-activation)

זיהית **נושא חדש**, **מעבר נושא**, או ש**משהו גדול מתחיל** — גם אם המשתמש לא הפעיל Plan ולא `/scale`:

1. היכנס ל-Plan mode (EnterPlanMode).
2. נתב לפי `~/.claude/scale-prefs` ו**הכרז** על הניתוב.
3. שאל שאלות מכוונות (AskUserQuestion — רק מה שמשנה את התוצר).
4. כתוב תוכנית.
5. בצע לפי הסקייל: תכנון ב-🔵, מימוש ב-🟢, קטעים קיצוניים ב-🟣.

**"גדול"** = פיצ'ר חדש · ≥3 צעדים תלויים · שינוי רב-קבצים · אזור לא מוכר בריפו.
**לא מפעילים** על המשך ישיר של עבודה קיימת, תיקון קטן, או שאלה שיחתית.

## מה נחשב מורכב (🟣) — כל השאר → 🟢

- מיגרציה רב-מערכתית החוצה **≥3 שירותים/ריפואים**
- ארכיטקטורת אבטחה קריטית: auth, crypto, secrets, שינוי קונפיגורציה fleet-wide
- דיבוג ששרד **2 ניסיונות תיקון כושלים**
- ריפקטור **>20 קבצים** שבו חובה להוכיח שימור התנהגות
- תכנון פעולה בלתי-הפיכה/הרסנית (מיגרציית דאטה, rollout לכל הצי)

**כלל ההכרעה: ספק? → 🟢. השכבה המורכבת היא הסלמה, לא העדפה** — והיא מקבלת עבודה רק אחרי תוכנית כתובה מה-🔵.

## Handoff commands

```bash
# 🟢 Build — Codex (ברירת מחדל; כבר מוצמד ל-gpt-5.6-codex, בלי -m)
om "<task>"                                  # = omx team 3:executor "<task>"
codex exec --skip-git-repo-check "<task>"    # raw non-interactive
# 🟢 Build — Grok 4.6 (כשה-SCALE_EXEC=grok)
scale-env grok -p "<task>" -m grok-4.6       # xAI Grok CLI, headless; scale-env טוען מפתחות מ-~/.devops-secrets
```

🔵 על Opus 5 (כשה-SCALE_PLAN=opus5): שגר plan-subagent — Agent tool עם `model: opus` ו-prompt
תכנון (explore → design → החזר תוכנית); ה-session הראשי מתזמר ומבצע.
🟣 Complex: subagent עם `model: opus` (Opus 5) או `model: fable` (Fable 5; אם הערך לא נתמך בגרסה — inherit,
ה-session הראשי הוא ממילא Fable) + התוכנית מה-🔵. אין agent ייעודי לשכבה — מכוון (סוכן ריק על מודל יקר = פיתוי לבזבוז).

## Fallback — OpenRouter הוא רשת הביטחון של כולם

הזיהוי הוא במעמד ה-handoff: exit≠0 / 401 / 429 / 5xx בפלט ⇒ עבור שורה בטבלה, **הכרז על המעבר** בשורה אחת.

| ערוץ נפל | נסה | ואם גם הוא |
|---|---|---|
| 🟢 Codex (OpenAI) | `scale-env codex exec --profile or` (= `openai/gpt-5.6-codex` דרך OpenRouter; `-m <slug>` לדריסה) | הערוץ השני: `scale-env grok -p …` |
| 🟢 Grok (xAI) | `scale-env codex exec --profile grok-or` (= `x-ai/grok-4.6` דרך OpenRouter) | הערוץ השני: `om` / `codex exec` |
| 🔵/🟣 Anthropic | המודל-האח (Fable↔Opus 5) דרך subagent | בצע ב-🟢 ודווח |

שני הפרופילים נושאים מודל מלא (vendor-qualified) — עובדים בלי `-m` גם כשההנחיה מגיעה מכלל #8 המקוצר.

הפרופילים `or`/`grok-or` נכתבים ל-`~/.codex/config.toml` ע"י `setup-scale.sh` (בלי לגעת בפין).
מפתחות ב-`~/.devops-secrets`: `XAI_API_KEY=` · `OPENROUTER_API_KEY=`. שרת בלי מפתח ⇒ הערוץ רדום,
`--check` מדווח, והניתוב עוקף אותו.

**מפתחות אישיים — "ה-auth שלי, בכל שרת, רק אני":** `~/.devops-secrets` הוא פר-משתמש פר-שרת
ו-kit-push **בכוונה לא מסנכרן אותו**. הפצת המפתחות של היוזר שלך לכל חשבונות-המשתמש *שלך* בצי
(ורק שלך — קובץ 600 בבעלותך, משתמשים אחרים מקבלים Permission denied):

```bash
bash ~/DevOPS/secrets-push.sh XAI_API_KEY=xai-... OPENROUTER_API_KEY=sk-or-...   # מהמרכזי בלבד
bash ~/DevOPS/secrets-push.sh --check XAI_API_KEY OPENROUTER_API_KEY             # מי מחזיק מה (שמות בלבד)
bash ~/DevOPS/secrets-push.sh --remove XAI_API_KEY                               # rotation/ביטול
```

הערכים לא עוברים ב-argv המרוחק (גלוי ב-`ps`) — הם זורמים על ה-stdin של ערוץ ה-SSH. שרת בלי
חשבון בשם שלך פשוט מדווח unreachable. משתמש אחר שרוצה ערוץ חי — מפתח משלו או login אינטראקטיבי (`/grok`).

## Bootstrap — לפני handoff ראשון (חובה)

```bash
# Codex (🟢 ברירת מחדל)
command -v codex || npm i -g @openai/codex@latest oh-my-codex@latest   # kit-update עושה זאת אוטומטית
codex login status                                                     # exit 0 = מחובר
# Grok (🟢 חלופי)
command -v grok  || npm i -g @xai-official/grok@latest                 # kit-update עושה זאת אוטומטית
scale-env grok -p 'Reply ok' -m grok-4.6                               # עובד רק עם XAI_API_KEY או login
```

**לא מחובר?** שלב אינטראקטיבי חד-פעמי שרק המשתמש יכול לבצע — הדרך אותו, אל תעקוף:
`codex login` (OAuth של ChatGPT) · Grok: מפתח מ-console.x.ai אל `~/.devops-secrets` (`XAI_API_KEY=`) או `grok` login אינטראקטיבי.
עד שיש — **אל תנתב לערוץ הזה**; בצע בערוץ הזמין ואמור למשתמש מה חסר.

## The fleet pin + knobs

`setup-scale.sh` (רץ ע"י kit-update פעמיים ביום) מצמיד `model = "gpt-5.6-codex"` **top-level**
ב-`~/.codex/config.toml` — אטומית, מעל כל `[section]`, בלי לגעת ב-`[mcp_servers.*]` — ומוסיף
append-if-missing את `[model_providers.openrouter]` + `[profiles.grok-or]` + `[profiles.or]`
(לעולם לא `model_provider` top-level — זה היה דורס את הפין). גיבוי מתגלגל: `config.toml.bak-scale`.

Override פר-שרת (שורד את ה-re-pin), ב-env או ב-`~/.devops-secrets`:
`SCALE_CODEX_MODEL=<model>` (ברירת מחדל `gpt-5.6-codex`) · `SCALE_GROK_MODEL=<model>` (ברירת מחדל `grok-4.6`).
עריכה ידנית של המפתח נדרסת by design.

**MCP**: Grok CLI הוא *לקוח* MCP (כמו Claude Code) — אין "grok MCP server" להתקין. ל-OpenRouter אין MCP
רשמי, ושרת קהילתי היה מעמיס LLM שני מאחורי tool-call — **החלטה מכוונת: לא מתקינים** (מדיניות אפס-MCP-מיותרים).

## Anti-patterns

- ❌ החלפת מודל באמצע session (`/model`) כחלק מהניתוב — הניתוב הוא per-task, בהעברה/האצלה
- ❌ שליחת תכנון/ארכיטקטורה ל-🟢 — זה 🔵
- ❌ ברירת מחדל 🟣 "ליתר ביטחון" — קיצון בלבד, אחרי תוכנית
- ❌ לשאול את המשתמש איזה מודל בכל משימה — מכריזים; שואלים רק בספק
- ❌ לשכוח לעדכן `scale-prefs` כשהמשתמש הביע העדפה — זו הדביקות

## Verify

```bash
bash ~/DevOPS/setup-scale.sh --check                          # full state report (pin, profiles, codex, grok, prefs)
awk '/^\[/{exit} /^model/' ~/.codex/config.toml               # top-level pin present
grep -c '^\[mcp_servers' ~/.codex/config.toml                 # MCP blocks intact
grep -A2 'profiles.grok-or' ~/.codex/config.toml              # OpenRouter fallback profile present
codex exec --skip-git-repo-check 'Reply ok'                   # 🟢 codex live (needs codex login)
scale-env grok -p 'Reply ok' -m grok-4.6                      # 🟢 grok live (needs XAI key/login)
cat ~/.claude/scale-prefs 2>/dev/null                         # sticky prefs (empty = defaults)
```

| בעיה | פתרון |
|------|-------|
| אין `model =` ב-config | `bash ~/DevOPS/setup-scale.sh` (או המתן ל-kit-update) |
| מחרוזת מודל שגויה/עודכנה | `SCALE_CODEX_MODEL=`/`SCALE_GROK_MODEL=` ב-`~/.devops-secrets` + הרצת הסקריפט |
| MCP נשבר אחרי עריכה | שחזר מ-`~/.codex/config.toml.bak-scale` |
| `401 Unauthorized` מ-codex/grok | לא בעיית מודל — `codex login` / מפתח `XAI_API_KEY` בשרת |
| פרופיל `grok-or` לא עובד | ודא `OPENROUTER_API_KEY` ב-`~/.devops-secrets`; הפרופיל דורש `wire_api="responses"` (נכתב אוטומטית) |

## מדיה גנרטיבית — שכבה נפרדת מהניתוב הזה

שלוש השכבות למעלה מנתבות **מודלי קוד**. יצירת מדיה (תמונה/וידאו/קול/3D) לא רצה על אף אחד מהם — היא הולכת לספק ייעודי, ונשארת פעולה בתשלום שמישהו צריך לאשר:

| צורך | יעד | מסלול |
|---|---|---|
| תכנון קריאייטיב, בריף, בחירת מודל, ביקורת תוצאה | 🔵 Plan | ה-session הזה |
| הרצת ההפקה בסקריפט/batch | 🟢 Build | `om` + `higgsfield.sh` |
| יצירה/עריכה אינטראקטיבית, סטודיו, ויראליות | ספק | MCP `/higgsfield` |
| מחקר מודעות מתחרים לפני שמייצרים | ספק | MCP `/konvert` |

⚠️ שני ה-MCP האלה הם OAuth בלבד (אימות דפדפן per-host) ובתשלום — הרישום fleet-wide הוא opt-in דרך `HIGGSFIELD_MCP=1`/`KONVERT_MCP=1`. `higgsfield.sh` לא מחובר ל-kit-update: לנתיב הלילי אין משטח הוצאה.

## Related Skills

- `/codex` — ה-runtime של ערוץ Codex (om/omx, MCP wiring)
- `/grok` — ה-runtime של ערוץ Grok (התקנה, auth, headless)
- `/higgsfield`, `/konvert` — media generation + ad research (paid providers, OAuth MCP + CLI)
- `/ruflo` — dual-mode orchestration; טבלת WASM/Haiku/Sonnet שלה = שכבות פנימיות, לא זה
- `/master` — agent selection decision trees
- `/cost-optimization` — Claude-API model pricing/selection in app code

Slash: `/scale`.
