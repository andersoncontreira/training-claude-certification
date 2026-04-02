# Domain 3: Resources & References

## Official Documentation

- [Claude Code overview](https://docs.anthropic.com/en/docs/claude-code)
- [Claude Code hooks](https://docs.anthropic.com/en/docs/claude-code/hooks)
- [Claude Code settings](https://docs.anthropic.com/en/docs/claude-code/settings)
- [Custom slash commands](https://docs.anthropic.com/en/docs/claude-code/slash-commands)
- [CLAUDE.md reference](https://docs.anthropic.com/en/docs/claude-code/memory)
- [Permissions and security](https://docs.anthropic.com/en/docs/claude-code/security)
- [MCP in Claude Code](https://docs.anthropic.com/en/docs/claude-code/mcp)

## Key Files and Locations

| File | Location | Purpose |
|------|----------|---------|
| `settings.json` | `~/.claude/` or `.claude/` | Hooks, permissions, config |
| `settings.local.json` | `.claude/` | Local overrides (gitignored) |
| `CLAUDE.md` | `~/` or project root | Persistent instructions for Claude |
| Custom commands | `.claude/commands/*.md` | Project-specific slash commands |

## Hook Event Reference

| Event | When it fires | Can block? |
|-------|---------------|-----------|
| `PreToolUse` | Before tool execution | Yes (exit 2) |
| `PostToolUse` | After tool execution | No |
| `UserPromptSubmit` | On user prompt submit | Yes (exit 2) |
| `Stop` | When Claude stops | No |

## Permissions Syntax Examples

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run:*)",
      "Bash(git:*)",
      "Read(**)",
      "Write(src/**)"
    ],
    "deny": [
      "Bash(curl:*)",
      "Bash(rm:*)"
    ]
  }
}
```

## CLI Quick Reference

```bash
# Start interactive session
claude

# Run one-shot command
claude -p "Explain this file" path/to/file.py

# Continue last session
claude --continue

# Resume specific session
claude --resume <session-id>

# List available MCP servers
claude mcp list
```
