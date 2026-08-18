---
title: "graphify"
type: "skill"
tags: ["kit","skill","graphify","graphrag","knowledge graph code","code+docs graph","leiden communities","pr impact graph"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T01:08:05.47331+00:00"
id: "c9a0c950-15da-4a56-a507-4624d7d4674d"
---

> Deploy & operate Graphify (Graphify-Labs/graphify, MIT) — a GraphRAG tool that turns a codebase plus its docs, SQL schemas, configs, and PDFs into a queryable knowledge graph. Triggers - "graphify", "graphrag", "knowledge graph code", "code+docs graph", "leiden communities", "pr impact graph", "graph.json mcp", "גרף ידע קוד", "גרף מסמכים".

# Graphify — גרף ידע של קוד+מסמכים (GraphRAG)

[Graphify-Labs/graphify](https://github.com/Graphify-Labs/graphify) (MIT, PyPI `graphifyy`) — הופך קודבייס **וגם** מסמכים/SQL/configs/PDF ל-**knowledge graph** ניתן-לשאילתא. קוד מנותח **100% מקומית** ב-tree-sitter (דטרמיניסטי, בלי LLM, 36+ שפות); מסמכים/PDF עוברים ב-LLM שתבחר. קהילות Leiden, תיוג קשת `EXTRACTED` מול `INFERRED`, פלט `graph.html`/`GRAPH_REPORT.md`/`graph.json`, שאילתת NL + path-trace + PR-impact.

> **מול codebase-memory (fleet-wide):** codebase-memory = **מבנה קוד** דטרמיניסטי מהיר (בינארי C סטטי, בלי Node, בלי LLM, call-graph/impact). graphify = **GraphRAG** — מוסיף בליעת מסמכים/PDF, קשתות INFERRED, שאילתת NL, קהילות. **משלימים** (מבנה מול גרף-ידע סמנטי), רצים זה לצד זה. לכן graphify הוא deploy-on-demand — לא דוחפים שני כלי code-graph לכל 47 השרתים.

## התקנה ותפעול
```bash
bash ~/DevOPS/deploy-graphify.sh                     # uv tool install "graphifyy[mcp]" (Python≥3.10)
graphify extract <repo> --backend claude-cli         # keyless (מנוי Claude); הקוד לא יוצא מהמכונה
bash ~/DevOPS/deploy-graphify.sh --register-graph <repo>/graphify-out   # רישום MCP לגרף הספציפי
```
ה-MCP הוא **per-graph** (`python -m graphify.serve <graph.json>`) — משרת גרף אחד בנוי מראש, לכן הרישום הוא per-project (`--register-graph`), לא שרת גנרי לצי.

## אבטחה
- ניתוח קוד מקומי לגמרי (יתרון פרטיות); רק מסמכים/PDF פוגשים LLM.
- MCP מעל HTTP → תמיד `--api-key`, לעולם לא `--host 0.0.0.0` בלי אימות.

## Scale (כלל #8)
Backend ברירת-מחדל `claude-cli` (keyless, מנוי) תואם את מדיניות ה-premium-first; `--backend ollama` לריצה ללא egress.
