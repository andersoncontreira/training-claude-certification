# Exercise 01: First MCP Server

**Difficulty:** Beginner
**Domain:** MCP & Tool Design
**Goal:** Build and run a minimal MCP server with two tools using the Python SDK.

---

## Context

An MCP server exposes tools that Claude can call. This exercise walks you through the complete lifecycle: writing a server, running it, and connecting it to Claude.

---

## Task

Create an MCP server called `calculator` that exposes two tools:

1. `add(a: float, b: float) -> float` — adds two numbers
2. `multiply(a: float, b: float) -> float` — multiplies two numbers

---

## Requirements

- Use `mcp` Python SDK (`pip install mcp`)
- Server must run via stdio transport
- Both tools must have clear descriptions and proper input schemas
- Tools must return results as `TextContent`

---

## Starter Code

Create a file `calculator_server.py`:

```python
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp import types
import asyncio

app = Server("calculator")

@app.list_tools()
async def list_tools() -> list[types.Tool]:
    return [
        types.Tool(
            name="add",
            description="TODO: write a clear description",
            inputSchema={
                # TODO: define the input schema for (a: float, b: float)
            }
        ),
        types.Tool(
            name="multiply",
            description="TODO: write a clear description",
            inputSchema={
                # TODO: define the input schema for (a: float, b: float)
            }
        ),
    ]

@app.call_tool()
async def call_tool(name: str, arguments: dict) -> list[types.TextContent]:
    # TODO: implement add and multiply
    # Return: [types.TextContent(type="text", text=str(result))]
    raise ValueError(f"Unknown tool: {name}")

async def main():
    async with stdio_server() as (read, write):
        await app.run(read, write, app.create_initialization_options())

if __name__ == "__main__":
    asyncio.run(main())
```

---

## Testing Your Server

### Option A: Use MCP Inspector
```bash
npx @modelcontextprotocol/inspector python calculator_server.py
```
Open the inspector UI, list tools, and manually call `add` with `{"a": 3, "b": 4}`.

### Option B: Connect to Claude Desktop
Add to your `claude_desktop_config.json`:
```json
{
  "mcpServers": {
    "calculator": {
      "command": "python",
      "args": ["path/to/calculator_server.py"]
    }
  }
}
```
Restart Claude Desktop and ask: "What is 137 plus 489?"

---

## Expected Behavior

When asked "What is 3 multiplied by 7?", Claude should:
1. Call `multiply` with `{"a": 3, "b": 7}`
2. Receive `"21"`
3. Respond: "3 multiplied by 7 is 21."

---

## Extension Challenges

1. Add a `divide(a, b)` tool — handle division by zero gracefully and return an error message.
2. Add a `history` resource that returns the last 10 calculations performed.
3. Implement input validation: reject non-numeric inputs with a descriptive error.

---

## Key Concepts Practiced

- MCP server structure (`list_tools`, `call_tool` handlers)
- Tool input schema with JSON Schema
- stdio transport
- TextContent response format
