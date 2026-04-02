# Exercise 02: Memory Strategies

**Difficulty:** Intermediate
**Domain:** Context & Reliability
**Goal:** Implement persistent memory patterns to give Claude knowledge that survives context resets.

---

## Context

The context window is ephemeral — when a conversation ends, Claude "forgets" everything. For production systems that need persistent state (user preferences, accumulated knowledge, session history), you need external memory.

---

## Part A: File-Based Memory (Claude Code Style)

Implement a simple memory system using JSON files — similar to how Claude Code's memory works.

```python
import json
import os
from pathlib import Path
from datetime import datetime

MEMORY_DIR = Path("./memory")
MEMORY_DIR.mkdir(exist_ok=True)

def save_memory(key: str, value: dict) -> None:
    """Save a memory entry to disk."""
    entry = {
        "key": key,
        "value": value,
        "updated_at": datetime.utcnow().isoformat()
    }
    filepath = MEMORY_DIR / f"{key}.json"
    with open(filepath, "w") as f:
        json.dump(entry, f, indent=2)

def load_memory(key: str) -> dict | None:
    """Load a memory entry from disk."""
    filepath = MEMORY_DIR / f"{key}.json"
    if not filepath.exists():
        return None
    with open(filepath) as f:
        return json.load(f)["value"]

def load_all_memories() -> dict:
    """Load all memory entries — used to inject into system prompt."""
    memories = {}
    for filepath in MEMORY_DIR.glob("*.json"):
        key = filepath.stem
        memories[key] = load_memory(key)
    return memories

def build_system_prompt_with_memory() -> str:
    """Build a system prompt that includes all persisted memories."""
    memories = load_all_memories()

    if not memories:
        memory_section = "No memories stored yet."
    else:
        memory_section = json.dumps(memories, indent=2)

    return f"""You are a personal coding assistant with persistent memory.

<memory>
{memory_section}
</memory>

When you learn something important about the user (preferences, project details, decisions made),
use the save_memory tool to persist it for future conversations."""
```

### Add a save_memory tool

```python
import anthropic

client = anthropic.Anthropic()

memory_tool = {
    "name": "save_memory",
    "description": "Save an important piece of information to persistent memory for future conversations. Use this when you learn about user preferences, project decisions, or any context that should be remembered.",
    "input_schema": {
        "type": "object",
        "properties": {
            "key": {
                "type": "string",
                "description": "A short identifier for this memory, e.g. 'preferred_language', 'project_stack'"
            },
            "value": {
                "type": "object",
                "description": "The data to remember"
            }
        },
        "required": ["key", "value"]
    }
}

def run_agent_with_memory(user_message: str) -> str:
    system = build_system_prompt_with_memory()
    messages = [{"role": "user", "content": user_message}]

    # TODO: implement agent loop
    # When Claude calls save_memory: call save_memory(key, value)
    # Return the final text response
    pass

# Test session 1: teach Claude something
print(run_agent_with_memory("I prefer type hints in all Python functions and I'm using Python 3.11."))
print(run_agent_with_memory("My project uses FastAPI, SQLAlchemy, and PostgreSQL."))

# Test session 2: verify memory persists (simulate new conversation)
print(run_agent_with_memory("Write me a function to query users from the database."))
# Expected: Claude should use the remembered stack (FastAPI/SQLAlchemy) and add type hints
```

---

## Part B: RAG with Simple In-Memory Vector Store

Build a minimal RAG pipeline using numpy for similarity search (no external vector DB needed).

```python
import numpy as np
import anthropic
from typing import List

client = anthropic.Anthropic()

class SimpleVectorStore:
    """Minimal in-memory vector store using cosine similarity."""

    def __init__(self):
        self.documents: List[str] = []
        self.embeddings: List[np.ndarray] = []

    def embed(self, text: str) -> np.ndarray:
        """Get embeddings using Voyage AI via Anthropic (or mock for testing)."""
        # TODO: In production, use an embedding API
        # For this exercise, use a mock that gives random-but-consistent embeddings
        # (use hash of text as seed for reproducibility)
        np.random.seed(hash(text) % (2**32))
        return np.random.rand(1536)

    def add(self, document: str) -> None:
        embedding = self.embed(document)
        self.documents.append(document)
        self.embeddings.append(embedding)

    def search(self, query: str, k: int = 3) -> List[str]:
        """Return top-k most similar documents."""
        if not self.documents:
            return []

        query_embedding = self.embed(query)

        # Cosine similarity
        scores = []
        for doc_embedding in self.embeddings:
            similarity = np.dot(query_embedding, doc_embedding) / (
                np.linalg.norm(query_embedding) * np.linalg.norm(doc_embedding)
            )
            scores.append(similarity)

        # Return top-k
        top_indices = np.argsort(scores)[-k:][::-1]
        return [self.documents[i] for i in top_indices]

# Build a knowledge base
store = SimpleVectorStore()
knowledge_base = [
    "FastAPI is a modern Python web framework that is fast and easy to use.",
    "SQLAlchemy is a SQL toolkit and ORM for Python.",
    "PostgreSQL is an open-source relational database management system.",
    "JWT (JSON Web Tokens) are used for stateless authentication.",
    "Docker is a platform for containerizing applications.",
    "Python 3.11 introduced significant performance improvements.",
]

for doc in knowledge_base:
    store.add(doc)

def answer_with_rag(question: str) -> str:
    # Retrieve relevant context
    relevant_docs = store.search(question, k=3)
    context = "\n".join(f"- {doc}" for doc in relevant_docs)

    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=512,
        system=f"""Answer questions using only the context provided.
If the answer isn't in the context, say "I don't have that information."

<context>
{context}
</context>""",
        messages=[{"role": "user", "content": question}]
    )
    return response.content[0].text

# Test
questions = [
    "What is FastAPI good for?",
    "How does authentication work with JWT?",
    "What is the capital of France?"  # not in knowledge base
]

for q in questions:
    print(f"Q: {q}")
    print(f"A: {answer_with_rag(q)}\n")
```

---

## Key Concepts Practiced

- File-based persistent memory with system prompt injection
- Memory tools (save/load pattern)
- RAG pipeline: embed → store → retrieve → augment → generate
- Cosine similarity for document search
- Handling out-of-knowledge questions gracefully
