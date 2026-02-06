---
title: Development Guide
status: stable
owners: [documentation-team]
---

# Development Guide

## Adding a Service Function

1. Create function in `src/<pkg>/services/<domain>_service.py`
2. First parameter: `ServiceContext`
3. Add type hints and docstring
4. Write unit test with fake context
5. Wire to CLI command if needed

```python
# Follow the pattern in services/example_service.py.jinja:
def create_example(ctx: ServiceContext, name: str, value: int) -> Entity:
    entity = Entity(name=EntityName(text=name), value=EntityValue(amount=value))
    ctx.repository.save(entity)
    return entity
```

## Adding a Protocol + Adapter

1. Define protocol in `src/<pkg>/protocols/<name>.py`
2. Implement adapter in `src/<pkg>/adapters/<name>.py`
3. Add Protocol-typed field to `ServiceContext`
4. Create fake in `tests/fakes/fake_<name>.py`
5. Update `conftest.py` fixture

## Testing

**Philosophy**: Fakes over mocks. Unit tests with fakes, integration tests with real adapters.

```bash
uv run pytest                              # All tests
uv run pytest tests/unit/                  # Unit only
uv run pytest tests/integration/           # Integration only
uv run pytest --cov=<pkg> --cov-report=term-missing  # Coverage
uv run pytest -x                           # Stop on first failure
uv run pytest -v                           # Verbose
```

**Writing tests**:
- Unit tests: use `test_context` fixture (provides `FakeRepository`)
- Integration tests: use real adapters (e.g., `InMemoryRepository`)
- Fakes provide test helpers: `assert_saved()`, `get_saved_count()`, `reset()`

## CLI Usage

```bash
uv run <cli_command> --help                # Show commands
uv run <cli_command> create "name" 42      # Create entity
uv run <cli_command> get <uuid>            # Get by ID
uv run <cli_command> list                  # List all
uv run python -m <package_name> --help     # Alternative invocation
```

The CLI is a thin adapter — it builds `ServiceContext` with real adapters and delegates to service functions.

## Code Quality

```bash
uv run ruff check .          # Lint
uv run ruff check . --fix    # Lint + auto-fix
uv run ruff format .         # Format
```

## Dependencies

```bash
uv add <package>             # Add runtime dependency
uv add --dev <package>       # Add dev dependency
uv remove <package>          # Remove dependency
uv lock --upgrade            # Update lockfile
```

## Sources

- [Architecture](../architecture/architecture.md) — patterns and layer structure
- [services/example_service.py.jinja](../../src/%7B%7B%20package_name%20%7D%7D/services/example_service.py.jinja) — service pattern
- [tests/conftest.py.jinja](../../tests/conftest.py.jinja) — test fixtures
- [cli/main.py.jinja](../../src/%7B%7B%20package_name%20%7D%7D/cli/main.py.jinja) — CLI wiring
