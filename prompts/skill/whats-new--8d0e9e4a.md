---
title: "whats-new"
type: "skill"
tags: ["kit","skill","whats-new","what","whats","new"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T01:08:05.47331+00:00"
id: "8d0e9e4a-1b98-4e9b-b08a-5f51368c5f72"
---

> The kit's "while you were away" session greeting — a SessionStart hook (kit-hello.sh, registered fleet-wide by setup-whats-new.sh via kit-update) that injects two things into every new Claude Code session: the sticky /scale routing line (every session), and a ONE-TIME styled Hebrew greeting ("היי, בזמן שלא היית ליאור יצר... ושיתף אותך") listing skills distributed since this user's last session. State per user in ~/.claude/.kit-last-seen; skill set read consumer-side from ~/.claude/skills/<name>/SKILL.md after profile filtering, so each host greets only about skills that exist for it. First-ever run seeds silently except names in the whats-new-announce manifest. Use to understand, test, or troubleshoot the greeting, or to make a release announce itself. Triggers - "whats-new", "what's new", "בזמן שלא היית", "kit hello", "session greeting", "לא קיבלתי הודעה על skill", "greeting hook".

# What's New — "בזמן שלא היית" (ברכת session על הפצות חדשות) 👋

משתמש שמתחבר אחרי ש-`kit-push` הפיץ skills חדשים מקבל בתחילת ה-session הבא שלו ברכה
מעוצבת בעברית שמציגה אותם — פעם אחת בלבד. בנוסף, שורת הניתוב של `/scale` מוזרקת לקונטקסט
**בכל** session (החצי הדטרמיניסטי של "מכריזים, לא שואלים").

## איך זה עובד

| רכיב | תפקיד |
|------|-------|
| `setup-whats-new.sh` | רושם את ה-hook ב-`~/.claude/settings.json` → `hooks.SessionStart` (רץ מ-kit-update, אידמפוטנטי, לא נוגע ב-hooks אחרים) |
| `kit-hello.sh` | ה-payload: קורא את סט ה-skills בצד הצרכן (`~/.claude/skills/<name>/SKILL.md`), משווה ל-state, פולט `additionalContext` |
| `~/.claude/.kit-last-seen` | state פר-משתמש: `version=` + `skills=` (CSV ממוין). נכתב אטומית אחרי כל הרצה ⇒ הברכה לא חוזרת |
| `whats-new-announce` | manifest בשורש הקיט: שמות שיוכרזו גם בהרצה ראשונה-אי-פעם (בלי state). מתעדכן פר-release ע"י `/distribute-skill` |

- **זריעה שקטה**: אין state ⇒ לא שופכים 135 skills — מכריזים רק על שמות מה-manifest.
- **פר-משתמש אוטומטית**: ה-state ב-`$HOME` ⇒ כל משתמש בשרת מרובה-משתמשים מקבל ברכה משלו.
- **פר-פרופיל אוטומטית**: הסט נקרא אחרי סינון `EXCLUDE_PATTERN` ⇒ שרת n8n לא יקבל ברכה על skill עיצוב שלא הותקן אצלו.
- **race ידוע**: שני sessions ראשונים במקביל ⇒ ברכה כפולה (מקובל, לא מתוקן בכוונה).

## מה מוזרק בפועל

- `<scale-prefs>` — שורת הניתוב מ-`~/.claude/scale-prefs` (או ברירות מחדל), כל session.
- `<kit-whats-new>` — רק כשיש חדש: הנחיה לפתוח בברכה `"👋 היי, בזמן שלא היית ליאור יצר את הסקייל הבא ושיתף אותך בו:"` + עד 5 פריטים (פקודה + תיאור מה-frontmatter) + "+N נוספים".

## בדיקה / אבחון

```bash
bash ~/DevOPS/kit-hello.sh | python3 -m json.tool          # מה יוזרק ל-session הבא (וגם מעדכן state!)
cat ~/.claude/.kit-last-seen                                # מה המשתמש "כבר ראה"
jq '.hooks.SessionStart' ~/.claude/settings.json            # ה-hook רשום?
rm ~/.claude/.kit-last-seen                                 # איפוס — ה-session הבא יכריז לפי ה-manifest
```

⚠️ הרצה ידנית של `kit-hello.sh` **צורכת** את הברכה (מעדכנת state). לבדיקה בלי לצרוך:
`KIT_HELLO_STATE=/tmp/x bash ~/DevOPS/kit-hello.sh`.

| בעיה | סיבה שכיחה |
|------|------------|
| לא הופיעה ברכה אחרי הפצה | ה-skill סונן ע"י פרופיל השרת · מישהו הריץ kit-hello ידנית · ה-hook לא רשום (`setup-whats-new.sh`) |
| ברכה על skills ישנים | ה-state נמחק/אופס — זו התנהגות תקינה של הרצה-ראשונה + manifest |
| שום דבר לא מוזרק | אין python3 · settings.json לא תקין (ה-merge fail-closed) |

## Related

- `/scale` — שורת הניתוב שמוזרקת כל session
- `/distribute-skill` — שלב עדכון ה-manifest בהפצה
- `/kit-verify` — check #6 (hooks) מוודא שה-hook רץ ומגיע למודל

Slash: `/whats-new`.
