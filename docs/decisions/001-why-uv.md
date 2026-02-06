---
title: "ADR 001: Package Management with uv"
status: stable
owners: [architecture-team]
---

# ADR 001: Package Management with uv

## Status

Accepted.

## Context

Python package management has many tools (pip, pip-tools, Poetry, PDM). We need a single tool for dependency resolution, virtual environments, and lockfiles.

## Decision

Use **uv** for all package management.

## Rationale

- **Speed**: 10-100x faster than pip (Rust-based)
- **Unified**: Replaces pip + pip-tools + venv + virtualenv
- **Modern lockfile**: TOML-based `uv.lock` for reproducible installs
- **Astral ecosystem**: Same team behind ruff

## Alternatives Considered

- **pip + pip-tools**: Slow, requires multiple tools
- **Poetry**: Python-based (slower), custom lockfile format
- **PDM**: Less community adoption

## Usage

```bash
uv sync              # Install from lockfile
uv add <package>     # Add dependency
uv add --dev <pkg>   # Add dev dependency
uv run pytest        # Run in managed venv
uv lock --upgrade    # Update lockfile
```

## Sources

- [uv documentation](https://docs.astral.sh/uv/)
- [pyproject.toml.jinja](../../pyproject.toml.jinja) — project configuration
