# Chapter 1: Introduction & Installation

## Core Idea
Alembic is a lightweight database migration tool for SQLAlchemy. Install it in a virtualenv alongside your project, run `alembic init`, and set `sqlalchemy.url` in `alembic.ini`.

## Frameworks Introduced
- **Migration Environment**: A directory of scripts (`alembic/`) specific to your application, created once with `alembic init` and maintained alongside source code
- **Version Script Chain**: Migration files linked by `down_revision` pointers, forming a directed graph rather than sequential integers

## Key Concepts
- **alembic.ini**: Main configuration file — `script_location`, `sqlalchemy.url`, `file_template` are the critical settings
- **env.py**: Customizable Python script run on every Alembic invocation; controls DB connection and migration execution
- **script.py.mako**: Mako template for generating new migration files
- **versions/**: Directory holding individual migration scripts with GUID-based IDs
- **%(here)s**: ConfigParser token resolving to the absolute path of the config file

## Mental Models
- Use `alembic init --template pyproject` for PEP 621 compliant config (pyproject.toml + truncated alembic.ini)
- Use `alembic init --template multidb` for multi-database setups
- Pin Alembic to `major.minor` digits (e.g., `>=1.14,<1.15`) — middle digit is "Significant Minor Release" and may break APIs

## Anti-patterns
- **Editing alembic.ini percent signs**: Percent signs in `file_template` and `sqlalchemy.url` must be doubled (`%%`) for ConfigParser interpolation
- **Forgetting env.py customization**: The generic template's `env.py` must be edited to import your models and set `target_metadata`

## Worked Example
```ini
# Minimal alembic.ini for a PostgreSQL project
[alembic]
script_location = %(here)s/alembic
prepend_sys_path = .
sqlalchemy.url = postgresql+psycopg2://scott:P%%40ssw%%25rd@localhost:5432/testdb

[loggers]
keys = root,sqlalchemy,alembic
# ... logging config follows
```

## Key Takeaways
1. Install Alembic in the same virtualenv as your project so `env.py` can access your models
2. `script_location` is the only required config key — everything else has sensible defaults
3. For passwords with `@` or `%`, URL-encode with `urllib.parse.quote_plus` then double percent signs for ConfigParser

## Connects To
- **Ch 2**: Tutorial walks through environment creation step by step
- **Ch 3**: Autogenerate requires `target_metadata` set in env.py
