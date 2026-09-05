# Chapter 4: Generating SQL Scripts (Offline Mode)

## Core Idea
Alembic can generate pure SQL scripts instead of executing migrations directly. Use `--sql` flag with upgrade/downgrade commands. Critical for DBA review workflows and restricted DDL environments.

## Frameworks Introduced
- **Offline Mode**: No database connection required — Alembic generates DDL statements to stdout or a file
- **Start:End Range Syntax**: `alembic upgrade 1975ea:ae1027 --sql` to generate SQL for a specific revision range

## Key Concepts
- **--sql flag**: Switches any upgrade/downgrade command to output SQL instead of executing
- **output_buffer**: Configure in `env.py` to direct SQL output to specific files
- **transactional_ddl**: Set to `False` in offline mode for databases that don't support transactional DDL
- **Starting version**: In offline mode, defaults to base; use `start:end` syntax to specify range

## Mental Models
- Offline mode = "what SQL would these migrations generate?" — no reflection, no live connection
- Scripts generated must use only Alembic `op.*` directives, not raw Python DB-API calls
- For multi-database offline mode, loop through databases and set `output_buffer` per database

## Anti-patterns
- **Using SELECT in offline migrations**: No DB connection available — data queries will fail
- **Forgetting start version**: Offline mode defaults to base; use `start:end` to generate incremental SQL

## Worked Example
```bash
# Generate SQL for upgrading to a specific revision
$ alembic upgrade ae1027a6acf --sql > migration.sql

# Generate SQL for a specific range
$ alembic upgrade 1975ea83b712:ae1027a6acf --sql > partial.sql
```

```python
# env.py — multi-database offline mode
def run_migrations_offline():
    for name, engine, file_ in [
        ("db1", db_1, "db1.sql"),
        ("db2", db_2, "db2.sql"),
    ]:
        context.configure(
            url=engine.url,
            transactional_ddl=False,
            output_buffer=open(file_, 'w'))
        context.execute("-- running migrations for '%s'" % name)
        context.run_migrations(name=name)
```
- **What it demonstrates**: Generating separate SQL files for multiple databases

## Key Takeaways
1. `alembic upgrade head --sql` generates a complete SQL script from base to head
2. Use `start:end` syntax to generate SQL for a specific revision range
3. Migration scripts must use only `op.*` directives for offline compatibility
4. For multi-DB offline, use `output_buffer` to write per-database SQL files

## Connects To
- **Ch 6**: Batch mode with `copy_from` is required for offline SQLite migrations
- **Ch 2**: Online mode is the default — offline is opt-in via `--sql`
