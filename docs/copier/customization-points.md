---
title: Template Customization Points
status: draft
owners: [template-team]
---

# Template Customization Points

Guide to what users can customize when generating projects from this Copier template.

## Philosophy

This template balances **fixed architecture** with **flexible customization**.

**Fixed** (Architecture - Cannot Change):
- Functional service layer pattern
- Protocol-based dependency injection
- Context object pattern
- src/ layout
- Testing strategy (fakes over mocks)
- Tool choices (uv, ruff, pytest, Typer)

**Flexible** (Project Details - User Customizes):
- Project name and package name
- Author information
- Python version (within supported range)
- License
- Optional features (example code, CI, etc.)

**Rationale**: Architecture decisions provide value. Project details must match user needs.

## User Inputs (Interactive)

### Required Inputs

Users answer these questions during generation:

#### 1. Project Name
**Prompt**: "Project name (human-readable)"
**Example**: `"User Management API"`
**Used for**: README title, documentation headers

#### 2. Package Name
**Prompt**: "Python package name (snake_case)"
**Default**: Auto-generated from project name
**Example**: `"user_management_api"`
**Validation**: Must be valid Python identifier
**Used for**: Directory name (`src/user_management_api/`), imports, CLI entry point

#### 3. Project Description
**Prompt**: "Short project description"
**Example**: `"RESTful API for managing user accounts"`
**Used for**: README, pyproject.toml description field

#### 4. Author Name
**Prompt**: "Author name"
**Default**: From git config or "Your Name"
**Example**: `"Jane Smith"`
**Used for**: pyproject.toml authors, LICENSE file

#### 5. Author Email
**Prompt**: "Author email"
**Default**: From git config or "you@example.com"
**Example**: `"jane.smith@example.com"`
**Used for**: pyproject.toml authors

#### 6. Python Version
**Prompt**: "Minimum Python version"
**Choices**: `["3.10", "3.11", "3.12"]`
**Default**: `"3.11"`
**Used for**: `.python-version`, pyproject.toml `requires-python`, ruff `target-version`

### Optional Features

#### 7. Include Example Code
**Prompt**: "Include example domain code?"
**Type**: Boolean
**Default**: `true`

**When true**:
- Generates complete example domain (Entity, value objects)
- Example service functions (create, list, get)
- Example CLI commands
- Full unit and integration tests

**When false**:
- Generates empty package structure
- Only `__init__.py` files
- Minimal tests (structure only)
- Users start from blank slate

**Use case**:
- `true` - Learn from examples, adapt to your domain
- `false` - Experienced users who want minimal boilerplate

#### 8. Include GitHub Actions
**Prompt**: "Include GitHub Actions workflows?"
**Type**: Boolean
**Default**: `true`

**When true**:
- `.github/workflows/test.yml` - Run tests on push/PR
- `.github/workflows/lint.yml` - Run ruff checks

**When false**:
- No `.github/` directory
- Users configure their own CI

**Use case**:
- `true` - Using GitHub, want ready CI/CD
- `false` - Using GitLab/other, or custom CI

#### 9. Include Development Scripts
**Prompt**: "Include development scripts?"
**Type**: Boolean
**Default**: `true`

**When true**:
- `scripts/setup.sh` - Project setup script
- `scripts/test.sh` - Test runner with coverage

**When false**:
- No `scripts/` directory

**Use case**:
- `true` - Want convenient scripts
- `false` - Prefer manual commands or custom scripts

#### 10. License
**Prompt**: "Project license"
**Choices**: `["MIT", "Apache-2.0", "BSD-3-Clause", "GPL-3.0", "Proprietary"]`
**Default**: `"MIT"`

**When not "Proprietary"**:
- Generates LICENSE file with selected license text
- Adds license to pyproject.toml

**When "Proprietary"**:
- No LICENSE file
- pyproject.toml license field omitted

## What Stays Fixed

These cannot be customized during generation:

### Architecture Patterns

**Functional Service Layer**:
- Services as pure functions (not classes)
- Context objects for dependency injection
- All generated code follows this pattern

**Protocol-Based Interfaces**:
- All dependencies defined with `typing.Protocol`
- Structural typing throughout
- No ABC or inheritance-based interfaces

**Context Objects**:
- `ServiceContext` as frozen dataclass
- All adapters injected via context
- Pattern enforced in all service functions

**src/ Layout**:
- Package always in `src/` directory
- Tests in top-level `tests/` directory
- Cannot generate flat layout

### Tooling

**Package Management**: uv only
- No pip-tools, Poetry, or PDM option
- `pyproject.toml` configured for uv
- Lockfile strategy: uv.lock

**Linting/Formatting**: ruff only
- No Black + Flake8 option
- Configuration in pyproject.toml
- Line length: 100 (can edit after generation)

**Testing**: pytest only
- No unittest or nose option
- Test structure follows pytest conventions

**CLI**: Typer only
- No Click or argparse option
- Commands use Typer decorators

### Documentation Structure

**meta-knowledge-base conventions**:
- Handbook strategy (dual entrypoints)
- docs/knowledge-base.yaml
- Architecture docs, ADRs, guides, reference
- YAML frontmatter required

**File organization**:
- `docs/architecture/` - Architecture documentation
- `docs/decisions/` - ADRs
- `docs/guides/` - User guides
- `docs/reference/` - Technical reference
- `docs/copier/` - Template documentation

### Testing Strategy

**Fakes over mocks**:
- `tests/fakes/` with in-memory implementations
- No mock/patch in generated tests
- Fixture-based test organization

