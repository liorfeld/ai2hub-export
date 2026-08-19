---
title: "nano-banana"
type: "skill"
tags: ["kit","skill","nano banana","nano-banana","gemini image","generate image","transparent png","transparent background"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T06:06:43.030774+00:00"
id: "aae601c6-7d31-4582-93df-da8141ef9d28"
---

> Generate images with the Gemini API and turn them into TRANSPARENT PNGs for websites and video overlays. Triggers - "nano banana", "nano-banana", "gemini image", "generate image", "transparent png", "transparent background", "green screen", "chroma key", "remove background", "overlay element", "רקע שקוף", "מסך ירוק", "הסרת רקע", "אלמנט גרפי".

# nano-banana — תמונות Gemini על רקע שקוף

יצירת אלמנטים גרפיים בכל סגנון, שמשתלבים בכל אתר או וידאו. הכלי: `~/DevOPS/nano-banana.sh` (bash + curl + jq + ffmpeg בלבד).

> 💰 **בתשלום, ללא free tier.** כל תמונה מחויבת לחשבון ה-Gemini שלך: `gemini-2.5-flash-image` **$0.039** · `gemini-3.1-flash-image` **$0.067** · `gemini-3-pro-image` **$0.134**. הכלי מדפיס את העלות **לפני** כל חיוב, מייצר **תמונה אחת** בכל הרצה, ואין לו flag של batch/loop. **הוא לא מחווט ל-kit-update** — לריצה הלילית אין משטח חיוב.

> 🔍 **SynthID.** כל תמונה שיוצאת מ-Gemini נושאת watermark בלתי-נראה. אסור להציג את הפלט כ"נקי מ-watermark".

## למה green-screen ולא "רקע שקוף"?

מודלי התמונה של Gemini מחזירים **RGB שטוח, בלי ערוץ אלפא**. בקשה ל"רקע שקוף" מחזירה לבן מלא או ציור של משבצות שקיפות — לא שקיפות אמיתית. הפתרון היחיד האמין:

1. הפרומפט מבקש רקע ירוק **אחיד** `#00FF00`, מואר אחיד, בלי צללים על הרקע.
2. `ffmpeg -vf "colorkey=0x00FF00:0.30:0.15,despill"` מסיר את הירוק ומנקה את ההילה בקצוות.

`--transparent` עושה את שניהם אוטומטית.

## שימוש

```bash
bash ~/DevOPS/nano-banana.sh --check                     # תלויות + מפתח + טבלת עלות (אפס חיוב)

bash ~/DevOPS/nano-banana.sh "כוכב מפץ גדול, סגנון קומיקס, קווי מתאר עבים" \
  -o /tmp/star.png --transparent                          # PNG שקוף

bash ~/DevOPS/nano-banana.sh "product shot of a red ceramic mug, studio lighting" \
  -o /tmp/mug.png --model gemini-3-pro-image              # איכות גבוהה, בלי שקיפות
```

⚠️ אל תריץ בתוך `~/DevOPS` — `*.png` לא ב-gitignore. השתמש ב-`-o` לנתיב scratch.

## הקמה

```bash
echo 'GEMINI_API_KEY=your_key' >> ~/.devops-secrets && chmod 600 ~/.devops-secrets   # https://aistudio.google.com/apikey
bash ~/DevOPS/deploy-creative-stack.sh --install-ffmpeg    # רק במארח שבו באמת מייצרים גרפיקה
bash ~/DevOPS/nano-banana.sh --check
```

## אימות שהשקיפות אמיתית

```bash
ffprobe -v error -select_streams v:0 -show_entries stream=pix_fmt -of csv=p=0 /tmp/star.png
# צפוי: rgba (ולא rgb24)
```

## פתרון תקלות

| בעיה | פתרון |
|------|-------|
| `GEMINI_API_KEY not set` | הוסף ל-`~/.devops-secrets` (או `export`) — מפתח ב-https://aistudio.google.com/apikey |
| `--transparent needs ffmpeg` | `bash ~/DevOPS/deploy-creative-stack.sh --install-ffmpeg` |
| הילה ירוקה סביב האובייקט | העלה סבילות: ערוך את `colorkey=0x00FF00:0.30:0.15` — הפרמטר השני (0.30) הוא similarity |
| חורים שקופים בתוך האובייקט | האובייקט מכיל ירוק. בקש בפרומפט "no green in the subject", או השתמש ברקע מגנטה `#FF00FF` |
| `no image in response` | הפרומפט נחסם ע"י מסנני בטיחות, או שם מודל שגוי — הכלי מדפיס את `.error.message` |
| הרקע לא אחיד | הוסף לפרומפט "flat, evenly lit, single-tone background, no gradient" |

## Related Skills

`/creative-stack` (master) · `/pexels` · `/remotion` · `/design`
