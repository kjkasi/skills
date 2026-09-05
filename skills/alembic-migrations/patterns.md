# Patterns & Techniques

## Fresh Database Setup (Stamp Pattern)
**When to use**: New database deployment — skip migration history, use SQLAlchemy's `create_all()` + Alembic stamp.
**How**: Run `MetaData.create_all(engine)` then `command.stamp(Config, "head")`.
**Trade-offs**: Fast setup vs. loses migration history — combine with pruning old migrations.

## Conditional Data Migrations
**When to use**: Separate schema changes from data backfills; make data migrations opt-in.
**How**: Use `-x data=true` flag; in migration, check `context.get_x_argument(as_dictionary=True).get('data')`.
**Trade-offs**: Cleaner migration files vs. requires flag management.

## Connection Sharing (Programmatic)
**When to use**: Running multiple Alembic commands in one transaction from Python.
**How**: Pass `Connection` via `Config.attributes['connection']`; env.py checks for it before creating engine.
**Trade-offs**: Transactional consistency vs. more complex env.py.

## Naming Convention Integration
**When to use**: Any project — ensures constraints are named, portable, and visible to autogenerate.
**How**: Define `MetaData(naming_convention={...})` on your declarative base; pass to `context.configure()`.
**Trade-offs**: Slight setup effort vs. eliminates manual naming and cross-DB portability issues.

## Batch Mode for SQLite
**When to use**: Any project targeting SQLite — required for ALTER operations beyond ADD COLUMN.
**How**: Set `render_as_batch=True` in env.py; autogenerate wraps mutations in `batch_alter_table`.
**Trade-offs**: Automatic table recreation vs. slower migrations; PRAGMA FOREIGN KEYS must be disabled.

## Offline SQL Generation
**When to use**: DBA review workflows, restricted DDL environments, audit trails.
**How**: `alembic upgrade head --sql > migration.sql`; use `start:end` for ranges.
**Trade-offs**: No live execution vs. scripts must use only `op.*` directives.

## Post-Write Hooks (Code Formatting)
**When to use**: Enforce code style on generated migration files automatically.
**How**: Configure `[post_write_hooks]` in alembic.ini with Black, ruff, or custom functions.
**Trade-offs**: Consistent formatting vs. adds generation time; use `pre-commit` as alternative.

## Replaceable Objects (Views, Stored Procedures)
**When to use**: Managing database objects that must be dropped and recreated as a whole.
**How**: Create `ReplaceableObject(name, sqltext)` value objects; use `@Operations.register_operation()` for custom ops.
**Trade-offs**: Full control vs. requires custom operation implementation.

## Pruning Old Migrations
**When to use**: Large projects with hundreds of migration files slowing startup.
**How**: Delete old files; set first remaining file's `down_revision = None` to rebase.
**Trade-offs**: Faster startup vs. loses incremental history — use `create_all()` + stamp pattern first.
