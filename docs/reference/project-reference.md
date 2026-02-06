---
title: Project Reference
status: stable
owners: [documentation-team]
---

# Project Reference

## Directory Structure

```
src/{{ package_name }}/
├── domain/
│   ├── entities.py         # Entity with identity (id, name, value)
│   ├── value_objects.py    # EntityName, EntityValue (frozen dataclasses)
│   └── exceptions.py       # DomainError, ValidationError, EntityNotFoundError
├── protocols/
│   └── repository.py       # Repository[T] protocol (save, get, list_all, delete)
├── services/
│   ├── context.py          # ServiceContext (frozen dataclass, holds Repository[Entity])
│   └── example_service.py  # create_example, get_example, list_examples
├── adapters/
│   └── repository.py       # InMemoryRepository[T]
└── cli/
    ├── main.py             # Typer app, get_context(), version command
    └── commands/
        └── example.py      # create, get, list commands

tests/
├── conftest.py             # Fixtures: fake_repository, test_context
├── fakes/
│   └── fake_repository.py  # FakeRepository[T] with test helpers
├── unit/
│   ├── test_domain.py      # Entity, value object, validation tests
│   └── test_services.py    # Service function tests with fakes
└── integration/
    └── test_adapters.py    # InMemoryRepository tests
```

## pyproject.toml Configuration

Key sections (see [pyproject.toml.jinja](../../pyproject.toml.jinja)):

- `[project]` — name, version, dependencies (typer)
- `[project.scripts]` — CLI entry point
- `[tool.uv]` — dev dependencies (pytest, ruff, coverage)
- `[tool.ruff]` — linting rules (`E`, `F`, `I`, `UP`), line length 88
- `[tool.pytest.ini_options]` — test paths, markers
- `[tool.coverage]` — source path, omit patterns

## Tooling Quick Reference

| Tool | Purpose | Key Commands |
|------|---------|-------------|
| uv | Package management | `uv sync`, `uv add`, `uv run` |
| ruff | Lint + format | `uv run ruff check .`, `uv run ruff format .` |
| pytest | Testing | `uv run pytest`, `uv run pytest --cov` |
| Typer | CLI framework | `uv run {{ cli_command }} --help` |

## Import Conventions

Dependencies flow inward: `cli → services → protocols ← adapters`. Domain has no imports from other layers.

```python
# Services import domain + protocols:
from {{ package_name }}.domain.entities import Entity
from {{ package_name }}.protocols.repository import Repository

# Adapters import domain only (satisfy protocols structurally):
from {{ package_name }}.domain.entities import Entity

# CLI imports services + adapters (wiring layer):
from {{ package_name }}.services.context import ServiceContext
from {{ package_name }}.adapters.repository import InMemoryRepository
```

## Sources

- [pyproject.toml.jinja](../../pyproject.toml.jinja) — project configuration
- [copier.yml](../../copier.yml) — template variables and defaults
- [Architecture](../architecture/architecture.md) — design patterns
