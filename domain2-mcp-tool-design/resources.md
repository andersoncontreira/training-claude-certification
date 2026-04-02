# Domain 2: Resources & References

## Official MCP Documentation

- [Model Context Protocol — official site](https://modelcontextprotocol.io)
- [MCP specification](https://spec.modelcontextprotocol.io)
- [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk)
- [MCP TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk)

## Anthropic Documentation

- [MCP overview in Claude docs](https://docs.anthropic.com/en/docs/build-with-claude/mcp)
- [Tool use reference](https://docs.anthropic.com/en/docs/build-with-claude/tool-use)

## Example MCP Servers

- [MCP servers repository](https://github.com/modelcontextprotocol/servers) — official reference implementations
- Notable examples: filesystem, git, GitHub, Postgres, Slack

## Minimal MCP Server (Python)

```python
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp import types

app = Server("my-server")

@app.list_tools()
async def list_tools() -> list[types.Tool]:
    return [
        types.Tool(
            name="hello",
            description="Returns a greeting for the given name.",
            inputSchema={
                "type": "object",
                "properties": {
                    "name": {"type": "string", "description": "Name to greet"}
                },
                "required": ["name"]
            }
        )
    ]

@app.call_tool()
async def call_tool(name: str, arguments: dict) -> list[types.TextContent]:
    if name == "hello":
        return [types.TextContent(type="text", text=f"Hello, {arguments['name']}!")]
    raise ValueError(f"Unknown tool: {name}")

async def main():
    async with stdio_server() as (read, write):
        await app.run(read, write, app.create_initialization_options())

if __name__ == "__main__":
    import asyncio
    asyncio.run(main())
```

## Claude Desktop MCP Config

```json
{
  "mcpServers": {
    "my-server": {
      "command": "python",
      "args": ["path/to/server.py"]
    }
  }
}
```

## JSON-RPC 2.0 Reference

MCP uses JSON-RPC 2.0 as its wire protocol. Key message types:
- `initialize` / `initialized`
- `tools/list`
- `tools/call`
- `resources/list`
- `resources/read`
