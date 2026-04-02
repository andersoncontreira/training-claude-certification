# Exercise 02: Chain-of-Thought Techniques

**Difficulty:** Intermediate
**Domain:** Prompt Engineering
**Goal:** Apply CoT techniques to improve accuracy on reasoning-heavy tasks.

---

## Context

Chain-of-thought (CoT) prompting asks Claude to reason through a problem step-by-step before giving a final answer. This consistently improves accuracy on tasks that require multi-step logic, analysis, or evaluation.

---

## Part A: Compare Direct vs. CoT Responses

For each task below, make two API calls: one direct, one with CoT. Compare the quality.

### Task 1: Security Analysis

**Direct prompt:**
```
Is this code safe?

def login(username, password):
    query = f"SELECT * FROM users WHERE username='{username}' AND password='{password}'"
    return db.execute(query)
```

**CoT prompt (you write this):**
```
TODO: Rewrite the prompt to instruct Claude to:
1. Identify all user-controlled inputs
2. Trace each input through the code
3. Evaluate each usage for security risks
4. Conclude with a safety verdict and severity rating
```

Write both prompts and compare the outputs. Which is more useful and why?

---

### Task 2: Business Decision

**Direct prompt:**
```
Should we migrate our database from PostgreSQL to MongoDB?
```

**CoT prompt:**
```
TODO: Rewrite to guide Claude through a structured analysis:
- Current situation assessment
- Pros and cons of migration
- Risk evaluation
- Final recommendation with confidence level
```

---

## Part B: Scratchpad Pattern

The scratchpad pattern asks Claude to reason in `<thinking>` tags before giving the final answer. This keeps the response clean while preserving the reasoning benefit.

```python
import anthropic

client = anthropic.Anthropic()

def analyze_with_scratchpad(code_snippet: str) -> dict:
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=2048,
        messages=[{
            "role": "user",
            "content": f"""Analyze this code for bugs.

<code>
{code_snippet}
</code>

First, reason through the code carefully in <thinking> tags.
Then provide your final analysis outside the thinking tags.

Format your final analysis as:
- Bugs found: (list)
- Severity: (high/medium/low)
- Recommended fix: (brief description)"""
        }]
    )

    text = response.content[0].text

    # TODO: Parse the response to separate thinking from final answer
    # Hint: look for <thinking>...</thinking> in the text

    return {
        "thinking": "TODO: extract thinking section",
        "analysis": "TODO: extract final analysis"
    }

# Test with buggy code
buggy_code = """
def divide(a, b):
    return a / b

result = divide(10, 0)
print(result)
"""

result = analyze_with_scratchpad(buggy_code)
print("Reasoning:", result["thinking"])
print("\nAnalysis:", result["analysis"])
```

---

## Part C: Extended Thinking API

For the most complex reasoning tasks, use the `thinking` parameter:

```python
import anthropic

client = anthropic.Anthropic()

def complex_analysis(problem: str) -> dict:
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=16000,
        thinking={
            "type": "enabled",
            "budget_tokens": 8000
        },
        messages=[{"role": "user", "content": problem}]
    )

    thinking_text = ""
    answer_text = ""

    for block in response.content:
        if block.type == "thinking":
            thinking_text = block.thinking
        elif block.type == "text":
            answer_text = block.text

    return {"thinking": thinking_text, "answer": answer_text}

# TODO: Test with a complex architecture decision problem
problem = """
We have a Python microservice that processes 10,000 events/second.
Currently it's single-threaded and we're hitting CPU limits.
We're considering: async/await, multiprocessing, or switching to Go.
Each option has tradeoffs around developer experience, performance, and maintenance.
What should we do?
"""

result = complex_analysis(problem)
print(f"Tokens used for thinking: {len(result['thinking'].split())}")
print("\nAnswer:", result["answer"])
```

---

## Part D: When NOT to Use CoT

Not every task benefits from CoT. Complete this table based on your observations:

| Task | Use CoT? | Reason |
|------|----------|--------|
| Classify an email into 5 categories | | |
| Determine if a sentence is grammatically correct | | |
| Design a database schema for a new product | | |
| Extract all email addresses from a document | | |
| Evaluate whether a business plan is viable | | |
| Convert temperature from Celsius to Fahrenheit | | |

---

## Key Concepts Practiced

- Zero-shot CoT ("think step by step")
- Explicit CoT (structured reasoning steps in the prompt)
- Scratchpad pattern (`<thinking>` tags)
- Extended thinking API parameter
- Recognizing when CoT adds value vs. adds latency without benefit
