# Domain 2: MCP & Tool Design (18%)

Covers the Model Context Protocol (MCP) — the open standard for connecting AI models to external tools and data sources.

## What is MCP?

MCP (Model Context Protocol) is an open protocol that standardizes how AI applications connect to external tools, data sources, and services. Think of it as a USB standard for AI integrations.

```
Claude (MCP Client)  <-->  MCP Server  <-->  External Service
                     JSON-RPC 2.0
```

## Core Components

### MCP Server
- Exposes **tools**, **resources**, and **prompts** to clients
- Runs as a separate process (stdio or HTTP transport)
- Declares capabilities during initialization handshake
- Stateless per request (each tool call is independent)

### MCP Client
- Discovers available tools from the server
- Routes tool calls from the model to the appropriate server
- Manages the connection lifecycle

### Three Primitives

| Primitive | Description | Example |
|-----------|-------------|---------|
| **Tools** | Functions the model can call | `search_database()`, `send_email()` |
| **Resources** | Data the model can read | File contents, DB records |
| **Prompts** | Reusable prompt templates | Pre-built instruction sets |

## Tool Schema Design

A well-designed tool schema is the single most important factor in whether Claude uses a tool correctly.

### Anatomy of a Good Tool
```json
{
  "name": "search_products",
  "description": "Search the product catalog by keyword. Returns up to 10 matching products with name, price, and stock status. Use this when the user asks about product availability or pricing.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "query": {
        "type": "string",
        "description": "Search keyword or phrase"
      },
      "max_results": {
        "type": "integer",
        "description": "Maximum number of results to return (1-10)",
        "default": 5
      }
    },
    "required": ["query"]
  }
}
```

### Description Best Practices
- Describe **what** the tool does AND **when to use it**
- Mention what the tool returns
- Be explicit about limitations (e.g., "only returns public data")
- Avoid ambiguity between similar tools

## MCP Lifecycle

```
1. Client spawns server process (stdio) or connects (HTTP)
2. initialize handshake → server declares capabilities
3. Client lists available tools/resources
4. Model requests tool call
5. Client routes call to server
6. Server executes and returns result
7. Client passes result back to model
```

## Transport Options

| Transport | Use Case |
|-----------|----------|
| **stdio** | Local tools, CLI integrations |
| **HTTP + SSE** | Remote services, web APIs |

## Security Considerations
- Validate all tool inputs before execution
- Never expose secrets in tool descriptions or responses
- Use least-privilege: only expose what the model needs
- Sanitize outputs before returning to the model (prevent prompt injection)

## Common Exam Topics
- Distinguishing tools vs. resources vs. prompts
- Designing unambiguous tool descriptions
- Understanding the MCP handshake and lifecycle
- Choosing the right transport for a given scenario
- Input validation and security patterns
