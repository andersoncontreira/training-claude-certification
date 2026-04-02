# Exercise 03: Reliability and Fallbacks

**Difficulty:** Advanced
**Domain:** Context & Reliability
**Goal:** Build a production-grade wrapper around the Claude API with retry logic, output validation, fallbacks, and circuit breakers.

---

## Context

Production AI systems fail in predictable ways: rate limits, server errors, malformed outputs, timeouts. A reliable system handles all of these gracefully.

---

## Part A: Retry with Exponential Backoff

```python
import anthropic
import time
import random
from typing import Callable, Any

client = anthropic.Anthropic()

def call_with_retry(
    fn: Callable,
    max_retries: int = 3,
    base_delay: float = 1.0,
    max_delay: float = 60.0
) -> Any:
    """
    Call fn with exponential backoff on retryable errors.

    Retryable: RateLimitError (429), InternalServerError (500), APIStatusError (529)
    Non-retryable: AuthenticationError, BadRequestError, NotFoundError
    """
    for attempt in range(max_retries + 1):
        try:
            return fn()
        except anthropic.RateLimitError as e:
            if attempt == max_retries:
                raise
            # Use the Retry-After header if available
            retry_after = float(e.response.headers.get("retry-after", base_delay * (2 ** attempt)))
            jitter = random.uniform(0, 0.1 * retry_after)
            wait = min(retry_after + jitter, max_delay)
            print(f"Rate limited. Waiting {wait:.1f}s (attempt {attempt + 1}/{max_retries})")
            time.sleep(wait)
        except anthropic.InternalServerError:
            if attempt == max_retries:
                raise
            wait = min(base_delay * (2 ** attempt) + random.uniform(0, 1), max_delay)
            print(f"Server error. Waiting {wait:.1f}s (attempt {attempt + 1}/{max_retries})")
            time.sleep(wait)
        except (anthropic.AuthenticationError, anthropic.BadRequestError):
            raise  # don't retry client errors

    raise RuntimeError("Unreachable")

# Usage
def ask_claude(question: str) -> str:
    return call_with_retry(
        lambda: client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=512,
            messages=[{"role": "user", "content": question}]
        ).content[0].text
    )
```

---

## Part B: Output Validation

Never trust Claude's output blindly when you expect a specific format.

```python
import json
from typing import Any

class OutputValidationError(Exception):
    pass

def get_json_output(prompt: str, expected_schema: dict, max_retries: int = 2) -> dict:
    """
    Get a JSON output from Claude, retrying with corrective feedback if invalid.
    """
    messages = [{"role": "user", "content": prompt}]

    for attempt in range(max_retries + 1):
        response = call_with_retry(
            lambda: client.messages.create(
                model="claude-sonnet-4-6",
                max_tokens=1024,
                temperature=0,
                messages=messages
            )
        )

        text = response.content[0].text

        # Try to parse JSON
        try:
            # Handle markdown code blocks
            if "```json" in text:
                text = text.split("```json")[1].split("```")[0].strip()
            elif "```" in text:
                text = text.split("```")[1].split("```")[0].strip()

            data = json.loads(text)

            # TODO: Validate against expected_schema using jsonschema
            # from jsonschema import validate, ValidationError
            # validate(instance=data, schema=expected_schema)

            return data

        except json.JSONDecodeError as e:
            if attempt == max_retries:
                raise OutputValidationError(f"Could not parse JSON after {max_retries + 1} attempts: {e}")

            # Add corrective feedback to the conversation
            messages.append({"role": "assistant", "content": response.content[0].text})
            messages.append({
                "role": "user",
                "content": f"Your response was not valid JSON. Error: {e}. Please respond with only valid JSON, no markdown."
            })

    raise OutputValidationError("Unreachable")

# Test
schema = {
    "type": "object",
    "properties": {
        "sentiment": {"type": "string", "enum": ["positive", "neutral", "negative"]},
        "score": {"type": "number", "minimum": 0, "maximum": 1}
    },
    "required": ["sentiment", "score"]
}

