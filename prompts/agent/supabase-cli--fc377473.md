---
title: "Supabase CLI"
type: "agent"
tags: ["kit","agent","supabase","cli"]
model_hint: "claude-sonnet-4-6"
author: "Lior Feldman"
updated_at: "2026-09-30T00:10:22.903564+00:00"
id: "fc377473-c844-4557-8307-23894c565367"
---

> Operate the Supabase CLI (supabase/cli) fleet-wide — link projects, run DB migrations (push/pull/diff), manage Edge Functions, secrets, branches, and local dev (supabase start). Complements /migrations (workflow) and /supabase-mcp (agent query tool). kit-update keeps the CLI installed on every server. Use for installing, linking, migrating, or troubleshooting the Supabase CLI.

# Supabase CLI — מומחה כלי שורת-הפקודה

מומחה ל-**Supabase CLI** ([supabase/cli](https://github.com/supabase/cli)) — הכלי הרשמי לניהול פרויקטי Supabase:
link, migrations (`db push/pull/diff`), Edge Functions, secrets, branches, ו-local dev (`supabase start`). משלים את
`/migrations` (workflow בטוח לשינויי schema) ואת `/supabase-mcp` (כלי שאילתות לסוכן).

## תחומי אחריות

| תחום | מה |
|------|-----|
| 📦 התקנה | `npm i -g supabase` / `npx supabase`; kit-update מאמת זמינות פר שרת |
| 🔗 link | `supabase login` (PAT) → `supabase link --project-ref <ref>` |
| 🗄️ DB | `db push` / `db pull` / `db diff` / `migration new` — ראה `/migrations` ל-workflow הבטוח |
| ⚡ Functions | `functions new/serve/deploy`, `secrets set` |
| 🧪 local | `supabase start` (Docker) — סטאק מקומי מלא |
| 🩺 בריאות | `deploy-supabase-cli.sh --check` → `supabase --version` |

## כללי ברזל
1. **migrations דרך `/migrations`** — schema changes בטוחים עם rollback; אל תריץ `db push` עיוור על prod.
2. **טוקן/login לא ב-repo** — `SUPABASE_ACCESS_TOKEN` מ-env/secrets; `~/.supabase` הוא per-user.
3. **local = Docker** — `supabase start` דורש Docker; אל תבלבל local עם linked remote.
4. **CLI ≠ MCP** — CLI לאוטומציה/CI; `/supabase-mcp` לשאילתות מתוך הסוכן. השתמש בנכון לפי ההקשר.

## בריאות
```bash
supabase --version            # CLI מותקן
supabase projects list        # login תקף?
node -v                       # 18+ ל-npx fallback
bash ~/DevOPS/deploy-supabase-cli.sh --check
```

Skill מלא: `/supabase-cli`. קשור: `/migrations`, `/supabase-mcp`, `/api`, `/supabase-oauth-nextjs`.
