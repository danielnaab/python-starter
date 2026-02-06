---
title: Python Starter Template - Agent Reference
status: stable
owners: [template-team]
---

# Agent Reference

## Project Metadata

- **Type**: Copier template for Python projects
- **Python**: 3.11+ | **Pkg mgr**: uv | **Lint/fmt**: ruff | **Test**: pytest | **CLI**: Typer
- **Architecture**: Functional services + protocol-based DI + frozen dataclass context

## Configuration

See [knowledge-base.yaml](knowledge-base.yaml) for entrypoints, canonical sources, and write permissions.

## Core Pattern

```python
# Actual code from services/context.py.jinja:
@dataclass(frozen=True)
class ServiceContext:
    repository: Repository[Entity]

# Actual code from services/example_service.py.jinja:
def create_example(ctx: ServiceContext, name: str, value: int) -> Entity:
    entity = Entity(name=EntityName(text=name), value=EntityValue(amount=value))
    ctx.repository.save(entity)
    return entity
```

Key rules:
- Services are **pure functions**, first param is `ServiceContext`
- Interfaces use `typing.Protocol` — no inheritance required
- Tests use **fakes** (not mocks), stored in `tests/fakes/`

## File Organization

```
src/{{ package_name }}/
├── domain/          # Entities, value objects, exceptions
├── protocols/       # Repository[T] protocol (typing.Protocol)
├── services/        # ServiceContext + pure service functions
├── adapters/        # InMemoryRepository[T]
└── cli/             # Typer CLI (thin adapter over services)
```

## Common Operations

**Add service function**: Create in `services/`, accept `ServiceContext` as first param, add unit test with fake context.

**Add protocol + adapter**: Define protocol in `protocols/`, implement in `adapters/`, add field to `ServiceContext`, create fake in `tests/fakes/`.

**Run tests**: `uv run pytest` | **Lint**: `uv run ruff check .` | **Format**: `uv run ruff format .`

## Authority

**Canonical** (source of truth): `src/`, `pyproject.toml`, `tests/`
**Interpretive** (explains canonical): `docs/`

When code and docs conflict, **code is correct**.

## Documentation

- [Architecture](architecture/architecture.md) — layers, patterns, testing strategy
- [Decisions](decisions/) — 7 ADRs (uv, src layout, protocols, services, context, Typer, ruff)
- [Guides](guides/getting-started.md) — setup and [development](guides/development.md)
- [Reference](reference/project-reference.md) — structure, config, tooling

## Sources

- [knowledge-base.yaml](knowledge-base.yaml) — KB configuration
- [services/context.py.jinja](../src/%7B%7B%20package_name%20%7D%7D/services/context.py.jinja) — ServiceContext
- [services/example_service.py.jinja](../src/%7B%7B%20package_name%20%7D%7D/services/example_service.py.jinja) — service functions
- [protocols/repository.py.jinja](../src/%7B%7B%20package_name%20%7D%7D/protocols/repository.py.jinja) — Repository protocol
