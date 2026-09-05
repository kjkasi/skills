---
name: pydantic-settings
description: "Knowledge base from \"Pydantic Settings Documentation\" by Pydantic Team. Use when applying pydantic-settings patterns for configuration management, environment variables, CLI arguments, secrets, or referencing its concepts."
---

<!-- argument-hint: [topic, framework name, or chapter number] -->

# Pydantic Settings
**Author**: Pydantic Team | **Chapters**: 8 | **Generated**: 2026-09-03

## How to Use This Skill

- **Without arguments** — load core frameworks for reference
- **With a topic** — ask about `env_prefix`, `nested models`, `CLI`, or another indexed topic; I find and read the relevant chapter
- **With chapter** — ask for `ch01`; I load that specific chapter
- **Browse** — ask "what chapters do you have?" to see the full index

When you ask about a topic not covered in Core Frameworks below, I will read
the relevant chapter file before answering.

---

## Core Frameworks & Mental Models

### BaseSettings Pattern
Inherit from `BaseSettings` instead of `BaseModel` to enable automatic environment variable loading. The model initialiser attempts to determine field values from environment when not provided as kwargs.

```python
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    model_config = SettingsConfigDict(env_prefix='MYAPP_')
    database_url: str
    debug: bool = False
```

**Use when:** Your config should come from environment variables
**How:** Replace `BaseModel` with `BaseSettings`; add `model_config = SettingsConfigDict(...)`

### env_prefix & env_prefix_target
`env_prefix` adds a prefix to all environment variable names. `env_prefix_target` controls where the prefix applies: `'variable'` (default), `'alias'`, or `'all'`.

**Use when:** You have multiple services with potential naming collisions
**How:** Set `env_prefix='MYAPP_'` and choose target based on alias needs

### AliasChoices for Fallback
Allows multiple environment variable names for a single field; first found wins. Use `validation_alias` for env var lookup without affecting serialization.

```python
api_key: str = Field(
    validation_alias=AliasChoices('API_KEY', 'LEGACY_API_KEY', 'auth_key')
)
```

**Use when:** You need backward compatibility or multiple naming conventions

### env_nested_delimiter
Flattens nested models into environment variables (e.g., `FOO__BAR=1` → `Foo(bar=1)`). Use with `env_nested_max_split` when field names contain the delimiter.

**Use when:** You have hierarchical configuration that should flatten to env vars
**How:** Set `env_nested_delimiter='__'`; add `env_nested_max_split=1` if field names contain `_`

### nested_model_default_partial_update
Controls whether env vars partially update nested model defaults or replace them entirely. `True` merges; `False` replaces.

**Use when:** You want to override only some fields of a nested model
**How:** Set `nested_model_default_partial_update=True` to preserve unset defaults

### CLI Integration
Enable CLI parsing with `cli_parse_args=True`. CLI is the topmost source by default. Use `CliSubCommand` for subcommands and `CliPositionalArg` for positional args.

```python
class App(BaseSettings, cli_parse_args=True, cli_enforce_required=True):
    verbose: bool = False
    deploy: CliSubCommand[Deploy]
```

**Use when:** Building CLI tools that need config from multiple sources
**How:** Enable CLI parsing; define subcommands with `cli_cmd` methods

### Secrets Management
Secrets are lowest priority; env vars and dotenv always override them. Use `NestedSecretsSettingsSource` for directory-based organization. Cloud providers (AWS, Azure, GCP) have dedicated sources.

**Use when:** Storing sensitive configuration in files or cloud secret managers
**How:** Set `secrets_dir='/run/secrets'`; use `NestedSecretsSettingsSource` for nested layouts

### Source Priority Control
Override `settings_customise_sources` to reorder or add/remove sources. Default: CLI > init > env > dotenv > secrets.

**Use when:** Default priority doesn't match your testing or deployment needs
**How:** Implement class method returning tuple of sources in desired order

### In-place Reload
Call `settings.from_env()` to update values without creating a new instance. Useful for long-running processes.

**Use when:** You need to refresh config in long-running processes
**How:** Call `settings.from_env()` after env vars change

---

## Chapter Index

| # | Title | Key Frameworks |
|---|-------|----------------|
| [ch01](chapters/ch01-getting-started.md) | Getting Started | BaseSettings, SettingsConfigDict, Field Aliases |
| [ch02](chapters/ch02-environment-variables.md) | Environment Variables | env_prefix, env_nested_delimiter, case_sensitive |
| [ch03](chapters/ch03-nested-models.md) | Nested Models | nested_model_default_partial_update, env_nested_delimiter |
| [ch04](chapters/ch04-dotenv-support.md) | Dotenv Support | env_file, env_file_depth, dotenv_filtering |
| [ch05](chapters/ch05-cli-support.md) | CLI Support | CliSubCommand, CliPositionalArg, CliApp |
| [ch06](chapters/ch06-secrets.md) | Secrets Management | SecretsSettingsSource, NestedSecretsSettingsSource |
| [ch07](chapters/ch07-other-sources.md) | Other Sources | TomlConfigSettingsSource, deep_merge, table headers |
| [ch08](chapters/ch08-advanced.md) | Advanced Topics | settings_customise_sources, caching, in-place reload |

## Topic Index

- **BaseSettings** → ch01
- **env_prefix** → ch01, ch02
- **AliasChoices** → ch01
- **env_nested_delimiter** → ch02, ch03
- **case_sensitive** → ch02
- **nested_model_default_partial_update** → ch03
- **env_file** → ch04
- **dotenv_filtering** → ch04
- **CliSubCommand** → ch05
- **CliPositionalArg** → ch05
- **CliApp** → ch05
- **SecretsSettingsSource** → ch06
- **NestedSecretsSettingsSource** → ch06
- **secrets_dir** → ch06
- **TomlConfigSettingsSource** → ch07
- **deep_merge** → ch07
- **settings_customise_sources** → ch08
- **cache_settings** → ch08
- **from_env()** → ch08

## Supporting Files

- [glossary.md](glossary.md) — all key terms with definitions
- [patterns.md](patterns.md) — all techniques and design patterns
- [cheatsheet.md](cheatsheet.md) — quick reference tables and decision guides

---

## Scope & Limits

This skill covers the pydantic-settings library configuration patterns only. For hands-on implementation in your codebase, combine with project-specific tools. For topics beyond this library, check related skills or ask the agent directly.
