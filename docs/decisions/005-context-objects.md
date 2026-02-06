---
title: "ADR 005: Context Objects for Dependency Injection"
status: stable
owners: [architecture-team]
---

# ADR 005: Context Objects for Dependency Injection

## Status

Accepted.

## Context

Service functions need access to dependencies (repositories, external services). We need a mechanism to pass these without long parameter lists or global state.

## Decision

Use **frozen dataclasses** as context objects containing all dependencies typed as Protocols.

## Rationale

- **Explicit**: One object shows all dependencies at a glance
- **Immutable**: `frozen=True` prevents accidental mutation
- **Type-safe**: IDE autocomplete and type checker support
- **Testable**: Construct test contexts with fake implementations
- **No framework**: Plain dataclass, no registration or wiring magic

## Actual Implementation

```python
# services/context.py.jinja
@dataclass(frozen=True)
class ServiceContext:
    repository: Repository[Entity]
```

The template starts with one dependency. Add more Protocol-typed fields as your application grows.

## Testing Pattern

```python
# tests/conftest.py.jinja
@pytest.fixture
def test_context(fake_repository) -> ServiceContext:
    return ServiceContext(repository=fake_repository)
```

## Alternatives Considered

- **Individual parameters**: Repetitive signatures, hard to refactor when adding dependencies
- **Global config / singletons**: Hidden state, hard to test
- **DI framework containers**: Magic wiring, learning curve, overkill for most projects
- **Dict / NamedTuple**: No type safety or IDE support

## Sources

- [services/context.py.jinja](../../src/%7B%7B%20package_name%20%7D%7D/services/context.py.jinja) — ServiceContext definition
- [tests/conftest.py.jinja](../../tests/conftest.py.jinja) — test context fixture
