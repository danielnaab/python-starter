---
title: "ADR 004: Functional Service Layer"
status: stable
owners: [architecture-team]
---

# ADR 004: Functional Service Layer

## Status

Accepted.

## Context

The service layer contains business logic and orchestrates domain objects and adapters. Common approaches: service classes with injected dependencies, or standalone functions.

## Decision

Services are **pure functions** taking `ServiceContext` as first parameter.

## Rationale

- **Pythonic**: Functions are the natural unit for stateless operations
- **Testable**: Pure functions with explicit inputs/outputs
- **Composable**: Functions compose without lifecycle management
- **Explicit**: All dependencies visible in the context parameter
- **Simple**: No `__init__`, no instance state, no inheritance hierarchy

## Example

```python
def create_example(ctx: ServiceContext, name: str, value: int) -> Entity:
    entity = Entity(name=EntityName(text=name), value=EntityValue(amount=value))
    ctx.repository.save(entity)
    return entity
```

## When to Use Classes

Use classes when there is genuine instance state (connection pools, caches, stateful protocols). For stateless request-handling logic, prefer functions.

## Alternatives Considered

- **Service classes**: More boilerplate (`__init__`, `self`), instance state temptation
- **Module-level functions with globals**: Hidden dependencies, untestable
- **Higher-order functions / currying**: Overly complex for Python

## Sources

- [services/example_service.py.jinja](../../src/%7B%7B%20package_name%20%7D%7D/services/example_service.py.jinja) — service functions
- [Architecture Patterns with Python](https://www.cosmicpython.com/book/) — service layer patterns
