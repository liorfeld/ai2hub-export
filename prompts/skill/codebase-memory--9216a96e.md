---
title: "codebase-memory"
type: "skill"
tags: ["kit","skill","codebase memory","codebase-memory-mcp","code graph","call graph","index repository","impact analysis"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T01:08:05.47331+00:00"
id: "9216a96e-b6c1-4cf9-94b4-2b395f142f5f"
---

> Code-intelligence memory MCP (DeusData/codebase-memory-mcp, MIT) — indexes a repo into a persistent knowledge graph via tree-sitter AST across 158 languages, giving agents semantic code search, call-graph tracing, and impact/architecture analysis with sub-ms queries. Triggers - "codebase memory", "codebase-memory-mcp", "code graph", "call graph", "index repository", "impact analysis", "trace path", "who calls", "architecture map", "code intelligence", "מפת קוד", "גרף קריאות", "מי קורא ל".

# codebase-memory-mcp — זיכרון מבנה-קוד

[DeusData/codebase-memory-mcp](https://github.com/DeusData/codebase-memory-mcp) (MIT) — MCP server שממפה ריפו ל-**knowledge graph** דרך tree-sitter AST (158 שפות): חיפוש קוד סמנטי, מעקב call-graph, ניתוח impact/ארכיטקטורה, שאילתות תת-מילישנייה. **בינארי C סטטי — אפס תלויות, בלי Node** (לכן רץ גם על הוסטים עם Node-18), 100% מקומי, בלי מפתחות.

> **מול הזיכרונות האחרים:** `MEMORY.md` = זיכרון **ידע** per-project (כלל #6) · AgentMemory = זיכרון agent עשיר מקומי (on-demand דרך `mcp-on agentmemory`). codebase-memory זוכר **מבנה קוד** — וזה ה-MCP היחיד שדלוק תמיד (ALWAYS_ON). רץ לצדם — לא מחליף.

## הכלים (MCP)

| כלי | מה |
|-----|-----|
| `index_repository` | בונה/מרענן את הגרף לריפו. **on-demand** — כלום לא מאונדקס אוטומטית. |
| `search_code` / `search_graph` | חיפוש סמנטי בקוד / בגרף. |
| `query_graph` | שאילתת גרף (relationships, dependencies). |
| `trace_path` | מסלול בין שתי ישויות — "מה מחבר את A ל-B". |
| `get_architecture` | תמונת ארכיטקטורה על. |
| `get_code_snippet` | שליפת קטע לפי ישות. |
| `detect_changes` / `index_status` | מה השתנה מאז אינדוקס / מצב האינדקס. |
| `manage_adr` | Architecture Decision Records. |
| `list_projects` / `delete_project` | ניהול פרויקטים מאונדקסים. |

## מתי

- להיכנס לקודבייס לא מוכר → `get_architecture` + `search_code`.
- "מי קורא לפונקציה הזו / מה יישבר אם אשנה" → `trace_path` / `query_graph` (blast-radius).
- לחסוך טוקנים בשאלות קוד — הגרף מחזיר בדיוק את הרלוונטי במקום להזרים קבצים.

## הקמה ותפעול

מותקן **fleet-wide** ע"י kit-update (בינארי נעוץ-גרסה + אימות SHA-256 ל-`~/.local/bin`, רישום MCP ל-Claude ול-Codex). בלי Node, בלי מפתחות.

```bash
bash ~/DevOPS/setup-codebase-memory.sh --check    # binary, version, disk, MCP registration
bash ~/DevOPS/setup-codebase-memory.sh            # install/refresh + register (idempotent)
```

- **שער דיסק:** מדלג בקול על הוסטים disk-full (cache האינדקס עולה דיסק). override: `CODEBASE_MEMORY_MIN_DISK_MB=0`.
- **אבטחה:** לא מריצים את install.sh שלהם (עורך ~43 קונפיגים) — הבינארי מותקן ידנית ורק שתי רשומות MCP שאנחנו שולטים בהן נכתבות. SHA-256 נעוץ נבדק לפני התקנה → asset מזויף נכשל closed.
- אחרי הקמה — restart ל-Claude/Codex כדי לטעון את ה-MCP.

Slash: `/codebase-memory`.
