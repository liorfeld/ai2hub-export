---
title: "cli-anything"
type: "skill"
tags: ["kit","skill","cli-anything","generate cli","agent-native cli","harness.md","software for agents","hkuds"]
model_hint: null
author: "Lior Feldman"
updated_at: "2026-08-18T01:08:05.47331+00:00"
id: "53b1296a-1a10-4881-95d0-808557f3007d"
---

> CLI-Anything — מסגרת להפיכת תוכנה בעלת source code ל-CLI agent-native. 7-phase pipeline, HARNESS.md SOP, Click + REPL, JSON output, 2,130+ tests על 16+ אפליקציות. Triggers - "cli-anything", "generate cli", "agent-native cli", "HARNESS.md", "wrap software as CLI", "software for agents", "HKUDS", "click cli", "repl cli".

# CLI-ANYTHING.md — Software → Agent-Native CLI Guide

מבוסס על [HKUDS/CLI-Anything](https://github.com/HKUDS/CLI-Anything) — Claude Code plugin שהופך כל תוכנה עם source code ל-CLI מובנה ל-AI agents.

---

## מתי להפעיל

- יש תוכנה (GUI/desktop/server) שרוצים שסוכן AI יפעיל בלי UI automation
- יש בקשה "תעטוף את X ב-CLI" / "תפוך את X ל-agent-native"
- מתחילים פרויקט לבנות CLI wrapper מעל אפליקציה קיימת
- רוצים להוסיף commands ל-CLI שנוצר דרך CLI-Anything (`/cli-anything:refine`)
- צריך לוודא עמידה ב-HARNESS.md standard

---

## Installation Flow

```bash
# בתוך Claude Code:
/plugin marketplace add HKUDS/CLI-Anything
/plugin install cli-anything

# הפעלה על target:
/cli-anything ./my-software
```

**Prerequisites:**
- Python 3.10+
- Target software מותקנת על המכונה (GIMP/Blender/LibreOffice...)
- Source code של הפרויקט נגיש
- Claude Opus 4.6+ (frontier model חובה)

---

## 7-Phase Pipeline

| Phase | שם | Input | Output |
|-------|-----|-------|--------|
| 1 | Codebase Analysis | source tree | capability map |
| 2 | Command Architecture | capability map | `HARNESS.md` (SOP) |
| 3 | Click Implementation | HARNESS.md | `agent-harness/` package + `repl_skin.py` |
| 4 | Test Planning | commands | test matrix (unit + e2e + subprocess) |
| 5 | Test Writing | test matrix | pytest suite (real backends) |
| 6 | Documentation | all above | README + SKILL.md (Phase 6.5 auto-gen) |
| 7 | PyPI Publishing | package | `pip install cli-anything-<software>` |

**Refine loop**: אחרי Phase 7, אפשר לחזור עם `/cli-anything:refine` להרחיב coverage.

---

## HARNESS.md Standard

כל CLI שנוצר חייב `HARNESS.md` ב-root של `agent-harness/` עם:

```markdown
# <Software> CLI Harness

## Purpose
<מה ה-CLI עושה + מה הוא לא עושה>

## Command Taxonomy
<hierarchy מלא של subcommands>

## State Model
<מה שמור בין commands ב-REPL>

## JSON Schema
<מבנה ה-output לכל command>

## Backend Invocation
<איך ה-CLI קורא ל-backend האמיתי>

## Test Strategy
<unit + e2e + output verification>

## Known Limitations
<מה לא נתמך ולמה>
```

`/cli-anything:validate` בודק ש-HARNESS.md תקין.

---

## Click Conventions

### חובה על כל command
```python
@click.command()
@click.option('--json', 'output_json', is_flag=True, help='Structured output')
@click.option('--project', type=click.Path(), help='Project state file')
def my_command(output_json, project):
    result = invoke_real_backend(...)
    if output_json:
        click.echo(json.dumps(result))
    else:
        click.echo(human_readable(result))
```

### Dual-Mode Pattern
```bash
# Subcommand mode (scripting)
cli-anything-gimp --json image new --width 1920 --height 1080

# REPL mode (interactive agent session)
cli-anything-gimp
>>> image new --width 1920
>>> layer add --name "sky"
>>> export --format png
```

### REPL Skin
כל CLI משתמש ב-`repl_skin.py` אחיד — consistency בין כל ה-CLIs שנוצרו.

---

## Testing Strategy

### 3 שכבות חובה

| שכבה | מטרה | דוגמה |
|------|------|--------|
| **Unit** | logic עם synthetic data | `test_color_parsing()` |
| **E2E** | real backend invocation | `test_blender_renders_actual_png()` |
| **Subprocess** | CLI via shell | `subprocess.run(['cli-anything-gimp', '--json', 'image', 'new'])` |

### Output Verification (לא רק exit code)
```python
# PNG magic bytes
assert output.read(8) == b'\x89PNG\r\n\x1a\n'

# Audio RMS
assert 0.01 < calculate_rms(wav_path) < 0.99

# PDF structure
assert pdf.pages[0].extract_text() == expected
```

### כלל ברזל: אין skip
אם backend לא זמין — הבדיקה נכשלת. לא `@pytest.skipif`, לא fallback.

---

## Plugin Commands Reference

| Command | מטרה | דוגמה |
|---------|------|--------|
| `/cli-anything <path>` | pipeline מלא (7 שלבים) | `/cli-anything ./gimp` |
| `/cli-anything:refine <path>` | הרחבת coverage עם gap analysis | `/cli-anything:refine ./gimp "I want batch image processing"` |
| `/cli-anything:test <path>` | run tests + update results | `/cli-anything:test ./gimp` |
| `/cli-anything:validate <path>` | בדיקת HARNESS.md compliance | `/cli-anything:validate ./gimp` |

---

## Supported Software Matrix

| Category | Software | Tests |
|----------|----------|-------|
| Creative | GIMP | 107 |
| Creative | Blender | 208 |
| Creative | Inkscape | 202 |
| Creative | Krita | ✓ |
| Audio | Audacity | 161 |
| Video | Kdenlive | 155 |
| Video | Shotcut | 154 |
| Video | VideoCaptioner | ✓ |
| Productivity | LibreOffice | 158 |
| Productivity | Zotero | ✓ |
| Productivity | Obsidian | ✓ |
| Streaming | OBS Studio | 153 |
| Streaming | Openscreen | ✓ |
| Diagramming | Draw.io | 138 |
| AI/ML | ComfyUI | ✓ |
| AI/ML | Ollama | ✓ |
| AI/ML | Exa | ✓ |
| AI/ML | NotebookLM | ✓ |
| Infra | AdGuard Home | ✓ |
| Infra | n8n | ✓ |
| Infra | Dify Workflow | ✓ |
| Game Dev | Godot | ✓ |
| Meeting | Zoom | ✓ |
| Music | MuseScore | ✓ |

**סה"כ**: 2,130+ passing tests, 100% pass rate.

---

## Generated CLI Usage Patterns

### Install
```bash
cd <target>/agent-harness
pip install -e .
```

### Subcommand Mode (scripting / CI)
```bash
cli-anything-libreoffice document new -o report.json
cli-anything-libreoffice --project report.json writer add-heading "Q1 Results"
cli-anything-libreoffice --project report.json writer export --format pdf
```

### REPL Mode (interactive agent)
```bash
cli-anything-blender
>>> scene new --name ProductShot
>>> mesh import sphere.obj
>>> render execute --output render.png
>>> exit
```

### Discovery
```bash
cli-anything-gimp --help                # רשימת commands
cli-anything-gimp image --help          # subcommands של image
cli-anything-gimp --json capabilities   # full JSON dump
```

---

## Iron Rules

1. **Authentic Integration** — `invoke_real_backend()` תמיד. אף פעם mock/emulation.
2. **JSON Everywhere** — `--json` flag on every command. output דטרמיניסטי.
3. **Tests Fail, Don't Skip** — backend missing = test failure.
4. **Source Required** — compiled-only → decompile first.
5. **Frontier Model** — Opus 4.6+ בלבד. Sonnet/Haiku יוצרים CLIs חצי-אפויים.
6. **Dual-Mode Always** — REPL + subcommand שניהם, אין ברירה.
7. **Output Verification** — magic bytes / pixels / RMS, לא exit code.
8. **Python 3.10+** — TypeAlias, match/case, modern typing.
9. **No Fallbacks** — אם יכולת X לא זמינה, זרוק שגיאה ברורה.
10. **SKILL.md Auto-gen** — Phase 6.5 מייצר את זה מ-Click decorators.

---

## שילוב עם סקילים אחרים

| מצב | טען גם |
|-----|--------|
| לפני Phase 1 | `/spec-driven` — ACs ב-Given/When/Then |
| Phase 3 — code quality | `/code-reviewer` + `/engineering-pro` |
| Phase 5 — tests | `/qa` + `/webapp-testing` |
| Phase 6 — docs | `/doc-coauthoring` |
| Phase 6.5 — SKILL.md | `/skill-creator` |
| Phase 7 — packaging | `/docker-dev` + `/dependency-auditor` |
| לחשוף כ-MCP server | `/mcp-builder` |
| אבטחה של ה-CLI שנוצר | `/engineering-pro` + `/skill-security-auditor` |
| Observability לסוכן | `/observability` |

---

## Unique Capabilities

1. **Refine Loop** — הרחבה ממוקדת: `"I want more CLIs on image batch processing"` → gap analysis → commands חדשים.
2. **SKILL.md Auto-generation** — כל CLI מקבל skill definition אוטומטי ל-AI discovery.
3. **CLI-Hub Meta-Skill** — סוכנים יכולים לגלות + להתקין CLIs מ-registry חי ללא אדם.
4. **Cross-Platform** — Windows via Cygwin guards + WSL/Git Bash.
5. **Rendering Validation** — בודק את ה-output האמיתי (PNG pixels, PDF bytes, audio RMS), לא רק exit code.

---

## Limitations (דע את הגבולות)

- דורש frontier LLM — Opus 4.6+ (Sonnet/Haiku יוצרים CLIs חלקיים)
- דורש source code — binaries בלבד → צריך decompile
- בדרך-כלל צריך `/cli-anything:refine` 1-2 פעמים כדי להגיע ל-production quality
- התוכנה חייבת להיות מותקנת מקומית — אין graceful degradation
- Python 3.10+ חובה — אין backport

---

## Quick Reference — לזכור

- **Plugin**: `/plugin install cli-anything`
- **Run**: `/cli-anything <path>`
- **Refine**: `/cli-anything:refine <path>`
- **Test**: `/cli-anything:test <path>`
- **Validate**: `/cli-anything:validate <path>`
- **Standard**: `HARNESS.md` at root of `agent-harness/`
- **CLI package**: `cli-anything-<software>` on PyPI
