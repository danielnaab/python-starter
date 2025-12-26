---
title: Python Starter Template - Agent Reference
status: stable
owners: [template-team]
---

# Python Starter Template - Agent Reference

Structured technical reference for AI agents working with this Python starter template.

## Project Metadata

- **Type**: Domain-agnostic Python starter template
- **Python**: 3.11+
- **Package Manager**: uv
- **Linter/Formatter**: ruff
- **Testing**: pytest
- **CLI Framework**: Typer
- **Architecture**: Functional service layer with context objects

## Configuration

See [knowledge-base.yaml](knowledge-base.yaml) for complete configuration including:
- Entrypoints (human/agent)
- Documentation paths
- Canonical sources (src/, pyproject.toml, tests/)
- Write permissions
- Lifecycle statuses

## Critical Patterns

### 1. Functional Service Layer

Services are **pure functions**, not classes.

**Pattern**:
```python
from dataclasses import dataclass
from typing import Protocol

@dataclass(frozen=True)
class ServiceContext:
    """Immutable context containing all dependencies."""
    repository: Repository[Entity]
    config: Config
    logger: Logger

def my_use_case(ctx: ServiceContext, param: str, value: int) -> Result:
    """Service function - pure, testable, composable.

    Args:
        ctx: Service context with all dependencies
        param: Business parameter
        value: Business parameter

    Returns:
        Result of the operation

    Raises:
        ValidationError: If business rules violated
    """
    # Validate inputs
    if value < 0:
        raise ValueError("Value must be non-negative")

    # Domain logic
    entity = Entity(name=param, value=value)

    # Adapter calls via context
    ctx.repository.save(entity)
    ctx.logger.info(f"Created: {entity.id}")

    return entity
```

**Location**: `src/starter/services/`

**Key characteristics**:
- First parameter: `ServiceContext`
- Pure functions (no hidden state)
- Explicit dependencies via context
- Return results or raise exceptions

**Documentation**: [architecture/functional-services.md](architecture/functional-services.md)

**Decision rationale**: [decisions/004-functional-services.md](decisions/004-functional-services.md)

### 2. Context Object Pattern

Dependencies injected via **frozen dataclass**.

**Pattern**:
```python
@dataclass(frozen=True)
class ServiceContext:
    """Service layer dependency container.

    Immutable to ensure thread safety and predictable behavior.
    All fields use Protocol types for flexibility.
    """
    repository: Repository[Entity]
    external_api: ExternalAPI
    config: Config
    logger: Logger
```

**Benefits**:
- Immutable (`frozen=True`)
- Type-safe (all fields type-hinted)
- Explicit (no hidden dependencies)
- Easy to test (partial contexts with fakes)

**Location**: `src/starter/services/context.py`

**Testing pattern**:
```python
def test_context() -> ServiceContext:
    """Build test context with fakes."""
    return ServiceContext(
        repository=FakeRepository(),
        external_api=FakeExternalAPI(),
        config=Config(debug=True),
        logger=NullLogger(),
    )
```

**Documentation**: [architecture/dependency-injection.md](architecture/dependency-injection.md)

**Decision rationale**: [decisions/005-context-objects.md](decisions/005-context-objects.md)

### 3. Protocol-Based Interfaces

Use `typing.Protocol` for structural typing.

**Pattern**:
```python
from typing import Protocol, Generic, TypeVar, Optional

T = TypeVar("T")

class Repository(Protocol, Generic[T]):
    """Repository interface using structural typing.

    Any class implementing these methods satisfies this protocol,
    no inheritance required.
    """

    def save(self, entity: T) -> None:
        """Persist entity."""
        ...

    def get(self, entity_id: str) -> Optional[T]:
        """Retrieve entity by ID."""
        ...

    def list_all(self) -> list[T]:
        """List all entities."""
        ...

    def delete(self, entity_id: str) -> bool:
        """Delete entity by ID. Returns True if deleted."""
        ...
```

**Location**: `src/starter/protocols/`

**Benefits**:
- Structural subtyping (duck typing with type safety)
- No inheritance required
- Easy to create fakes for testing
- Modern Python idiom (PEP 544)

**Implementation example**:
```python
class InMemoryRepository(Generic[T]):
    """In-memory repository implementation.

    Satisfies Repository[T] protocol without inheriting.
    """

    def __init__(self) -> None:
        self._storage: dict[str, T] = {}

    def save(self, entity: T) -> None:
        self._storage[entity.id] = entity

    def get(self, entity_id: str) -> Optional[T]:
        return self._storage.get(entity_id)

    def list_all(self) -> list[T]:
        return list(self._storage.values())

    def delete(self, entity_id: str) -> bool:
        if entity_id in self._storage:
            del self._storage[entity_id]
            return True
        return False
```

