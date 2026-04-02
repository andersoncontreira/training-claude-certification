# Domain 1: Agentic Architecture (27%)

The largest domain. Covers how to design, build, and orchestrate AI agents using Claude.

## Key Concepts

### 1. The Agent Loop
An agent operates in a continuous loop: **think → act → observe → repeat**.

```
User Input
    ↓
System Prompt + Context
    ↓
Claude reasons → selects tool
    ↓
Tool executes → returns result
    ↓
Claude observes result → reasons again
    ↓
Final response (or next tool call)
```

### 2. Tool Use
- Tools are functions the model can call during reasoning
- Defined via JSON schema: `name`, `description`, `input_schema`
- Claude decides *when* and *which* tool to call based on context
- Tool results are injected back into the conversation as `tool_result` blocks

### 3. Orchestration Patterns

| Pattern | Description | Use Case |
|---------|-------------|----------|
| **Single agent** | One Claude instance with multiple tools | Simple tasks |
| **Orchestrator + Subagents** | Main agent delegates to specialized agents | Complex, parallelizable tasks |
| **Pipeline** | Output of one agent feeds next | Sequential processing |
| **Supervisor** | Agent validates output of other agents | Quality control |

### 4. Multi-Agent Systems
- **Orchestrator**: plans and delegates work, synthesizes results
- **Subagent**: executes a specific scoped task, returns structured output
- Communication happens via tool calls or message passing
- Each subagent has its own system prompt and context

### 5. Human-in-the-Loop (HITL)
- Inject human approval at critical decision points
- Use `confirmation_required` pattern before irreversible actions
- Escalation: agent pauses and surfaces decision to user
- Important for: financial operations, data deletion, external API calls

### 6. Agent Reliability
- **Retry logic**: handle transient tool failures gracefully
- **Fallback tools**: alternative path when primary tool fails
- **Max iterations**: prevent infinite loops with a step budget
- **Output validation**: verify tool results before proceeding

## Design Principles
- Give agents the **minimum permissions** needed (least privilege)
- Write system prompts that define **scope and constraints** clearly
- Make tool descriptions self-explanatory — Claude relies heavily on them
- Prefer **deterministic tools** (same input → same output) for critical paths
- Log all tool calls for auditability

## Common Exam Topics
- Identifying the right orchestration pattern for a given scenario
- Designing tool schemas for specific use cases
- Knowing when to use HITL vs. fully automated flows
- Understanding context accumulation in long agent runs
- Tradeoffs between single-agent and multi-agent architectures
