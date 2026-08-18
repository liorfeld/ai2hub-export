---
title: "figma"
type: "skill"
tags: ["kit","skill","figma"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T01:08:05.47331+00:00"
id: "60cd0596-88a0-4234-ac73-2d5b1c5c0568"
---

> Figma MCP integration - Extract designs, tokens, components, screenshots. Convert Figma to code. Design system sync.

# FIGMA.md - Figma MCP Integration Guide

## MCP Tools Reference

### Read Tools (תמיד זמינים)

| Tool | תיאור | פרמטרים |
|------|--------|---------|
| `get_design_context` | חילוץ עיצוב + styling → React + Tailwind קוד | `fileKey`, `nodeId?` |
| `get_variable_defs` | משתנים ו-design tokens (צבעים, spacing, typo) | `fileKey`, `nodeId?` |
| `get_metadata` | מבנה שכבות XML (IDs, שמות, מיקומים, גדלים) | `fileKey`, `nodeId?` |
| `get_screenshot` | צילום מסך של אלמנט (base64) | `fileKey`, `nodeId?` |
| `get_code_connect_map` | מיפוי node IDs → קומפוננטות בקוד | `fileKey`, `nodeId?` |
| `get_code_connect_suggestions` | הצעות אוטומטיות ל-Code Connect | `fileKey`, `nodeId?` |
| `get_context_for_code_connect` | הגדרות props של קומפוננטה | `fileKey`, `nodeId`, `clientFrameworks`, `clientLanguages` |
| `search_design_system` | חיפוש ב-design library (קומפוננטות, variables, styles) | query text |
| `get_figjam` | מטאדאטה של FigJam diagram | `fileKey`, `nodeId` |
| `whoami` | זהות המשתמש והתוכנית | — |

### Write Tools (Remote MCP בלבד)

| Tool | תיאור |
|------|--------|
| `use_figma` | יצירה/עריכה/מחיקה של כל אובייקט ב-Figma |
| `generate_figma_design` | המרת דף web לשכבות עיצוב ב-Figma |
| `create_new_file` | יצירת קובץ Design/FigJam חדש |
| `generate_diagram` | יצירת FigJam diagram מ-Mermaid/טקסט |
| `create_design_system_rules` | יצירת קובצי כללים ל-design system |
| `add_code_connect_map` | מיפוי node → קומפוננטה בקוד |
| `send_code_connect_mappings` | אישור מיפויים batch |

---

## פרמטרים - איך למצוא

### fileKey
מתוך URL של Figma:
```
https://www.figma.com/design/do4pJqHwNwH1nBrrscu6Ld/My-Project
                              ^^^^^^^^^^^^^^^^^^^^^^^^
                              זה ה-fileKey
```

### nodeId
מתוך URL (פרמטר `node-id`):
```
?node-id=42:100  →  nodeId = "42:100"
?node-id=0-1     →  nodeId = "0:1"   (מחליפים - ב-:)
```

---

## Workflows

### 1. Figma → Code (הנפוץ ביותר)

```
שלב 1: get_screenshot        → ראה את העיצוב
שלב 2: get_design_context    → קבל מבנה + styling
שלב 3: get_variable_defs     → קבל design tokens
שלב 4: בנה קומפוננטה עם הנתונים
```

**חובה לפני בנייה:**
- בדוק קומפוננטות קיימות בפרויקט (`ls src/components/`)
- בדוק צבעים קיימים (`grep -roh "bg-\[#..." src/`)
- התאם ל-design system של הפרויקט, לא רק ל-Figma

### 2. Design System Sync

```
שלב 1: get_variable_defs     → חלץ כל הטוקנים
שלב 2: search_design_system  → מצא קומפוננטות
שלב 3: עדכן tailwind.config / CSS variables
```

### 3. Code Connect (סנכרון דו-כיווני)

```
שלב 1: get_code_connect_suggestions  → מצא קומפוננטות לא ממופות
שלב 2: get_context_for_code_connect  → קבל props
שלב 3: add_code_connect_map          → צור מיפוי
```

### 4. ביקורת עיצוב (Design Review)

```
שלב 1: get_screenshot     → צילום של הדף
שלב 2: get_metadata       → מבנה שכבות
שלב 3: get_variable_defs  → טוקנים בשימוש
שלב 4: השווה מול הקוד הקיים
```

---

## כללים

1. **לא להמציא** — אם ה-design token לא קיים ב-Figma, אל תמציא אחד
2. **RTL First** — כל תרגום מ-Figma חייב להתאים ל-RTL (ps/pe, flex-row-reverse)
3. **Tailwind Tokens** — העדף CSS variables / Tailwind config על inline values
4. **Screenshot First** — תמיד תתחיל עם `get_screenshot` כדי לראות מה בונים
5. **פרויקט גובר** — design system של הפרויקט גובר על מה שב-Figma אם יש סתירה
6. **Responsive** — Figma הוא static, הקוד חייב להיות responsive

## שילוב עם סקילים אחרים

| מצב | טען גם |
|-----|--------|
| בניית קומפוננטה מ-Figma | `/design` + `/figma` |
| עמוד שלם מ-Figma | `/design-pro` + `/figma` |
| ביקורת Figma vs קוד | `/uiux-review` + `/figma` |
| Design tokens sync | `/figma` בלבד |
