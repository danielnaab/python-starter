---
title: Getting Started
status: stable
owners: [documentation-team]
---

# Getting Started

## Prerequisites

- Python 3.11+
- [uv](https://docs.astral.sh/uv/) (package manager)
- [Copier](https://copier.readthedocs.io/) (template engine)

## Generate a Project

```bash
pip install copier
copier copy path/to/python-starter my-project
cd my-project
```

Copier will prompt for: project name, package name, author, Python version, and optional features (example code, GitHub Actions, dev scripts).

## Project Layout

```
my-project/
├── src/<package_name>/
│   ├── domain/          # Entity, value objects, errors, repository port
│   ├── services/        # Pure functions + ServiceContext
│   ├── adapters/        # Port implementations (InMemoryRepository)
│   └── cli/             # Typer CLI commands
├── tests/
│   ├── fakes/           # Fake implementations for testing
│   ├── unit/            # Unit tests (entity, services)
│   └── integration/     # Integration tests (adapters)
└── pyproject.toml       # All configuration
```

## Essential Commands

```bash
uv sync                    # Install dependencies
uv run pytest              # Run tests
uv run ruff check .        # Lint
uv run ruff format .       # Format
uv run <cli_command> --help  # CLI help
```

## Update from Template

```bash
cd my-project
copier update
```

## Next Steps

- [Development Guide](development.md) — adding features, testing, CLI usage
- [Architecture](../architecture/architecture.md) — patterns and design decisions
- [Decisions](../decisions/) — ADRs explaining tool and pattern choices

## Sources

- [copier.yml](../../copier.yml) — template configuration
- [pyproject.toml.jinja](../../pyproject.toml.jinja) — project configuration template
