---
title: "mission-control"
type: "skill"
tags: ["kit","skill","mission control","mission-control","builderz","agent fleet dashboard","fleet ops console","dispatch tasks"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T01:08:05.47331+00:00"
id: "080fc9fe-b52d-4227-af89-51ba57452c6d"
---

> Deploy & operate Mission Control (builderz-labs) — a self-hosted Next.js dashboard for managing AI agent FLEETS — dispatch tasks (Kanban), track token cost, audit skill/agent security, and coordinate multi-agent workflows. SQLite, zero external deps, MIT. NOT a web3 tool. Use when deploying, configuring, or operating a fleet-ops console. Triggers - "mission control", "mission-control", "builderz", "agent fleet dashboard", "fleet ops console", "dispatch tasks", "agent cost tracking", "kanban agents".

# Mission Control — Agent Fleet Ops Console

[builderz-labs/mission-control](https://github.com/builderz-labs/mission-control) (MIT) is a **self-hosted dashboard
for managing AI agent fleets**: dispatch tasks on a Kanban board, track token/cost, audit agent + skill security,
and coordinate multi-agent workflows across frameworks (Claude Code, OpenClaw, CrewAI, LangGraph, AutoGen…).
**Self-hosted, zero external deps, powered by SQLite.** **Not web3** — pure agent-ops.

> **Why it fits this kit:** we already run a *server* fleet (kit-push → 29 hosts, Netdata, watchdogs). Mission Control
> is the *agent* fleet's control room — task board, cost, security scoring — and it auto-discovers local Claude Code
> sessions. Think of it as the dashboard layer over the agents you spawn.

## Stack
Next.js 16 + React 19 + TypeScript + TailwindCSS + Zustand · SQLite (WAL, better-sqlite3) · WebSocket + SSE live
updates · Recharts/Reagraph · 101 REST APIs (OpenAPI 3.1 at `/api-docs`). **node ≥ 22, pnpm@10.x.**

## Deploy (pick one)
```bash
# A) installer (recommended)
git clone https://github.com/builderz-labs/mission-control.git ~/mission-control
cd ~/mission-control && bash install.sh --local      # or: bash install.sh --docker

# B) docker compose (auto-generates credentials, persists across restarts)
cd ~/mission-control && docker compose up -d

# C) prebuilt image
docker run -d --name mission-control --restart unless-stopped -p 3055:3000 \
  ghcr.io/builderz-labs/mission-control:latest

# D) kit helper (clone/pull + compose + --check)
bash ~/DevOPS/deploy-mission-control.sh
bash ~/DevOPS/deploy-mission-control.sh --check
```
- **UI on port 3000.** ⚠️ both Mission Control *and* `/hermes-workspace` default to 3000 — if co-located, map MC to a
  different host port (e.g. `-p 3055:3000`) or set its env port.
- First run → open `http://<host>:3000/setup` to create the admin account.
- **Manual dev:** `nvm use 22 && pnpm install && pnpm dev`.

## Production hardening (required before exposing)
- Set **all** default credentials (don't ship the auto-generated ones publicly).
- `MC_ALLOWED_HOSTS` — allowlist your host(s); requests with other Host headers are rejected.
- Use `docker-compose.hardened.yml` (read-only fs, dropped capabilities, HSTS) behind a **TLS reverse proxy**.
- Keep it **Tailscale-only** unless it's behind auth + TLS. Never bare `0.0.0.0` on the public internet.

## What you get (32 panels)
- **Tasks** — 6-stage Kanban (inbox → assigned → in progress → review → quality review → done), drag-drop, priorities, threaded comments, inline sub-agent spawning, multi-project ticket prefixes.
- **Security suite** — real-time trust scoring (0–100), secret/credential detection in agent comms, MCP tool-call auditing, injection tracking, hook profiles (minimal/standard/strict). Complements `/skill-security-auditor`.
- **Skills registry** — browse/install agent skills with a built-in scanner (prompt injection, credential leaks, exfiltration, dangerous shell) across `~/.agents/skills`, `~/.codex/skills`, `~/.openclaw/skills`, project-local.
- **Cost / Claude Code** — auto-discovers local Claude Code sessions, surfaces token usage + cost, extracts team tasks.
- **Scheduling** — natural language ("every morning at 9am") → cron + dated child tasks. Webhooks w/ HMAC-SHA256, GitHub Issues sync.

## CLI / API
```bash
export MC_URL=http://localhost:3000
export MC_API_KEY=...        # shown in Settings after first login
# OpenAPI docs (Scalar UI):  http://localhost:3000/api-docs
```

## Manage / troubleshoot
```bash
cd ~/mission-control && docker compose ps | logs -f | restart | down
pnpm typecheck && pnpm test          # 282 unit + 295 e2e
```
| בעיה | פתרון |
|------|-------|
| 403 / "host not allowed" | הגדר `MC_ALLOWED_HOSTS` לכלול את ה-host |
| התנגשות פורט עם hermes-workspace | מפה את MC ל-`3055:3000` או שנה פורט |
| node version error | דורש node ≥ 22 (`nvm use 22`) |
| credentials ברירת מחדל בפרודקשן | החלף הכל + reverse-proxy TLS לפני חשיפה |

## Related Skills
- `/skill-security-auditor` — Mission Control's scanner complements the kit's pre-install audit
- `/engineering-pro`, `/observability` — reliability + monitoring for the console
- `/ruflo` — orchestrate the agents that Mission Control then tracks
- `/hermes-workspace`, `/agent-zero` — agent platforms you can monitor from here
