# Exercise 03: Human-in-the-Loop

**Difficulty:** Advanced
**Domain:** Agentic Architecture
**Goal:** Build an agent that pauses for human approval before executing irreversible actions.

---

## Context

Some actions are irreversible: deleting files, sending emails, executing payments. A well-designed agentic system should surface these actions to the user before executing them, rather than proceeding autonomously.

This pattern is called **Human-in-the-Loop (HITL)**.

---

## Task

Build a **file management agent** that can:
- List files in a directory (safe — no approval needed)
- Read file contents (safe — no approval needed)
- **Delete a file** (requires human approval before execution)
- **Move a file** (requires human approval before execution)

The agent must ask for confirmation in the terminal before executing any destructive operation.

---

## Architecture

```
User request
      ↓
Agent decides to call delete_file(path)
      ↓
Before executing: print confirmation prompt to terminal
User types "yes" or "no"
      ↓
If yes: execute deletion, return result
If no: return cancellation message to agent
      ↓
Agent informs user of outcome
```

---

## Requirements

- Approval prompt must show: tool name, parameters, and a clear warning
- If user denies, the tool must return a `{"status": "cancelled", "reason": "User denied"}` result — the agent should handle this gracefully
- Safe tools (`list_files`, `read_file`) must NOT trigger approval
- The HITL mechanism must be in the tool implementation layer, not in the agent prompt

---

## Starter Code

```python
import anthropic
import os

client = anthropic.Anthropic()

# --- HITL gate ---

DESTRUCTIVE_TOOLS = {"delete_file", "move_file"}

def request_human_approval(tool_name: str, tool_input: dict) -> bool:
    """Prompt the user for approval before executing a destructive tool."""
    print("\n" + "="*50)
    print("APPROVAL REQUIRED")
    print(f"Tool: {tool_name}")
    print(f"Parameters: {tool_input}")
    print("This action may be irreversible.")
    print("="*50)
    answer = input("Approve? (yes/no): ").strip().lower()
    return answer == "yes"

# --- Tool implementations ---

def list_files(directory: str) -> dict:
    try:
        files = os.listdir(directory)
        return {"files": files}
    except FileNotFoundError:
        return {"error": f"Directory not found: {directory}"}

def read_file(path: str) -> dict:
    try:
        with open(path) as f:
            return {"content": f.read()}
    except FileNotFoundError:
        return {"error": f"File not found: {path}"}

def delete_file(path: str) -> dict:
    # TODO: implement deletion (already gated by HITL dispatcher)
    pass

def move_file(source: str, destination: str) -> dict:
    # TODO: implement move (already gated by HITL dispatcher)
    pass

# --- Tool dispatcher with HITL gate ---

TOOL_REGISTRY = {
    "list_files": list_files,
    "read_file": read_file,
    "delete_file": delete_file,
    "move_file": move_file,
}

def dispatch_tool(tool_name: str, tool_input: dict) -> dict:
    """Execute a tool, requesting human approval if it's destructive."""
    if tool_name in DESTRUCTIVE_TOOLS:
        approved = request_human_approval(tool_name, tool_input)
        if not approved:
            return {"status": "cancelled", "reason": "User denied the operation."}

    fn = TOOL_REGISTRY.get(tool_name)
    if fn is None:
        return {"error": f"Unknown tool: {tool_name}"}
    return fn(**tool_input)

# --- Tool schemas ---

tools = [
    # TODO: Define schemas for list_files, read_file, delete_file, move_file
]

# --- Agent ---

def file_manager_agent(user_request: str) -> str:
    system = """You are a file management assistant. Help users manage their files.
Always list files before attempting to delete or move them.
If an operation is cancelled by the user, acknowledge it and ask what else you can help with."""

    messages = [{"role": "user", "content": user_request}]

    # TODO: Implement the agent loop using dispatch_tool
    pass

if __name__ == "__main__":
    # Test with a safe operation first
    answer = file_manager_agent(
        "List the files in /tmp, then delete any file ending in .log"
    )
    print(answer)
```

---

## Test Scenarios

1. Ask the agent to list files in a directory — no approval prompt should appear
2. Ask the agent to delete a specific file — approval prompt must appear
3. Deny the approval — agent should handle cancellation gracefully
4. Approve — agent should confirm the deletion

---

## Extension Challenges

1. Implement a **timeout** on the approval prompt: if the user doesn't respond in 30 seconds, auto-deny.
2. Add an **audit log** that records all approval decisions (approved/denied) with timestamps to a file.
3. Implement a **"dry run" mode**: agent explains what it *would* do without actually doing it, then asks for a single "execute all" confirmation.

---

## Key Concepts Practiced

- Separating HITL logic from agent prompting
- Tool dispatcher pattern
- Graceful handling of cancelled operations
- Designing agent system prompts for resilience
