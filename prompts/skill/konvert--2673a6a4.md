---
title: "konvert"
type: "skill"
tags: ["kit","skill","konvert","usekonvert","ad research","competitor ads","winning ads","ad library"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T06:04:49.917818+00:00"
id: "2673a6a4-0363-489c-8950-de1215829b7c"
---

> Konvert (usekonvert.com) — paid ad-creative intelligence - search a library of real running ads by niche/angle/format/runtime, pull competitor brands and their top performers, analyze hooks and layouts, and generate new ad images anchored on what already converts. Triggers - "konvert", "usekonvert", "ad research", "competitor ads", "winning ads", "ad library", "swipe file", "creative brief", "hooks", "ad angles", "מודעות מתחרים", "מחקר קריאייטיב", "מודעה שעובדת".

# Konvert — מודיעין קריאייטיב פרסומי (MCP)

[usekonvert.com](https://usekonvert.com) הוא מאגר מודעות רצות אמיתיות עם חיפוש לפי נישה, זווית, פורמט ומשך-ריצה, פלוס יצירת מודעות חדשות שמעוגנות במה שכבר עובד. הקיט מחבר אותו כ-MCP.

> **MCP הוא הממשק היחיד.** אין REST API ציבורי, אין CLI רשמי, ואין תיעוד מפתחים — `usekonvert.com/docs` ו-`/api` מחזירים 404, ול-`api.usekonvert.com` תעודת TLS פגה (נמדד 2026-07-28). **חבילת ה-npm בשם `konvert` היא ממיר מטבעות של מישהו אחר — לא הכלי הזה. לא להתקין.**

## הקמה

```bash
bash ~/DevOPS/setup-media-mcp.sh --check
bash ~/DevOPS/setup-media-mcp.sh --wire konvert
#   Claude Code:  /mcp → konvert → Authenticate
#   Codex:        codex mcp login konvert
```

`https://mcp.usekonvert.com/mcp` · OAuth 2.1 (DCR + PKCE, scopes `mcp:read`/`mcp:write`). fleet-wide זה opt-in (`KONVERT_MCP=1` ב-`~/.devops-secrets`) — אימות הוא פעולת דפדפן לכל מארח, ובלעדיו ה-MCP רק מחזיר 401.

**אם החשבון כבר מחובר כקונקטור של claude.ai** (‏`claude.ai konvert` ב-`claude mcp list`), Claude Code עובד דרכו והסקריפט לא מוסיף רישום מקומי כפול. ההתחברות נחוצה ל-Codex ול-`mcp-http.sh`:

```bash
# 1. פותחים את פורט ה-callback מהמחשב עם הדפדפן (VS Code: PORTS → Forward a Port → 1455)
ssh -L 1455:127.0.0.1:1455 <user>@<host>
# 2. על השרת — משאירים רץ, הוא ממתין ל-callback:
codex mcp login konvert
```

הפורט נעוץ ל-1455 ב-`~/.codex/config.toml` (`mcp_oauth_callback_port`) ע"י `setup-media-mcp.sh`; בלי נעיצה codex מאזין על פורט אקראי ואי אפשר לתעל אליו. ה-token נוחת ב-`~/.codex/.credentials.json` (600) ומשם `mcp-http.sh` קורא אותו.

## CLI — דרך `mcp-http.sh`

```bash
bash ~/DevOPS/mcp-http.sh --check
bash ~/DevOPS/mcp-http.sh tools konvert
bash ~/DevOPS/mcp-http.sh call  konvert get_usage '{}'
bash ~/DevOPS/mcp-http.sh call  konvert search_ads '{"query":"skincare before-after hook","limit":5}'
bash ~/DevOPS/mcp-http.sh login konvert            # מעביר ל-codex mcp login
```

הסקריפט לא מממש OAuth בעצמו — הוא צורך token שהלקוח (Codex/Claude) כבר הנפיק, לפי הסדר: `KONVERT_MCP_TOKEN` ב-env/`~/.devops-secrets` → סריקת מאגר ה-OAuth של Codex. מארח headless: `ssh -L 1455:127.0.0.1:1455 <host>` לפני ה-login.

## הכלים

| קבוצה | כלים |
|---|---|
| חיפוש מודעות | `search_ads` · `browse_ads` · `get_ad` · `get_ad_card` |
| מותגים | `get_brands` · `get_brand_by_name` · `find_competitor_brands` · `scout_new_brand` · `list_scouted_brands` |
| מה עובד | `get_brand_top_ads` · `get_top_ads_across_brands` · `analyze_ad_image` |
| אוספים | `create_collection` · `save_ads_to_collection` · `list_collections` · `get_collection` |
| המותג שלי | `create_my_brand` · `scrape_my_brand` · `update_my_brand` · `set_my_brand_logo` · `attach_my_brand_asset` · `list_my_brands` · `get_my_brand` · `get_brand_deletion_link` |
| יצירה | `generate_ad_image` → `poll_image_generation` → `get_image_generation` |
| העלאה | `request_image_upload` · `open_upload_widget` |
| חשבון | `get_usage` · `report_missing_capability` |

## מכסות — הן קטנות, בדוק לפני שמייצרים

`get_usage` הוא הפקודה הראשונה בכל סשן יצירה. המסלולים: Starter 50 · Premium 150 · Pro 300 יצירות לחודש. **אין רובד חינמי.** כל `generate_ad_image` שורף מהמכסה — `search_ads`/`browse_ads` לא. לכן: מחקר ראשון, יצירה רק על בריף סגור.

⚠️ **שני נתיבי ההתחברות עלולים להיות שני חשבונות שונים.** נמדד ב-2026-07-28 על המרכזי, באותו רגע: דרך הקונקטור של claude.ai — `Starter, 40/50, מתאפס 2026-08-18, my_brands 1/1`; דרך ה-OAuth של Codex — `Starter, 0/50, מתאפס 2026-08-28, my_brands 0/1`. כלומר ההתחברות מ-Codex נחתה על workspace/חשבון אחר, עם מותג ומכסה נפרדים. לפני שמסתמכים על מספר — לבדוק `get_usage` **בנתיב שבו עובדים**, ולא להניח שהמותגים והאוספים משותפים.

## הזרימה שעובדת

```
get_usage                                   → כמה נשאר
find_competitor_brands / scout_new_brand    → מי המתחרים בנישה
get_brand_top_ads / get_top_ads_across_brands → מה רץ הכי הרבה זמן (= מה עובד)
analyze_ad_image                            → פירוק הוק, זווית, היררכיה, CTA
save_ads_to_collection                      → swipe file לפרויקט
create_my_brand + scrape_my_brand           → המותג שלנו (טון, צבעים, לוגו)
generate_ad_image → poll_image_generation   → וריאציות מעוגנות בדאטה
```

אחרי זה: `/ad-copy-best-practices` לקופי, `/creative-direction` לנעילת קול ווויזואל, `/image-prompt-engineering` לניסוח הפרומפט, ו-`/higgsfield` או `/nano-banana` לנכס הסופי (וידאו / PNG שקוף).

## מה לשמור עליו

1. **השראה, לא העתקה.** המודעות מגיעות מספריות מודעות ציבוריות. ללמוד מבנה, הוק וזווית — כן. לשכפל סימן מסחרי, לוגו, סלוגן או צילום של מותג אחר — לא. `generate_ad_image` ישמח לייצר משהו נגזר מדי אם תבקש; הסינון הוא באחריותך.
2. **עברית בתוך תמונה לא אמינה.** מודלי תמונה שוברים RTL וניקוד. הפק את הרקע/הקומפוזיציה, והוסף את הטקסט העברי כשכבה (`/design`, `/image-prompt-engineering`).
3. **כלל ברזל #3 חל גם על מודעה.** טקסט על תמונה חייב יחס ניגודיות ≥ 4.5:1 — overlay כהה מתחת לטקסט בהיר, לא ניחוש.
4. **`get_brand_deletion_link` הוא מחיקה.** קישור לפעולה בלתי הפיכה על נתוני מותג — אל תפתח בלי בקשה מפורשת.
5. **דאטה של לקוחות.** `scrape_my_brand` מושך את האתר של הלקוח לשרת של הספק. זו העברת מידע לצד ג׳ — קבל אישור לפני שמזינים מותג של לקוח.

## פתרון תקלות

| תסמין | סיבה / פתרון |
|---|---|
| `401 invalid_token` | אין token או שפג. `/mcp` → Authenticate, או `bash ~/DevOPS/mcp-http.sh login konvert`. |
| `no token for 'konvert'` ב-CLI | ה-token של Codex לא נמצא ב-`~/.codex/.credentials.json` — התחבר (`codex mcp login konvert`), או שים `KONVERT_MCP_TOKEN=` ב-`~/.devops-secrets`. |
| `codex mcp login` פותח URL ואז נתקע | ה-callback חוזר ל-`127.0.0.1:1455` **על השרת** — בלי `ssh -L 1455:127.0.0.1:1455` (או port-forward ב-VS Code) הדפדפן לא מגיע אליו. וקישור מריצה שנהרגה כבר לא תקף — הרץ שוב וקח URL חדש. |
| שתי שורות ב-`claude mcp list`, אחת `Needs authentication` | כפילות מול הקונקטור של claude.ai. `claude mcp remove konvert -s user`. מ-v1.25.1 הסקריפט לא יוצר אותה. |
| יצירה נכשלת בלי שגיאה ברורה | בדוק `get_usage` — כנראה נגמרה המכסה החודשית. |
| חיפוש מחזיר ריק | הנישה לא מכוסה. `scout_new_brand` מכניס מותג חדש לאינדקס, ו-`report_missing_capability` מדווח פער. |
| התקנת `npm i konvert` "לא עשתה כלום" | זו חבילת המרת מטבעות זרה. הסר; הממשק הוא MCP בלבד. |

Slash: `/konvert`. סוכן: `@konvert`. הקמה: `setup-media-mcp.sh`. CLI: `mcp-http.sh`.
