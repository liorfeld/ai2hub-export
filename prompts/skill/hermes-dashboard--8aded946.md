---
title: "hermes-dashboard"
type: "skill"
tags: ["kit","skill","hermes dashboard","hermes-dashboard","chrisryugj","hermes dashboard hub","hermes admin ui","hermes gateway dashboard"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T01:08:05.47331+00:00"
id: "8aded946-3420-4006-bc8b-598bbac6c0cd"
---

> Deploy & operate Hermes Dashboard Hub (chrisryugj/hermes-dashboard) — a lightweight single-file HTML/Python admin dashboard that PATCHES the Hermes gateway API server and installs to ~/.hermes/dashboard/, served at :8642/dashboard. Triggers - "hermes dashboard", "hermes-dashboard", "chrisryugj", "hermes dashboard hub", "hermes admin ui", "hermes gateway dashboard", "hermes mcp manager".

# Hermes Dashboard Hub — in-gateway admin UI

[chrisryugj/hermes-dashboard](https://github.com/chrisryugj/hermes-dashboard) (MIT) is a **lightweight admin dashboard
for the Hermes Agent gateway**. Unlike the separate-app GUIs, it **extends the gateway itself**: an installer patches
the Hermes API server and copies a single-file dashboard into `~/.hermes/dashboard/`, served from **`:8642/dashboard`**.

> **Relationship to the other Hermes UIs (read first):**
> - `/hermes` deploys the **agent** (gateway `:8642`). **This dashboard patches THAT gateway** — it cannot run without it (needs Hermes **v0.7.0+**).
> - `/hermes-workspace` (outsourc-e) and `/patterstage` (Daniel-Parke) are **separate web apps** on their own ports.
> - This one is the **lightest** option — no separate server/port, no Node build. Pick one Hermes GUI per host.

## Stack
Single-file HTML frontend (no build deps) · Python backend that integrates into the Hermes API server · Shell installer.
Dark Linear-style UI with XSS protection + path-traversal guards.

## Features
Model switching · MCP server management · cron job scheduling · config editing · real-time logging — all against the
local Hermes gateway.

## Deploy
```bash
# kit helper: clone, run the installer (patches gateway + copies dashboard), verify
bash ~/DevOPS/deploy-hermes-dashboard.sh
bash ~/DevOPS/deploy-hermes-dashboard.sh --check     # status only

# manual equivalent:
git clone https://github.com/chrisryugj/hermes-dashboard.git ~/hermes-dashboard
cd ~/hermes-dashboard && bash install.sh             # patches ~/.hermes gateway, copies dashboard files
# then restart the Hermes gateway so the patched API server serves /dashboard
```
- **Prerequisite:** `/hermes` agent running, `hermes --version` ≥ 0.7.0, `~/.hermes/` present.
- After install, restart the gateway and open `http://127.0.0.1:8642/dashboard`.

## Remote / exposed
The dashboard is served by the gateway, so its exposure follows the gateway's. Keep Hermes **Tailscale-only behind a
TLS reverse proxy with auth**; never expose `:8642` publicly. (Upstream adds XSS + path-traversal protection, but that
is not a substitute for not exposing the port.)

## Verify
```bash
curl -s -m5 http://127.0.0.1:8642/health             # gateway up?
ls ~/.hermes/dashboard/                               # dashboard files installed?
curl -s -m5 -o /dev/null -w '%{http_code}\n' http://127.0.0.1:8642/dashboard   # 200?
bash ~/DevOPS/deploy-hermes-dashboard.sh --check
```
| בעיה | פתרון |
|------|-------|
| `/dashboard` מחזיר 404 | הגייטוויי לא הופעל מחדש אחרי ה-patch, או גרסת Hermes < 0.7.0 |
| installer נכשל | `~/.hermes` חסר → הרץ `/hermes` קודם; אמת הרשאות כתיבה |
| נגיש מבחוץ בלי auth | הגבל את `:8642` ל-Tailscale + TLS proxy |

## Related Skills
- `/hermes` — **deploy the agent first**; this patches its gateway
- `/patterstage`, `/hermes-workspace`, `/mission-control` — heavier sibling GUIs (pick one per host)
