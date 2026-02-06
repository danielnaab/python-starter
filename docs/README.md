---
title: Python Starter Template Documentation
status: stable
owners: [template-team]
---

# Python Starter Template

Copier template for Python projects with functional service architecture, protocol-based DI, and modern tooling (uv, ruff, pytest, Typer).

## Quick Start

```bash
pip install copier
copier copy path/to/python-starter my-project
```

See [Getting Started](guides/getting-started.md) for details.

## Architecture

Services are pure functions taking a frozen `ServiceContext` dataclass. Interfaces use `typing.Protocol` (no inheritance). Tests use fakes, not mocks.

See [Architecture](architecture/architecture.md) for the full pattern.

## Navigation

### Guides
- [Getting Started](guides/getting-started.md) — installation, project layout, essential commands
- [Development](guides/development.md) — adding features, testing, CLI, code quality

### Architecture & Decisions
- [Architecture](architecture/architecture.md) — layers, patterns, technology choices
- [ADR 001: uv](decisions/001-why-uv.md) | [002: src layout](decisions/002-src-layout.md) | [003: Protocol DI](decisions/003-protocol-di.md) | [004: Functional services](decisions/004-functional-services.md) | [005: Context objects](decisions/005-context-objects.md) | [006: Typer](decisions/006-typer-cli.md) | [007: Ruff](decisions/007-ruff-tooling.md)

### Reference
- [Project Reference](reference/project-reference.md) — directory structure, configuration, tooling

### Template
- [Template Design](copier/template-design.md) — variables, conditional content, updates
- [Dependency Pattern](copier/dependency-pattern.md) — template-as-dependency approach

### For AI Agents
- [Agent Reference](agents.md) — structured technical reference for AI tools

## Authority

Code is canonical. When docs conflict with code, update the docs. See [knowledge-base.yaml](knowledge-base.yaml).

## Sources

- [Architecture Patterns with Python](https://www.cosmicpython.com/book/)
- [uv](https://docs.astral.sh/uv/) | [ruff](https://docs.astral.sh/ruff/) | [Typer](https://typer.tiangolo.com/) | [pytest](https://docs.pytest.org/)
- [PEP 544 — Protocols](https://peps.python.org/pep-0544/)
