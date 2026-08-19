---
title: "agent-zero"
type: "skill"
tags: ["kit","skill","agent zero","agent-zero","agent0ai","autonomous agent platform","deploy agent zero","a0"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T05:51:45.314559+00:00"
id: "8e2426e7-6da2-49de-bea4-dd2c60c2264d"
---

> Deploy & manage Agent Zero (agent0ai) — an autonomous, "organic" multi-agent framework that ships as a Docker container with a full Linux desktop, code execution, browser use, persistent memory, and MCP. Calls an external LLM (Anthropic Claude / OpenRouter / Ollama), no GPU. Use when deploying, configuring, exposing, isolating, or troubleshooting Agent Zero. Triggers - "agent zero", "agent-zero", "agent0ai", "autonomous agent platform", "deploy agent zero", "a0", "subordinate agents", "agent zero dashboard".

# Agent Zero — Deploy & Manage

[agent0ai/agent-zero](https://github.com/agent0ai/agent-zero) (MIT) is an **open, "organic" multi-agent framework**.
The primary agent breaks a task into subtasks and delegates to **subordinate agents** (developer/researcher/
reviewer…), each with its own context, then synthesizes. It ships as a **Docker container** with a full Linux/
XFCE desktop, terminal + Python code execution, a built-in browser, persistent (time-travel) memory, a plugin hub,
and **MCP** support. It calls an **external LLM** over API — **no GPU**. Official image: `agent0ai/agent-zero`.

> **Agent Zero vs `/hermes`:** Hermes is a stateless **inference gateway** (OpenAI-compatible API). Agent Zero is a
> stateful **agent OS** (multi-agent, desktop, code-exec, memory, plugins). They're complementary — Agent Zero can
> even use a Hermes endpoint as its provider. For pure model serving use `/hermes`; for autonomous multi-step work use this.

## ⚠️ Security first — it runs code
Agent Zero executes terminal commands and Python **as the agent**. The Docker container is the isolation boundary.
- **Always run in Docker.** Running it bare = the agent gets your host's filesystem + shell. Never do that.
- **Mount only `/a0/usr`** (config, projects, memory). Never mount `/a0` (breaks upgrades) or `/` (host escape).
- Cap resources (`--memory`, `--cpus`). `privileged: true` is only for nested Docker-in-Docker — avoid unless needed.
- Secrets via env/Settings, **never** echoed to logs.

## Deploy (resource-capped, idempotent)
```bash
bash ~/DevOPS/deploy-agent-zero.sh                       # defaults: 2 cpus / 4GB, port 50080, Tailscale-only
bash ~/DevOPS/deploy-agent-zero.sh --cpus 4 --memory 6
bash ~/DevOPS/deploy-agent-zero.sh --provider anthropic --model claude-sonnet-4-5
bash ~/DevOPS/deploy-agent-zero.sh --check               # status
```
Manual equivalent:
```bash
docker pull agent0ai/agent-zero:latest
docker run -d --name agent-zero --restart unless-stopped \
  --memory 4g --cpus 2 \
  -p 50080:80 -v a0_usr:/a0/usr \
  -e A0_SET_chat_model_provider=anthropic \
  -e A0_SET_chat_model_name=claude-sonnet-4-5 \
  agent0ai/agent-zero
```
- **Web UI:** container port `80` → host `50080`. **Bind to Tailscale/loopback**, not `0.0.0.0`.
- **Multi-instance:** run more containers on `50081`, `50082`… each with its own volume.

## Provider setup (favor Anthropic Claude)
Three model slots (Settings → Model, or `A0_SET_*` env):
- **Chat model** (primary reasoning) → `anthropic` / `claude-sonnet-4-5`
- **Utility model** (memory/summarization, cheaper) → e.g. `claude-haiku-...`
- **Embedding model** → local by default

| Provider | env / setting |
|----------|---------------|
| Anthropic | `A0_SET_chat_model_provider=anthropic` + `A0_SET_chat_model_name=claude-sonnet-4-5` + `ANTHROPIC_API_KEY` |
| OpenRouter | provider `openrouter`, name `anthropic/claude-...` |
| Ollama (local) | provider `ollama`, API URL `http://host.docker.internal:11434` |

API keys go in Settings → External Services (or env). A paid ChatGPT/Claude **subscription ≠ API credits** — you need an API key.

## Dashboard auth & exposure
- Set a **username/password** in Settings on first run.
- Keep the UI on **Tailscale only** (the deploy script binds there). For LAN, front it with a `caddy` reverse proxy + `basic_auth` (same pattern as `/hermes`).
- Remote without exposing the port: SSH tunnel `ssh -L 50080:localhost:50080 <host>` → `http://localhost:50080`.

## MCP — expose the sandbox to Claude
Agent Zero can run as an MCP server (`agent0ai/code-execution-mcp`) so Claude/other clients execute code inside its
isolated sandbox. Only connect **trusted** clients — the MCP layer delegates security to the caller.

## Manage / troubleshoot
```bash
docker logs -f agent-zero
docker stats --no-stream agent-zero          # verify caps are applied
docker exec -it agent-zero bash              # shell inside the sandbox
docker restart agent-zero
docker inspect -f '{{.HostConfig.Memory}} {{.HostConfig.NanoCpus}}' agent-zero   # caps in bytes/nanocpus
```
| בעיה | פתרון |
|------|-------|
| UI לא עולה | בדוק `docker ps`, port mapping `50080:80`, ו-`docker logs` |
| המודל לא עונה | חסר `ANTHROPIC_API_KEY` / provider שגוי ב-Settings |
| נפילה אחרי upgrade | מאונט שגוי — חייב להיות **רק** `a0_usr:/a0/usr` |
| צריך docker-in-docker | `privileged: true` ב-compose (סיכון — רק אם הכרחי) |

## Self-heal
Deploy with `--restart unless-stopped`. Update via Settings → Update → Self Update (or re-pull the image + recreate).
For fleet self-heal, add a `--check` cron that re-runs the deploy script if the container is missing.

## Related Skills
- `/hermes` — Nous Hermes inference gateway (can be Agent Zero's provider)
- `/ruflo` — dual-mode orchestration (Claude Code 🔵 + Codex 🟢)
- `/engineering-pro` — security audit before exposing a code-exec service
- `/observability` — monitor the container (Netdata fleet stack)
