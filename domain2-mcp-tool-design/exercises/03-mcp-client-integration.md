# Exercise 03: MCP Client Integration

**Difficulty:** Advanced
**Domain:** MCP & Tool Design
**Goal:** Build a Python client that connects to an MCP server and uses its tools in a Claude agent loop.

---

## Context

In production, Claude Code and Claude Desktop act as MCP clients. But you can also build your own MCP client to programmatically connect to servers and use their tools in custom agentic workflows.

---

## Task

Build a system with:
1. An MCP server that exposes a `search_notes(query)` tool and a `create_note(title, content)` tool
2. A client that:
   - Connects to the server via stdio
   - Discovers available tools
   - Passes those tools to a Claude agent
   - Runs the agent loop, routing tool calls to the MCP server

---

## Part 1: The MCP Server

Create `notes_server.py`:

```python
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp import types
import asyncio
import json

app = Server("notes")

# In-memory store
notes = {
    "meeting-2024-01": {"title": "Q1 Planning", "content": "Discussed roadmap for Q1..."},
    "ideas-001": {"title": "Product Ideas", "content": "1. Dark mode 2. Export to PDF..."},
}

@app.list_tools()
async def list_tools() -> list[types.Tool]:
    return [
        types.Tool(
            name="search_notes",
            description="Search notes by keyword. Returns matching note titles and IDs. Use when user wants to find or recall a note.",
            inputSchema={
                "type": "object",
                "properties": {
                    "query": {"type": "string", "description": "Search keyword"}
                },
                "required": ["query"]
            }
        ),
        types.Tool(
            name="create_note",
            description="Create a new note with a title and content. Returns the ID of the created note.",
            inputSchema={
                "type": "object",
                "properties": {
                    "title": {"type": "string"},
                    "content": {"type": "string"}
                },
                "required": ["title", "content"]
            }
        ),
    ]

@app.call_tool()
async def call_tool(name: str, arguments: dict) -> list[types.TextContent]:
    if name == "search_notes":
        query = arguments["query"].lower()
        results = [
            {"id": k, "title": v["title"]}
            for k, v in notes.items()
            if query in v["title"].lower() or query in v["content"].lower()
        ]
        return [types.TextContent(type="text", text=json.dumps(results))]

    if name == "create_note":
        import uuid
        note_id = str(uuid.uuid4())[:8]
        notes[note_id] = {"title": arguments["title"], "content": arguments["content"]}
        return [types.TextContent(type="text", text=json.dumps({"id": note_id, "status": "created"}))]

    raise ValueError(f"Unknown tool: {name}")

async def main():
    async with stdio_server() as (read, write):
        await app.run(read, write, app.create_initialization_options())

if __name__ == "__main__":
    asyncio.run(main())
```

---

## Part 2: The MCP Client + Agent

Create `notes_client.py`:

```python
import asyncio
import anthropic
import json
from mcp import ClientSession
from mcp.client.stdio import stdio_client

client = anthropic.Anthropic()

async def run_agent_with_mcp(user_message: str, server_script: str):
    """Connect to MCP server, discover tools, run agent loop."""

    # Start MCP server as subprocess and connect
    server_params = {
        "command": "python",
        "args": [server_script]
    }

    async with stdio_client(server_params) as (read, write):
        async with ClientSession(read, write) as session:
            # Initialize the MCP session
            await session.initialize()

            # TODO: List tools from the MCP server
            tools_result = await session.list_tools()

            # TODO: Convert MCP tool definitions to Anthropic tool format
            # MCP tools have: name, description, inputSchema
            # Anthropic tools have: name, description, input_schema
            anthropic_tools = []  # convert tools_result.tools here

            # TODO: Implement the agent loop
            # When Claude calls a tool, use session.call_tool(name, arguments)
            # Convert the MCP result back to a tool_result message

            messages = [{"role": "user", "content": user_message}]

            while True:
                response = client.messages.create(
                    model="claude-sonnet-4-6",
                    max_tokens=1024,
                    tools=anthropic_tools,
                    messages=messages
                )

                # TODO: handle tool_use and end_turn
                pass

async def main():
    answer = await run_agent_with_mcp(
        "Search my notes for anything about the roadmap",
        "notes_server.py"
    )
    print(answer)

if __name__ == "__main__":
    asyncio.run(main())
```

---

## Key Conversion Reference

```python
# MCP tool → Anthropic tool format
def mcp_to_anthropic_tool(mcp_tool) -> dict:
    return {
        "name": mcp_tool.name,
        "description": mcp_tool.description,
        "input_schema": mcp_tool.inputSchema  # note: camelCase → snake_case
    }

# Call MCP tool and get result
result = await session.call_tool(tool_name, tool_arguments)
text_result = result.content[0].text  # TextContent
```

---

## Expected Output

```
I found 1 note matching "roadmap":
- "Q1 Planning" (ID: meeting-2024-01)

This note discusses the Q1 roadmap planning session.
```

---

## Extension Challenges

1. Add a `get_note(id)` tool to the server and update the client to support it.
2. Implement **tool caching**: after the first `list_tools` call, cache the result and reuse it across agent invocations without reconnecting.
3. Add **error handling**: if the MCP server crashes mid-session, the agent should catch the error and return a graceful failure message.

---

## Key Concepts Practiced

- MCP client session lifecycle (`initialize`, `list_tools`, `call_tool`)
- Converting between MCP and Anthropic tool formats
- Embedding MCP tool calls inside a Claude agent loop
- Async Python with MCP SDK
