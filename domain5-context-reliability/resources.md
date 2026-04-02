# Domain 5: Resources & References

## Official Anthropic Documentation

- [Context windows](https://docs.anthropic.com/en/docs/build-with-claude/context-windows)
- [Long context tips](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/long-context-tips)
- [Model overview and context limits](https://docs.anthropic.com/en/docs/about-claude/models/overview)
- [Error handling and retries](https://docs.anthropic.com/en/api/errors)
- [Rate limits](https://docs.anthropic.com/en/api/rate-limits)

## Model Context Limits (as of 2025)

| Model | Context Window | Output Tokens |
|-------|---------------|---------------|
| claude-opus-4-6 | 200K | 32K |
| claude-sonnet-4-6 | 200K | 64K |
| claude-haiku-4-5 | 200K | 8K |

## Token Counting API

```python
# Count tokens before sending to manage costs and context limits
response = client.messages.count_tokens(
    model="claude-sonnet-4-6",
    system="Your system prompt here",
    messages=[{"role": "user", "content": "Your message"}]
)
print(f"Input tokens: {response.input_tokens}")
```

## Error Types and Retry Strategy

```python
import anthropic

# Errors that warrant a retry (transient)
RETRYABLE_ERRORS = [
    anthropic.RateLimitError,      # 429: slow down
    anthropic.APIStatusError,       # 529: overloaded
    anthropic.InternalServerError,  # 500: server error
]

# Errors that should NOT be retried (client errors)
NON_RETRYABLE_ERRORS = [
    anthropic.AuthenticationError,  # 401: fix your key
    anthropic.PermissionDeniedError, # 403: not allowed
    anthropic.BadRequestError,      # 400: fix your request
    anthropic.NotFoundError,        # 404: wrong endpoint
]
```

## RAG Pattern (Retrieval-Augmented Generation)

```python
def answer_with_rag(question: str, vector_store) -> str:
    # 1. Embed the question
    query_embedding = embed(question)

    # 2. Retrieve top-k relevant documents
    relevant_docs = vector_store.search(query_embedding, k=5)

    # 3. Build context from retrieved docs
    context = "\n\n".join([doc.content for doc in relevant_docs])

    # 4. Ask Claude with retrieved context
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        system="Answer questions using only the provided context. If the answer is not in the context, say so.",
        messages=[{
            "role": "user",
            "content": f"<context>\n{context}\n</context>\n\nQuestion: {question}"
        }]
    )
    return response.content[0].text
```

## Useful Libraries

- `tiktoken` — approximate token counting (OpenAI tokenizer, close enough for estimation)
- `jsonschema` — validate Claude's JSON output against a schema
- `tenacity` — retry library with exponential backoff