## Customization After Generation

Users can modify these post-generation:

### Easy Customizations

**Add dependencies**:
```bash
uv add requests  # Add runtime dependency
uv add --dev mypy  # Add dev dependency
```

**Change Python version**:
- Edit `.python-version`
- Edit `pyproject.toml` `requires-python`
- Edit `pyproject.toml` `tool.ruff.target-version`

**Modify ruff settings**:
```toml
[tool.ruff]
line-length = 120  # Change from default 100
```

**Add pytest plugins**:
```bash
uv add --dev pytest-asyncio  # For async tests
uv add --dev pytest-xdist  # For parallel tests
```

### Domain Customization

**Replace example domain**:
1. Delete `src/{package}/domain/entities.py`
2. Create your own domain models
3. Update service functions
4. Update tests

**Add new services**:
1. Create `src/{package}/services/your_service.py`
2. Add service functions following pattern
3. Add tests in `tests/unit/test_your_service.py`

**Add new adapters**:
1. Define protocol in `src/{package}/protocols/`
2. Implement in `src/{package}/adapters/`
3. Add to `ServiceContext`
4. Create fake in `tests/fakes/`

**Add CLI commands**:
1. Create `src/{package}/cli/commands/your_commands.py`
2. Import in `cli/main.py`
3. Follow Typer patterns

### Advanced Customizations

**Multiple contexts**:
- Split `ServiceContext` into domain-specific contexts
- Useful for larger applications
- See: [Context Objects Reference](../reference/context-objects.md#multiple-contexts)

**Add type checking**:
```bash
uv add --dev mypy
# Add mypy configuration to pyproject.toml
```

**Add pre-commit hooks**:
```bash
uv add --dev pre-commit
# Create .pre-commit-config.yaml
uv run pre-commit install
```

**Switch to different adapters**:
- Replace in-memory repository with database
- Implement same Protocol interface
- Swap in `cli/main.py` `get_context()`

## Non-Customizable (By Design)

These are intentionally fixed:

**Architecture patterns**: Cannot switch to class-based services or different DI approach without rewriting

**Directory structure**: src/ layout is enforced

**Tool choices**: Switching from uv/ruff/pytest requires significant configuration changes

**Documentation approach**: Removing docs/ structure loses meta-knowledge-base benefits

**Rationale**: These decisions provide the template's value. Changing them means you don't need this template.

## Template Updates

### Updating Generated Projects

When template receives updates:

```bash
# Update project from latest template
copier update

# Review changes
git diff

# Accept or reject changes
git add .
git commit -m "chore: update from template"
```

**What gets updated**:
- Documentation improvements
- Tool configuration updates
- New best practices

**What stays yours**:
- Domain code
- Custom services
- Your tests
- Project-specific configuration

### Opting Out of Updates

Add to `.copier-answers.yml`:
```yaml
_exclude:
  - "docs/**"  # Don't update documentation
  - "pyproject.toml"  # Don't update configuration
```

## Validation Rules

Template validates inputs:

**Package name**:
```python
# Must be valid Python identifier
package_name.isidentifier()  # True

# Examples:
"my_package"  # ✅ Valid
"my-package"  # ❌ Invalid (hyphens)
"123_package"  # ❌ Invalid (starts with digit)
"package"  # ✅ Valid
```

**Author email**:
```python
# Must contain @
"@" in author_email

# Examples:
"user@example.com"  # ✅ Valid
"user"  # ❌ Invalid
```

**Python version**:
- Must be one of: `["3.10", "3.11", "3.12"]`
- Cannot specify arbitrary versions

## Configuration Presets (Future)

Future versions may support presets:

```bash
# Minimal preset
copier copy gh:org/python-starter my-project \
  --data preset=minimal

# API preset (with FastAPI, SQLAlchemy)
copier copy gh:org/python-starter my-project \
  --data preset=api

# CLI preset (focus on CLI tools)
copier copy gh:org/python-starter my-project \
  --data preset=cli
```

**Not yet implemented** - currently all configuration is explicit.

## Examples

### Example 1: Minimal Project

**Inputs**:
- project_name: "My Tool"
- package_name: "my_tool"
- include_example_code: `false`
- include_github_actions: `false`
- include_dev_scripts: `false`

**Result**: Bare skeleton with architecture but no examples

**Use case**: Experienced user who knows the patterns, wants minimal starting point

### Example 2: Full Featured

**Inputs**:
- project_name: "User Management API"
- package_name: "user_management_api"
- python_version: "3.11"
- include_example_code: `true`
- include_github_actions: `true`
- include_dev_scripts: `true`
- license: "MIT"

**Result**: Complete example with CI, scripts, full documentation

**Use case**: Learning the patterns, want working example to adapt

### Example 3: Library (No CLI)

**Post-generation customization**:
1. Generate with defaults
2. Delete `src/{package}/cli/`
3. Remove CLI entry point from pyproject.toml
4. Remove CLI documentation from docs/

**Use case**: Building a library, not an application

## Related Documentation

- [Template Design](template-design.md) - Overall template strategy
- [Variable Mapping](variable-mapping.md) - All template variables
- [Getting Started](../guides/getting-started.md) - Using generated project
- [Development Workflow](../guides/development-workflow.md) - Customizing workflow

## Sources

- [Copier: Template Configuration](https://copier.readthedocs.io/en/stable/creating/)
- [Copier: Updating Projects](https://copier.readthedocs.io/en/stable/updating/)
- [Jinja2: Template Designer Documentation](https://jinja.palletsprojects.com/en/3.1.x/templates/)
