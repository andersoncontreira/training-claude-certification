# Domain 5: Context & Reliability (15%)

Covers managing Claude's context window effectively and building reliable, predictable AI systems.

## The Context Window

Claude processes everything within a single "context window" — a finite number of tokens that includes the system prompt, conversation history, tool results, and any documents provided.

### Key Numbers (as of 2025)
- Claude models support up to **200K tokens** context
- 1 token ≈ 0.75 words (rough estimate)
- 200K tokens ≈ ~150,000 words ≈ ~500 pages of text

### What Consumes Context
```
System prompt          (fixed cost per conversation)
Conversation history   (grows over time)
Tool call + results    (can be large — e.g., file contents)
Documents              (user-provided data)
Claude's responses     (output tokens)
```

### Context Window Limits in Practice
- Long conversations will eventually exceed the limit
- Large tool results (e.g., reading a 10,000-line file) consume significant context
- Multiple tool calls accumulate quickly in an agent loop

## Memory Strategies

Since the context window is finite, production systems use external memory:

| Strategy | Description | Best For |
|----------|-------------|----------|
| **Conversation summary** | Periodically summarize old messages, replace with summary | Long ongoing conversations |
| **RAG** | Retrieve relevant documents at query time | Large knowledge bases |
| **External key-value store** | Persist structured data (facts, preferences) externally | User preferences, session state |
| **File-based memory** | Write notes to files, read them back when needed | Claude Code workflows |
| **Sliding window** | Keep only the last N turns of conversation | Simple chatbots |

### Implementing Conversation Summary

```python
def summarize_conversation(messages: list) -> str:
    """Condense old messages into a summary to free up context."""
    old_messages = messages[:-10]  # keep last 10 turns
    summary_prompt = f"Summarize this conversation history concisely:\n{old_messages}"

    response = client.messages.create(
        model="claude-haiku-4-5-20251001",  # cheaper model for summaries
        max_tokens=512,
        messages=[{"role": "user", "content": summary_prompt}]
    )
    return response.content[0].text

def trim_context(messages: list, max_turns: int = 20) -> list:
    if len(messages) <= max_turns:
        return messages

    summary = summarize_conversation(messages[:-max_turns])
    summary_message = {
        "role": "user",
        "content": f"[Previous conversation summary: {summary}]"
    }
    return [summary_message] + messages[-max_turns:]
```

## Reliability Patterns

### 1. Determinism
For tasks that must produce consistent results, use:
- `temperature=0`
- Fixed model version (avoid `claude-sonnet-latest` in production — pin to `claude-sonnet-4-6`)
- Consistent system prompt (version-control it)

### 2. Output Validation
Never trust Claude's output blindly in production:

```python
import json
from jsonschema import validate

def get_structured_output(prompt: str, schema: dict) -> dict:
    response = client.messages.create(...)
    try:
        data = json.loads(response.content[0].text)
        validate(instance=data, schema=schema)
        return data
    except (json.JSONDecodeError, ValidationError) as e:
        # Retry with more explicit instructions, or raise
        raise ValueError(f"Invalid output: {e}")
```

### 3. Retry with Exponential Backoff
For transient API failures:

```python
import time
import anthropic

def call_with_retry(fn, max_retries=3):
    for attempt in range(max_retries):
        try:
            return fn()
        except anthropic.RateLimitError:
            wait = 2 ** attempt
            time.sleep(wait)
        except anthropic.APIError as e:
            if e.status_code >= 500:
                time.sleep(2 ** attempt)
            else:
                raise  # don't retry 4xx errors
    raise RuntimeError("Max retries exceeded")
```

### 4. Fallbacks
Design systems with graceful degradation:

```python
def get_answer(question: str) -> str:
    try:
        # Primary: full model with tools
        return run_agent_with_tools(question)
    except Exception:
        try:
            # Fallback: simpler model, no tools
            return run_simple_completion(question)
        except Exception:
            # Last resort: static response
            return "I'm unable to process your request right now. Please try again later."
```

### 5. Step Budget
Prevent infinite agent loops:

```python
MAX_STEPS = 10

def run_agent(messages, step=0):
    if step >= MAX_STEPS:
        return "I reached the step limit without completing the task."

    response = client.messages.create(...)
    if response.stop_reason == "end_turn":
        return response.content[0].text

    # handle tool use...
    return run_agent(new_messages, step + 1)
```

## Common Exam Topics
- Identifying when a context window will be exceeded
- Choosing the right memory strategy for a scenario
- Implementing output validation for structured responses
- Understanding why pinning model versions matters in production
- Retry vs. fallback vs. fail-fast patterns
