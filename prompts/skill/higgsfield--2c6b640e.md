---
title: "higgsfield"
type: "skill"
tags: ["kit","skill","higgsfield","higgsfield.ai","generate video","text to video","image to video","kling"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T01:08:05.47331+00:00"
id: "2c6b640e-87ed-486d-b198-7e370cd86200"
---

> Higgsfield AI — generative media platform (image, video, audio, voice, 3D, TikTok publishing, marketing/DTC ad studio, virality prediction) wired into the kit through BOTH of its interfaces - the hosted OAuth MCP at mcp.higgsfield.ai/mcp for interactive work, and a real REST CLI (higgsfield.sh, curl+jq, no Node) over platform.higgsfield.ai for scripted/batch generation. Triggers - "higgsfield", "higgsfield.ai", "generate video", "text to video", "image to video", "kling", "veo", "sora", "seedance", "upscale video", "remove background", "lipsync", "voice clone", "3d model from image", "virality", "tiktok publish", "יצירת וידאו", "וידאו מתמונה", "הכפלת רזולוציה".

# Higgsfield — יצירת מדיה (MCP + CLI)

[higgsfield.ai](https://higgsfield.ai) הוא צומת אחד לעשרות מודלים גנרטיביים (image · video · audio/voice · 3D) פלוס סטודיו שיווקי, חיזוי ויראליות ופרסום ל-TikTok. הקיט מחבר אותו בשני מסלולים — **MCP** לעבודה אינטראקטיבית ו-**CLI** לסקריפטים ו-batch.

> **שני ארנקים נפרדים.** מנוי/קרדיטים של `mcp.higgsfield.ai` (הקונקטור) ≠ קרדיטי API של `platform.higgsfield.ai` (ה-CLI). קניית האחד לא מטעינה את השני. `balance` דרך ה-MCP מדווח רק על הראשון.

## 1. MCP — הקונקטור (אינטראקטיבי)

```bash
bash ~/DevOPS/setup-media-mcp.sh --check              # מצב בלבד
bash ~/DevOPS/setup-media-mcp.sh --wire higgsfield    # רישום ב-Claude + Codex
# ואז אימות פעם אחת לכל מארח:
#   Claude Code:  /mcp → higgsfield → Authenticate
#   Codex:        codex mcp login higgsfield
```

`https://mcp.higgsfield.ai/mcp` · OAuth 2.1 בלבד (DCR + PKCE, דרך דפדפן). **אין API key ל-MCP** — לכן ההרשמה fleet-wide היא opt-in (`HIGGSFIELD_MCP=1` ב-`~/.devops-secrets`) ולא ברירת מחדל: שרת headless שאף אחד לא יאמת בו יקבל MCP שמחזיר 401 בכל session.

**אין כפילות ב-Claude.** אם החשבון כבר חושף את השרת כקונקטור של claude.ai (‏`claude.ai higgsfield` ב-`claude mcp list`), Claude Code **כבר מחובר דרכו** — רישום HTTP מקומי נוסף רק מוסיף שורה שנייה שתקועה ב-`Needs authentication` ליד אחת שעובדת. `setup-media-mcp.sh` מזהה את זה ומדלג על צד Claude; Codex כן נרשם, כי לו אין קונקטורים.

**התחברות Codex ממארח מרוחק:** פורט ה-callback נעוץ ל-1455 (`mcp_oauth_callback_port` ב-`~/.codex/config.toml`, נכתב ע"י הסקריפט). בלי נעיצה codex מאזין על פורט אקראי ואי אפשר לתעל אליו — אז זה החסם האמיתי בכל השרתים. הזרימה:

```bash
# מהמחשב עם הדפדפן, לא מהשרת:
ssh -L 1455:127.0.0.1:1455 <user>@<host>
# ובאותו חלון, על השרת:
codex mcp login higgsfield        # פותח URL — להעתיק לדפדפן ולאשר
```

### מה יש בו (המשפחות שבאמת בשימוש)

| משפחה | כלים |
|---|---|
| יצירה | `generate_image` · `generate_video` · `generate_audio` · `generate_3d` · `models_explore` (בחירת מודל לפי מטרה) |
| עריכה על נכס קיים | `upscale_image`/`upscale_video` (2K/4K) · `outpaint_image` · `reframe` (יחס תצוגה) · `remove_background` · `motion_control` (recast/puppeteer/motion-transfer) |
| קול | `create_voice` · `list_voices` · `voice_change` · `dubbing` |
| סטודיו | `shorts_studio_*` · `personal_clipper_*` · `show_marketing_studio` (DTC Ads) · `explainer_video` + `get_explainer_presets` |
| ניתוח | `virality_predictor` · `video_analysis_*` |
| חשבון | `balance` · `transactions` · `show_generations` · `show_medias` · `job_status`/`job_display` |
| הפצה | `tiktok_connect` · `tiktok_prepare_publish` · `tiktok_publish` · `tiktok_music_trending` |
| בנייה | `create_website`/`deploy_website`/`publish_website` · `deploy_game`/`publish_game` · `sandbox_exec` · `website_db`/`website_secrets` |

**דפוס חובה של הספק:** לפני וידאו רב-שלבי לפי בריף (explainer, פרסומת, UGC, פודקאסט) קוראים `get_workflow_instructions` בלי ארגומנט (קטלוג) ואז עם השם; לפני משחק — `get_game_creation_instructions`. אל תמציא את הזרימה, היא מגיעה מהשרת.

### מה דורש אישור אנושי לפני הרצה

- `tiktok_publish` — **פרסום פומבי** בחשבון אמיתי. תמיד לאשר את הקליפ, הכיתוב וההאשטגים מול המשתמש לפני קריאה.
- `deploy_website` / `publish_website` / `deploy_game` / `publish_game` — מעלים תוכן לתשתית של הספק ופותחים כתובת ציבורית.
- `sandbox_exec` / `website_db` / `website_secrets` — הרצת קוד וסודות אצל צד ג׳. אל תעביר סודות של לקוחות.
- `show_plans_and_credits` / `confirm_billing_purchase` — מסכי רכישה. פותחים רק כשהמשתמש ביקש לקנות.

## 2. CLI — `higgsfield.sh` (סקריפטים, cron, batch)

Higgsfield לא מפרסם CLI רשמי (יש SDK ל-Python ו-`@higgsfield/client` ל-JS), אז זה ה-CLI: `curl` + `jq`, בלי Node, עובד גם על מארחי Node-18.

```bash
bash ~/DevOPS/higgsfield.sh --check                    # תלויות + מפתחות + עלות. לא מוציא כסף
bash ~/DevOPS/higgsfield.sh models                     # מזהי מודלים מוכרים
bash ~/DevOPS/higgsfield.sh image "ספל אדום על שיש" -o mug.jpg --aspect 1:1
bash ~/DevOPS/higgsfield.sh video --image https://…/a.jpg "מצלמה נעה שמאלה" -o clip.mp4
bash ~/DevOPS/higgsfield.sh submit <model_id> '{"prompt":"…"}'   # כל מודל מהגלריה
bash ~/DevOPS/higgsfield.sh status <request_id>
bash ~/DevOPS/higgsfield.sh wait   <request_id> -o out.mp4       # המשך אחרי timeout
bash ~/DevOPS/higgsfield.sh cancel <request_id>                  # רק בזמן queued
```

מפתחות: `HIGGSFIELD_API_KEY` + `HIGGSFIELD_API_SECRET` ב-`~/.devops-secrets` (או env), מ-[cloud.higgsfield.ai](https://cloud.higgsfield.ai). הכותרת היא `Authorization: Key <key>:<secret>` — שניהם, לא רק המפתח.

**עובדות חיוב שנמדדו מהתיעוד של הספק:**
1. ה-API הוא **אסינכרוני**: POST מחזיר `request_id` ואז pollים את `/requests/{id}/status`. הסטטוסים: `queued` · `in_progress` · `nsfw` · `failed` · `completed`.
2. **רק `completed` מחויב.** `failed` ו-`nsfw` מוחזרים לקרדיט אוטומטית.
3. **הקבצים נשמרים 7 ימים בלבד (מינימום).** תמיד `-o` — התוצאה תרד מהשרת שלהם ואי אפשר לשחזר.
4. ביטול אפשרי רק ב-`queued`. ברגע `in_progress` — משלמים.
5. מודלי וידאו הם **image-to-video**: חייבים `--image <URL ציבורי>`, לא נתיב מקומי. אין תמונה? קודם `higgsfield.sh image … -o` והעלאה, או `generate_image` דרך ה-MCP.

**לא מחובר ל-kit-update.** בדיוק כמו `nano-banana.sh` — לנתיב הלילי האוטומטי אסור שיהיה משטח הוצאה כספית.

### CLI מעל ה-MCP (כשיש מנוי ולא מפתח API)

```bash
bash ~/DevOPS/mcp-http.sh tools higgsfield
bash ~/DevOPS/mcp-http.sh call  higgsfield balance '{}'
```
משתמש ב-token שהלקוח (Codex/Claude) כבר הנפיק. פירוט ב-[/konvert](KONVERT.md) — שם זה המסלול היחיד.

## 3. ניתוב (סקייל)

| שלב | מסלול |
|---|---|
| בריף יצירתי, בחירת מודל, סטוריבורד, ביקורת תוצאה | 🔵 Plan (Fable 5 / Opus 5) — ה-session הזה |
| batch, לולאות, סקריפט הפקה, שילוב ב-pipeline | 🟢 Build — `gpt-5.6-codex` דרך `om` (ברירת מחדל) או Grok 4.6 דרך `/grok`, + `higgsfield.sh` |
| שיחה עם הקונקטור (סטודיו, ויראליות, TikTok) | 🔵 עם ה-MCP — הכלים אינטראקטיביים ומחזירים widgets |

`/scale` הוא ההחלטה, לא ה-session. אל תחליף מודל באמצע.

## 4. איפה זה יושב מול שאר הקיט

- **`/creative-stack`** — GSAP (אנימציה), Pexels (B-roll קיים, קרדיט חובה), nano-banana (PNG שקוף). Higgsfield משלים אותם ביצירת חומר **חדש** (וידאו/קול/3D) שאין לו מקור stock.
- **`/konvert`** — מחקר קריאייטיב פרסומי. הזרימה הטבעית: konvert מוצא מה עובד → higgsfield מייצר את הנכס.
- **`/remotion`** — הרכבת וידאו ב-React. Higgsfield מייצר shots, Remotion מרכיב אותם (⚠️ רישיון קנייני).
- **`/video-script-writing`, `/image-prompt-engineering`** — הטקסט והפרומפט לפני היצירה.

## 5. פתרון תקלות

| תסמין | סיבה / פתרון |
|---|---|
| `401` מה-MCP | ה-token פג. `/mcp` → Authenticate, או `codex mcp login higgsfield`. הספק ממליץ להסיר ולהוסיף מחדש אם reconnect לא מפעיל login. |
| שתי שורות ב-`claude mcp list` — אחת Connected ואחת `Needs authentication` | הקונקטור של claude.ai כבר עובד, והרישום המקומי הוא כפילות מיותרת. `claude mcp remove higgsfield -s user`. מ-v1.25.1 הסקריפט לא יוצר את הכפילות מלכתחילה. |
| `codex mcp login` פותח URL ואז נתקע | ה-callback חוזר ל-`127.0.0.1:1455` **על השרת**. בלי `ssh -L 1455:127.0.0.1:1455` הדפדפן שלך לא מגיע אליו. ודא שהפורט נעוץ: `setup-media-mcp.sh --check`. |
| `timed out waiting for OAuth callback` | לקישור יש דדליין של כמה דקות, ומרגע שהתהליך מת הקישור לא תקף גם אם הדף עדיין פתוח. הרץ שוב וקח URL חדש. |
| קריאה ל-שרת אחד מחזירה 401 אחרי שהתחברת לשניים | היה באג ב-`mcp-http.sh` עד v1.25.2: הוא לקח את ה-token הראשון ב-`~/.codex/.credentials.json` במקום את זה של ה-`server_url` המבוקש. עדכן את הקיט. |
| `isError: true` בתשובת כלי | שגיאת כלי, לא תעבורה — קרא את `structuredContent.error`. `mcp-http.sh` מזהה זאת ומחזיר rc=1. |
| `no request_id in response` ב-CLI | מפתח/סוד שגויים או `model_id` לא קיים. `higgsfield.sh models` ואז הגלריה ב-cloud.higgsfield.ai. |
| נתקע ב-`queued` עד timeout | לא אבד — `higgsfield.sh wait <request_id> -o out.mp4` ממשיך מאותה נקודה. |
| הקישור שהוחזר מת | עברו 7 ימים. חובה `-o` בכל הרצה. |
| `nsfw` | מודרציה — לא חויבת. נסח מחדש; המדיניות שונה בין מודלים. |
| התשובה דוחפת שדרוג/קרדיטים | `show_plans_and_credits` הוא כלי מכירה בכוונה. פתח רק לבקשת המשתמש. |
| `codex mcp add` "נכשל" אבל הרישום קיים | התנהגות מוכרת: codex כותב את הרשומה ואז **פותח מיד זרימת OAuth** וממתין ל-callback. בהרצה לא-אינטראקטיבית זה נחתך ב-timeout וקוד היציאה מדווח על ה-login שנכשל, לא על הרישום. `setup-media-mcp.sh` בודק את קיום הרשומה ולא את קוד היציאה. |

Slash: `/higgsfield`. סוכן: `@higgsfield`. הקמה: `setup-media-mcp.sh`. CLI: `higgsfield.sh`, `mcp-http.sh`.
