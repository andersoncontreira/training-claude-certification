# Python Code Style

- Always use classes instead of standalone functions.
- Each class must have a single responsibility (SRP).
- Use dataclasses or Pydantic models for data structures — never plain dicts as return types.
- Private methods must be prefixed with `_`.
- Use type hints on all method signatures (parameters and return types).
- Do not use `*args` or `**kwargs` unless there is a clear justification.
- Prefer composition over inheritance.
- Never use `print()` for logging — use the `logging` module.
- All classes must have a docstring describing their responsibility.