result = get_json_output(
    'Analyze the sentiment of: "This product is amazing!" Return JSON with keys: sentiment and score (0-1).',
    schema
)
print(result)
```

---

## Part C: Circuit Breaker

A circuit breaker stops calling a service that's consistently failing, preventing cascading failures.

```python
from enum import Enum
from dataclasses import dataclass, field
from datetime import datetime, timedelta

class CircuitState(Enum):
    CLOSED = "closed"       # Normal operation
    OPEN = "open"           # Failing — reject all calls
    HALF_OPEN = "half_open" # Testing if service recovered

@dataclass
class CircuitBreaker:
    failure_threshold: int = 5          # failures before opening
    recovery_timeout: float = 60.0      # seconds before trying again
    success_threshold: int = 2          # successes to close from half-open

    _failures: int = field(default=0, init=False)
    _successes: int = field(default=0, init=False)
    _state: CircuitState = field(default=CircuitState.CLOSED, init=False)
    _last_failure_time: datetime | None = field(default=None, init=False)

    def call(self, fn: Callable) -> Any:
        if self._state == CircuitState.OPEN:
            # Check if recovery timeout has passed
            if datetime.now() - self._last_failure_time > timedelta(seconds=self.recovery_timeout):
                self._state = CircuitState.HALF_OPEN
                self._successes = 0
                print("Circuit: HALF_OPEN — testing recovery")
            else:
                raise RuntimeError("Circuit is OPEN — service unavailable")

        try:
            result = fn()
            self._on_success()
            return result
        except Exception as e:
            self._on_failure()
            raise

    def _on_success(self):
        if self._state == CircuitState.HALF_OPEN:
            self._successes += 1
            if self._successes >= self.success_threshold:
                self._state = CircuitState.CLOSED
                self._failures = 0
                print("Circuit: CLOSED — service recovered")
        elif self._state == CircuitState.CLOSED:
            self._failures = 0  # reset on success

    def _on_failure(self):
        self._failures += 1
        self._last_failure_time = datetime.now()
        if self._failures >= self.failure_threshold:
            self._state = CircuitState.OPEN
            print(f"Circuit: OPEN after {self._failures} failures")

circuit = CircuitBreaker(failure_threshold=3, recovery_timeout=10.0)
```

---

## Part D: Full Reliability Stack

Combine retry, output validation, circuit breaker, and fallback into a single production-ready client.

```python
class ReliableClaudeClient:
    def __init__(self):
        self.client = anthropic.Anthropic()
        self.circuit = CircuitBreaker()

    def complete(
        self,
        messages: list,
        system: str = "",
        model: str = "claude-sonnet-4-6",
        max_tokens: int = 1024,
        temperature: float = 0,
        fallback_response: str | None = None
    ) -> str:
        def _call():
            return self.circuit.call(
                lambda: call_with_retry(
                    lambda: self.client.messages.create(
                        model=model,
                        max_tokens=max_tokens,
                        temperature=temperature,
                        system=system,
                        messages=messages
                    )
                )
            )

        try:
            response = _call()
            return response.content[0].text
        except RuntimeError as e:
            if "Circuit is OPEN" in str(e) and fallback_response:
                print(f"Using fallback: {e}")
                return fallback_response
            raise

# Usage
reliable = ReliableClaudeClient()

answer = reliable.complete(
    messages=[{"role": "user", "content": "What is 2+2?"}],
    fallback_response="I'm temporarily unavailable. Please try again in a moment."
)
print(answer)
```

---

## Testing Your Reliability Stack

1. Simulate a rate limit error by patching the client — verify retry with backoff
2. Simulate 5 consecutive failures — verify circuit opens
3. Wait for `recovery_timeout` — verify circuit transitions to half-open
4. Simulate 2 successes — verify circuit closes again
5. Test JSON output validation with a prompt that returns invalid JSON first

---

## Key Concepts Practiced

- Exponential backoff with jitter
- Distinguishing retryable vs non-retryable errors
- Self-correcting JSON output via conversation
- Circuit breaker pattern (CLOSED → OPEN → HALF_OPEN → CLOSED)
- Layered reliability: retry + circuit breaker + fallback
