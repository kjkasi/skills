# Chapter 3: Auto-generating Migrations

## Core Idea
Alembic compares your ORM's MetaData (proposed schema) against the live database (current schema) and generates candidate migration scripts. Always review and adjust the output — autogenerate is not perfect.

## Frameworks Introduced
- **Autogenerate Detection Pipeline**: Scans database tables → reflects columns/indexes/constraints → compares against MetaData → emits detected changes
- **include_name Hook**: Callable `(name, type_, parent_names) -> bool` to filter schemas, tables, columns from autogeneration
- **include_object Hook**: Callable `(object, name, type_, reflected, compare_to) -> bool` for fine-grained object-level filtering

## Key Concepts
- **target_metadata**: Set in `env.py` — the MetaData object(s) representing your desired schema
- **compare_type**: Enable/disable column type comparison (default: True since 1.12.0)
- **compare_server_default**: Enable/disable server default comparison (default: False)
- **autogenerate_plugins**: List of plugin names for extensible detection (e.g., CHECK constraint detection)
- **render_as_batch**: Enables batch mode output for SQLite-compatible migrations

## Mental Models
- Autogenerate produces **candidates**, not finals — always review before applying
- Use `include_name` when you need to **exclude** schemas/tables from comparison (filtering by name)
- Use `include_object` when you need to **exclude** based on object properties (e.g., `Column.info` flags)

## Anti-patterns
- **Trusting autogenerate blindly**: It cannot detect table/column renames (shows as add+drop), anonymous constraints, or ENUM types on non-native backends
- **Forgetting to set target_metadata**: Without it, autogenerate has nothing to compare against
- **Not naming constraints**: Unnamed constraints are invisible to autogenerate — use naming conventions

## Code Examples
```python
# env.py — enable autogenerate
from myapp.models import Base
target_metadata = Base.metadata

def run_migrations_online():
    engine = engine_from_config(
        config.get_section(config.config_ini_section), prefix='sqlalchemy.')
    with engine.connect() as connection:
        context.configure(
            connection=connection,
            target_metadata=target_metadata,
            compare_type=True,
            compare_server_default=False,
        )
        with context.begin_transaction():
            context.run_migrations()
```
- **What it demonstrates**: Configuring env.py for autogenerate with type comparison enabled

```python
# Filter tables to only those in your MetaData
def include_name(name, type_, parent_names):
    if type_ == "table":
        return name in target_metadata.tables
    return True

context.configure(
    connection=connection,
    target_metadata=target_metadata,
    include_name=include_name,
)
```
- **What it demonstrates**: Preventing autogenerate from generating DROP TABLE for tables not in your models

## Reference Tables

### What Autogenerate Detects

| Change Type | Detection |
|---|---|
| Table add/drop | Always |
| Column add/drop | Always |
| Nullable change | Always |
| Index/unique constraint changes | Always (named only) |
| Foreign key changes | Always (named only) |
| CHECK constraint add/drop | Named only (since 1.19.0) |
| Column type change | Optional (`compare_type=True`) |
| Server default change | Optional (`compare_server_default=True`) |
| Table/column rename | **Not detected** — shows as add+drop |

## Worked Example
```bash
# Generate a migration from model differences
$ alembic revision --autogenerate -m "Added account table"
INFO [alembic.context] Detected added table 'account'
Generating /path/to/alembic/versions/27c6a30d7c24.py...done

# Check if new operations exist without generating
$ alembic check
No new upgrade operations detected.
```

## Key Takeaways
1. Always set `target_metadata = Base.metadata` in env.py before using autogenerate
2. Use `include_name` to prevent false DROP TABLE for tables outside your models
3. `alembic check` (since 1.9.0) tests for pending changes without generating files
4. Name all constraints — unnamed ones are invisible to autogenerate
5. Configure `sqlalchemy_module_prefix` and `user_module_prefix` to control how types render in migrations

## Connects To
- **Ch 5**: Naming conventions make constraints visible to autogenerate
- **Ch 6**: `render_as_batch=True` for SQLite-compatible autogenerate output
- **Ch 9**: Cookbook has patterns for conditional migrations and empty migration prevention
