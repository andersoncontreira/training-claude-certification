# Exercise 01: Basic Agent Loop

**Difficulty:** Beginner
**Domain:** Agentic Architecture
**Goal:** Implement a minimal agent loop with one tool using the Anthropic Python SDK.

---

## Context

An agent loop is the core pattern of agentic systems: Claude reasons, calls a tool, receives the result, and reasons again until it has enough information to answer the user.

---

## Task

Build a Python script that implements a basic agent loop with a single tool: `get_weather`.

The agent should:
1. Receive a user question about weather
2. Call the `get_weather` tool with the appropriate city
3. Receive the (mocked) result
4. Return a natural language answer to the user

---

## Requirements

- Use the `anthropic` Python SDK
- Define the tool with a proper JSON schema
- Handle the full loop: `tool_use` → `tool_result` → final response
- The loop must terminate when Claude stops calling tools

---

## Starter Code

```python
import anthropic
import json

client = anthropic.Anthropic()

# Mock tool implementation
def get_weather(city: str) -> dict:
    mock_data = {
        "São Paulo": {"temp_c": 28, "condition": "Partly cloudy"},
        "London": {"temp_c": 12, "condition": "Rainy"},
        "Tokyo": {"temp_c": 22, "condition": "Clear"},
    }
    return mock_data.get(city, {"temp_c": 20, "condition": "Unknown"})

# Tool schema
tools = [
    {
        "name": "get_weather",
        "description": "Get current weather for a given city.",
        "input_schema": {
            "type": "object",
            "properties": {
                "city": {
                    "type": "string",
                    "description": "The city name, e.g. 'São Paulo'"
                }
            },
            "required": ["city"]
        }
    }
]

def run_agent(user_message: str) -> str:
    messages = [{"role": "user", "content": user_message}]

    while True:
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=1024,
            tools=tools,
            messages=messages
        )

        # TODO: Check stop_reason
        # TODO: If "tool_use", extract tool name and input
        # TODO: Call the appropriate function
        # TODO: Append assistant message and tool_result to messages
        # TODO: If "end_turn", extract and return the text response

        pass  # replace with your implementation

if __name__ == "__main__":
    answer = run_agent("What's the weather like in Tokyo right now?")
    print(answer)
```

---

## Expected Output

```
The weather in Tokyo is currently clear with a temperature of 22°C.
```

---

## Extension Challenges

1. Add a second tool `convert_temperature(celsius: float) -> float` that converts to Fahrenheit, and observe how Claude chains two tool calls.
2. Add a `max_iterations` limit to the loop and raise an error if exceeded.
3. Print each tool call and result to the console so you can trace the loop execution.

---

## Key Concepts Practiced

- `stop_reason`: `"tool_use"` vs `"end_turn"`
- Building the `messages` array incrementally
- `tool_use` and `tool_result` content block types
