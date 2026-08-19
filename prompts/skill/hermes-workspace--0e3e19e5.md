---
title: "hermes-workspace"
type: "skill"
tags: ["kit","skill","hermes workspace","hermes-workspace","outsourc-e","hermes ui","hermes dashboard ui","agent swarm dashboard"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T06:02:49.623432+00:00"
id: "0e3e19e5-488a-41b4-9e9d-b9993c14e772"
---

> Deploy & run Hermes Workspace (outsourc-e) — a web + Electron control plane that sits ON TOP OF the Nous Hermes Agent — unified chat, file browser/Monaco editor, terminal, agent memory, skills catalog, and a multi-agent swarm dashboard. Complements the /hermes skill (which deploys the agent itself). Use when setting up, exposing, or troubleshooting the workspace UI. Triggers - "hermes workspace", "hermes-workspace", "outsourc-e", "hermes ui", "hermes dashboard ui", "agent swarm dashboard", "hermes control plane", "swarm.yaml".

# Hermes Workspace — Orchestration Control Plane

[outsourc-e/hermes-workspace](https://github.com/outsourc-e/hermes-workspace) (MIT) is a **native web + Electron
workspace for the Hermes Agent** — "not a chat wrapper; a complete workspace." One UI for: streaming chat with tool
rendering, a file browser + Monaco editor, a terminal, agent memory browse/edit, a 2,000+ skills catalog, and a
**multi-agent swarm dashboard** with MCP integration.

> **Relationship to `/hermes` (read this first):**
> - `/hermes` deploys the **Nous Hermes Agent** itself — the gateway at `:8642` (+ dashboard `:9119`).
> - **`/hermes-workspace` is the UI layer over it** — it *connects to* an existing Hermes gateway, it does not replace it.
> - Typical setup: deploy the agent with `/hermes`, then point this workspace at `HERMES_API_URL=http://<host>:8642`.
> - It also works against any OpenAI-compatible backend, so it's not locked to Nous.

## Stack
React 19 + Vite + TanStack · **Electron** (Mac/Win desktop) · Tailwind + Framer Motion · Monaco editor · xterm
terminal · Zustand · Three.js/R3F (visuals) · Playwright/Puppeteer · **pnpm**. PWA support for mobile.

## Ports
| Port | Service | Owner |
|------|---------|-------|
| `8642` | Hermes gateway API | the agent (`/hermes`) |
| `9119` | Hermes dashboard API | the agent (`/hermes`) |
| `3000` | **this workspace UI** | hermes-workspace |

## Deploy
```bash
# 1) get it (the kit helper clones/pulls, installs, scaffolds .env, builds)
bash ~/DevOPS/deploy-hermes-workspace.sh
bash ~/DevOPS/deploy-hermes-workspace.sh --check

# manual equivalent:
git clone https://github.com/outsourc-e/hermes-workspace.git ~/hermes-workspace
cd ~/hermes-workspace && pnpm install
cp .env.example .env
printf '\nHERMES_API_URL=http://127.0.0.1:8642\nHERMES_DASHBOARD_URL=http://127.0.0.1:9119\n' >> .env
pnpm dev            # http://localhost:3000  (override: PORT=4000 pnpm dev)
```
- Point it at the agent: `HERMES_API_URL` (gateway) + `HERMES_DASHBOARD_URL` (config/sessions/skills/jobs).
- On the agent side ensure `API_SERVER_ENABLED=true` + `API_SERVER_HOST=0.0.0.0` in `~/.hermes/.env`, and `hermes dashboard` running.
- ⚠️ Port 3000 collides with `/mission-control` — use `PORT=` to move one if co-located.

## LLM providers (in `.env`)
At least one: `ANTHROPIC_API_KEY` (Claude — favored), `OPENAI_API_KEY`, `OPENROUTER_API_KEY`, `GOOGLE_API_KEY`, or local Ollama (no key).
The committed `.env.example` only ships `VITE_PLAYGROUND_*`; add provider keys + Hermes URLs yourself.

## Remote / exposed deploy (mandatory hardening)
Binding beyond loopback **requires** auth — the server refuses to start on a non-loopback host without it:
```bash
echo 'HOST=0.0.0.0'                         >> .env   # expose beyond localhost
echo 'HERMES_PASSWORD=<32+ char secret>'    >> .env   # required on non-loopback
echo 'HERMES_API_TOKEN=<token>'             >> .env   # if the gateway sets API_SERVER_KEY
echo 'TRUST_PROXY=1'                         >> .env   # behind a reverse proxy
```
Prefer **Tailscale-only** + a TLS reverse proxy. Never expose without `HERMES_PASSWORD`.

## Swarm architecture
Routing via `swarm.yaml` — 10 semantic roles (orchestrator, builder, reviewer, qa, researcher, km-agent, ops-watch,
maintainer, strategist, inbox-triage), context-sensitive decisions through **GBrain**, and **persistent tmux-backed
workers** that keep context across tasks. Edit `swarm.yaml` to tune task distribution.

## Desktop & verify
```bash
pnpm electron:dev                 # desktop dev
pnpm electron:build               # Mac/Win builds (electron:build:mac | :win)
# health checks:
curl http://127.0.0.1:8642/health        # gateway ok
curl http://127.0.0.1:9119/api/status    # dashboard metadata
curl http://127.0.0.1:3000/api/sessions  # workspace booted (sessions payload / [])
```
| בעיה | פתרון |
|------|-------|
| chat ריק / "no backend" | `HERMES_API_URL` שגוי או הסוכן לא רץ — אמת `curl :8642/health` |
| מסרב לעלות על host חיצוני | חסר `HERMES_PASSWORD` (חובה ל-non-loopback) |
| התנגשות פורט עם mission-control | `PORT=4000 pnpm dev` או הזז את MC |
| תכונות מתקדמות חסרות | ה-gateway לא חושף את כל ה-Hermes APIs — הפעל `API_SERVER_ENABLED=true` |

## Related Skills
- `/hermes` — **deploy the agent first**; this is the UI on top of it
- `/ruflo` — dual-mode orchestration; pairs with the swarm model
- `/agent-zero`, `/mission-control` — sibling agent platforms in the kit
