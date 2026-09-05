---
name: alembic-migrations
description: "Knowledge base from \"Alembic Documentation\" by the SQLAlchemy project. Use when working with Alembic database migrations, configuring autogenerate, managing branches, batch operations for SQLite, naming conventions, or generating SQL scripts."
---

<!-- argument-hint: [migration topic, command name, or chapter number] -->

# Alembic Documentation
**Author**: Mike Bayer / SQLAlchemy project | **Chapters**: 9 | **Generated**: 2026-09-03

## How to Use This Skill

- **Without arguments** — load core frameworks for reference
- **With a topic** — ask about `autogenerate`, `batch`, `branches`, or another indexed topic; I find and read the relevant chapter
- **With chapter** — ask for `ch03`; I load that specific chapter
- **Browse** — ask "what chapters do you have?" to see the full index

When you ask about a topic not covered in Core Frameworks below, I will read
the relevant chapter file before answering.

---

## Core Frameworks & Mental Models

### Autogenerate Detection Pipeline
Compare ORM MetaData (proposed) against live database (current) → generate candidate migrations. **Always review candidates** — autogenerate is not perfect. Cannot detect: table/column renames, anonymous constraints, ENUM types on non-native backends.

Use `include_name` to filter schemas/tables by name. Use `include_object` to filter by object properties (e.g., `Column.info` flags).

### Revision Chain as DAG
Migrations form a **linked list** via `down_revision` pointers. Branches turn it into a **directed acyclic graph**. Alembic traverses via topological sort. Multiple heads = ambiguous upgrade target → merge or specify branch.

### Batch Operations (SQLite)
SQLite lacks ALTER TABLE. `op.batch_alter_table()` performs "move and copy" — creates temp table with changes, copies data, drops old, renames. On other backends, runs normal ALTERs unless `recreate='always'`. **Must disable PRAGMA FOREIGN KEYS** during batch ops.

### Naming Conventions
Define `MetaData(naming_convention={...})` to auto-name all constraints. Makes constraints visible to autogenerate, portable across databases, and targetable by `op.drop_constraint()`. Use `op.f("name")` to bypass convention for literal names.

### Offline Mode
Generate SQL scripts with `--sql` flag. No database connection needed. Scripts must use only `op.*` directives. Use `start:end` syntax for ranges. Critical for DBA review workflows.

### Stamp Pattern
For fresh databases: run `MetaData.create_all()` then `command.stamp(Config, "head")`. Skips migration history iteration — much faster than running hundreds of migrations.

### Conditional Migrations
Separate schema changes from data backfills. Use `-x data=true` flag; check `context.get_x_argument(as_dictionary=True).get('data')` in migration scripts.

---

## Chapter Index

| # | Title | Key Frameworks |
|---|-------|----------------|
| [ch01](chapters/ch01-intro.md) | Introduction & Installation | env.py, alembic.ini, %(here)s |
| [ch02](chapters/ch02-tutorial.md) | Getting Started Tutorial | revision chain, op.* directives, upgrade/downgrade |
| [ch03](chapters/ch03-autogenerate.md) | Auto-generating Migrations | target_metadata, include_name, include_object, compare_type |
| [ch04](chapters/ch04-offline.md) | Generating SQL Scripts | --sql, output_buffer, start:end syntax |
| [ch05](chapters/ch05-naming.md) | Naming Conventions | naming_convention, op.f(), constraint visibility |
| [ch06](chapters/ch06-batch.md) | Batch Migrations for SQLite | batch_alter_table, render_as_batch, recreate='always', copy_from |
| [ch07](chapters/ch07-branches.md) | Branching & Merging | branch_labels, merge points, depends_on, version_locations |
| [ch08](chapters/ch08-ops.md) | Operations Reference | op.* namespace, Operations.register_operation(), BatchOperations |
| [ch09](chapters/ch09-cookbook.md) | Recipes & Patterns | stamp pattern, conditional migrations, connection sharing |

## Topic Index

- **autogenerate** → ch03
- **alembic.ini** → ch01
- **batch mode** → ch06
- **branches** → ch07
- **check (alembic check)** → ch03
- **column types** → ch03
- **connection sharing** → ch09
- **conditional migrations** → ch09
- **cookbook** → ch09
- **create_all** → ch09
- **create_table** → ch08
- **custom operations** → ch08, ch09
- **data migrations** → ch09
- **depends_on** → ch07
- **down_revision** → ch02
- **downgrade** → ch02
- **env.py** → ch01
- **exclude/include** → ch03
- **file_template** → ch01
- **foreign keys** → ch06
- **head/heads** → ch02, ch07
- **include_name** → ch03
- **include_object** → ch03
- **merge** → ch07
- **MetaData** → ch03
- **multiple databases** → ch07, ch09
- **naming conventions** → ch05
- **offline mode** → ch04
- **operations** → ch08
- **op.f()** → ch05, ch08
- **PRAGMA FOREIGN KEYS** → ch06
- **post_write_hooks** → ch03
- **pyproject.toml** → ch01
- **render_as_batch** → ch06
- **revision** → ch02
- **stamp** → ch09
- **target_metadata** → ch03
- **version_locations** → ch07
- **versions/** → ch01

## Supporting Files

- [glossary.md](glossary.md) — all key terms with definitions
- [patterns.md](patterns.md) — all techniques and design patterns
- [cheatsheet.md](cheatsheet.md) — quick reference tables and decision guides

---

## Scope & Limits

This skill covers the Alembic documentation content only. For hands-on implementation in your codebase, combine with project-specific tools and SQLAlchemy documentation. For topics beyond Alembic, check related skills or ask the agent directly.
