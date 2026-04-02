# Exercise 01: System Prompt Design

**Difficulty:** Beginner
**Domain:** Prompt Engineering
**Goal:** Write, test, and refine effective system prompts for different use cases.

---

## Context

The system prompt is the single most impactful element of a Claude deployment. A well-structured system prompt results in consistent, on-brand, accurate responses. A poorly written one leads to inconsistency, hallucination, and off-task behavior.

---

## Part A: Diagnose the Bad System Prompt

The following system prompt is poor. Identify all its problems and rewrite it.

**Original:**
```
You are a helpful assistant. Answer questions about our product.
Be nice and professional. Don't say anything bad.
```

**Problems to identify:**
1. `TODO`
2. `TODO`
3. `TODO`
4. `TODO`

**Context for rewrite:** This is for a SaaS product called "DataFlow" — a data pipeline tool for engineers. Users are typically senior developers troubleshooting integration issues or asking for best practices.

**Your improved version:**
```xml
<system>
TODO: Write a complete system prompt with:
- Role definition
- Context about DataFlow
- Task scope
- Output format preferences
- Constraints (what NOT to do)
</system>
```

---

## Part B: System Prompt for a Specific Task

Write a system prompt for each of the following use cases. Then test each with 3 sample inputs.

### Use Case 1: Code Review Bot

Constraints:
- Only review Python code
- Focus on: correctness, security, and readability
- Do not rewrite code unless explicitly asked
- Return structured output: Summary, Issues (with severity), Suggestions

```xml
<system>
TODO
</system>
```

**Test inputs:**
1. A function with a SQL injection vulnerability
2. A function that works correctly but has poor naming
3. A JavaScript snippet (should be rejected gracefully)

---

### Use Case 2: Customer Support Triage

Constraints:
- Classify tickets into: `billing`, `technical`, `account`, `other`
- Assign priority: `urgent`, `normal`, `low`
- Never provide refunds or account changes directly — escalate to human
- Always be empathetic in tone

```xml
<system>
TODO
</system>
```

**Test inputs:**
1. "My service is completely down and I'm losing money!"
2. "How do I update my billing address?"
3. "Can you explain how your API rate limits work?"

---

## Part C: Output Format Specification

One of the most common failures in prompt engineering is under-specifying the output format.

For each scenario, write the output format specification section of a system prompt:

**Scenario 1:** You want Claude to return a risk assessment as JSON with keys: `risk_level` (low/medium/high), `factors` (list of strings), `recommendation` (string).

```
TODO: Write the output format instruction
```

**Scenario 2:** You want Claude to return a structured report in Markdown with exactly these sections: Executive Summary, Findings, Recommendations.

```
TODO: Write the output format instruction
```

---

## Part D: Test and Iterate

For your system prompt from Part B, Use Case 1 (Code Review Bot):

1. Test with the 3 inputs above
2. For each output, score it 1–5 on: accuracy, format compliance, usefulness
3. Identify the lowest-scoring aspect
4. Modify the system prompt to improve that aspect
5. Retest

Document your iterations:

| Version | Change made | Avg score before | Avg score after |
|---------|------------|-----------------|----------------|
| v1 | Initial | - | `?` |
| v2 | `TODO` | `?` | `?` |

---

## Key Concepts Practiced

- System prompt structure (role, context, task, format, constraints)
- Output format specification
- Iterative prompt refinement
- Handling out-of-scope requests gracefully
