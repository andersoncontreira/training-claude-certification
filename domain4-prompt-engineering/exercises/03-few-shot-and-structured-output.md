# Exercise 03: Few-Shot Prompting & Structured Outputs

**Difficulty:** Advanced
**Domain:** Prompt Engineering
**Goal:** Use few-shot examples and tool_choice to reliably extract structured data from unstructured text.

---

## Context

Two of the most powerful techniques for production-grade prompts:
1. **Few-shot prompting** — teach Claude the exact format by example
2. **Forced tool use** — use `tool_choice` to guarantee JSON output via a tool schema

---

## Part A: Few-Shot for Classification

Build a sentiment + intent classifier for product reviews.

**Target output:**
```json
{
  "sentiment": "negative",
  "intent": "return_request",
  "urgency": "high",
  "key_issue": "product arrived damaged"
}
```

### Step 1: Zero-shot baseline

```python
import anthropic, json

client = anthropic.Anthropic()

def classify_review_zero_shot(review: str) -> dict:
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=256,
        temperature=0,
        system="Classify the following product review. Return JSON with keys: sentiment (positive/neutral/negative), intent (praise/question/return_request/complaint/other), urgency (high/normal/low), key_issue (string or null).",
        messages=[{"role": "user", "content": review}]
    )
    return json.loads(response.content[0].text)

# Test
reviews = [
    "Great product, arrived on time, very happy!",
    "The screen cracked on first use. I need to return this immediately.",
    "Does this come in blue?"
]

for r in reviews:
    print(classify_review_zero_shot(r))
```

### Step 2: Add few-shot examples

```python
FEW_SHOT_EXAMPLES = """
<examples>

<example>
<review>Absolutely love this product! Best purchase I've made this year.</review>
<classification>{"sentiment": "positive", "intent": "praise", "urgency": "low", "key_issue": null}</classification>
</example>

<example>
<review>This broke after 2 days. I want a refund NOW. This is unacceptable!</review>
<classification>{"sentiment": "negative", "intent": "return_request", "urgency": "high", "key_issue": "product broke after 2 days"}</classification>
</example>

<example>
<review>It's okay, not what I expected but it works.</review>
<classification>{"sentiment": "neutral", "intent": "complaint", "urgency": "low", "key_issue": "did not meet expectations"}</classification>
</example>

</examples>
"""

def classify_review_few_shot(review: str) -> dict:
    system = f"""You are a product review classifier.
{FEW_SHOT_EXAMPLES}
Classify the review in the same JSON format as the examples.
Return only valid JSON, no explanation."""

    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=256,
        temperature=0,
        system=system,
        messages=[{"role": "user", "content": review}]
    )
    return json.loads(response.content[0].text)
```

**Compare the outputs of zero-shot vs. few-shot for edge cases:**
- "Arrived broken but the seller sorted it out quickly"
- "meh"
- "WORST PRODUCT EVER DO NOT BUY!!!"

---

## Part B: Forced Structured Output via Tool Use

The most reliable way to guarantee JSON output is to define a tool with the exact schema you want and force Claude to call it.

```python
import anthropic, json

client = anthropic.Anthropic()

# Define the output schema as a tool
extraction_tool = {
    "name": "submit_review_classification",
    "description": "Submit the classification of a product review",
    "input_schema": {
        "type": "object",
        "properties": {
            "sentiment": {
                "type": "string",
                "enum": ["positive", "neutral", "negative"]
            },
            "intent": {
                "type": "string",
                "enum": ["praise", "question", "return_request", "complaint", "other"]
            },
            "urgency": {
                "type": "string",
                "enum": ["high", "normal", "low"]
            },
            "key_issue": {
                "type": ["string", "null"],
                "description": "The main issue mentioned, or null if none"
            }
        },
        "required": ["sentiment", "intent", "urgency", "key_issue"]
    }
}

def classify_review_tool_forced(review: str) -> dict:
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=512,
        temperature=0,
        tools=[extraction_tool],
        tool_choice={"type": "tool", "name": "submit_review_classification"},  # FORCE it
        system="Classify the product review.",
        messages=[{"role": "user", "content": review}]
    )

    # With forced tool use, the first content block is always tool_use
    tool_call = response.content[0]
    return tool_call.input  # already a dict, no json.loads needed

# Test
print(classify_review_tool_forced("This is garbage. I want my money back."))
```

---

## Part C: Batch Entity Extraction

Extract structured data from unstructured text at scale.

```python
from typing import List

invoice_tool = {
    "name": "extract_invoice_data",
    "description": "Extract structured data from an invoice",
    "input_schema": {
        "type": "object",
        "properties": {
            "invoice_number": {"type": "string"},
            "date": {"type": "string", "description": "ISO 8601 format"},
            "vendor_name": {"type": "string"},
            "total_amount": {"type": "number"},
            "currency": {"type": "string"},
            "line_items": {
                "type": "array",
                "items": {
                    "type": "object",
                    "properties": {
                        "description": {"type": "string"},
                        "quantity": {"type": "number"},
                        "unit_price": {"type": "number"}
                    }
                }
            }
        },
        "required": ["invoice_number", "date", "vendor_name", "total_amount", "currency"]
    }
}

SAMPLE_INVOICE = """
Invoice #INV-2024-0892
Date: March 15, 2024
From: Acme Software Ltd.

Items:
- Pro License x3 @ $299.00 each
- Support Package x1 @ $150.00

Total: $1,047.00 USD
"""

def extract_invoice(text: str) -> dict:
    # TODO: Implement using invoice_tool with forced tool_choice
    pass

result = extract_invoice(SAMPLE_INVOICE)
print(json.dumps(result, indent=2))
```

---

## Part D: Reliability Comparison

Run 5 different reviews through all three methods (zero-shot, few-shot, forced tool). Fill in:

| Review | Zero-shot valid JSON? | Few-shot valid JSON? | Tool-forced valid JSON? |
|--------|-----------------------|----------------------|------------------------|
| Review 1 | | | |
| Review 2 | | | |
| Review 3 | | | |
| Review 4 | | | |
| Review 5 | | | |

**Conclusion:** Which method is most reliable for production use, and why?

---

## Key Concepts Practiced

- Few-shot example format with XML tags
- `tool_choice: {"type": "tool", "name": "..."}` for forced tool use
- JSON Schema for output validation
- `enum` constraints to limit output values
- Comparing reliability across prompting strategies
