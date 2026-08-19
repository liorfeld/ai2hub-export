---
title: "grok"
type: "skill"
tags: ["kit","skill","grok","grok cli","grok 4.6","xai","x.ai","grok build"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T06:00:32.16885+00:00"
id: "1fc49d60-da8c-4102-85c3-5feea544257c"
---

> The Grok 4.6 build channel of /scale — xAI's official Grok CLI (@xai-official/grok, binary `grok`) as the alternative 🟢 execution engine alongside Codex. Headless handoff via `scale-env grok -p "<task>" -m grok-4.6` (scale-env loads XAI_API_KEY/GROK_CODE_XAI_API_KEY from ~/.devops-secrets); interactive `grok` for exploratory work. MCP client (like Claude Code) — there is no "grok MCP server" to install. Fallback when xAI is down: `scale-env codex exec --profile grok-or` (x-ai/grok-4.6 through OpenRouter). Installed fleet-wide by kit-update; auth is per-host (API key from console.x.ai or one-time interactive login). Use to run Grok tasks, wire auth, pick grok vs codex, or troubleshoot the CLI. Triggers - "grok", "grok cli", "grok 4.6", "xai", "x.ai", "grok build", "run on grok", "גרוק", "להריץ על גרוק".

# Grok — ערוץ הביצוע החלופי של הסקייל (xAI Grok CLI) 🟢

Grok 4.6 הוא האפשרות השנייה בשכבת ה-Build של `/scale` (לצד Codex). ה-CLI הרשמי של xAI
(`@xai-official/grok`) מותקן fleet-wide ע"י kit-update; הבחירה בין codex ל-grok היא ההעדפה
הדביקה `SCALE_EXEC` ב-`~/.claude/scale-prefs`.

## התקנה ואימות

```bash
command -v grok || npm i -g @xai-official/grok@latest   # kit-update עושה זאת אוטומטית
grok --version
bash ~/DevOPS/setup-scale.sh --check                     # שורות grok CLI / grok auth
```

**Auth (חד-פעמי, per-host):** מפתח API מ-console.x.ai אל `~/.devops-secrets`:

```bash
echo 'XAI_API_KEY=xai-...' >> ~/.devops-secrets && chmod 600 ~/.devops-secrets
```

`scale-env` ממפה אוטומטית `XAI_API_KEY` → `GROK_CODE_XAI_API_KEY` (המפתח שה-CLI קורא).
חלופה אינטראקטיבית: הרצת `grok` בטרמינל ו-login דרך הדפדפן (OAuth של SuperGrok/X Premium).
בלי מפתח ולא מחובר — הערוץ **רדום**: אל תנתב אליו, בצע ב-codex ודווח מה חסר.

## Handoff (מתוך Claude Code / סקריפטים)

```bash
scale-env grok -p "<task>" -m grok-4.6      # headless, מודל נעוץ per-invocation
scale-env grok -p "<task>" -m grok-4.6 --output-format streaming-json   # פלט מובנה
```

- הצמדת המודל היא **per-invocation** (`-m`) — ל-CLI אין pin בקונפיג כמו של codex. ברירת המחדל
  של הצי: `grok-4.6`; override: `SCALE_GROK_MODEL=` ב-`~/.devops-secrets`.
- `grok` הוא לקוח MCP מלא (קורא AGENTS.md, skills, hooks) — **אין** "grok MCP server" להתקין,
  וזו החלטה מכוונת של הקיט (מדיניות אפס-MCP-מיותרים; ר' `/scale`).

## Fallback — xAI נפל?

```bash
scale-env codex exec --profile grok-or "<task>"   # x-ai/grok-4.6 דרך OpenRouter (wire_api=responses)
```

הפרופיל `grok-or` נכתב ל-`~/.codex/config.toml` ע"י `setup-scale.sh`; דורש `OPENROUTER_API_KEY=`
ב-`~/.devops-secrets`. גם הוא נפל → הערוץ השני של השכבה: `om` / `codex exec`.

## מתי grok ומתי codex (כשאין העדפת משתמש)

ברירת המחדל היא **codex**. grok נבחר כשהמשתמש ביקש (`SCALE_EXEC=grok`), כש-codex מושבת
(אין login / OpenAI down), או כשהמשתמש רוצה השוואת מימושים (שני הערוצים על אותה משימה).

| בעיה | פתרון |
|------|-------|
| `grok: command not found` | `npm i -g @xai-official/grok@latest` או המתן ל-kit-update |
| 401 מ-grok | אין מפתח — `XAI_API_KEY=` ב-`~/.devops-secrets` (או `grok` login אינטראקטיבי) |
| xAI down / 5xx | `scale-env codex exec --profile grok-or "<task>"` |
| רוצים מודל אחר | `SCALE_GROK_MODEL=<model>` ב-`~/.devops-secrets` + `bash ~/DevOPS/setup-scale.sh` |

## Related Skills

- `/scale` — מדיניות הניתוב המלאה (שכבות, העדפות, fallback)
- `/codex` — הערוץ המקביל בשכבת ה-Build

Slash: `/grok`.
