---
title: "ADR 003: Protocol-Based Dependency Injection"
status: stable
owners: [architecture-team]
---

# ADR 003: Protocol-Based Dependency Injection

## Status

Accepted.

## Context

Services need interfaces to decouple from concrete implementations. Python offers Abstract Base Classes, typing.Protocol, DI frameworks, and ad-hoc approaches.

## Decision

Use `typing.Protocol` (PEP 544) for interface definitions. No DI framework — dependencies are passed via context objects (see [ADR 005](005-context-objects.md)).

Protocols (ports) live in `domain/` because the domain defines what it needs from the outside world. Adapters satisfy these protocols via structural typing.

## Rationale

- **Structural typing**: Any class with matching methods satisfies the protocol — no inheritance required
- **Type safety**: Full IDE support, mypy/pyright checking
- **No framework**: Standard library only, no magic
- **Easy fakes**: Test implementations don't inherit from protocols

## Example

```python
class Repository(Protocol, Generic[T]):
    def save(self, entity: T) -> None: ...
    def get(self, entity_id: str) -> Optional[T]: ...
    def list_all(self) -> list[T]: ...
    def delete(self, entity_id: str) -> bool: ...

# Satisfies Repository without inheriting:
class InMemoryRepository(Generic[T]):
    def save(self, entity: T) -> None: ...
    def get(self, entity_id: str) -> Optional[T]: ...
    def list_all(self) -> list[T]: ...
    def delete(self, entity_id: str) -> bool: ...
```

## Alternatives Considered

- **ABC**: Requires inheritance; couples implementations to interface
- **DI frameworks** (dependency-injector): Adds complexity, magic, and a learning curve
- **Service Locator**: Global state anti-pattern; hides dependencies
- **Separate `protocols/` package**: Adds a directory for one file; ports conceptually belong to the domain

## Sources

- [PEP 544 — Protocols: Structural subtyping](https://peps.python.org/pep-0544/)
- [domain/repository.py.jinja](../../src/%7B%7B%20package_name%20%7D%7D/domain/repository.py.jinja) — Repository protocol
