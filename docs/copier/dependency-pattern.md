---
title: Template as Dependency Pattern
status: stable
owners: [template-team]
---

# Template as Dependency Pattern

This document describes the pattern where generated projects treat the template as a **dependency** rather than a one-time scaffold.

## Problem

Traditional Copier templates copy all documentation into generated projects, creating:
- **Duplication**: Same docs exist in template and every project
- **Drift**: Template docs improve, but projects have stale copies
- **Noise**: Projects contain generic pattern docs mixed with domain docs
- **Maintenance burden**: Fix docs in N places instead of one

## Solution

Treat the template as a **dependency that projects import**:

1. **Template docs stay in template** (single source of truth)
2. **Generated projects reference template docs** (via relative paths)
3. **Knowledge base imports** make docs discoverable to agents
4. **Copier updates** can push improvements to downstream projects

## Implementation

### Template Configuration (copier.yml)

Exclude template-specific documentation from generation:

```yaml
_exclude:
  - copier.yml
  - .git
  # ... other standard excludes ...

  # Template documentation stays in template, not generated in projects
  - docs/architecture
  - docs/decisions
  - docs/guides
  - docs/reference
  - knowledge-base.yaml  # Template KB config

_skip_if_exists:
  # ... project files that users modify ...
  - README.md
  - docs/agents.md
  - knowledge-base.yaml  # Project KB config
```

### Generated Documentation

Create **minimal, parameterized docs** that link to template:

**docs/README.md.jinja**:
```markdown
# {{ project_name }} Documentation

{{ project_description }}

## Architecture & Patterns

This project uses the [Python Starter Template](../python-starter) patterns.

**Core patterns**:
- [Functional Services](../python-starter/docs/architecture/functional-services.md)
- [Protocol-based DI](../python-starter/docs/architecture/dependency-injection.md)
- [Testing Strategy](../python-starter/docs/architecture/testing-strategy.md)

See [Template Documentation](../python-starter/docs/) for complete guides.
```

**docs/agents.md.jinja**:
```markdown
# Agent Entrypoint - {{ project_name }}

## Architecture

This project uses **Python Starter Template** patterns:
- [Functional Services](../python-starter/docs/architecture/functional-services.md)
- [Protocol-based DI](../python-starter/docs/architecture/dependency-injection.md)

## Guides

See template documentation:
- [Adding Features](../python-starter/docs/guides/adding-features.md)
- [Testing Guide](../python-starter/docs/guides/testing-guide.md)
```

### Knowledge Base Import

**Project knowledge-base.yaml**:
```yaml
apiVersion: kb/v1
name: myproject
description: "My awesome project"

imports:
  - kind: local
    path: ../meta-knowledge-base
    entrypoint: meta.yaml
  - kind: local
    path: ../python-starter  # ← Import template KB
    entrypoint: knowledge-base.yaml

# ... project-specific config ...
```

**Agent discovery flow**:
1. Agent reads project's `knowledge-base.yaml`
2. Sees import of `../python-starter`
3. Reads `../python-starter/knowledge-base.yaml`
4. Discovers all template docs (architecture, decisions, guides, reference)
5. Can navigate and use template documentation

## Benefits

### Single Source of Truth
- Template patterns documented **once** in python-starter
- All projects reference the same docs
- Updates benefit everyone immediately

### Cleaner Projects
- Generated projects contain **only project-specific docs**
- No clutter from generic pattern documentation
- Clear separation: template = patterns, project = domain

### Living Dependency
- Template docs can improve over time
- Projects get improvements via `copier update`
- No manual synchronization needed

### Agent Discoverability
- Agents discover template docs via KB imports
- No special configuration needed
- Works with any KB-aware tooling

### Update Lifecycle
- `copier update` pushes new patterns to projects
- Protected files (`_skip_if_exists`) preserve customizations
- Clean merge process for updates

## File Categories

### Never Generate (template-only)
- `docs/architecture/*` - Pattern documentation
- `docs/decisions/*` - Template ADRs
- `docs/guides/*` - How-to guides
- `docs/reference/*` - Technical reference
- `knowledge-base.yaml` - Template KB config

### Generate Once, Protect Forever
- `README.md` - Project README
- `docs/agents.md` - Project agent entrypoint
- `knowledge-base.yaml` - Project KB config
- Source files users will modify

### Always Update
- `pyproject.toml` - Build config (managed carefully)
- `TEMPLATE_STATUS.md` - Template metadata
- Core infrastructure files

## Usage Example

### Initial Generation

```bash
copier copy ../python-starter my-project
cd my-project
```

Result:
- Minimal `docs/README.md` linking to template
- Minimal `docs/agents.md` with template references
- No duplicated architecture/decisions/guides/reference docs
- `knowledge-base.yaml` imports python-starter KB

### Agent Workflow

Agent working on my-project:
1. Reads `my-project/knowledge-base.yaml`
2. Follows import to `../python-starter/knowledge-base.yaml`
3. Discovers `../python-starter/docs/architecture/`
4. Reads pattern documentation
5. Applies patterns to project-specific code

### Template Updates

```bash
# In template repository
git commit -m "Improve testing guide"

# In project repository
copier update --trust
```

Result:
- Project gets updated `docs/README.md` (links may change)
- Project's `docs/agents.md` protected (user customizations preserved)
- Template docs available immediately (no need to regenerate)

## Migration from Duplicated Docs

If you already generated projects with duplicated docs:

1. **Backup custom content** from project docs
2. **Remove template docs** from project:
   ```bash
   rm -rf docs/architecture docs/decisions docs/guides docs/reference
   rm docs/knowledge-base.yaml
   ```
3. **Update `knowledge-base.yaml`** to import python-starter
4. **Update `docs/README.md`** with minimal content + template links
5. **Restore custom content** to project-specific docs
6. **Commit changes**
7. **Run `copier update`** to sync with template

## Considerations

### Advantages
- ✅ No duplication
- ✅ Single source of truth
- ✅ Template improvements benefit all projects
- ✅ Clearer project structure
- ✅ Agent discoverability built-in

### Requirements
- ⚠️ Projects must have access to template repository (local or remote)
- ⚠️ Relative path structure must be maintained
- ⚠️ KB import system must support local paths (it does)

### When Not to Use
- ❌ Projects deployed without template access
- ❌ Template is not stable (frequent breaking changes)
- ❌ Projects need to fork patterns significantly

## Related

- [Template Design](template-design.md) - Overall Copier template strategy
- [Variable Mapping](variable-mapping.md) - Template variable reference
- [Customization Points](customization-points.md) - User customization options

## Sources

- [Copier documentation](https://copier.readthedocs.io/)
- [Meta knowledge base](../../meta-knowledge-base/docs/meta.md)
- [copier.yml](../../copier.yml) - Template configuration
- [knowledge-base.yaml](../../knowledge-base.yaml) - Template KB config
