# Chapter 9: Recipes & Patterns

## Core Idea
Common Alembic patterns: stamp a new DB at head after `create_all`, conditional migrations via `-x` flags, sharing connections programmatically, and managing replaceable objects (views, stored procedures).

## Frameworks Introduced
- **Stamp Pattern**: Run `MetaData.create_all()` then `command.stamp(Config, "head")` — new databases skip migration history
- **Conditional Migrations**: Use `context.get_x_argument(as_dictionary=True)` to read `-x key=value` flags and conditionally run schema vs. data migrations
- **Connection Sharing**: Pass a `Connection` via `Config.attributes['connection']` — env.py uses it instead of creating its own engine
- **Replaceable Objects**: Views/stored procedures as value objects with `ReplaceableObject(name, sqltext)` — use operation plugins for CREATE/DROP/REPLACE

## Key Concepts
- **command.stamp()**: Mark a database at a specific revision without running migrations
- **get_x_argument()**: Read custom `-x` flags from command line in migration scripts
- **Config.attributes**: Share objects (like connections) between Python code and env.py
- **Operations.register_operation()**: Extend `op.*` namespace with custom operations
- **Pruning old migrations**: Delete files, set first remaining file's `down_revision = None`

## Mental Models
- Stamp = "this DB is already at this version" — no migration execution, just version tracking
- Conditional migrations separate schema changes from data changes — run independently via flags
- Replaceable objects need the "previous version" pattern — reference old migration files for downgrade

## Anti-patterns
- **Running all migrations for fresh DBs**: Use `create_all()` + `stamp("head")` instead — much faster
- **Hardcoding data migrations**: Use `-x data=true` flags so data migrations are opt-in
- **Not pruning old migrations**: Thousands of migration files slow startup — prune and rebase

## Code Examples
```python
# Stamp pattern — new database setup
my_metadata.create_all(engine)

from alembic.config import Config
from alembic import command
alembic_cfg = Config("/path/to/yourapp/alembic.ini")
command.stamp(alembic_cfg, "head")
```
- **What it demonstrates**: Creating a fresh DB without running migration history

```python
# Conditional migration via -x flag
def upgrade():
    schema_upgrades()
    if context.get_x_argument(as_dictionary=True).get('data', None):
        data_upgrades()

def schema_upgrades():
    op.create_table("my_table", Column('data', String))

def data_upgrades():
    my_table = table('my_table', column('data', String))
    op.bulk_insert(my_table, [
        {'data': 'data 1'},
        {'data': 'data 2'},
    ])
```
- **What it demonstrates**: Separating schema and data migrations with opt-in data via `-x data=true`

```python
# Programmatic connection sharing
from alembic import command, config

cfg = config.Config("/path/to/yourapp/alembic.ini")
with engine.begin() as connection:
    cfg.attributes['connection'] = connection
    command.upgrade(cfg, "head")
```
- **What it demonstrates**: Running migrations within a Python-managed transaction

## Key Takeaways
1. Use `create_all()` + `stamp("head")` for fresh databases — skip migration iteration
2. Separate schema and data migrations — use `-x data=true` to run data migrations optionally
3. Share connections via `Config.attributes['connection']` for transactional programmatic usage
4. Prune old migrations periodically — set first remaining file's `down_revision = None`

## Connects To
- **Ch 2**: Tutorial covers basic migration creation
- **Ch 3**: Autogenerate can be combined with stamp for new databases
- **Ch 7**: Branch management patterns for complex environments