**Documentation**: [reference/protocols.md](reference/protocols.md)

**Decision rationale**: [decisions/003-protocol-di.md](decisions/003-protocol-di.md)

### 4. Testing Strategy

**Philosophy**: Unit tests with fakes, integration tests with real adapters.

**Unit testing pattern**:
```python
def test_create_example(test_context):
    """Test service function with fake context."""
    # Arrange
    fake_repo = FakeRepository()
    ctx = ServiceContext(repository=fake_repo, config=test_config(), logger=test_logger())

    # Act
    entity = create_example(ctx, name="test", value=100)

    # Assert
    assert entity.name == "test"
    assert entity.value == 100
    assert fake_repo.get(entity.id) == entity
```

**Fakes over mocks**:
- Create in-memory implementations (e.g., `FakeRepository`)
- Implement full protocol interface
- Provide test helpers (reset, assert_saved, etc.)
- Store in `tests/fakes/`

**Location**: `tests/`
- `tests/unit/` - Unit tests
- `tests/integration/` - Integration tests
- `tests/fakes/` - Fake implementations
- `tests/conftest.py` - Pytest fixtures

**Documentation**: [architecture/testing-strategy.md](architecture/testing-strategy.md)

## File Organization

```
src/starter/
├── __init__.py           # Package exports
├── __main__.py           # Entry point for `python -m starter`
│
├── domain/               # Pure domain logic
│   ├── __init__.py
│   ├── entities.py       # Domain entities (dataclasses)
│   └── value_objects.py  # Value objects (immutable)
│
├── services/             # Service layer (FUNCTIONS)
│   ├── __init__.py
│   ├── context.py        # ServiceContext definition
│   └── example_service.py # Service functions
│
├── adapters/             # External system implementations
│   ├── __init__.py
│   ├── repository.py     # Repository implementations
│   └── external_api.py   # External API adapters
│
├── protocols/            # Protocol interface definitions
│   ├── __init__.py
│   ├── repository.py     # Repository protocol
│   └── external.py       # External service protocols
│
└── cli/                  # CLI layer (Typer)
    ├── __init__.py
    ├── main.py           # Typer app + context wiring
    └── commands/         # Command groups
        ├── __init__.py
        └── example.py    # Example commands
```

**Detailed reference**: [reference/project-structure.md](reference/project-structure.md)

## Configuration Files

### pyproject.toml

Central configuration for:
- Project metadata (name, version, dependencies)
- uv configuration (`[tool.uv]`)
- ruff linting/formatting (`[tool.ruff]`)
- pytest testing (`[tool.pytest.ini_options]`)
- Coverage settings (`[tool.coverage]`)
- Entry points (`[project.scripts]`)

**Reference**: [reference/configuration.md](reference/configuration.md)

### .python-version

Python version specification (3.11) for uv.

### uv.lock

Auto-generated lockfile for reproducible installs. Do not edit manually.

## Common Operations

### Add a new service function

1. Define function in `src/starter/services/<domain>_services.py`
2. Accept `ServiceContext` as first parameter
3. Add type hints for all parameters and return value
4. Write comprehensive docstring (Args, Returns, Raises)
5. Add unit test in `tests/unit/test_services.py` with fake context
6. Wire to CLI in `src/starter/cli/commands/` if needed

**Example**:
```python
# src/starter/services/user_services.py
def create_user(ctx: ServiceContext, email: str, name: str) -> User:
    """Create new user.

    Args:
        ctx: Service context
        email: User email address
        name: User full name

    Returns:
        Created user entity

    Raises:
        ValidationError: If email invalid
    """
    user = User(email=email, name=name)
    ctx.repository.save(user)
    return user

# tests/unit/test_services.py
def test_create_user(test_context):
    user = create_user(test_context, email="test@example.com", name="Test User")
    assert user.email == "test@example.com"
    assert test_context.repository.get(user.id) == user
```

**Guide**: [guides/adding-features.md](guides/adding-features.md)

### Add a new adapter

1. Define protocol in `src/starter/protocols/<name>.py`
2. Implement adapter in `src/starter/adapters/<name>.py`
3. Add protocol field to `ServiceContext` in `src/starter/services/context.py`
4. Create fake implementation in `tests/fakes/fake_<name>.py`
5. Update `tests/conftest.py` fixture to include fake
6. Add integration test in `tests/integration/test_adapters.py`

