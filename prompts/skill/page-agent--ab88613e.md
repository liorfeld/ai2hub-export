---
title: "page-agent"
type: "skill"
tags: ["kit","skill","page-agent","page agent","in-page agent","dom agent","drive the page","browser gui agent"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T01:08:05.47331+00:00"
id: "ab88613e-4f98-4a14-94f6-45ebe45ec844"
---

> Deploy & operate Page Agent (alibaba/page-agent, MIT) — an IN-PAGE GUI agent that reads the live DOM as text (no screenshots, no vision model, no headless browser) and drives a web page from inside the tab. Triggers - "page-agent", "page agent", "in-page agent", "dom agent", "drive the page", "browser gui agent", "@page-agent/mcp", "סוכן בדף", "סוכן דפדפן".

# Page Agent — הסוכן שחי בתוך הדף

[alibaba/page-agent](https://github.com/alibaba/page-agent) (MIT) — סוכן GUI **בתוך הדף**: קורא את ה-DOM החי כטקסט, מפרש פקודה בשפה טבעית ומפעיל את הדף מבפנים. **בלי screenshots, בלי מודל ראייה, בלי headless browser, בלי extension חובה** לליבה. רכיבי ה-DOM והפרומפט נגזרים מ-`browser-use`.

> **מול שאר צי הדפדפן:** `agent-browser` = CLI headless (batch/CI) · Playwright MCP = headless חיצוני · **page-agent = בתוך הדף**, עם ה-session/auth של המשתמש, זול ומהיר (בלי vision). לא מחליף — ממלא נישה.

## שני מצבי שימוש

**1. ספרייה בתוך פרויקט (per-project):**
```js
import { PageAgent } from 'page-agent'
const agent = new PageAgent({ model: 'qwen3.5-plus', baseURL: 'https://<PROXY>/v1', apiKey: '<via-proxy>', language: 'he-IL' })
await agent.execute('לחץ על כפתור ההתחברות')
```

**2. MCP server (Beta) — שליטה מבחוץ:** `bash ~/DevOPS/deploy-page-agent.sh` מתקין את `@page-agent/mcp` (Node≥20) ורושם אותו ל-Claude+Codex. ה-MCP **מגשר לתוסף Chrome של Page Agent דרך WebSocket** — צריך דפדפן פתוח + התוסף מותקן.

## עקרונות אבטחה (אכיפה)
1. **מפתח LLM אף פעם לא בצד-לקוח.** בכל אפליקציה אמיתית — `baseURL` מפנה ל-proxy צד-שרת, המפתח נשאר בשרת.
2. **הדף = מקור prompt-injection.** סוכן שקורא DOM ולוחץ פועל עם הרשאות המשתמש → שער אישור לפעולות הרסניות.
3. **לא ל-demo CDN עם דאטה אמיתי** — נקודת הבדיקה החינמית של Alibaba מעבירה prompt+DOM אליהם.

## Scale (כלל #8)
`baseURL` מנותב ל-proxy שמפנה ל-`gpt-5.6-codex` (🟢 בנייה) או ל-Claude — לפי הטסק. page-agent הוא *כלי*, לא מנתב.

## תפעול
`deploy-page-agent.sh --check` (node/bin/registration) · `--uninstall`. Node-18 hosts → דילוג נקי.
