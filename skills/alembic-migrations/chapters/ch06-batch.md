# Chapter 6: Batch Migrations for SQLite

## Core Idea
SQLite lacks ALTER TABLE support. Alembic's `batch_alter_table()` performs "move and copy" — creates a temp table with changes, copies data, drops old, renames. On other backends, it runs normal ALTERs unless `recreate='always'`.

## Frameworks Introduced
- **Batch Operations Context**: `op.batch_alter_table("table_name")` returns a `BatchOperations` context that accumulates mutations, then executes the move-and-copy workflow
- **Copy-From Pattern**: Pass a pre-fabricated `Table` object via `copy_from` for offline mode or when reflection is impractical

## Key Concepts
- **render_as_batch=True**: Configure in `env.py` so autogenerate wraps mutations in `batch_alter_table` blocks
- **recreate='always'**: Force move-and-copy on all backends (useful for long-lock avoidance on PostgreSQL/MySQL)
- **naming_convention parameter**: Required on `batch_alter_table` to name reflected unnamed constraints (especially foreign keys)
- **PRAGMA FOREIGN KEYS**: Must be disabled during batch operations on SQLite when referential integrity is enforced

## Mental Models
- Batch mode is a **transaction wrapper** — all mutations in the block are applied atomically via table recreation
- On SQLite: batch = table recreate; on PostgreSQL/MySQL: batch = normal ALTERs (unless `recreate='always'`)
- Self-referencing foreign keys are handled specially — they reference the original table name through the rename

## Anti-patterns
- **Using batch with PRAGMA FOREIGN KEYS enabled**: The drop will fail — disable the pragma first
- **Forgetting existing_type for Boolean/Enum**: These schema types have associated CHECK constraints that must be explicitly specified
- **Not providing table_args for unnamed CHECK constraints**: Unnamed constraints are silently dropped during recreation

## Code Examples
```python
# Basic batch migration
with op.batch_alter_table("some_table") as batch_op:
    batch_op.add_column(Column('foo', Integer))
    batch_op.drop_column('bar')
```
- **What it demonstrates**: Simple column add/drop in batch mode

```python
# Batch with autogenerate (env.py)
context.configure(
    connection=connection,
    target_metadata=target_metadata,
    render_as_batch=True
)
```
- **What it demonstrates**: Enabling batch mode for autogenerate output

```python
# Offline batch mode — must provide table structure
meta = MetaData()
some_table = Table(
    'some_table', meta,
    Column('id', Integer, primary_key=True),
    Column('bar', String(50))
)

with op.batch_alter_table("some_table", copy_from=some_table) as batch_op:
    batch_op.add_column(Column('foo', Integer))
    batch_op.drop_column('bar')
```
- **What it demonstrates**: Batch migration without database reflection (offline mode)

## Reference Tables

### Batch Mode Behavior by Backend

| Backend | Default Behavior | With `recreate='always'` |
|---|---|---|
| SQLite | Move-and-copy (if non-ADD COLUMN ops) | Move-and-copy always |
| PostgreSQL | Normal ALTERs | Move-and-copy (drops/recreates constraints) |
| MySQL/InnoDB | Normal ALTERs | Move-and-copy (manual FK handling needed) |

## Key Takeaways
1. `render_as_batch=True` in env.py makes autogenerate produce batch-compatible output
2. Use `naming_convention` parameter on `batch_alter_table` for SQLite unnamed constraint handling
3. `copy_from` is required for offline batch mode — no reflection available
4. Disable `PRAGMA FOREIGN KEYS` before batch operations on SQLite
5. `recreate='always'` forces move-and-copy on all backends — useful for minimizing table lock time

## Connects To
- **Ch 4**: Offline batch mode requires `copy_from` since no DB connection
- **Ch 5**: Naming conventions are critical for batch mode constraint handling
