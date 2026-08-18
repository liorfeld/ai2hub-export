---
title: "supabase-mcp"
type: "skill"
tags: ["kit","skill","supabase mcp","supabase-mcp","register supabase mcp","supabase claude tool","supabase","mcp"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T01:08:05.47331+00:00"
id: "cbc53774-c137-45b7-87f9-9044f6313e93"
---

> Register & operate the official Supabase MCP server (@supabase/mcp-server-supabase) so Claude Code + Codex can manage tables, fetch project config, run scoped read-only SQL, and introspect schema directly from Supabase. Triggers - "supabase mcp", "supabase-mcp", "@supabase/mcp-server-supabase", "register supabase mcp", "mcp_servers.supabase", "supabase claude tool", "query supabase from claude".

# Supabase MCP — official MCP server registration

[supabase/mcp](https://github.com/supabase/mcp) (Apache-2.0) is Supabase's **official MCP server**
(`@supabase/mcp-server-supabase`). It connects Claude Code, Codex, Cursor, and Windsurf to a Supabase project for
**managing tables, fetching config, querying data, and schema introspection** from inside the agent.

> **This is a registration, not a deployment.** There is no container to run. You **register** the server in
> `~/.claude.json` (`mcpServers`) and `~/.codex/config.toml` (`[mcp_servers.supabase]`) — exactly the pattern the kit
> already uses for the Stitch MCP (`configure_stitch_mcp` in `kit-update`). A hosted variant
> (`https://mcp.supabase.com/mcp`, OAuth) exists as an alternative to the local stdio server.

## Security defaults (do NOT simplify away)
1. **`--read-only`** by default — only enable writes when the user explicitly asks.
2. **`--project-ref=<ref>`** — scope to one project, not the whole org/account.
3. **`SUPABASE_ACCESS_TOKEN` from env / `~/.devops-secrets`** — never hardcode it into a config that lands in git.
4. Prefer a **least-privilege Personal Access Token** over a service-role key.

## Register
```bash
# kit auto-registers via kit-update -> configure_supabase_mcp(). Manual:
export SUPABASE_ACCESS_TOKEN=sbp_xxx            # from env / ~/.devops-secrets, never committed
bash ~/DevOPS/deploy-supabase-mcp.sh --project-ref <ref>     # read-only + scoped, writes Claude + Codex
bash ~/DevOPS/deploy-supabase-mcp.sh --check                 # verify registration + package resolves
```
Codex `~/.codex/config.toml`:
```toml
[mcp_servers.supabase]
command = "npx"
args = ["-y", "@supabase/mcp-server-supabase@latest", "--read-only", "--project-ref=<ref>"]
[mcp_servers.supabase.env]
SUPABASE_ACCESS_TOKEN = "<from env/secrets>"
```
Claude `~/.claude.json` → same shape under `mcpServers.supabase` (`command`/`args`/`env`).

## Transport & auth
- **stdio** (local `npx`, the default here) or **HTTP** (`https://mcp.supabase.com/mcp`, OAuth 2.0 login).
- Optional `--features=<groups>` to enable/disable tool groups; `--read-only` strongly recommended for agent use.

## Verify
```bash
node -v                                                 # 18+ for npx
jq '.mcpServers.supabase' ~/.claude.json                # registered for Claude?
grep -A2 '\[mcp_servers.supabase\]' ~/.codex/config.toml # registered for Codex?
npx -y @supabase/mcp-server-supabase@latest --help      # package resolves?
bash ~/DevOPS/deploy-supabase-mcp.sh --check
```
| בעיה | פתרון |
|------|-------|
| הכלי לא מופיע ב-Claude | אין `mcpServers.supabase` ב-`~/.claude.json` או חסר token — הרץ deploy עם `SUPABASE_ACCESS_TOKEN` |
| `unauthorized` / 401 | PAT שגוי/חסר הרשאות, או `--project-ref` לא תואם לטוקן |
| כתיבה נחסמת | ברירת מחדל `--read-only` — הסר רק אם באמת צריך כתיבה |

## Related Skills
- `/supabase-cli` — the Supabase CLI (migrations, local dev, functions) — complements MCP
- `/migrations` — safe schema-change workflow
- `/api` — Next.js + Supabase backend patterns
- `/mcp-builder` — building your own MCP servers
