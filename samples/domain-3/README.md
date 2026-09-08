# Domain 3.1 — CLAUDE.md Hierarchy, Scope, and Modular Organization

## Key Concepts

### Configuration levels (all combine; most specific wins on conflict)

| Level | File | Versioned? |
|---|---|---|
| User | `~/.claude/CLAUDE.md` | No — personal |
| Project root | `CLAUDE.md` or `.claude/CLAUDE.md` | Yes — shared with team |
| Subdirectory | `<subdir>/CLAUDE.md` | Yes — context-specific |
| Personal override | `CLAUDE.local.md` (in `.gitignore`) | No — personal |

### Inheritance behavior

Claude Code walks up the directory tree from the current working directory,
loading every CLAUDE.md found along the path. Opening Claude Code inside
`project/frontend/templates/site/` (no CLAUDE.md there) will still load:

1. `~/.claude/CLAUDE.md` (user level)
2. `project/CLAUDE.md` (project root)
3. `project/frontend/CLAUDE.md` (subdirectory)

No duplication needed. A CLAUDE.md in `site/` would only be necessary to
add or override rules specific to that context.

### Performance thresholds (exam-relevant)

| Lines | Rule compliance rate |
|---|---|
| < 200 | > 92% |
| > 400 | 71% |

> **Product behavior (real, but separate from exam content):**
> The Claude Code CLI shows a warning when a CLAUDE.md exceeds **40,000 characters**:
> `⚠ Large CLAUDE.md will impact performance (41.1k chars > 40.0k)`
> This is a UI/performance warning from the tool. The exam tests the **line threshold**,
> not the character limit. Keep both in mind but don't confuse them in exam questions.

### `@import` syntax

Keeps the main CLAUDE.md short and acts as a table of contents.
Referenced files are loaded as if their content were inline.

```
@import .claude/rules/python-code.md
@import .claude/rules/testing.md
```

### `.claude/rules/` directory

Alternative to a monolithic CLAUDE.md. Each file has a single responsibility
and stays under 200 lines. The main CLAUDE.md imports them all.

---

## Sample Project

See `my-python-project/` for a complete working example with:

- `CLAUDE.md` — root file using `@import`
- `CLAUDE.local.md` — personal overrides (would be in `.gitignore`)
- `.claude/rules/python-code.md` — OOP code style rules
- `.claude/rules/documentation.md` — docs language and format rules
- `.claude/rules/testing.md` — unittest rules
- `.claude/rules/project-structure.md` — directory layout rules
- `.claude/rules/docker.md` — Docker and container rules
