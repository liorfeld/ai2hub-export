---
title: "agentmemory"
type: "skill"
tags: ["kit","skill","agentmemory","agent memory","ריבוי זיכרון","multi memory","persistent memory","local memory"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T05:51:57.715428+00:00"
id: "3bb88800-10cf-4fcb-9684-9d24a059ea11"
---

> Deploy & manage AgentMemory (rohitg00) — a rich LOCAL memory service for AI coding agents (npm/CLI). Triggers - "agentmemory", "agent memory", "ריבוי זיכרון", "multi memory", "persistent memory", "local memory", "memory viewer", "deploy agentmemory".

# AgentMemory — Deploy & Manage (local rich memory)

[rohitg00/agentmemory](https://github.com/rohitg00/agentmemory) is a **persistent memory service for AI coding
agents** that captures and retrieves context across sessions. It runs as a **background Node service** and exposes
**53 MCP tools** over a local HTTP server, so Claude Code (and 20+ other agents) can remember prior work without
re-explanation. **92% token reduction** vs context-pasting, **95.2% R@5** on LongMemEval-S.

What makes it rich: **4-tier consolidation** (working → episodic → semantic → procedural), **hybrid retrieval**
(BM25 + vector cosine + knowledge-graph traversal, fused with Reciprocal Rank Fusion), **session replay** with a
timeline viewer, and a **privacy filter** that strips secrets before storage. Storage is **local SQLite** with
**local embeddings** (`all-MiniLM-L6-v2`) — **no external database**, no GPU, LLM calls **disabled by default**.

> **AgentMemory in the kit's memory map:** knowledge memory lives in each project's `MEMORY.md` (iron rule #6),
> CODE memory in `codebase-memory` (the only ALWAYS_ON MCP). AgentMemory is the **local, per-host rich** layer
> (SQLite, 4-tier, hybrid search, replay), with **zero central dependency**, loaded **on-demand** via
> `mcp-on agentmemory`. Its old registration into `~/.claude/settings.json` was a **dead key Claude Code
> never read** — removed in 1.35.0.

## ⚠️ Security first
- **Bind to Tailscale/loopback only.** The API (3111) + viewer (3113) must never listen on a public `0.0.0.0`.
  Verify after deploy: `ss -tlnp | grep -E '3111|3113'`.
- **Protect the REST API** with `AGENTMEMORY_SECRET` (the deploy script generates one into `~/.agentmemory/.env`).
- **Privacy filter on** — secrets are stripped before storage; never disable it on shared hosts.
- **LLM calls off by default.** Only set `ANTHROPIC_API_KEY` + `AGENTMEMORY_AUTO_COMPRESS=true` if you actually
  want LLM-based compression/summarization. Keys live in `~/.agentmemory/.env`, never in logs.

## Deploy (idempotent, per-host local)
```bash
bash ~/DevOPS/deploy-agentmemory.sh            # install + start daemon + register MCP (Tailscale/loopback)
bash ~/DevOPS/deploy-agentmemory.sh --no-mcp   # install + start only, skip settings.json wiring
bash ~/DevOPS/deploy-agentmemory.sh --check     # status: service, health, both MCP entries, ports
```
What it does: `npm install -g @agentmemory/agentmemory`, starts the `agentmemory` server as a **systemd user unit**
(falls back to `nohup`), scaffolds `~/.agentmemory/.env` (generates `AGENTMEMORY_SECRET`). The MCP itself is wired
**on-demand** with `mcp-on agentmemory` (the old `~/.claude/settings.json` merge targeted a dead key — removed 1.35.0).

Ports: **3111** API+MCP-HTTP · **3113** viewer · 3112/49134 internal.
The iii engine binds `0.0.0.0:49134`; the setup adds a loopback-only firewall rule so it is never publicly reachable.

## Fleet rollout (automatic)
`kit-update` runs **`setup-agentmemory.sh`** on every server, so AgentMemory
installs + starts **automatically fleet-wide** and self-heals on each update/cron run. The MCP itself is
**on-demand**: enable it per project with `mcp-on agentmemory` (re-run after `kit-update`, which resets Codex servers to disabled). It is
**resource-guarded**: hosts with less than `AGENTMEMORY_MIN_RAM_MB` (default 1024) free **self-skip** with a loud
`[agentmemory] SKIP:` line (override per-host with `AGENTMEMORY_MIN_RAM_MB=0`). `deploy-agentmemory.sh` is the
manual/verbose front-end over the same core.

## MCP registration (on-demand)
The MCP layer is a **stdio bridge** (`@agentmemory/mcp`, `AGENTMEMORY_URL=http://127.0.0.1:3111`) that talks to
the local server. Registration goes through the on-demand catalog:
```bash
mcp-on agentmemory     # enable for the current project (Claude + Codex)
mcp-off agentmemory    # disable
```
Restart Claude Code, then `claude mcp list` (or `/mcp`) should show `agentmemory`.
> The old registration into `~/.claude/settings.json` → `mcpServers` was a **dead key Claude Code never read**
> (it reads only `~/.claude.json`) — removed in 1.35.0. `agentmemory connect claude-code` writes its own config;
> prefer `mcp-on` so the server stays under the catalog's control.

## Manage / troubleshoot
```bash
systemctl --user status agentmemory            # service state (or: pgrep -af agentmemory)
journalctl --user -u agentmemory -f            # logs
curl -s http://127.0.0.1:3111/agentmemory/health   # health probe
# viewer: http://127.0.0.1:3113  (SSH tunnel: ssh -L 3113:localhost:3113 <host>)
systemctl --user restart agentmemory
```
| בעיה | פתרון |
|------|-------|
| `agentmemory` לא ב-`/mcp` | השרת לא רץ — `systemctl --user status agentmemory`; ה-bridge מתחבר ל-`AGENTMEMORY_URL` |
| נרשם אבל לא מופיע ב-`/mcp` | נרשם ל-`~/.claude/settings.json` (מפתח מת) — הרץ `mcp-on agentmemory`; Claude Code קורא רק `~/.claude.json` |
| פורט 3111 תפוס | שירות כבר רץ / התנגשות — בדוק `ss -tlnp` לפורט 3111, ואז `systemctl --user restart agentmemory` |
| מאזין על 0.0.0.0 | חשיפה ציבורית — הגבל ל-loopback/Tailscale + ודא `AGENTMEMORY_SECRET` ב-`~/.agentmemory/.env` |
| השירות לא שורד reboot | `loginctl enable-linger $USER` (user unit ללא linger נעצר בלוגאאוט) |
| מודל ה-embeddings לא יורד | ריצה ראשונה מורידה `all-MiniLM-L6-v2` — דורש רשת; בדוק logs |

## Self-heal
`systemctl --user enable agentmemory` + `loginctl enable-linger`. For fleet self-heal, add a `--check` cron that
re-runs the deploy script if the health probe fails (same pattern as the other deploy scripts).

## Related Skills
- `/self-improving` — MEMORY.md → CLAUDE.md promotion (file-tier knowledge memory, iron rule #6)
- `/engineering-pro` — security audit before exposing a network service
- `/observability` — monitor the daemon via the Netdata fleet stack
- `/hermes`, `/agent-zero` — other per-host services that follow this same deploy/`--check` pattern
