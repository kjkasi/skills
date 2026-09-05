# Quick Reference & Decision Guide

## Decision Rules

| Situation | Action | Why |
|---|---|---|
| Fresh database | `create_all()` + `stamp("head")` | Skip migration iteration — instant setup |
| Multiple heads exist | `alembic merge` or specify `branchname@head` | Ambiguous upgrade target |
| SQLite + ALTER beyond ADD COLUMN | Use `batch_alter_table` | SQLite lacks ALTER TABLE support |
| Need SQL scripts for DBA | `--sql` flag on upgrade/downgrade | Generates DDL without execution |
| Constraints not visible to autogenerate | Add naming conventions to MetaData | Unnamed constraints are invisible |
| Data migration needed | Separate into `data_upgrades()`, use `-x data=true` | Keep schema and data changes independent |
| Multiple databases | Use `version_locations` + branch labels | Separate migration streams cleanly |
| Constraint name is database-generated | Use `op.f("literal_name")` to bypass convention | Convention would mangle the name |

## Quick Commands

```bash
# Setup
alembic init alembic                    # Create migration environment
alembic init --template pyproject ...   # PEP 621 config
alembic init --template multidb ...     # Multi-database

# Migration lifecycle
alembic revision -m "description"       # Create empty migration
alembic revision --autogenerate -m "…"  # Generate from model diff
alembic upgrade head                    # Apply all pending
alembic downgrade base                  # Reverse all
alembic upgrade +2                      # Move 2 forward
alembic downgrade -1                    # Move 1 back

# Inspection
alembic current                         # Current revision(s)
alembic history --verbose               # Full chain
alembic heads                           # All head revisions
alembic branches                        # Show branch points
alembic check                           # Test for pending changes (1.9+)

# Branching
alembic merge -m "merge" rev1 rev2      # Merge branches
alembic revision --head branch@head     # Add to specific branch
alembic upgrade branch@head             # Upgrade specific branch

# SQL generation
alembic upgrade head --sql > out.sql    # Full SQL script
alembic upgrade start:end --sql         # Range SQL

# Programmatic
from alembic import command, config
cfg = Config("alembic.ini")
command.upgrade(cfg, "head")
command.stamp(cfg, "head")
```

## Key Config Options

| Option | Purpose | Default |
|---|---|---|
| `script_location` | Path to alembic directory | Required |
| `sqlalchemy.url` | Database connection string | Required for generic template |
| `file_template` | Migration filename pattern | `%%(rev)s_%%(slug)s` |
| `render_as_batch` | Enable batch mode for autogenerate | False |
| `compare_type` | Detect column type changes | True (since 1.12) |
| `compare_server_default` | Detect server default changes | False |

## Percent-Sign Escaping

```
# In alembic.ini / pyproject.toml:
%% = literal %
# Double percent signs for ConfigParser interpolation

# In sqlalchemy.url:
%40 = @ (URL-encoded)
%25 = % (URL-encoded)
# Then double % for ConfigParser:
P@ss%rd → P%40ss%25rd → P%%40ss%%25rd
```

## Taste: When to Use What

- **Simple single-DB project**: Generic template, naming conventions, autogenerate
- **SQLite target**: Add `render_as_batch=True`, disable PRAGMA FOREIGN KEYS
- **Multi-database**: multidb template or `version_locations` + branch labels
- **CI/CD pipeline**: Add `alembic check` to detect pending migrations
- **DBA review required**: Generate SQL with `--sql`, submit for review
- **Large migration history**: Periodically prune old files, rebase with `create_all()` + stamp
