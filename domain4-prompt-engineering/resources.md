# Domain 4: Resources & References

## Official Anthropic Documentation

- [Prompt engineering overview](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview)
- [System prompts](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/system-prompts)
- [Use XML tags](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/use-xml-tags)
- [Chain of thought](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/chain-of-thought)
- [Extended thinking](https://docs.anthropic.com/en/docs/build-with-claude/extended-thinking)
- [Reduce hallucinations](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/reduce-hallucinations)
- [Prompt library](https://docs.anthropic.com/en/prompt-library)

## API Parameters Reference

```python
client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    temperature=0,        # 0 = deterministic
    top_p=1,              # nucleus sampling
    top_k=0,              # vocabulary limit (0 = disabled)
    system="...",         # system prompt
    messages=[...]
)
```

## Structured Output with Tool Use

```python
import anthropic, json

client = anthropic.Anthropic()

extraction_tool = {
    "name": "extract_entities",
    "description": "Extract structured entities from text",
    "input_schema": {
        "type": "object",
        "properties": {
            "people": {"type": "array", "items": {"type": "string"}},
            "organizations": {"type": "array", "items": {"type": "string"}},
            "locations": {"type": "array", "items": {"type": "string"}}
        },
        "required": ["people", "organizations", "locations"]
    }
}

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=512,
    tools=[extraction_tool],
    tool_choice={"type": "tool", "name": "extract_entities"},  # force tool use
    messages=[{
        "role": "user",
        "content": "Elon Musk visited Google headquarters in Mountain View."
    }]
)

result = json.loads(response.content[0].input)
```

## Extended Thinking

```python
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=16000,
    thinking={
        "type": "enabled",
        "budget_tokens": 10000  # tokens available for thinking
    },
    messages=[{"role": "user", "content": "Solve this step by step: ..."}]
)

# Response has thinking blocks + text blocks
for block in response.content:
    if block.type == "thinking":
        print("Reasoning:", block.thinking)
    elif block.type == "text":
        print("Answer:", block.text)
```

## Prompt Evaluation Checklist

Before using a prompt in production, verify:
- [ ] System prompt defines role, context, task, format, constraints
- [ ] User-provided content is wrapped in XML tags
- [ ] Output format is explicitly specified
- [ ] Temperature is appropriate for the task (0 for extraction, higher for creative)
- [ ] Few-shot examples are included if output format is non-trivial
- [ ] Prompt injection is mitigated if processing untrusted input
