# Exercise 01: Context Window Management

**Difficulty:** Beginner
**Domain:** Context & Reliability
**Goal:** Measure context consumption and implement a strategy to keep conversations within limits.

---

## Context

The context window has a finite size. In long agent runs or extended conversations, you'll exhaust it if you don't actively manage what stays in the context.

---

## Part A: Measure Token Usage

Use the token counting API to understand how context accumulates.

```python
import anthropic

client = anthropic.Anthropic()

system_prompt = """You are a helpful coding assistant.
You have access to the user's codebase and help them write, debug, and refactor Python code."""

def count_tokens(messages: list) -> int:
    """Count tokens for a given message list + system prompt."""
    response = client.messages.count_tokens(
        model="claude-sonnet-4-6",
        system=system_prompt,
        messages=messages
    )
    return response.input_tokens

# Simulate a growing conversation
messages = []
token_counts = []

conversation_turns = [
    ("user", "I'm working on a FastAPI application. Can you help me?"),
    ("assistant", "Of course! I'd be happy to help with your FastAPI application. What do you need?"),
    ("user", "I need to add authentication. Should I use JWT or sessions?"),
    ("assistant", "For FastAPI, JWT is typically the better choice for APIs..."),
    ("user", "Can you show me a full JWT implementation with refresh tokens?"),
    ("assistant", "Here's a complete JWT implementation: [imagine 500 lines of code here]"),
    ("user", "Now add rate limiting to each endpoint"),
    ("assistant", "Here's how to add rate limiting: [more code]"),
]

for role, content in conversation_turns:
    messages.append({"role": role, "content": content})
    count = count_tokens(messages)
    token_counts.append(count)
    print(f"After turn {len(messages):2d} ({role}): {count:,} tokens")

print(f"\nTotal growth: {token_counts[0]:,} → {token_counts[-1]:,} tokens")
print(f"Growth factor: {token_counts[-1] / token_counts[0]:.1f}x")
```

**Questions to answer:**
1. How fast does the context grow with code-heavy responses?
2. At this rate, after how many turns would you hit 200K tokens?

---

## Part B: Sliding Window Strategy

Implement a sliding window that keeps only the last N turns:

```python
def sliding_window(messages: list, max_turns: int = 10) -> list:
    """Keep only the last max_turns messages."""
    # TODO: implement
    # Important: always keep pairs (user + assistant) together
    # If max_turns is odd, round down to even to keep complete pairs
    pass

# Test: verify the window works correctly
long_history = [
    {"role": "user", "content": f"Question {i}"}
    if i % 2 == 0
    else {"role": "assistant", "content": f"Answer {i}"}
    for i in range(1, 21)
]

windowed = sliding_window(long_history, max_turns=6)
assert len(windowed) == 6, f"Expected 6, got {len(windowed)}"
assert windowed[0]["role"] == "user", "Should start with user message"
print("Sliding window works correctly")
```

---

## Part C: Summarization Strategy

Implement a smarter approach: summarize old turns instead of discarding them.

```python
def summarize_old_turns(messages: list, keep_recent: int = 8) -> list:
    """
    Summarize the older part of the conversation and return a trimmed messages list.

    Strategy:
    - Keep the most recent `keep_recent` messages as-is
    - Summarize everything before that into a single system-injected message
    """
    if len(messages) <= keep_recent:
        return messages

    old_messages = messages[:-keep_recent]
    recent_messages = messages[-keep_recent:]

    # Build a prompt to summarize the old conversation
    history_text = "\n".join([
        f"{m['role'].upper()}: {m['content'][:200]}..."  # truncate for the summary prompt
        for m in old_messages
    ])

    summary_response = client.messages.create(
        model="claude-haiku-4-5-20251001",  # cheaper model for summarization
        max_tokens=256,
        messages=[{
            "role": "user",
            "content": f"Summarize this conversation history in 3-5 bullet points, focusing on key decisions and context:\n\n{history_text}"
        }]
    )

    summary = summary_response.content[0].text

    # Inject summary as a user message at the start
    summary_message = {
        "role": "user",
        "content": f"[Conversation summary from earlier: {summary}]"
    }

    # TODO: return the trimmed messages list with the summary prepended
    pass

# Test
trimmed = summarize_old_turns(long_history, keep_recent=6)
tokens_before = count_tokens(long_history)
tokens_after = count_tokens(trimmed)
print(f"Tokens before: {tokens_before:,}")
print(f"Tokens after:  {tokens_after:,}")
print(f"Reduction:     {(1 - tokens_after/tokens_before)*100:.0f}%")
```

---

## Part D: Choose the Right Strategy

For each scenario, recommend a context management strategy and explain why:

| Scenario | Recommended Strategy | Reason |
|----------|---------------------|--------|
| Customer support bot — each conversation is independent | | |
| Long-running coding assistant — user works on same codebase for hours | | |
| Document Q&A — 1000-page manual | | |
| Agent loop processing 100 files one by one | | |
| Legal document review requiring full document comprehension | | |

---

## Key Concepts Practiced

- Token counting API
- Context growth patterns
- Sliding window vs. summarization tradeoffs
- Choosing strategies based on use case requirements
