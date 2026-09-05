# Chapter 2: Getting Started Tutorial

## Core Idea
Create a migration environment with `alembic init`, write upgrade/downgrade functions using `op.*` directives, and run `alembic upgrade head` to apply. The revision chain is a linked list via `down_revision`.

## Frameworks Introduced
- **Revision Chain**: Each migration file has `revision` (its ID) and `down_revision` (parent ID). `None` = base. Alembic traverses this chain to determine execution order.
- **op.* Operations**: Minimalistic directives (`create_table`, `add_column`, `drop_table`, etc.) that work in both online and offline modes

## Key Concepts
- **head**: The most recent revision(s) in the chain
- **base**: The starting revision (`down_revision = None`)
- **Partial revision IDs**: Use the shortest unique prefix (e.g., `ae1` for `ae1027a6acf`)
- **Relative identifiers**: `+2` / `-1` for moving N steps from current; `ae10+2` for relative to a specific revision
- **alembic_version table**: Auto-created table tracking current revision(s) — can have multiple rows during branch merges

## Mental Models
- Think of migrations as a **linked list** not sequential integers — branches create a DAG (directed acyclic graph)
- `alembic upgrade head` = traverse from current version(s) to target, executing each `upgrade()` function
- `alembic downgrade base` = reverse traversal executing `downgrade()` functions

## Anti-patterns
- **Using `alembic upgrade head` with multiple heads**: Fails with ambiguous target — use `branchname@head` or merge first
- **Skipping downgrade()**: Always implement it for reversibility — you'll need it for rollbacks

## Worked Example
```python
# Migration: create account table
revision = '1975ea83b712'
down_revision = None

from alembic import op
import sqlalchemy as sa

def upgrade():
    op.create_table(
        'account',
        sa.Column('id', sa.Integer, primary_key=True),
        sa.Column('name', sa.String(50), nullable=False),
        sa.Column('description', sa.Unicode(200)),
    )

def downgrade():
    op.drop_table('account')
```

## Key Takeaways
1. `alembic revision -m "description"` creates a new migration file
2. `alembic upgrade head` applies all pending migrations; `alembic downgrade base` reverses them
3. `alembic current` shows the database's current revision; `alembic history --verbose` shows the full chain
4. History ranges: `alembic history -r1975ea:ae1027` or `-r-3:current`

## Connects To
- **Ch 1**: Environment setup and configuration
- **Ch 3**: Autogenerate replaces manual upgrade/downgrade writing
