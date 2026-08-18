---
title: "trend-scout"
type: "skill"
tags: ["kit","skill","trend-scout","trends digest","trending repos","trendshift","github trending","daily repo report"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T01:08:05.47331+00:00"
id: "98b9c8e6-22c2-4ab2-b377-69ef52ac1f23"
---

> Deploy & operate Trend-Scout — a twice-daily (07:00+19:00 Asia/Jerusalem) digest that reads trendshift.io + github.com/trending + a GitHub topic sweep (15 AI/agent topics, so repos that are already big — not only ones spiking today — are seen), dedups against what the kit already adopted or explicitly skipped (skills/ + agents/ + CHANGELOG), enriches the genuinely-new repos (license/lang/stars/what-it-does via the GitHub API), and emails/messages (email always; Telegram + WhatsApp when creds are set) a report of what's new · trending on BOTH · worth installing. Triggers - "trend-scout", "trends digest", "trending repos", "trendshift", "github trending", "daily repo report", "what's trending", "מה טרנדי", "דוח ריפואים", "סריקת טרנדים".

# Trend-Scout — דוח טרנדים יומי

מנוע שקורא **trendshift.io** + **github.com/trending** פעמיים ביום (07:00/19:00 שעון ישראל), מנפה מול מה שכבר בקיט (adopted) או נדחה (skipped, עם הסיבה מה‑CHANGELOG), מעשיר את החדשים (רישיון/שפה/כוכבים/מה‑זה דרך GitHub API), ושולח דוח: **מה חדש · מה בשני המקורות · מה שווה להתקין**.

## שלושה מקורות — ולמה השלישי חובה
| מקור | מה הוא רואה |
|------|-------------|
| `github.com/trending` (daily+weekly) | מה **עולה** היום — דירוג לפי כוכבים שנצברו בחלון |
| `trendshift.io` | אותו דבר, רשימה מקבילה |
| **GitHub Search API** (15 topics) | מה **גדול וחי** — `topic:claude-code/agent-skills/mcp-server` (ליבת קיט) + AI רחב (`llm`, `rag`, `ai-agent`, `agentic-ai`, `llmops`…), `stars:>500`, `pushed:>30d` |

שני הראשונים עיוורים לכל מה שכבר עשה את הספייק שלו: `affaan-m/ECC` (240K★, נדחף יומית, בול בתחום)
לא הופיע בהם **אף פעם**. החיפוש הנושאי הוא מה שתופס אותו. ~194 ריפואים לריצה, הדוח מציג 20 —
ולכן יש **ניקוד**: `log10(כוכבים)` + בונוס ריבוי-מקורות + בונוס topic ליבתי + טריות `pushed`, ארכיון יורד.
Search API = 10 בקשות/דקה בלי טוקן → 7 שניות בין שאילתות, ריצה ~2 דקות. `GITHUB_TOKEN` מקצר ל-2 שניות.

## התקנה (deploy-on-demand, נוד נבחר — בד"כ central)
```bash
bash ~/DevOPS/deploy-trend-scout.sh            # מתקין cron 07:00+19:00, אימייל עובד מיד
bash ~/DevOPS/deploy-trend-scout.sh --run-now  # + הרצה מיידית
bash ~/DevOPS/deploy-trend-scout.sh --check
```
Python stdlib בלבד (בלי venv/תלויות). לוגיקה: `trend-scout/trend-scout.py` (capture+dedup+דוח) · `send.sh` (ערוצים) · `run.sh` (cron).

## ערוצים (מדורג — אימייל תמיד, השאר עם מפתחות)
`~/.config/trend-scout/env`:
- **אימייל** — עובד מיד דרך `lib-notify.sh` (creds קיימים ב-central).
- **Telegram** — `TELEGRAM_BOT_TOKEN` + `TELEGRAM_CHAT_ID` (BotFather).
- **WhatsApp** — `GREEN_API_ID/TOKEN/URL` + `ALERT_CHAT` (כמו site-health) או OpenWA.

## כפתור "install intake" (אבטחה)
כל מועמד בדוח נושא קישור שמפעיל **intake מבוקר‑אישור** על central (מחקר+רישיון+אבטחה → branch לעיון), **לא** התקנה עיוורת לצי. מופעל רק כש‑`TREND_ENDPOINT_BASE` מוגדר (Stage 3). ההפצה לצי נשארת פעולת‑אדם.

## Scale (כלל #8)
capture = דטרמיניסטי (urllib+regex). נרטיב "worth installing" = ניתוח דרך `claude -p` המקומי (🔵) עם fallback דטרמיניסטי. dedup מבוסס slug מול הקיט.
