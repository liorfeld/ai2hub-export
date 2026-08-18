---
title: "codex"
type: "skill"
tags: ["kit","skill","codex","openai codex","omx","oh-my-codex","om","omd"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T01:08:05.47331+00:00"
id: "864803fe-43dd-44fb-949d-633c49e58cab"
---

> OpenAI Codex CLI (@openai/codex) under the oh-my-codex (omx) runtime — the default 🟢 build channel of the kit's Dual-Mode workflow (Claude Code 🔵 plans/reviews, Codex 🟢 implements/refactors/optimizes/boilerplate; the second build channel is Grok 4.6 via /grok). Covers om/omr/omd aliases, omx team runs, ~/.codex/config.toml MCP wiring, and the kit-update fleet distribution (codex + omx install, ~/.codex/skills sync). Use to run Codex/omx, do dual-mode handoff, or troubleshoot the Codex runtime. Triggers - "codex", "openai codex", "@openai/codex", "omx", "oh-my-codex", "om ", "omd", "omx team", "dual mode", "codex runtime", "~/.codex".

# Codex (OMX) — the 🟢 implementation runtime

[openai/codex](https://github.com/openai/codex) (`@openai/codex`) is OpenAI's coding-agent CLI. In this kit it runs under
**oh-my-codex (omx)** as the **default 🟢 build channel** of the Dual-Mode workflow (iron-rule #8; the second 🟢 channel is Grok 4.6 via `/grok`):

| Track | Owns |
|-------|------|
| 🔵 Claude Code | architecture, security, tests, code review, PRD decomposition |
| 🟢 Codex (OMX) | implementation, refactoring, optimization, boilerplate |

This is the canonical `/codex` skill. Full dual-mode orchestration (Claude + Codex via claude-flow v3) is `/ruflo`.

## Runtime commands
```bash
om "<task>"                 # primary shortcut = omx team 3:executor "<task>"
omx team status <team>      # control: status | resume <team> | shutdown <team>
omd                         # = omx doctor --team   (run before any swarm)
codex --version             # the underlying @openai/codex CLI
```
- Broad, multi-file, refactor-heavy or handoff-heavy work → default to **`omx team`**.
- `/prompts:planner`, `/prompts:architect`, `/prompts:executor`, `/prompts:verifier` are the default OMX surfaces.
- **Do not** run `omx agents-init .` in a normal KIT project — the kit templates (`CLAUDE.md`/`AGENTS.md`) are the source of truth.

## MCP & config
`~/.codex/config.toml` holds `[mcp_servers.*]` blocks (e.g. Stitch, `/supabase-mcp`). The kit writes these via
`kit-update` (`configure_stitch_mcp`, `configure_supabase_mcp`). Keep provider/API keys in env or the config's `env`
table — never in the repo.

## Fleet distribution (already automatic)
`kit-update` (twice daily) installs `@openai/codex@latest` + `oh-my-codex@latest`, syncs DevOPS skills into
`~/.codex/skills/`, pulls central `omx-*` skills, and configures the Codex MCP servers on every server. No manual deploy
needed; `deploy-codex.sh --check` reports the local state.

## Verify
```bash
codex --version                       # CLI installed
omx version && omd                    # omx present + doctor green
ls ~/.codex/skills | head             # skills synced (omx-* + DevOPS skills)
grep -c '^\[mcp_servers' ~/.codex/config.toml   # MCP servers registered
bash ~/DevOPS/deploy-codex.sh --check
```
| בעיה | פתרון |
|------|-------|
| `codex: command not found` | kit-update לא רץ / Node ישן — `npm i -g @openai/codex@latest` ואז `kit-update` |
| `omx` חסר | `npm i -g oh-my-codex@latest`; אמת PATH; `omd` |
| MCP לא נטען ב-Codex | בדוק `[mcp_servers.*]` ב-`~/.codex/config.toml`; הרץ kit-update לרישום מחדש |

## Related Skills
- `/ruflo` — full Dual-Mode orchestration (Claude 🔵 + Codex 🟢) via claude-flow v3
- `/supabase-mcp`, `/migrations` — MCP + DB tooling Codex consumes
- `/engineering-pro`, `/ponytail` — quality gates over what Codex generates
