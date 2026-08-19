---
title: "simplex"
type: "skill"
tags: ["kit","skill","simplex","simplex-chat","private alert","metadata-free messaging","simplex bot","encrypted notification"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T06:11:19.405046+00:00"
id: "dc8bcc6a-1d68-4ba3-8e19-a0d1a147b818"
---

> Deploy a private, metadata-free ops-alert bot on SimpleX Chat (simplex-chat/simplex-chat, AGPLv3, Trail-of-Bits audited) — a messaging network with NO user identifiers (no phone number, no user id). Triggers - "simplex", "simplex-chat", "private alert", "metadata-free messaging", "no phone number chat", "simplex bot", "encrypted notification", "התראה פרטית", "ערוץ מוצפן".

# SimpleX — בוט התראות פרטי (ללא מזהים)

[simplex-chat/simplex-chat](https://github.com/simplex-chat/simplex-chat) (AGPLv3, מבוקר Trail of Bits) — רשת מסרים **ללא שום מזהה משתמש** (בלי טלפון, בלי user id), E2E עם post-quantum. אנחנו מתקינים את ה-**CLI ה-headless** (WebSocket control server) + בוט Node מינימלי = **ערוץ התראות פרטי** שני לצד OpenWA.

> **מול OpenWA:** WhatsApp = כולם כבר שם, אבל metadata חשוף. SimpleX = פרטי מטא-דאטה לגמרי, אבל הנמען חייב אפליקציית SimpleX + חיבור חד-פעמי. משלים, לא מחליף — לשימוש בהתראות רגישות.

## הקמה — deploy-on-demand (הוסט נבחר)

```bash
bash ~/DevOPS/deploy-simplex-bot.sh            # CLI + בוט + שני user services
bash ~/DevOPS/deploy-simplex-bot.sh --check    # binary, services, loopback bind, node, address
```

דרישות: Linux amd64/arm64, **Node ≥ 20** (לבוט; השרת עולה גם בלי), systemd --user.

## שימוש

```bash
# 1) אחרי deploy — קבל את קישור החיבור החד-פעמי:
journalctl --user -u simplex-bot -n 20 --no-pager     # או: cat ~/.simplex/address.txt
# 2) הוסף אותו כאיש-קשר פעם אחת מאפליקציית SimpleX (נייד/דסקטופ).
# 3) שלח התראה (loopback):
simplex-send "disk full on web-01"                    # → לכל איש-קשר מחובר
```

חיווט אופציונלי ל-site-health: הוסף `simplex-send "$msg"` ליד ערוץ ההתראות הקיים.

## אבטחה (קריטי!)

1. **ה-WS control server = שליטה מלאה בזהות.** נקשר ל-**127.0.0.1 בלבד** (מאומת) — **לעולם לא לחשוף**. `--check` מוודא loopback לשני הפורטים (5225 שרת, 5226 send).
2. בינארי **נעוץ-גרסה + SHA-256** לפני התקנה.
3. **AGPLv3** — שימוש פנימי בסדר; לא מפיצים גרסה מוקנפת כשירות לצד ג'.

## תקרות (ponytail — מסומנות)

- **אין הצפנת DB at-rest**: מפתח ה-`-k` של SimpleX עובר רק ב-argv (היה דולף ב-`ps`) — סירבנו לסוד-ב-argv. ההגנה = הרשאות `~/.simplex` (0700). צריך הצפנת דיסק? הוסף `-k` והפעלה אינטראקטיבית ידנית.
- **הבוט שולח לכל אנשי-הקשר** (בלי ניתוב פר-איש-קשר) ולא מטפל בהודעות נכנסות. להוסיף כשיהיה צורך.

Slash: `/simplex`.
