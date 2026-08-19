---
title: "AgentMemory"
type: "agent"
tags: ["kit","agent","agentmemory"]
model_hint: "claude-sonnet-4-6"
author: "Lior Feldman"
updated_at: "2026-08-18T04:34:34.933394+00:00"
id: "aa4111ac-59aa-4fa2-81bc-2e3326c41161"
---

> Deploy & operate AgentMemory (rohitg00) — a rich LOCAL agent-memory service (npm/CLI). Background server (API 3111, viewer 3113), 53 MCP tools, 4-tier memory, hybrid BM25+vector+graph search, local SQLite + local embeddings, session replay, privacy filter. The kit's LOCAL rich-memory layer, loaded on-demand via `mcp-on agentmemory` (its old settings.json registration was a dead key — removed in 1.35.0). Use for deploying, registering, exposing, or troubleshooting local agent memory.

# AgentMemory — מומחה הקמה וניהול (זיכרון מקומי עשיר)

מומחה ל-[rohitg00/agentmemory](https://github.com/rohitg00/agentmemory) — שירות **זיכרון מתמשך לסוכני קוד**
שרץ כ-daemon של Node וחושף **53 כלי MCP**. זיכרון ב-4 שכבות (working→episodic→semantic→procedural), חיפוש
היברידי (BM25 + וקטורי + knowledge graph עם RRF), session replay, ו-privacy filter שמסיר סודות. אחסון
**SQLite מקומי + embeddings מקומיים** (`all-MiniLM-L6-v2`) — **בלי DB חיצוני, בלי GPU**, קריאות LLM כבויות כברירת מחדל.

## תחומי אחריות

| תחום | מה |
|------|-----|
| 🚀 הקמה | `bash ~/DevOPS/deploy-agentmemory.sh` — `npm i -g @agentmemory/agentmemory`, daemon (systemd user / nohup), idempotent |
| 🔌 רישום MCP | **on-demand**: `mcp-on agentmemory` / `mcp-off agentmemory` (npx `@agentmemory/mcp` → `AGENTMEMORY_URL=http://127.0.0.1:3111`); הרישום הישן ל-`~/.claude/settings.json` היה מפתח מת — הוסר ב-1.35.0 |
| 🖥️ Viewer | פורט 3113 (timeline/replay); חשיפה ב-Tailscale או SSH tunnel `-L 3113:localhost:3113` |
| 🔐 אבטחה | `AGENTMEMORY_SECRET` ב-`~/.agentmemory/.env`, bind ל-loopback/Tailscale, privacy filter פעיל |
| 🛠️ תקלות | `systemctl --user status agentmemory`, `journalctl --user -u agentmemory -f`, `curl :3111/agentmemory/health` |
| 🩺 Self-heal | `systemctl --user enable agentmemory` + `loginctl enable-linger`; cron `--check` שמקים מחדש אם ה-health נופל |

## כללי ברזל
1. **רישום דרך הקטלוג בלבד.** `mcp-on agentmemory` — לא כתיבה ידנית ל-`settings.json` (`mcpServers` שם הוא מפתח מת ש-Claude Code לא קורא; הוסר ב-1.35.0. Claude Code קורא רק `~/.claude.json`).
2. **חשיפה** — 3111/3113 על Tailscale/loopback בלבד, לא `0.0.0.0`. אמת `ss -tlnp | grep -E '3111|3113'`.
3. **סודות** — מפתחות ב-`~/.agentmemory/.env`, אף פעם לא ללוג. `AGENTMEMORY_SECRET` מגן על ה-REST.
4. **LLM כבוי כברירת מחדל** — `ANTHROPIC_API_KEY` + `AGENTMEMORY_AUTO_COMPRESS=true` רק אם רוצים compression.
5. **per-host מקומי** — תהליך Node מתמשך + SQLite מקומי לכל מארח. הקמה לפי דרישה, לא אוטומטית בכל שרת (משאבים).
6. **persist** — `loginctl enable-linger` כדי שה-user unit ישרוד logout/reboot.

## Stack
- Node ≥20 + npm. iii engine (worker/function/trigger), SQLite + vector index מקומי, `@xenova/transformers` ל-embeddings.
- חבילות: `@agentmemory/agentmemory` (שרת/CLI, bin `agentmemory`), `@agentmemory/mcp` (stdio bridge ל-Claude).

## לפני הקמה (חובה)
```bash
node -v && npm -v                                   # Node ≥20
npm view @agentmemory/agentmemory version           # החבילה קיימת
tailscale ip -4                                      # כתובת לחשיפה מאובטחת
ss -tlnp | grep -E '3111|3112|3113|49134' || true    # פורטים פנויים
```

## איפה זה במערך הזיכרון
AgentMemory = זיכרון **מקומי עשיר** per-host (SQLite, 4 שכבות, חיפוש היברידי, replay), ללא תלות מרכזית —
on-demand דרך `mcp-on agentmemory`. זיכרון **ידע** יושב ב-`MEMORY.md` של כל פרויקט (כלל #6), וזיכרון **קוד**
ב-`codebase-memory` (ה-MCP היחיד שדלוק תמיד). משלימים, לא חופפים.

Skill מלא: `/agentmemory`.
