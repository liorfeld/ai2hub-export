---
title: "site-health"
type: "skill"
tags: ["kit","skill","site health","site-health","האתר לא עולה","ניטור אתרים","uptime","healthcheck לאתרים"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T01:08:05.47331+00:00"
id: "014e294b-fe56-4b1d-a1ac-0cc8afdb46fa"
---

> End-to-end site monitoring + self-heal for every app on a server — probes each site through a DB-touching endpoint (not just the homepage), restarts the right service/pm2 process after 2 consecutive failures with cooldowns, alerts via WhatsApp (Green API) or kit email, and refuses to restart apps when the shared DB/pooler layer is the real fault. Triggers: "site health", "site-health", "האתר לא עולה", "ניטור אתרים", "uptime", "healthcheck לאתרים", "self-heal", "המנגנון אתחל", "probe".

# Site Health — ניטור + ריפוי עצמי לכל האתרים בשרת

מנגנון host-level שנולד מתקלת production אמיתית (2026-07-06): `supabase-db`
אותחל, ה-pooler (Supavisor) נשאר עם DNS ישן, וכל האפליקציות איבדו DB —
בזמן שכל ה-healthchecks הפנימיים, `systemctl` ו-`docker ps` הראו ירוק.
דף הבית ענה 307, וה-DB היה מת. הלקח: **בודקים את המסלול שהמשתמש באמת
עובר — עד ה-DB — לא את התהליך.**

---

## ארכיטקטורה — שתי שכבות

| שכבה | מה בודקת | מי מרפא |
|------|-----------|---------|
| **pooler-autoheal** (אם יש Supabase מקומי) | `SELECT 1` דרך פורט ה-pooler עצמו (docker healthcheck ב-override) | `pooler-autoheal.timer` — `docker restart supabase-pooler` על unhealthy |
| **site-health** | probe HTTP לכל אפליקציה דרך endpoint שנוגע ב-DB | `site-health.timer` — restart ל-service/pm2 + התראה |

## קבצים

| קובץ | תפקיד |
|------|-------|
| `~/DevOPS/site-health.sh` | ה-payload (מקור אמת בקיט) |
| `/usr/local/sbin/site-health.sh` | העותק המותקן שה-timer מריץ |
| `/etc/site-health/sites.conf` | רשימת האתרים (פורמט למטה) |
| `/etc/site-health/env` | creds ל-WhatsApp (root 0600, אופציונלי) |
| `/var/lib/site-health/` | מוני כשלים + חותמות cooldown |
| `~/DevOPS/deploy-site-health.sh` | התקנה אידמפוטנטית |

## התקנה בשרת חדש

```bash
~/DevOPS/deploy-site-health.sh   # מתקין הכל; timer קם רק כש-sites.conf מלא
```

## בניית sites.conf — הכלל הקריטי

פורמט שורה: `name|METHOD|probe_url|expected_codes|restart_cmd[|cookie]`

מיפוי אתרים בשרת:
```bash
grep -H proxy_pass /etc/nginx/sites-enabled/*        # אתר → פורט
systemctl list-units --type=service --state=running   # services
pm2 jlist | jq -r '.[].name'                          # pm2 apps
```

**בחירת probe — לפי סדר עדיפות:**
1. **GET ל-endpoint מוגן + cookie session מזויף** (למשל
   `/api/auth/me` עם `almog_sid=<uuid-אפסים>`) → מכריח lookup ב-DB
   ומחזיר 401. זה probe אמיתי: DB מת = timeout/500 = כשל.
2. **GET ל-API מוגן** (`/api/invoices` → 401) — מוכיח שהאפליקציה חיה
   וה-middleware עובד.
3. **GET לדף הבית** (`200 307`) — process-level בלבד; מקובל כשההתחברות
   היא server actions ואין API לבדוק.

**❌ לעולם לא probe ל-route של login עם סיסמה שגויה** — זה מפעיל הגנת
brute-force (429), ממלא את `auth_rate_limits`, והמנגנון "מרפא" אתר בריא.
זו הייתה תקלה אמיתית ביום ההקמה — billing אותחל לשווא בגלל probe כזה.

## התראות

`/etc/site-health/env` עם `GREEN_API_ID/TOKEN/ALERT_CHAT` → WhatsApp.
ריק/חסר → fallback ל-`devops_notify` (אימייל הקיט). אין כלום → journal.
חילוץ token מ-billing (מוצפן AES-256-GCM ב-`whatsapp_instances`): פענוח
עם `SETTINGS_ENC_KEY` מ-`/etc/billing/billing.env` — ראה הסקריפט בגוף
ה-commit שהקים את המנגנון.

## בלמי בטיחות (מובנים בסקריפט)

- **2 כשלונות רצופים** לפני פעולה (עבירה חולפת ≠ תקלה).
- **Cooldown אתחול 15 דק'** לאתר — אין restart-storm.
- **Cooldown התראות 30 דק'** לאתר — אין spam.
- **Shared-layer guard:** רוב האתרים נופלים יחד ⇒ לא מאתחלים כלום,
  התראה אחת שמפנה ל-DB/pooler.
- **הודעת התאוששות** ✅ כשאתר חוזר.

## בדיקת המנגנון אחרי התקנה

```bash
sudo /usr/local/sbin/site-health.sh          # ריצה ידנית — הכל ירוק?
# סימולציית כשל: מוסיפים אתר מזויף, מריצים פעמיים, מצפים לאתחול+התראה:
echo '__test__|GET|http://127.0.0.1:9/|200|true' | sudo tee -a /etc/site-health/sites.conf
sudo /usr/local/sbin/site-health.sh && sudo /usr/local/sbin/site-health.sh
sudo sed -i '/^__test__|/d' /etc/site-health/sites.conf
sudo rm -f /var/lib/site-health/__test__.*
```

## תחקור התראה

```bash
journalctl -t site-health --since '-1h'       # מה נכשל ומה נעשה
cat /var/lib/site-health/<site>.fails          # מונה נוכחי
docker logs supabase-pooler --since 30m        # אם ההתראה על שכבה משותפת
```
