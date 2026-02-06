---
title: Copier Template Design
status: stable
owners: [template-team]
---

# Copier Template Design

This repository is a [Copier](https://copier.readthedocs.io/) template. Users run `copier copy` to generate projects from it.

## Template Variables

### User Inputs

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `project_name` | str | "My Python Project" | Human-readable name |
| `package_name` | str | (derived) | Python package name (snake_case) |
| `author_name` | str | "Your Name" | Author |
| `author_email` | str | "you@example.com" | Email (validated) |
| `python_version` | choice | "3.11" | Min Python: 3.10, 3.11, 3.12 |
| `include_example_code` | bool | true | Full working examples |
| `include_github_actions` | bool | true | CI/CD workflows |
| `include_dev_scripts` | bool | true | Setup/test scripts |
| `license` | choice | "MIT" | MIT, Apache-2.0, BSD-3-Clause, GPL-3.0, Proprietary |

### Computed

- `cli_command` — derived from `package_name`
- `module_path` — `<package_name>.cli.main`
- `current_year` — auto-generated

## What Stays Fixed

- Architecture pattern (functional services, protocols, context objects)
- Directory structure (`domain/`, `protocols/`, `services/`, `adapters/`, `cli/`)
- Tooling choices (uv, ruff, pytest, Typer)
- Testing strategy (fakes over mocks)

## Conditional Content

Files use `{% if include_example_code %}` to include or omit working examples. When disabled, files contain skeleton code with comments explaining the pattern.

Conditional features:
- `include_github_actions` — `.github/workflows/` lint and test workflows
- `include_dev_scripts` — `scripts/setup.sh` and `scripts/test.sh`

## Post-Generation

Copier runs automatically after generation:
1. `uv sync` — install dependencies
2. `git init` — initialize repository
3. `git add .` && `git commit` — initial commit

## Template Updates

Generated projects can pull template updates:

```bash
copier update
```

Protected files (`_skip_if_exists`): `.gitignore`, `README.md`

## Sources

- [copier.yml](../../copier.yml) — template configuration
- [Copier documentation](https://copier.readthedocs.io/)
- [Dependency pattern](dependency-pattern.md) — template-as-dependency approach
