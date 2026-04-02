# Exercise 02: Tool Schema Design

**Difficulty:** Intermediate
**Domain:** MCP & Tool Design
**Goal:** Practice writing tool descriptions and schemas that guide Claude to use tools correctly.

---

## Context

The quality of your tool description directly determines whether Claude uses the tool correctly, uses the wrong tool, or skips the tool entirely. This exercise focuses on the craft of schema design through a series of comparison cases.

---

## Part A: Fix the Bad Descriptions

Each tool below has a poor description. Rewrite it to be clear and unambiguous.

### Tool 1 (bad)
```json
{
  "name": "get_data",
  "description": "Gets data.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "id": { "type": "string" }
    },
    "required": ["id"]
  }
}
```

**Problems:** What data? From where? When should Claude use this?

**Your improved version:**
```json
{
  "name": "get_data",
  "description": "TODO: rewrite this",
  ...
}
```

---

### Tool 2 (bad)
```json
{
  "name": "search",
  "description": "Search for things.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "q": { "type": "string" }
    },
    "required": ["q"]
  }
}
```

**Your improved version:**
```json
{
  "name": "search",
  "description": "TODO: rewrite this",
  ...
}
```

---

## Part B: Design From Scratch

Design the complete tool schema for the following specifications.

### Tool 3: Send Email

**Spec:** Sends an email via the company's internal SMTP relay. Supports plain text and HTML bodies. BCC is optional. Maximum 10 recipients in TO field. Should only be used when the user explicitly asks to send an email — do not use for drafting.

```json
{
  "name": "send_email",
  "description": "TODO",
  "inputSchema": {
    "TODO": "define all required and optional properties"
  }
}
```

---

### Tool 4: Query Database

**Spec:** Executes a read-only SQL SELECT query against the company's PostgreSQL database. Returns up to 100 rows as a list of objects. The database contains tables: `orders`, `customers`, `products`. For write operations, a different tool must be used.

```json
{
  "name": "query_database",
  "description": "TODO",
  "inputSchema": {
    "TODO": "define properties"
  }
}
```

---

## Part C: Disambiguation

When two tools do similar things, descriptions must help Claude choose the right one. Write descriptions for this pair so Claude never confuses them.

### Tool pair: `search_products` vs `get_product_by_id`

- `search_products`: full-text search, returns multiple results, used when user describes what they want
- `get_product_by_id`: exact lookup, returns one result, used when user provides a specific ID

Write descriptions that make the distinction crystal clear.

---

## Part D: Schema Validation

The following schema has 3 bugs. Find and fix them.

```json
{
  "name": "create_ticket",
  "description": "Creates a support ticket.",
  "inputSchema": {
    "type": "Object",
    "properties": {
      "title": { "type": "string" },
      "priority": {
        "type": "string",
        "enum": ["low", "medium", "high", "critical"]
      },
      "tags": {
        "type": "array",
        "items": "string"
      }
    }
  }
}
```

Bugs:
1. `TODO`
2. `TODO`
3. `TODO`

---

## Key Concepts Practiced

- Writing descriptions that include: what it does, what it returns, when to use it
- JSON Schema: `type`, `properties`, `required`, `enum`, `items`
- Disambiguation between similar tools
- Common schema mistakes
