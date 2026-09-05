# Chapter 8: Operations Reference

## Core Idea
All migration directives are methods on `Operations`, accessed via `alembic.op.*`. They generate minimal SQLAlchemy metadata internally — you specify strings and flags, not full Table/Constraint objects (except `add_column` and `create_table`).

## Frameworks Introduced
- **Operations Proxy**: `alembic.op` is a module-level proxy to `Operations` instance — import symbols directly from `alembic.op`
- **BatchOperations**: Separate operations context for batch mode — `op.batch_alter_table()` returns this
- **Operation Plugins**: Extensible via `@Operations.register_operation()` and `@Operations.implementation_for()`

## Key Concepts
- **Schema operations**: `create_table`, `drop_table`, `add_column`, `drop_column`, `alter_column`, `create_index`, `drop_index`
- **Constraint operations**: `create_unique_constraint`, `drop_constraint`, `create_foreign_key`, `create_check_constraint`
- **Data operations**: `execute()`, `bulk_insert()`, `bulk_update()` for raw SQL or bulk data changes
- **Naming integration**: Constraint operations use naming conventions from `target_metadata` when names are `None`
- **op.f()**: Bypass naming convention tokenization for literal constraint names

## Mental Models
- `op.*` methods are **DDL generators** — they produce SQL statements, not ORM operations
- Each method works in both online (execute against DB) and offline (print SQL) modes
- `Operations.register_operation()` + `@Operations.implementation_for()` = custom op extensions

## Anti-patterns
- **Using raw SQL for DDL**: Prefer `op.*` directives — they work in offline mode and are database-agnostic
- **Forgetting `type_` parameter**: `op.drop_constraint()` needs `type_="foreignkey"` etc. to generate correct DDL
- **Not using naming conventions**: Without them, `op.drop_constraint(None, ...)` won't work

## Code Examples
```python
def upgrade():
    # Create table
    op.create_table(
        'account',
        sa.Column('id', sa.Integer, primary_key=True),
        sa.Column('name', sa.String(50), nullable=False),
    )

    # Add column
    op.add_column('account', sa.Column('email', sa.String(100)))

    # Drop constraint (with naming convention)
    op.drop_constraint('fk_account_email', 'account', type_='foreignkey')

    # Execute raw SQL (works in both online and offline)
    op.execute("UPDATE account SET name = 'unknown' WHERE name IS NULL")
```
- **What it demonstrates**: Core operations patterns

## Key Takeaways
1. All DDL operations go through `op.*` — they work in both online and offline modes
2. `add_column` and `create_table` require full `Column` objects; other ops use strings
3. `op.f("name")` bypasses naming convention for literal constraint names
4. Custom operations: register with `@Operations.register_operation()`, implement with `@Operations.implementation_for()`

## Connects To
- **Ch 3**: Autogenerate produces `op.*` calls in migration files
- **Ch 6**: Batch operations are a separate context from `op.*`
- **Ch 9**: Cookbook shows custom operation patterns
