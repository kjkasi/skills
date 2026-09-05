# Cheatsheet — Pydantic Settings

## Quick Reference: Config Options

| Option | Default | Description |
|---|---|---|
| `env_prefix` | `''` | Prefix for all env var names |
| `env_prefix_target` | `'variable'` | Where prefix applies: `'variable'`, `'alias'`, `'all'` |
| `env_nested_delimiter` | `None` | Flattens nested models to env vars |
| `env_nested_max_split` | `None` | Limits nesting depth |
| `case_sensitive` | `False` | Env var names case-sensitive |
| `env_ignore_empty` | `False` | Skip empty env vars |
| `validate_default` | `True` | Validate default values |
| `nested_model_default_partial_update` | `False` | Partial update nested defaults |
| `cli_parse_args` | `False` | Enable CLI parsing |
| `cli_enforce_required` | `False` | Required fields strictly required at CLI |
| `cache_settings` | `False` | Cache source values |

## Decision Rules

**When to use `alias` vs `validation_alias`:**
- `alias` → affects both validation and serialization
- `validation_alias` → only affects validation; serialization uses field name

**When to use `env_prefix_target`:**
- `'variable'` (default) → prefix only on env var names
- `'alias'` → prefix only on aliases
- `'all'` → prefix on both

**When to use `nested_model_default_partial_update`:**
- `True` → env vars merge into existing defaults (preserve unset fields)
- `False` → new instance created; only env-set fields change

**When to use `env_nested_max_split`:**
- Set when field names contain the delimiter (e.g., `GENERATION_LLM_API_KEY`)
- Prevents delimiter collision: `GENERATION_LLM_API_KEY` → `llm.api_key` not `llm.api.key`

## Source Priority (Default)

1. CLI (if `cli_parse_args=True`)
2. `init_settings` (kwargs)
3. `env_settings` (environment variables)
4. `dotenv_settings` (.env files)
5. `file_secret_settings` (secrets files)

**Override priority:** Use `settings_customise_sources` to reorder

## Common Patterns

**Namespace settings:**
```python
class Settings(BaseSettings):
    model_config = SettingsConfigDict(env_prefix='MYAPP_')
```

**Multiple alias fallback:**
```python
api_key: str = Field(validation_alias=AliasChoices('API_KEY', 'LEGACY_KEY'))
```

**Nested config:**
```python
model_config = SettingsConfigDict(env_nested_delimiter='__')
# Env var: APP__DB__HOST=localhost
```

**CLI app:**
```python
class Settings(BaseSettings, cli_parse_args=True, cli_enforce_required=True):
    ...
```

## Anti-patterns to Avoid

| Anti-pattern | Why It's Bad | Better Approach |
|---|---|---|
| Using `BaseModel` for config | No env var loading | Use `BaseSettings` |
| `NUMBERS=1,2,3` for list | Not valid JSON | Use `NUMBERS=[1,2,3]` |
| Ignoring `env_nested_max_split` | Delimiter collision | Set max split when field names contain delimiter |
| Not caching expensive sources | Repeated network calls | Use `cache_settings=True` |
| Async at parent commands | Extra threads/loops | Only async at leaf subcommands |

## Debugging

**Check which source provided each value:**
```python
settings = Settings()
print(settings.settings_sources_tasks)
```

**Inspect source loading:**
```python
settings = Settings()
print(settings._settings_build_values.sources)
```

## File Paths

| Source | Typical Path |
|---|---|
| Dotenv | `.env`, `.env.local`, `.env.prod` |
| Secrets | `/run/secrets/`, `/var/run/` |
| Config | `config.toml`, `config.json`, `pyproject.toml` |
| AWS Secrets | Secret ID in AWS Secrets Manager |
| AWS SSM | Parameter path (e.g., `/prod/my-app`) |
| Azure Key Vault | `https://my-vault.vault.azure.net/` |
| GCP Secret Manager | Secret name in GCP project |
