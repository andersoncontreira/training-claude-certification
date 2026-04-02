# Domain 1: Resources & References

## Official Anthropic Documentation

- [Build with Claude — Agents overview](https://docs.anthropic.com/en/docs/build-with-claude/agents)
- [Tool use (function calling)](https://docs.anthropic.com/en/docs/build-with-claude/tool-use)
- [Computer use](https://docs.anthropic.com/en/docs/build-with-claude/computer-use)
- [Claude API Reference — Messages](https://docs.anthropic.com/en/api/messages)

## Anthropic Research & Blog

- [Building effective agents](https://www.anthropic.com/research/building-effective-agents)
- [Claude's extended thinking](https://docs.anthropic.com/en/docs/build-with-claude/extended-thinking)

## SDK References

- [Anthropic Python SDK](https://github.com/anthropics/anthropic-sdk-python)
- [Anthropic TypeScript SDK](https://github.com/anthropics/anthropic-sdk-typescript)

## Key API Concepts

### Tool definition schema
```json
{
  "name": "tool_name",
  "description": "What the tool does — be explicit",
  "input_schema": {
    "type": "object",
    "properties": {
      "param": { "type": "string", "description": "..." }
    },
    "required": ["param"]
  }
}
```

### Message roles in agent loop
- `user` — human or tool_result input
- `assistant` — Claude's response (may include tool_use blocks)
- `tool_result` — result of a tool call, sent as user message

## Additional Reading

- [ReAct: Synergizing Reasoning and Acting in Language Models](https://arxiv.org/abs/2210.03629) — foundational paper for agent loops
- [Toolformer paper](https://arxiv.org/abs/2302.04761) — background on tool-augmented LLMs
