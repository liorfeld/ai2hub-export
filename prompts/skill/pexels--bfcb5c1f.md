---
title: "pexels"
type: "skill"
tags: ["kit","skill","pexels","b-roll","broll","stock video","stock footage","transition clip"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T06:08:09.535352+00:00"
id: "bfcb5c1f-0e59-42bd-a946-08a87e68c2f2"
---

> Pexels — free royalty-free 4K stock video (B-roll) and photos for the agent to pull transition clips, backgrounds, and imagery by keyword, instead of generating AI slop. Triggers - "pexels", "b-roll", "broll", "stock video", "stock footage", "transition clip", "royalty free video", "background video", "סרטון רקע", "קטע מעבר", "צילומי מלאי".

# Pexels — B-roll וצילומים חינם

וידאו 4K וצילומים ללא זכויות יוצרים, בחינם. [pexels.com/api](https://www.pexels.com/api/) · חבילה: `mcp-pexels@1.0.2` (MIT, **Node≥20**, נעוץ אחרי audit).

> ⚖️ **חובת קרדיט — לא אופציונלית.** כל שימוש בנכס מחייב קישור בולט ל-Pexels וקרדיט לצלם: `Photo by <שם> on Pexels` (מקושר לעמוד הנכס). אין לשכפל את הפונקציונליות הליבתית של Pexels. אי-מתן קרדיט = הפרת ToS.

## הרשמה

מפתח חינם ומיידי ב-https://www.pexels.com/api/ → הוסף ל-`~/.devops-secrets`:
```bash
echo 'PEXELS_API_KEY=your_key_here' >> ~/.devops-secrets && chmod 600 ~/.devops-secrets
bash ~/DevOPS/deploy-creative-stack.sh    # ירשום את ה-MCP; restart ל-Claude/Codex
```
בלי מפתח — הקיט **לא רושם** את ה-MCP בכלל (אף פעם לא רושמים MCP בלי credential).

## מגבלות free tier

| מגבלה | ערך |
|-------|-----|
| בקשות לשעה | 200 |
| בקשות לחודש | 20,000 |
| עלות | 0 |
| הגדלה | לבקש מ-Pexels (מותנה בהצגת attribution) |

## כלי ה-MCP

`pexels_search_videos` · `pexels_popular_videos` · `pexels_get_video` · `pexels_search_photos` · `pexels_curated_photos` · `pexels_get_photo` · `pexels_featured_collections` · `pexels_collection_media` · `pexels_my_collections`

## Fallback ב-REST (בלי MCP)

```bash
KEY="$(grep -E '^PEXELS_API_KEY=' ~/.devops-secrets | cut -d= -f2-)"

# חיפוש וידאו (B-roll)
curl -s -H "Authorization: $KEY" \
  "https://api.pexels.com/videos/search?query=ocean+sunset&per_page=5&size=large" \
  | jq -r '.videos[] | "\(.user.name)\t\(.url)\t\([.video_files[]|select(.quality=="hd")][0].link)"'

# וידאו פופולרי
curl -s -H "Authorization: $KEY" "https://api.pexels.com/videos/popular?per_page=5" | jq '.videos[].url'

# חיפוש תמונות (שים לב: נתיב שונה — /v1/)
curl -s -H "Authorization: $KEY" "https://api.pexels.com/v1/search?query=coffee&per_page=5" \
  | jq -r '.photos[] | "\(.photographer)\t\(.url)\t\(.src.large2x)"'
```
כותרת `X-Ratelimit-Remaining` בתשובה מראה כמה בקשות נשארו בחלון.

## Workflow טיפוסי (B-roll לוידאו)

1. חפש לפי מילות מפתח + `size=large` (4K/HD).
2. סנן לפי `duration` ו-`width/height` (אנכי לרילס: `orientation=portrait`).
3. הורד את `video_files[].link` באיכות הגבוהה ביותר שסבירה למשקל.
4. **רשום את הקרדיט** (`user.name` + `url`) לתוך קרדיטים/תיאור.
5. הרכב עם `/remotion` (⚠️ בדוק רישוי) או ffmpeg.

## פתרון תקלות

| בעיה | פתרון |
|------|-------|
| `NOT registered (no PEXELS_API_KEY)` | הוסף את המפתח ל-`~/.devops-secrets` והרץ `deploy-creative-stack.sh` |
| `SKIP (key set, but node < 20)` | `mcp-pexels` דורש Node≥20 — שדרג את המארח |
| 429 / בקשות נגמרו | חרגת מ-200/שעה — המתן או בקש הגדלה |
| ה-MCP לא בסשן | restart ל-Claude/Codex אחרי הרישום |
| הקליפ לא בזכויות שרוצים | Pexels License מתירה מסחרי ללא קרדיט *חוקית*, אבל ה-ToS דורש קרדיט — תמיד לתת |

## Related Skills

`/creative-stack` (master) · `/nano-banana` · `/remotion` · `/qa`
