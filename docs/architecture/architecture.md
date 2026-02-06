---
title: Architecture
status: stable
owners: [architecture-team]
---

# Architecture

Functional service layer with protocol-based dependency injection.

## Layers

```
domain/        Pure business logic (entity, value objects, errors, repository port)
services/      Business operations (pure functions + ServiceContext)
adapters/      Port implementations (InMemoryRepository)
cli/           Typer CLI (thin adapter over services)
```

Dependencies flow inward: CLI → Services → Domain ← Adapters. Domain has no dependencies.

## Functional Services

Services are **pure functions** taking `ServiceContext` as first parameter:

```python
# services/context.py.jinja
@dataclass(frozen=True)
class ServiceContext:
    repository: Repository[Entity]

# services/example_service.py.jinja
def create_example(ctx: ServiceContext, name: str, value: int) -> Entity:
    entity = Entity(name=EntityName(text=name), value=EntityValue(amount=value))
    ctx.repository.save(entity)
    return entity
```

Why functions over classes:
- Pythonic — stateless operations are naturally functions
- Testable — pure functions with explicit dependencies
- Composable — no instance lifecycle to manage

Source: [services/context.py.jinja](../../src/%7B%7B%20package_name%20%7D%7D/services/context.py.jinja), [services/example_service.py.jinja](../../src/%7B%7B%20package_name%20%7D%7D/services/example_service.py.jinja)

## Protocol-Based DI

Interfaces use `typing.Protocol` for structural typing (PEP 544). Ports live in the domain layer because the domain defines what it needs from the outside world:

```python
# domain/repository.py.jinja
class Repository(Protocol, Generic[T]):
    def save(self, entity: T) -> None: ...
    def get(self, entity_id: str) -> Optional[T]: ...
    def list_all(self) -> list[T]: ...
    def delete(self, entity_id: str) -> bool: ...
```

Any class implementing these methods satisfies the protocol — no inheritance required.

Source: [domain/repository.py.jinja](../../src/%7B%7B%20package_name%20%7D%7D/domain/repository.py.jinja)

## Domain Model

- **Entities**: Identity-based (compared by `id`), mutable, contain business logic
- **Value objects**: Attribute-based (frozen dataclasses), immutable, self-validating
- **Errors**: Domain-specific error hierarchy (`DomainError` → `ValidationError`, `EntityNotFoundError`)
- **Ports**: Protocol interfaces the domain requires from the outside world (`Repository[T]`)

Source: [domain/entity.py.jinja](../../src/%7B%7B%20package_name%20%7D%7D/domain/entity.py.jinja), [domain/errors.py.jinja](../../src/%7B%7B%20package_name%20%7D%7D/domain/errors.py.jinja)

## Testing Strategy

- **Unit tests** with fakes (not mocks) — `FakeRepository` implements `Repository` protocol with test helpers
- **Integration tests** with real adapters — `InMemoryRepository` tested against protocol contract
- **Fixtures** in `conftest.py` provide `fake_repository` and `test_context`

```python
# tests/fakes/fake_repository.py.jinja
class FakeRepository(Generic[T]):
    # Implements Repository protocol + test helpers:
    # reset(), assert_saved(), get_saved_count(), etc.
```

Source: [tests/fakes/fake_repository.py.jinja](../../tests/fakes/fake_repository.py.jinja), [tests/conftest.py.jinja](../../tests/conftest.py.jinja)

## Technology Choices

| Tool | Purpose | Decision |
|------|---------|----------|
| uv | Package management | [ADR 001](../decisions/001-why-uv.md) |
| src layout | Project structure | [ADR 002](../decisions/002-src-layout.md) |
| typing.Protocol | Interfaces | [ADR 003](../decisions/003-protocol-di.md) |
| Pure functions | Services | [ADR 004](../decisions/004-functional-services.md) |
| Frozen dataclass | DI container | [ADR 005](../decisions/005-context-objects.md) |
| Typer | CLI framework | [ADR 006](../decisions/006-typer-cli.md) |
| ruff | Lint + format | [ADR 007](../decisions/007-ruff-tooling.md) |

## Sources

- [services/context.py.jinja](../../src/%7B%7B%20package_name%20%7D%7D/services/context.py.jinja) — ServiceContext definition
- [domain/repository.py.jinja](../../src/%7B%7B%20package_name%20%7D%7D/domain/repository.py.jinja) — Repository protocol
- [PEP 544](https://peps.python.org/pep-0544/) — Protocols: Structural subtyping
- [Architecture Patterns with Python](https://www.cosmicpython.com/book/) — Domain modeling patterns
