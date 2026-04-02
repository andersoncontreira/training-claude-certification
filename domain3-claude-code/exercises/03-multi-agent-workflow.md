# Exercise 03: Multi-Agent Workflow in Claude Code

**Difficulty:** Advanced
**Domain:** Claude Code
**Goal:** Design a multi-agent workflow using Claude Code's Agent tool for parallel code analysis.

---

## Context

Claude Code can spawn subagents (other Claude instances) using the `Agent` tool. Each subagent runs independently with its own context, enabling parallelization of tasks that don't depend on each other.

---

## Task

Create a CLAUDE.md-based workflow that instructs Claude Code to perform a **parallel code audit** of a Python project by spawning specialized subagents.

---

## The Workflow

When the user runs `/audit`, Claude Code should:

1. Spawn 3 subagents **in parallel**:
   - **Security agent**: looks for hardcoded secrets, SQL injection, unsafe deserialization
   - **Quality agent**: identifies long functions, missing docstrings, high complexity
   - **Dependencies agent**: reads `requirements.txt` and checks for outdated or vulnerable packages
2. Collect results from all 3 subagents
3. Synthesize a final audit report

---

## Part 1: The Slash Command

Create `.claude/commands/audit.md`:

```markdown
Perform a parallel code audit of this project. Follow these steps exactly:

1. First, identify all Python files in the project using the Glob tool.

2. Launch three subagents IN PARALLEL (in a single message with multiple Agent tool calls):
   - Security subagent: analyze all Python files for security issues (hardcoded credentials, SQL injection risks, use of eval/exec, unsafe deserialization). Return findings as a JSON list.
   - Quality subagent: analyze all Python files for code quality issues (functions over 50 lines, missing docstrings, duplicated code). Return findings as a JSON list.
   - Dependencies subagent: read requirements.txt (if present) and list all dependencies. Flag any that are known to have security advisories or appear outdated.

3. Wait for all three subagents to complete.

4. Synthesize a final report with sections:
   - Executive Summary (2-3 sentences)
   - Security Findings
   - Quality Findings
   - Dependency Notes
   - Recommended Actions (prioritized)
```

---

## Part 2: Test Project

Create a small test project with intentional issues for the audit to find:

```python
# test_project/app.py

import pickle
import sqlite3

# Security issue: hardcoded credential
API_KEY = "sk-prod-1234567890abcdef"

def get_user(user_id):
    # Security issue: SQL injection
    conn = sqlite3.connect("users.db")
    query = "SELECT * FROM users WHERE id = " + str(user_id)
    return conn.execute(query).fetchone()

def process_data(raw_bytes):
    # Security issue: unsafe deserialization
    return pickle.loads(raw_bytes)

def very_long_function(data):
    # Quality issue: no docstring, will be long
    result = []
    for item in data:
        if item > 0:
            result.append(item * 2)
    # ... imagine 50 more lines here
    return result
```

```
# test_project/requirements.txt
flask==0.12.0
requests==2.18.0
django==2.0.0
```

---

## Part 3: Run and Evaluate

Run `/audit` and check:

- [ ] Did Claude spawn all 3 subagents in a single parallel dispatch?
- [ ] Did the Security agent find the hardcoded key, SQL injection, and pickle issue?
- [ ] Did the Quality agent flag the missing docstring?
- [ ] Did the Dependencies agent flag the outdated Flask/Django versions?
- [ ] Is the final report well-structured and actionable?

---

## Part 4: CLAUDE.md Enhancement

Add instructions to your project's `CLAUDE.md` to make the audit workflow work better:

```markdown
## Code Audit Guidelines

When performing security analysis:
- Flag any string that matches patterns: sk-*, password=, api_key=, secret=
- Check all SQL queries for string concatenation
- Flag use of: eval, exec, pickle.loads, yaml.load (without Loader)

When performing quality analysis:
- Flag functions over 40 lines
- Flag files without module docstrings
- Flag functions without docstrings if they are public (not starting with _)
```

---

## Extension Challenges

1. Add a fourth subagent that checks test coverage by looking for test files and comparing them to source files.
2. Modify the workflow so that if the security agent finds a critical issue (hardcoded secret), it immediately surfaces that to the user without waiting for the other agents.
3. Have the orchestrator save the full report to `audit-report.md` after synthesis.

---

## Key Concepts Practiced

- Parallel agent dispatch (multiple Agent tool calls in one message)
- Subagent context isolation
- Orchestrator synthesis pattern
- CLAUDE.md for persistent workflow instructions
- Slash commands for multi-step workflows
