---
title: "ADR 007: Ruff for Linting and Formatting"
status: stable
owners: [architecture-team]
---

# ADR 007: Ruff for Linting and Formatting

## Status

Accepted.

## Context

Python linting/formatting historically requires multiple tools (Black, Flake8, isort, pyupgrade). Configuration drift and speed are ongoing problems.

## Decision

Use **ruff** as the single tool for both linting and formatting.

## Rationale

- **Speed**: 10-100x faster than Python-based tools (Rust implementation)
- **Unified**: Replaces Black + Flake8 + isort + pyupgrade + autoflake
- **800+ rules**: Comprehensive coverage from multiple linting traditions
- **Auto-fix**: Most issues fixed automatically
- **Black-compatible**: Formatting matches Black output
- **Astral ecosystem**: Same team behind uv

## Configuration

All in `pyproject.toml`:

```toml
[tool.ruff]
target-version = "py311"
line-length = 88

[tool.ruff.lint]
select = ["E", "F", "I", "UP"]
```

## Usage

```bash
uv run ruff check .          # Lint
uv run ruff check . --fix    # Lint + auto-fix
uv run ruff format .         # Format
```

## Alternatives Considered

- **Black + Flake8 + isort**: Three tools, three configs, slow
- **Pylint**: Very slow, noisy defaults, complex configuration

## Sources

- [ruff documentation](https://docs.astral.sh/ruff/)
- [pyproject.toml.jinja](../../pyproject.toml.jinja) — ruff configuration
