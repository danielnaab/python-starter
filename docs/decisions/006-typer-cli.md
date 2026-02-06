---
title: "ADR 006: Typer for CLI Framework"
status: stable
owners: [architecture-team]
---

# ADR 006: Typer for CLI Framework

## Status

Accepted.

## Context

The template includes a CLI interface. Options: argparse (stdlib), Click, Typer, Fire.

## Decision

Use **Typer** for the CLI layer.

## Rationale

- **Type-hint driven**: Parameters defined by function signatures, not decorators
- **Auto-generated help**: From type hints and docstrings
- **Built on Click**: Reliable, battle-tested foundation
- **Developer experience**: IDE autocomplete, type checking, minimal boilerplate

## CLI Architecture

The CLI is a thin adapter over services:

```python
# cli/main.py.jinja
app = typer.Typer()

def get_context() -> ServiceContext:
    return ServiceContext(repository=InMemoryRepository[Entity]())

@app.command()
def create(name: str, value: int) -> None:
    ctx = get_context()
    entity = create_example(ctx, name=name, value=value)
    typer.echo(f"Created: {entity}")
```

## Usage

```bash
uv run {{ cli_command }} --help
uv run {{ cli_command }} create "test" 100
uv run python -m {{ package_name }} --help
```

## Alternatives Considered

- **argparse**: Verbose, imperative, no type hints
- **Click**: More decorator boilerplate than Typer
- **Fire**: Auto-generates CLI from any object — too magical, limited customization

## Sources

- [Typer documentation](https://typer.tiangolo.com/)
- [cli/main.py.jinja](../../src/%7B%7B%20package_name%20%7D%7D/cli/main.py.jinja) — CLI entry point
