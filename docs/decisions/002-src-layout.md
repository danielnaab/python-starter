---
title: "ADR 002: src Layout for Project Structure"
status: stable
owners: [architecture-team]
---

# ADR 002: src Layout for Project Structure

## Status

Accepted.

## Context

Python projects can use flat layout (`package/` at root) or src layout (`src/package/`). The choice affects import behavior during development and testing.

## Decision

Use **src layout**: `src/{{ package_name }}/`.

## Rationale

- **Prevents accidental imports**: Can't import from source tree without installation
- **Forces proper installation**: Tests run against installed package, not raw source
- **Modern standard**: Recommended by Python Packaging Authority
- **Clear boundaries**: Separates production code from tests, scripts, and config

## Directory Structure

```
project/
├── src/{{ package_name }}/    # Production code
├── tests/                      # Test code
├── scripts/                    # Dev scripts
└── pyproject.toml              # Configuration
```

## Alternatives Considered

- **Flat layout**: Risk of importing uninstalled source; confuses `sys.path`
- **Namespace packages**: Unnecessary complexity for single-package projects

## Sources

- [Python Packaging User Guide — src layout](https://packaging.python.org/en/latest/discussions/src-layout-vs-flat-layout/)
- [pytest — good practices](https://docs.pytest.org/en/stable/explanation/goodpractices.html)
