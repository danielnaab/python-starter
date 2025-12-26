---
title: Template Variable Mapping
status: draft
owners: [template-team]
---

# Template Variable Mapping

Complete reference of Copier template variables and their usage.

## Core Variables

### project_name
**Type**: `str`
**Description**: Human-readable project name
**Default**: `"My Python Project"`
**Example**: `"User Management System"`

**Used in**:
- `README.md`: Project title
- `docs/README.md`: Documentation title
- `pyproject.toml`: Not directly (use `package_name`)

### package_name
**Type**: `str`
**Description**: Python package name (snake_case)
**Default**: Auto-generated from `project_name`
**Example**: `"user_management_system"`
**Validation**: Must be valid Python identifier

**Used in**:
- `pyproject.toml`: `[project] name`
- `src/{{ package_name }}/`: Package directory
- All imports: `from {{ package_name }} import ...`
- Entry points: `{{ package_name }}.cli.main:app`

**Transform**:
```yaml
package_name:
  default: "{{ project_name.lower().replace(' ', '_').replace('-', '_') }}"
```

### project_description
**Type**: `str`
**Description**: Short project description
**Default**: `"A Python project built with modern best practices"`
**Example**: `"RESTful API for user management"`

**Used in**:
- `pyproject.toml`: `description` field
- `README.md`: Project description
- `docs/README.md`: Overview section

### author_name
**Type**: `str`
**Description**: Author's full name
**Default**: `"Your Name"`
**Example**: `"Jane Smith"`

**Used in**:
- `pyproject.toml`: `authors` field
- LICENSE file
- Documentation copyright notices

### author_email
**Type**: `str`
**Description**: Author's email address
**Default**: `"you@example.com"`
**Example**: `"jane.smith@example.com"`

**Used in**:
- `pyproject.toml`: `authors` field
- Git configuration (optional)

### python_version
**Type**: `str`
**Description**: Minimum Python version
**Default**: `"3.11"`
**Choices**: `["3.10", "3.11", "3.12"]`

**Used in**:
- `.python-version`: Version for uv
- `pyproject.toml`: `requires-python` field
- `tool.ruff`: `target-version`
- GitHub Actions: Python version matrix

## Optional Feature Flags

### include_example_code
**Type**: `bool`
**Description**: Include example domain code
**Default**: `true`

**When true**:
- Include `src/{{ package_name }}/domain/entities.py` with `Entity` class
- Include example service functions
- Include example CLI commands
- Include example tests

**When false**:
- Create empty package structure
- Include only `__init__.py` files
- No example code or tests

**Usage**:
```python
{% if include_example_code %}
# Example code here
{% endif %}
```

### include_github_actions
**Type**: `bool`
**Description**: Include GitHub Actions workflows
**Default**: `true`

**When true**:
- Include `.github/workflows/test.yml`
- Include `.github/workflows/lint.yml`

**When false**:
- No `.github/` directory

### include_dev_scripts
**Type**: `bool`
**Description**: Include development scripts
**Default**: `true`

**When true**:
- Include `scripts/setup.sh`
- Include `scripts/test.sh`

### license
**Type**: `str`
**Description**: Project license
**Default**: `"MIT"`
**Choices**: `["MIT", "Apache-2.0", "BSD-3-Clause", "GPL-3.0", "Proprietary"]`

**Used in**:
- `LICENSE` file (conditionally generated)
- `pyproject.toml`: `license` field

## File Mapping

### Direct Substitution

Files where variables are directly substituted:

| File | Variables Used |
|------|----------------|
| `pyproject.toml` | `package_name`, `project_description`, `author_name`, `author_email`, `python_version` |
| `README.md` | `project_name`, `project_description`, `package_name` |
| `.python-version` | `python_version` |
| `docs/README.md` | `project_name`, `package_name` |

### Directory Name Substitution

| Original | Template | Example Output |
|----------|----------|----------------|
| `src/starter/` | `src/{{ package_name }}/` | `src/my_project/` |

### Import Substitution

All Python imports are updated:

```python
# Template
from {{ package_name }}.domain.entities import Entity
from {{ package_name }}.services.context import ServiceContext

# Generated (package_name="my_project")
from my_project.domain.entities import Entity
from my_project.services.context import ServiceContext
```

### Entry Point Substitution

```toml
# Template
[project.scripts]
{{ package_name }}-cli = "{{ package_name }}.cli.main:app"

# Generated (package_name="my_project")
[project.scripts]
my_project-cli = "my_project.cli.main:app"
```

## Conditional Files

### GitHub Actions

```yaml
# copier.yml
_exclude:
  - "{% if not include_github_actions %}.github{% endif %}"
```

Excludes entire `.github/` directory if `include_github_actions` is `false`.

### Example Code

```python
# src/{{ package_name }}/domain/entities.py.jinja
{% if include_example_code %}
# Full example code
{% else %}
# Empty file with just docstring
"""Domain entities."""
{% endif %}
```

### License File

```yaml
# copier.yml
_exclude:
  - "{% if license == 'Proprietary' %}LICENSE{% endif %}"
```

## Advanced Variables

### Computed Variables

Variables computed from other variables:

```yaml
# copier.yml
cli_command:
  type: str
  default: "{{ package_name }}-cli"
  when: false  # Don't ask user, compute automatically

module_path:
  type: str
  default: "{{ package_name }}.cli.main"
  when: false
```

### Environment Variables

Access environment variables:

```yaml
git_user_name:
  type: str
  default: "{{ 'git config user.name' | shell }}"

git_user_email:
  type: str
  default: "{{ 'git config user.email' | shell }}"
```

## Variable Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Human-readable | `snake_case` with descriptive name | `project_name` |
| Boolean flags | `include_*` or `enable_*` | `include_example_code` |
| Version numbers | `*_version` | `python_version` |
| Computed values | Hidden from user (`when: false`) | `cli_command` |

## Validation

### Built-in Validation

```yaml
package_name:
  type: str
  validator: "{% if not package_name.isidentifier() %}Must be valid Python identifier{% endif %}"

python_version:
  type: str
  choices: ["3.10", "3.11", "3.12"]  # Validates automatically

author_email:
  type: str
  validator: "{% if '@' not in author_email %}Must be valid email{% endif %}"
```

### Custom Validation

```yaml
package_name:
  type: str
  validator: >
    {% if package_name.startswith('_') %}
    Cannot start with underscore
    {% elif package_name in ['test', 'tests'] %}
    Reserved package name
    {% elif not package_name.islower() %}
    Must be lowercase
    {% endif %}
```

## Testing Variables

### Test with Different Values

```bash
# Test minimal configuration
copier copy \
  --data project_name="Test Project" \
  --data package_name="test_project" \
  --data include_example_code=false \
  . test-minimal

# Test full configuration
copier copy \
  --data project_name="Full Test" \
  --data include_example_code=true \
  --data include_github_actions=true \
  . test-full
```

## Related Documentation

- [Template Design](template-design.md) - Overall design
- [Customization Points](customization-points.md) - What users can customize
- [Copier Documentation](https://copier.readthedocs.io/) - Official docs

## Sources

- [Copier: Template Variables](https://copier.readthedocs.io/en/stable/creating/#the-copieryml-file)
- [Jinja2 Templates](https://jinja.palletsprojects.com/en/3.1.x/templates/)
