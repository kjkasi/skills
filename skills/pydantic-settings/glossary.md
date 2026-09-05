# Glossary — Pydantic Settings

**AliasChoices** — Allows multiple environment variable names for a single field; first found wins (Ch 1)

**BaseSettings** — Core class that automatically loads values from environment variables (Ch 1)

**cache_settings** — Config option to cache source values for performance (Ch 8)

**case_sensitive** — Controls whether env var names are case-sensitive (default: False) (Ch 2)

**CliApp** — Utility class for running settings as CLI applications (Ch 5)

**CliExplicitFlag** — Annotation for boolean fields that require explicit value on CLI (Ch 5)

**CliImplicitFlag** — Annotation for boolean fields that derive value from flag presence (Ch 5)

**CliMutuallyExclusiveGroup** — Group fields that cannot be used together on CLI (Ch 5)

**CliPositionalArg** — Annotation for positional CLI arguments (Ch 5)

**CliSubCommand** — Annotation for CLI subcommands (Ch 5)

**CliToggleFlag** — Annotation for toggle-style boolean flags (Ch 5)

**CliDualFlag** — Annotation for dual-style boolean flags (Ch 5)

**CliUnknownArgs** — Annotation to capture unknown CLI arguments (Ch 5)

**deep_merge** — Config option to merge nested dicts instead of replacing (Ch 7)

**dotenv_filtering** — Controls which dotenv entries are passed to model (Ch 4)

**env_file** — Path(s) to dotenv files (Ch 4)

**env_file_depth** — Number of parent directories to search for dotenv files (Ch 4)

**env_ignore_empty** — Skip empty env vars, using defaults (Ch 2)

**env_nested_delimiter** — Flattens nested models into env vars (e.g., `FOO__BAR=1`) (Ch 2)

**env_nested_max_split** — Limits nesting depth when delimiter appears in field names (Ch 2)

**env_prefix** — Prefix applied to all environment variable names (Ch 1)

**env_prefix_target** — Controls where prefix applies: `'variable'`, `'alias'`, or `'all'` (Ch 2)

**enable_decoding** — Global toggle for JSON parsing of complex types (Ch 2)

**ForceDecode** — Annotation to force JSON parsing for a field (Ch 2)

**ImportString** — Type that imports an object from a string path (Ch 1)

**JsonConfigSettingsSource** — Load configuration from JSON files (Ch 7)

**NestedSecretsSettingsSource** — Drop-in replacement for SecretsSettingsSource with nested directory support (Ch 6)

**nested_model_default_partial_update** — Controls whether env vars partially update nested model defaults (Ch 3)

**NoDecode** — Annotation to disable JSON parsing for a field (Ch 2)

**PydanticBaseSettingsSource** — Base class for custom settings sources (Ch 8)

**PyprojectTomlConfigSettingsSource** — Load from pyproject.toml files (Ch 7)

**SecretsSettingsSource** — Load secrets from files where filename = key (Ch 6)

**secrets_dir** — Path to secrets directory (Ch 6)

**secrets_dir_max_size** — Limit total secrets size for security (Ch 6)

**secrets_dir_missing** — Behavior when secrets dir doesn't exist: `'ok'`, `'warn'`, `'error'` (Ch 6)

**secrets_nested_delimiter** — Delimiter for nested secrets (Ch 6)

**secrets_nested_subdir** — Use subdirectories for nested secrets (Ch 6)

**secrets_prefix** — Prefix for secret paths (Ch 6)

**settings_customise_sources** — Override source priority and add/remove sources (Ch 8)

**SettingsConfigDict** — Typed configuration dictionary for BaseSettings (Ch 1)

**TomlConfigSettingsSource** — Load configuration from TOML files (Ch 7)

**validate_default** — Whether default values are validated (default: True in BaseSettings) (Ch 1)

**YamlConfigSettingsSource** — Load configuration from YAML files (Ch 7)