**Example**:
```python
# src/starter/protocols/email.py
class EmailService(Protocol):
    def send(self, to: str, subject: str, body: str) -> bool: ...

# src/starter/adapters/email.py
class SMTPEmailService:
    def send(self, to: str, subject: str, body: str) -> bool:
        # Implementation
        return True

# src/starter/services/context.py
@dataclass(frozen=True)
class ServiceContext:
    repository: Repository[Entity]
    email_service: EmailService  # Add new protocol
    config: Config

# tests/fakes/fake_email.py
class FakeEmailService:
    def __init__(self):
        self.sent_emails = []

    def send(self, to: str, subject: str, body: str) -> bool:
        self.sent_emails.append((to, subject, body))
        return True
```

### Run tests

```bash
# All tests
uv run pytest

# With coverage
uv run pytest --cov=starter --cov-report=term-missing

# Specific file
uv run pytest tests/unit/test_services.py

# Verbose output
uv run pytest -v

# Stop on first failure
uv run pytest -x
```

### Run linting and formatting

```bash
# Check code
uv run ruff check .

# Format code
uv run ruff format .

# Both
uv run ruff check . && uv run ruff format .
```

## Dependency Management

```bash
# Install dependencies
uv sync

# Add dependency
uv add package-name

# Add dev dependency
uv add --dev package-name

# Update dependencies
uv lock --upgrade

# Remove dependency
uv remove package-name
```

## CLI Development

### Wiring context to CLI

**Pattern** (`src/starter/cli/main.py`):
```python
import typer
from starter.services.context import ServiceContext
from starter.adapters.repository import InMemoryRepository
from starter.services import example_service

app = typer.Typer()

def get_context() -> ServiceContext:
    """Build production context with real adapters."""
    return ServiceContext(
        repository=InMemoryRepository(),
        config=Config.from_env(),
        logger=setup_logger(),
    )

@app.command()
def create(name: str, value: int) -> None:
    """Create example entity."""
    ctx = get_context()
    entity = example_service.create_example(ctx, name=name, value=value)
    typer.echo(f"Created: {entity}")

if __name__ == "__main__":
    app()
```

### Running CLI

```bash
# Via entry point
uv run starter-cli --help
uv run starter-cli create "test" 100

# Via module
uv run python -m starter --help
```

## Authority and Sources

Per [knowledge-base.yaml](knowledge-base.yaml):

**Canonical sources** (source of truth):
- `src/` - Source code
- `pyproject.toml` - Project configuration
- `tests/` - Tests document behavior

**Interpretive sources** (explain canonical):
- `docs/` - Documentation

When code and docs conflict, **code is correct**. Documentation should be updated.

## Documentation Conventions

1. **Frontmatter**: All `.md` files require YAML frontmatter with `title`, `status`, `owners`
2. **Linking**: Use relative Markdown links `[text](path)`, not backticks
3. **Code references**: Link with line numbers: `src/file.py:123`
4. **Sources**: Include `## Sources` section in architecture and decision docs
5. **Lifecycle**: Use statuses: draft, working, stable, deprecated

## Related Documentation

### Architecture
- [Overview](architecture/overview.md) - High-level architecture
- [Functional Services](architecture/functional-services.md) - Service pattern
- [Dependency Injection](architecture/dependency-injection.md) - DI approach
- [Domain Model](architecture/domain-model.md) - Entities and value objects
- [Testing Strategy](architecture/testing-strategy.md) - Testing philosophy

### Decisions
- [001: Why uv?](decisions/001-why-uv.md)
- [002: src Layout](decisions/002-src-layout.md)
- [003: Protocol-based DI](decisions/003-protocol-di.md)
- [004: Functional Services](decisions/004-functional-services.md)
- [005: Context Objects](decisions/005-context-objects.md)
- [006: Typer CLI](decisions/006-typer-cli.md)
- [007: Ruff Tooling](decisions/007-ruff-tooling.md)

### Guides
- [Getting Started](guides/getting-started.md)
- [Development Workflow](guides/development-workflow.md)
- [Adding Features](guides/adding-features.md)
- [Testing Guide](guides/testing-guide.md)
- [CLI Usage](guides/cli-usage.md)

### Reference
- [Project Structure](reference/project-structure.md)
- [Configuration](reference/configuration.md)
- [Tooling](reference/tooling.md)
- [Protocols](reference/protocols.md)
- [Context Objects](reference/context-objects.md)

## Sources

- [knowledge-base.yaml](knowledge-base.yaml) - Configuration metadata
- [pyproject.toml](../pyproject.toml) - Project configuration
- [src/](../src/) - Source code (canonical)
- [tests/](../tests/) - Test suite (canonical)
