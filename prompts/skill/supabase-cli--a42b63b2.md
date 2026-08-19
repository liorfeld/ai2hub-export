---
title: "supabase-cli"
type: "skill"
tags: ["kit","skill","supabase cli","supabase-cli","supabase link","supabase db push","supabase migration","supabase functions"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T06:13:25.587719+00:00"
id: "a42b63b2-2025-4a05-9ab2-44aee2c93756"
---

> Operate the official Supabase CLI (supabase/cli) — link projects, run DB migrations (push/pull/diff/new), manage Edge Functions, secrets, branches, and a full local stack (supabase start, Docker). Triggers - "supabase cli", "supabase-cli", "supabase link", "supabase db push", "supabase migration", "supabase functions", "supabase start", "supabase login", "edge functions".

# Supabase CLI

[supabase/cli](https://github.com/supabase/cli) is Supabase's official command-line tool for managing projects: linking,
database migrations, Edge Functions, secrets, branches, and a full local dev stack. It is the **automation/CI** surface
for Supabase — distinct from `/supabase-mcp` (the agent's in-chat query tool) and `/migrations` (the safe
schema-change *workflow* that this CLI executes).

## Install (fleet-wide via kit-update)
```bash
npm install -g supabase        # global; or use `npx supabase ...` (Node 18+)
supabase --version
# kit-update verifies the CLI is present on every server; deploy-supabase-cli.sh --check reports it.
```

## Core workflow
```bash
supabase login                              # opens browser / uses SUPABASE_ACCESS_TOKEN (PAT)
supabase link --project-ref <ref>           # bind this repo to a remote project
supabase migration new <name>               # create a migration file
supabase db diff -f <name>                  # capture schema changes as a migration
supabase db push                            # apply migrations to the linked DB
supabase db pull                            # pull remote schema into local migrations
```
> ⚠️ Run schema changes through the **`/migrations`** workflow (review + rollback). Never `db push` blindly on prod.

## Edge Functions & secrets
```bash
supabase functions new <fn>
supabase functions serve <fn>               # local test
supabase functions deploy <fn>              # ship to the project
supabase secrets set KEY=value              # function env (never commit secrets)
```

## Local dev stack
```bash
supabase start          # full local Postgres + Studio + Auth + Storage (requires Docker)
supabase status         # ports / keys for the local stack
supabase stop
```

## Security
- `SUPABASE_ACCESS_TOKEN` (PAT) from env / `~/.devops-secrets`; never commit it. `~/.supabase` is per-user.
- Function secrets via `supabase secrets set`, not in code.

## Verify
```bash
supabase --version
supabase projects list                      # login valid?
bash ~/DevOPS/deploy-supabase-cli.sh --check
```
| בעיה | פתרון |
|------|-------|
| `supabase: command not found` | `npm i -g supabase` (או `npx supabase`); הרץ kit-update |
| `not logged in` | `supabase login` או `export SUPABASE_ACCESS_TOKEN=...` |
| `supabase start` נכשל | Docker לא רץ — התחל Docker; אמת פורטים פנויים |
| migration drift | `supabase db diff` ואז `/migrations` ל-apply בטוח |

## Related Skills
- `/migrations` — safe schema-change workflow (rollback, rolling migrations) — runs on top of this CLI
- `/supabase-mcp` — query/manage Supabase from inside the agent (MCP)
- `/api`, `/supabase-oauth-nextjs` — Next.js + Supabase app patterns
