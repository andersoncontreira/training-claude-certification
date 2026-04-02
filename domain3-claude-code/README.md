# Domain 3: Claude Code (20%)

Covers Claude Code — Anthropic's official CLI for Claude — including automation, hooks, slash commands, and multi-agent workflows.

## What is Claude Code?

Claude Code is an agentic coding tool that runs in the terminal and integrates with your development environment. It can read/write files, run shell commands, manage git, and call external tools.

## Core Architecture

```
User types prompt
      ↓
Claude Code (CLI)
  ├── Tools: Read, Write, Edit, Bash, Glob, Grep...
  ├── Hooks: shell scripts triggered on events
  ├── MCP servers: external tools via MCP
  └── Subagents: spawned for parallel tasks
```

## Hooks

Hooks are shell commands that execute automatically on specific Claude Code events. They run outside of Claude — in the OS shell — so they are deterministic and always execute regardless of Claude's output.

### Hook Types

| Hook | Trigger | Common Use |
|------|---------|-----------|
| `PreToolUse` | Before any tool call | Validation, logging, blocking |
| `PostToolUse` | After any tool call | Logging, notifications |
| `UserPromptSubmit` | When user submits a prompt | Pre-processing, context injection |
| `Stop` | When Claude stops responding | Notifications, cleanup |

### Hook Configuration (`.claude/settings.json`)

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "echo '[LOG] Bash tool called' >> /tmp/claude-audit.log"
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude finished\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```

### Hook Exit Codes
- `exit 0` — success, Claude continues
- `exit 2` — **block**: Claude is prevented from proceeding, hook's stderr shown to user
- Other non-zero — warning shown, Claude continues

## Custom Slash Commands

Custom commands are Markdown files stored in `.claude/commands/`. Claude reads the file content as a prompt when the command is invoked.

### Creating a command

File: `.claude/commands/review-pr.md`
```markdown
Review the current git diff and provide:
1. A summary of what changed
2. Potential issues or bugs
3. Suggestions for improvement

Focus on correctness and clarity.
```

Usage: `/review-pr`

### Dynamic commands with `$ARGUMENTS`

File: `.claude/commands/explain.md`
```markdown
Explain the following code in simple terms, suitable for a junior developer:

$ARGUMENTS
```

Usage: `/explain path/to/file.py`

## Permissions Model

Claude Code uses a layered permissions system:

```
Global settings  (~/.claude/settings.json)
      ↓
Project settings (.claude/settings.json)
      ↓
Local settings   (.claude/settings.local.json)  [gitignored]
```

### Allow/Deny rules

```json
{
  "permissions": {
    "allow": ["Bash(git:*)", "Read(**)", "Write(src/**)"],
    "deny": ["Bash(rm -rf:*)"]
  }
}
```

## Multi-Agent with Claude Code

Claude Code can spawn subagents (other Claude instances) for parallel work:

- Use the `Agent` tool from within Claude Code
- Each subagent has isolated context
- Results return to the parent agent
- Useful for: parallel file analysis, independent subtasks

## CLAUDE.md

`CLAUDE.md` files are automatically read by Claude Code at startup. Place them at:
- `~/CLAUDE.md` — global instructions
- `<project>/CLAUDE.md` — project-specific instructions

Use them to define: project conventions, architecture notes, forbidden commands, testing instructions.

## Common Exam Topics
- Hook event types and exit codes
- Custom slash command file format and `$ARGUMENTS`
- Permission allow/deny rules syntax
- CLAUDE.md loading hierarchy
- When to use hooks vs. prompts vs. CLAUDE.md
