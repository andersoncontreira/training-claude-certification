# Exercise 02: Multi-Agent Orchestration

**Difficulty:** Intermediate
**Domain:** Agentic Architecture
**Goal:** Build an orchestrator that delegates tasks to two specialized subagents.

---

## Context

Complex tasks often benefit from specialization. An orchestrator agent breaks down the work and delegates subtasks to agents that are focused on a specific domain, then synthesizes their results.

---

## Task

Build a **research pipeline** with three agents:

1. **Orchestrator** — receives the user's question, decides what research is needed, delegates to subagents, and writes the final answer
2. **FactFinder subagent** — given a topic, returns a list of key facts (mocked)
3. **Summarizer subagent** — given raw facts, returns a concise paragraph

---

## Architecture

```
User Question
      ↓
  Orchestrator
  ├── calls tool: delegate_to_fact_finder(topic)
  │       ↓ FactFinder agent runs → returns facts
  ├── calls tool: delegate_to_summarizer(facts)
  │       ↓ Summarizer agent runs → returns summary
  └── Final answer to user
```

---

## Requirements

- Each "subagent" is a separate `client.messages.create()` call with its own system prompt
- The orchestrator invokes subagents via tool calls (the tool implementation calls the subagent)
- Subagents must not share context with each other — only the orchestrator sees everything
- The orchestrator's final response must incorporate both subagent outputs

---

## Starter Code

```python
import anthropic

client = anthropic.Anthropic()

# --- Subagent implementations ---

def fact_finder_agent(topic: str) -> str:
    """Subagent specialized in finding facts about a topic."""
    response = client.messages.create(
        model="claude-haiku-4-5-20251001",
        max_tokens=512,
        system="You are a fact-finding assistant. Given a topic, return exactly 5 key facts as a numbered list. Be concise.",
        messages=[{"role": "user", "content": f"Find key facts about: {topic}"}]
    )
    return response.content[0].text

def summarizer_agent(facts: str) -> str:
    """Subagent specialized in summarizing facts into prose."""
    response = client.messages.create(
        model="claude-haiku-4-5-20251001",
        max_tokens=256,
        system="You are a summarization assistant. Given a list of facts, write a single coherent paragraph.",
        messages=[{"role": "user", "content": f"Summarize these facts:\n{facts}"}]
    )
    return response.content[0].text

# --- Tool implementations for orchestrator ---

def delegate_to_fact_finder(topic: str) -> dict:
    result = fact_finder_agent(topic)
    return {"facts": result}

def delegate_to_summarizer(facts: str) -> dict:
    result = summarizer_agent(facts)
    return {"summary": result}

# --- Orchestrator tools schema ---

orchestrator_tools = [
    # TODO: Define tool schema for delegate_to_fact_finder
    # TODO: Define tool schema for delegate_to_summarizer
]

# --- Orchestrator agent loop ---

def orchestrator(user_question: str) -> str:
    system = """You are a research orchestrator. To answer questions:
1. First use delegate_to_fact_finder to gather facts on the topic
2. Then use delegate_to_summarizer to turn the facts into a summary
3. Finally, provide your answer based on the summary."""

    messages = [{"role": "user", "content": user_question}]

    # TODO: Implement the orchestrator agent loop
    # Same pattern as Exercise 01, but dispatch to the correct subagent function

    pass

if __name__ == "__main__":
    answer = orchestrator("Tell me about the James Webb Space Telescope.")
    print(answer)
```

---

## Expected Behavior

1. Orchestrator calls `delegate_to_fact_finder("James Webb Space Telescope")`
2. FactFinder subagent returns 5 facts
3. Orchestrator calls `delegate_to_summarizer(facts)`
4. Summarizer returns a paragraph
5. Orchestrator returns final answer citing the summary

---

## Extension Challenges

1. Run both subagents **in parallel** using `concurrent.futures.ThreadPoolExecutor` and see if you can modify the orchestrator tool to accept multiple topics at once.
2. Add a **validator subagent** that checks if the summary is factually consistent with the facts — if not, the orchestrator should retry.
3. Add timing logs to compare latency with sequential vs. parallel subagent calls.

---

## Key Concepts Practiced

- Orchestrator/subagent architecture
- Isolating subagent context (each call is independent)
- Tool implementation calling external agents
- Synthesizing multiple subagent outputs
