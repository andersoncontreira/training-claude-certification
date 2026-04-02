# Exercise 01: Hooks and Automation

**Difficulty:** Beginner
**Domain:** Claude Code
**Goal:** Configure hooks to automate logging, validation, and notifications.

---

## Context

Hooks execute shell commands automatically when specific events occur in Claude Code. They are defined in `.claude/settings.json` and run outside of Claude — meaning they always execute regardless of what Claude says.

---

## Task

Set up a `.claude/settings.json` for this project with the following hooks:

1. **Audit log** — every time the `Bash` tool is called, append the command and timestamp to `/tmp/claude-audit.log`
2. **Block rm -rf** — if Claude tries to run `rm -rf`, block it with exit code 2 and show a clear error message
3. **Completion notification** — when Claude stops responding, print "Claude finished!" to the terminal

---

## Part 1: Create the settings file

Create `.claude/settings.json` in the project root:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "TODO: write bash command that appends to /tmp/claude-audit.log"
          }
        ]
      },
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "TODO: write bash command that blocks rm -rf (hint: read from stdin, check input, exit 2 if dangerous)"
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "TODO: print completion message"
          }
        ]
      }
    ]
  }
}
```

---

## Part 2: Understanding Hook Input

`PreToolUse` hooks receive the tool call information via **stdin as JSON**. For the `Bash` tool, the input looks like:

```json
{
  "tool_name": "Bash",
  "tool_input": {
    "command": "ls -la /tmp"
  }
}
```

Use `jq` to parse this in your hook command:

```bash
# Read the tool input and extract the command
input=$(cat)
command=$(echo "$input" | jq -r '.tool_input.command // ""')
```

---

## Part 3: The Blocking Hook

A hook that exits with code `2` blocks the tool call. Claude receives the hook's stderr as feedback.

```bash
#!/bin/bash
input=$(cat)
command=$(echo "$input" | jq -r '.tool_input.command // ""')

if echo "$command" | grep -q "rm -rf"; then
  echo "BLOCKED: rm -rf is not allowed in this project." >&2
  exit 2
fi

exit 0
```

**Your task:** Inline this as a one-liner in the settings.json `command` field, or reference an external script.

---

## Part 4: Verify It Works

1. Start Claude Code in this project: `claude`
2. Ask Claude to run `ls /tmp` — check that `/tmp/claude-audit.log` has an entry
3. Ask Claude to run `rm -rf /tmp/test` — verify the block works and Claude receives the error message
4. Complete any task and verify "Claude finished!" appears

---

## Hints

- In `settings.json`, the `command` value runs in a shell — you can use `bash -c '...'` for multiline logic
- For complex hooks, reference an external script: `"command": "bash .claude/hooks/audit.sh"`
- The `PostToolUse` hook receives both the input AND the output — useful for logging results

---

## Extension Challenges

1. Add a `PostToolUse` hook for the `Write` tool that logs the file path written to.
2. Create a `UserPromptSubmit` hook that rejects prompts containing the word "delete" and asks for confirmation instead.
3. Add a `PreToolUse` hook that blocks `Write` to any path outside the `src/` directory.

---

## Key Concepts Practiced

- `PreToolUse` vs `PostToolUse` vs `Stop` hooks
- Reading tool input from stdin with `jq`
- Exit code 2 for blocking
- Inline vs. external hook scripts
