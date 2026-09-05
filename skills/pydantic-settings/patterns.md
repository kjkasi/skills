# Patterns — Pydantic Settings

## Pattern 1: Environment Variable Namespace
**When to use**: When you have multiple services or apps that might have conflicting env var names
**How**: Use `env_prefix` to namespace all settings for your application
**Trade-offs**: Adds verbosity to env vars; prevents naming collisions

```python
class MyAppSettings(BaseSettings):
    model_config = SettingsConfigDict(env_prefix='MYAPP_')
    database_url: str
    debug: bool = False
# Reads from: MYAPP_DATABASE_URL, MYAPP_DEBUG
```

## Pattern 2: Multiple Alias Fallback
**When to use**: When you need backward compatibility or multiple naming conventions
**How**: Use `AliasChoices` to accept multiple env var names; first found wins
**Trade-offs**: More complex field definitions; clear fallback behavior

```python
class Settings(BaseSettings):
    api_key: str = Field(
        validation_alias=AliasChoices('API_KEY', 'LEGACY_API_KEY', 'auth_key')
    )
```

## Pattern 3: Nested Config with Delimiter
**When to use**: When you have hierarchical configuration that should flatten to env vars
**How**: Use `env_nested_delimiter` to flatten nested models (e.g., `APP_DB__HOST=localhost`)
**Trade-offs**: Env vars become longer; clear mapping to model structure

```python
class DbConfig(BaseModel):
    host: str = 'localhost'
    port: int = 5432

class Settings(BaseSettings):
    model_config = SettingsConfigDict(env_nested_delimiter='__')
    db: DbConfig
# Reads from: DB__HOST, DB__PORT
```

## Pattern 4: Partial Nested Update
**When to use**: When you want to override only some fields of a nested model
**How**: Set `nested_model_default_partial_update=True` to merge instead of replace
**Trade-offs**: Preserves defaults; might surprise if you expect full replacement

```python
class Settings(BaseSettings):
    model_config = SettingsConfigDict(
        env_nested_delimiter='__',
        nested_model_default_partial_update=True,
    )
    db: DbConfig = DbConfig()
# DB__PORT=3306 → DbConfig(host='localhost', port=3306)
```

## Pattern 5: CLI + Env Var Config
**When to use**: When you want settings from env vars with CLI override capability
**How**: Enable `cli_parse_args=True`; CLI is topmost source by default
**Trade-offs**: CLI args always win; might need to adjust priority for testing

```python
class Settings(BaseSettings, cli_parse_args=True):
    database_url: str
    debug: bool = False
# Can override any env var via CLI: --database_url=...
```

## Pattern 6: Subcommand Architecture
**When to use**: When building CLI tools with multiple operations (like git)
**How**: Use `CliSubCommand` for each operation; implement `cli_cmd` methods
**Trade-offs**: Clean separation; limited to single subparser per model

```python
class Deploy(BaseModel):
    environment: CliPositionalArg[str]
    def cli_cmd(self) -> None: ...

class App(BaseSettings):
    deploy: CliSubCommand[Deploy]
    def cli_cmd(self) -> None: CliApp.run_subcommand(self)
```

## Pattern 7: Multiple Config Files
**When to use**: When you want base config with environment-specific overrides
**How**: Pass list of files to config source; later files override earlier ones
**Trade-offs**: Clear precedence; shallow merge by default (use `deep_merge=True`)

```python
class Settings(BaseSettings):
    model_config = SettingsConfigDict(
        toml_file=['config.base.toml', 'config.prod.toml']
    )
```

## Pattern 8: Custom Source Priority
**When to use**: When default priority doesn't match your needs
**How**: Override `settings_customise_sources` to reorder or add sources
**Trade-offs**: Full control; must understand source lifecycle

```python
class Settings(BaseSettings):
    @classmethod
    def settings_customise_sources(cls, settings_cls, init_settings,
                                    env_settings, dotenv_settings,
                                    file_secret_settings):
        return (env_settings, init_settings, dotenv_settings, file_secret_settings)
```

## Pattern 9: Secret File Organization
**When to use**: When storing secrets in files (e.g., Docker Secrets)
**How**: Use `NestedSecretsSettingsSource` for directory-based organization
**Trade-offs**: Clear structure; requires custom source setup

```python
class Settings(BaseSettings):
    @classmethod
    def settings_customise_sources(cls, settings_cls, init_settings,
                                    env_settings, dotenv_settings,
                                    file_secret_settings):
        return (
            init_settings, env_settings, dotenv_settings,
            NestedSecretsSettingsSource(file_secret_settings),
        )
```

## Pattern 10: In-place Reload
**When to use**: When you need to refresh config in long-running processes
**How**: Call `settings.from_env()` to update values without new instance
**Trade-offs**: Avoids re-reading all sources; might miss some changes

```python
settings = Settings()
# ... time passes, env vars change ...
settings.from_env()  # Updates in place
```
