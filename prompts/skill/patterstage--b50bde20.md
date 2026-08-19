---
title: "patterstage"
type: "skill"
tags: ["kit","skill","patterstage","control hub","hermes control hub","daniel-parke","hermes command center","hermes gui"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T06:07:37.408399+00:00"
id: "b50bde20-430b-4103-ad77-b928cc20ddb9"
---

> Deploy & operate PatterStage "Control Hub" (Daniel-Parke/PatterStage) — a Next.js web command-center that sits ON TOP OF a running Nous Hermes Agent — mission dispatch, cron scheduling, agent-profile management, session browsing, and config editing without a terminal. Companion GUI to the /hermes agent (sibling of /mission-control & /hermes-workspace). Use when deploying, exposing, or troubleshooting the Hermes Control Hub UI. Triggers - "patterstage", "control hub", "hermes control hub", "daniel-parke", "hermes command center", "hermes gui", "agent control panel".

# PatterStage — Hermes "Control Hub"

[Daniel-Parke/PatterStage](https://github.com/Daniel-Parke/PatterStage) (MIT) is a **web command-centre built for the
Hermes Agent** — a Next.js dashboard to run and manage a locally-running Hermes agent **without the terminal**:
mission dispatch, cron scheduling, agent-profile management, session browsing, and live config editing.

> **Relationship to `/hermes` (read first):**
> - `/hermes` deploys the **Nous Hermes Agent** itself — gateway `:8642` (+ dashboard `:9119`).
> - **`/patterstage` is a UI layer over it** — connects to an existing gateway, does not replace it.
> - It is a **sibling** of `/mission-control` and `/hermes-workspace`. They overlap — pick one GUI per host.

## Stack
Next.js · Node 20+ · TypeScript (92%) · npm + Docker · Jest + Playwright (upstream CI). Runs on macOS/Linux.

## Ports
| Port | Service | Owner |
|------|---------|-------|
| `8642` | Hermes gateway API | the agent (`/hermes`) |
| `9119` | Hermes dashboard API | the agent (`/hermes`) |
| `4505` | **this Control Hub UI** | patterstage (kit default — avoids 3000/3055 clashes) |

## Deploy
```bash
# kit helper: clone/pull, install (npm or Docker), scaffold .env, point at the agent
bash ~/DevOPS/deploy-patterstage.sh
bash ~/DevOPS/deploy-patterstage.sh --port 4505 --api http://127.0.0.1:8642
bash ~/DevOPS/deploy-patterstage.sh --docker        # container instead of npm
bash ~/DevOPS/deploy-patterstage.sh --check         # status only

# manual equivalent:
git clone https://github.com/Daniel-Parke/PatterStage.git ~/patterstage
cd ~/patterstage && npm install && npm run build
echo 'HERMES_API_URL=http://127.0.0.1:8642' >> .env
PORT=4505 npm start         # http://localhost:4505/
```
- Point it at the agent with `HERMES_API_URL`; ensure `API_SERVER_ENABLED=true` on the agent side.
- ⚠️ Default host port `4505` avoids `/mission-control` (3055) and `/hermes-workspace` (3000). Use `--port` to move it.

## Remote / exposed deploy (mandatory hardening)
```bash
# Prefer Tailscale-only + a TLS reverse proxy. Never bind 0.0.0.0 without auth.
tailscale ip -4                 # bind to the tailnet IP, not the public NIC
```
Secrets (provider keys, Hermes tokens) live in `.env`/env — never in the repo.

## Verify
```bash
curl -s -m5 http://127.0.0.1:8642/health   # gateway up (agent running)?
curl -s -m5 http://127.0.0.1:4505/         # Control Hub booted (HTTP 200)?
bash ~/DevOPS/deploy-patterstage.sh --check
```
| בעיה | פתרון |
|------|-------|
| UI ריק / "no agent" | `HERMES_API_URL` שגוי או הסוכן לא רץ — אמת `curl :8642/health`, הרץ `/hermes` |
| התנגשות פורט | `--port` אחר; אל תריץ עוד GUI על אותו host |
| מסרב להתחבר מבחוץ | bind ל-Tailscale IP + TLS proxy; בלי auth אל תחשוף |

## Related Skills
- `/hermes` — **deploy the agent first**; this is the UI on top of it
- `/mission-control`, `/hermes-workspace` — sibling fleet/agent GUIs (pick one per host)
- `/ruflo` — dual-mode orchestration
