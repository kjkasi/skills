# Chapter 5: Naming Conventions for Constraints

## Core Idea
Use SQLAlchemy's `MetaData(naming_convention=...)` to auto-name all constraints and indexes. This makes constraints visible to autogenerate, portable across databases, and targetable by `op.drop_constraint()`.

## Frameworks Introduced
- **Naming Convention Dictionary**: Maps constraint types (`ix`, `uq`, `ck`, `fk`, `pk`) to name templates with tokens like `%(table_name)s`, `%(column_0_name)s`
- **op.f() Bypass**: `op.f("name")` produces a string that skips naming convention tokenization

## Key Concepts
- **Convention tokens**: `%(table_name)s`, `%(column_0_name)s`, `%(referred_table_name)s`, `%(constraint_name)s`, `%(column_0_label)s`
- **Unnamed constraints**: Database-generated names are unpredictable (e.g., Oracle `SYS_C0029334`) — always name your constraints
- **Convention integration**: Pass `target_metadata` with naming convention to `context.configure()` — autogenerate and operations inherit it

## Mental Models
- Think of naming conventions as a **regex for constraint names** — consistent, predictable, machine-generated
- `op.f()` is the escape hatch when you need to bypass the convention for a specific operation

## Anti-patterns
- **Manually naming every constraint**: Tedious and error-prone — use naming conventions instead
- **Using `column.unique=True` without conventions**: Creates unnamed UniqueConstraint — invisible to autogenerate
- **Forgetting convention for Boolean/Enum**: These create implicit CHECK constraints that need names

## Code Examples
```python
# In your model's metadata definition
from sqlalchemy import MetaData
from sqlalchemy.orm import DeclarativeBase

convention = {
    "ix": "ix_%(column_0_label)s",
    "uq": "uq_%(table_name)s_%(column_0_name)s",
    "ck": "ck_%(table_name)s_%(constraint_name)s",
    "fk": "fk_%(table_name)s_%(column_0_name)s_%(referred_table_name)s",
    "pk": "pk_%(table_name)s"
}

class Base(DeclarativeBase):
    metadata = MetaData(naming_convention=convention)
```
- **What it demonstrates**: Setting up automatic naming for all constraints

```python
# Bypass naming convention for a specific operation
def upgrade():
    op.drop_constraint(op.f("some_check_const"), "t1", type_="check")
```
- **What it demonstrates**: Using `op.f()` to drop a constraint by its literal name, skipping convention

## Key Takeaways
1. Define naming conventions on your `MetaData` — all constraints get predictable names
2. Pass `target_metadata` with conventions to `context.configure()` for autogenerate integration
3. `op.f("name")` bypasses naming convention tokenization when you need literal names
4. Boolean and Enum types need `.name` attribute set for CHECK constraint naming

## Connects To
- **Ch 3**: Autogenerate only detects named constraints — conventions ensure this
- **Ch 6**: Batch mode needs `naming_convention` parameter for unnamed SQLite constraints
