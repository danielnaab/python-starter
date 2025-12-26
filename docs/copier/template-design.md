---
title: Copier Template Design
status: draft
owners: [template-team]
---

# Copier Template Design

Strategy for converting this repository into a Copier template for project generation.

## Overview

**Current**: Fully functional Python starter project with documentation

**Future**: Copier template that generates customized projects

**Tool**: [Copier](https://copier.readthedocs.io/) - template-based project generator

## What is Copier?

Copier is a library and CLI for rendering project templates:

```bash
# Generate project from template
copier copy gh:org/python-starter my-project

# User answers questions
# Project is generated with customized values
```

## Template Philosophy

### Fixed Elements (Architecture)

Keep these **unchanged** across all generated projects:

- **Architecture patterns**: Functional services, protocol-based DI, context objects
- **Directory structure**: src/ layout, test organization
- **Documentation structure**: Handbook approach, ADRs, guides
- **Tooling**: uv, ruff, pytest, Typer
- **Best practices**: Code organization, testing strategies

**Rationale**: These represent architectural decisions that provide value.

### Variable Elements (Project-Specific)

Make these **customizable** via template variables:

- **Project name**: "Python Starter" → user's project name
- **Package name**: `starter` → user's package name
- **Author info**: Name, email
- **License**: MIT, Apache, etc.
- **Description**: Project description
- **Optional features**: Example domain code, GitHub Actions, etc.

**Rationale**: These are project-specific details that must change.

## Conversion Process

### Step 1: Add copier.yml

Create `copier.yml` in repository root:

```yaml
# Template metadata
_min_copier_version: "9.0.0"

# Template questions
project_name:
  type: str
  help: "Project name (human-readable)"
  default: "My Python Project"

package_name:
  type: str
  help: "Python package name (snake_case)"
  default: "{{ project_name.lower().replace(' ', '_').replace('-', '_') }}"
  validator: "{% if not package_name.isidentifier() %}Package name must be valid Python identifier{% endif %}"

author_name:
  type: str
  help: "Author name"
  default: "Your Name"

author_email:
  type: str
  help: "Author email"
  default: "you@example.com"

python_version:
  type: str
  help: "Minimum Python version"
  default: "3.11"
  choices:
    - "3.10"
    - "3.11"
    - "3.12"

include_example_code:
  type: bool
  help: "Include example domain code?"
  default: true

include_github_actions:
  type: bool
  help: "Include GitHub Actions workflows?"
  default: true

license:
  type: str
  help: "License"
  default: "MIT"
  choices:
    - "MIT"
    - "Apache-2.0"
    - "BSD-3-Clause"
    - "GPL-3.0"

# Files to exclude from template
_exclude:
  - "copier.yml"
  - ".git"
  - ".venv"
  - "**/__pycache__"
  - "*.pyc"
  - ".pytest_cache"
  - "uv.lock"

# Tasks to run after generation
_tasks:
  - "uv sync"
  - "git init"
  - "git add ."
  - 'git commit -m "Initial commit from template"'
```

### Step 2: Add Jinja2 Templates

Replace static values with Jinja2 templates:

#### pyproject.toml

```toml
[project]
name = "{{ package_name }}"
version = "0.1.0"
description = "{{ project_description }}"
authors = [
    {name = "{{ author_name }}", email = "{{ author_email }}"}
]
requires-python = ">={{ python_version }}"
```

#### README.md

```markdown
# {{ project_name }}

{{ project_description }}

## Installation

\```bash
uv sync
\```
```

#### src/{{ package_name }}/

Entire `src/starter/` directory becomes `src/{{ package_name }}/`:

- `src/starter/` → `src/{{ package_name }}/`
- All imports updated: `from starter.` → `from {{ package_name }}.`

### Step 3: Conditional Content

Use Jinja2 conditionals for optional features:

```python
# src/{{ package_name }}/domain/entities.py

{% if include_example_code %}
@dataclass
class Entity:
    """Example entity - remove or customize."""
    name: str
    value: int
    id: str = field(default_factory=lambda: str(uuid4()))
{% endif %}
```

```yaml
# .github/workflows/test.yml.jinja
{% if include_github_actions %}
name: Test

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      # ... rest of workflow
{% endif %}
```

### Step 4: Documentation Updates

Update documentation to be template-aware:

```markdown
<!-- docs/README.md -->
# {{ project_name }} Documentation

Welcome to the {{ project_name }} documentation.

This project was generated from the Python Starter Template.
```

## Directory Structure

```
template/
├── copier.yml                       # Template configuration
├── .copier-answers.yml.jinja        # Store user answers
├── {{ _copier_conf.answers_file }} # Answers file
│
├── src/
│   └── {{ package_name }}/          # Templated package name
│       ├── __init__.py.jinja        # Templated files
│       ├── domain/
│       ├── services/
│       └── cli/
│
├── tests/
│   └── ...                          # Tests (some conditional)
│
├── docs/
│   ├── README.md.jinja              # Templated docs
│   └── ...
│
├── pyproject.toml.jinja             # Templated config
├── README.md.jinja                  # Templated README
└── .gitignore                       # Static (no template)
```

## Variable Mapping

See [variable-mapping.md](variable-mapping.md) for complete list of template variables.

## Customization Points

See [customization-points.md](customization-points.md) for what users can customize.

## Testing Template

```bash
# Generate from template
copier copy path/to/template test-project

# Answer questions interactively
# Project: My Test Project
# Package: my_test_project
# Author: Test User
# ...

# Verify generated project
cd test-project
uv sync
uv run pytest
uv run ruff check .
```

## Migration Plan

### Phase 1: Setup
1. Add `copier.yml`
2. Test basic generation
3. Verify all variables work

### Phase 2: Templating
1. Convert static files to Jinja2
2. Add conditional sections
3. Update imports and references

### Phase 3: Testing
1. Generate test projects
2. Verify all configurations
3. Test with different options

### Phase 4: Documentation
1. Update README for template usage
2. Document customization options
3. Add troubleshooting guide

### Phase 5: Release
1. Tag template version
2. Publish template
3. Create usage examples

## Best Practices

### 1. Minimal Variables

Keep template variables minimal:
- Essential: project name, package name, author
- Optional: features, integrations

**Avoid**: Over-parameterization (too many options)

### 2. Smart Defaults

Provide good defaults:

```yaml
project_name:
  default: "My Python Project"

package_name:
  default: "{{ project_name.lower().replace(' ', '_') }}"  # Auto-generate
```

### 3. Validation

Validate user input:

```yaml
package_name:
  validator: "{% if not package_name.isidentifier() %}Invalid Python identifier{% endif %}"

python_version:
  choices: ["3.10", "3.11", "3.12"]  # Limit choices
```

### 4. Post-Generation Tasks

Automate setup:

```yaml
_tasks:
  - "uv sync"                        # Install dependencies
  - "git init"                       # Initialize git
  - "git add ."
  - 'git commit -m "Initial commit"'
```

## Maintenance

### Updating Template

1. Make changes to template repository
2. Test generation with changes
3. Update template version
4. Document breaking changes

### Versioning

Use semantic versioning for template:

- **Major**: Breaking changes (structure, major features)
- **Minor**: New features (optional components)
- **Patch**: Bug fixes, documentation updates

## Related Documentation

- [Variable Mapping](variable-mapping.md) - Template variables
- [Customization Points](customization-points.md) - User options
- [Copier Documentation](https://copier.readthedocs.io/) - Official docs

## Sources

- [Copier Documentation](https://copier.readthedocs.io/)
- [Jinja2 Templates](https://jinja.palletsprojects.com/)
- [Template Best Practices](https://copier.readthedocs.io/en/stable/creating/)
