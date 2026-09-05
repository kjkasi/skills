# Chapter 8: Advanced Topics

## Core Idea
Pydantic Settings provides advanced features for source priority control, debugging, custom sources, in-place reloading, caching, and async environment support.

## Frameworks Introduced
- **settings_customise_sources**: Override source priority and add/remove sources
- **PydanticBaseSettingsSource**: Base class for custom settings sources
- **In-place Reloading**: Update settings without recreating the instance
- **Caching**: Cache settings to avoid repeated source reads
- **Async Environments**: Support for async initialization and source loading

## Key Concepts
- **Source Priority**: Default order: init > env > dotenv > secrets; CLI is topmost if enabled
- **Custom Sources**: Subclass `PydanticBaseSettingsSource` for custom logic
- **In-place Reload**: `settings.from_env()` updates values without new instance
- **Caching**: `settings/cache_settings` caches source values; use `Settings(_cache_settings=False)` to disable
- **Async Support**: Async initialization and source loading for async environments

## Mental Models
- `settings_customise_sources` is the control plane for source priority
- Use caching when sources are expensive to read (network calls, file I/O)
- In-place reload is for long-running processes that need to refresh config

## Anti-patterns
- **Ignoring source priority**: Understand the default order before customizing
- **Not caching expensive sources**: Network-based sources should be cached
- **Using async at top level**: Only use async at leaf subcommands

## Code Examples
```python
from pydantic_settings import (
    BaseSettings,
    PydanticBaseSettingsSource,
    CliSettingsSource,
)

class Settings(BaseSettings):
    my_foo: str
    
    @classmethod
    def settings_customise_sources(
        cls,
        settings_cls: type[BaseSettings],
        init_settings: PydanticBaseSettingsSource,
        env_settings: PydanticBaseSettingsSource,
        dotenv_settings: PydanticBaseSettingsSource,
        file_secret_settings: PydanticBaseSettingsSource,
    ) -> tuple[PydanticBaseSettingsSource, ...]:
        return (
            init_settings,
            env_settings,
            dotenv_settings,
            file_secret_settings,
            CliSettingsSource(settings_cls, cli_parse_args=True),
        )
```

## Reference Tables
| Feature | Config/Method | Description |
|---|---|---|
| Source Priority | `settings_customise_sources` | Control source order |
| Caching | `model_config = {'cache_settings': True}` | Cache source values |
| In-place Reload | `settings.from_env()` | Update without new instance |
| Debug | `settings.settings_sources_tasks` | Inspect source loading |
| Async | `async def settings_customise_sources` | Async source loading |

### Default Source Priority (without CLI)
1. `init_settings` (kwargs)
2. `env_settings` (environment variables)
3. `dotenv_settings` (.env files)
4. `file_secret_settings` (secrets files)

### Default Source Priority (with CLI)
1. CLI settings (if `cli_parse_args=True`)
2. `init_settings`
3. `env_settings`
4. `dotenv_settings`
5. `file_secret_settings`

## Worked Example
```python
import os
from pydantic_settings import BaseSettings, CliSettingsSource

class Settings(BaseSettings):
    my_foo: str
    
    @classmethod
    def settings_customise_sources(cls, settings_cls, init_settings,
                                    env_settings, dotenv_settings,
                                    file_secret_settings):
        # Custom order: env > init > dotenv > secrets
        return (
            env_settings,
            init_settings,
            dotenv_settings,
            file_secret_settings,
        )

# Debug: Check which source provided each value
os.environ['MY_FOO'] = 'from_env'
settings = Settings()
print(settings.settings_sources_tasks)
# Shows which sources were consulted and their results

# In-place reload
os.environ['MY_FOO'] = 'updated'
settings.from_env()  # Updates settings without new instance
print(settings.my_foo)  # 'updated'
```

## Key Takeaways
1. `settings_customise_sources` controls source priority and composition
2. Default priority: init < env < dotenv < secrets; CLI is topmost if enabled
3. Cache expensive sources with `cache_settings=True` to avoid repeated reads
4. Use `from_env()` for in-place reloading in long-running processes
5. Debug with `settings_sources_tasks` to see which source provided each value

## Connects To
- **Ch 1**: Basic BaseSettings configuration
- **Ch 5**: CLI as a source with custom priority
- **Ch 6**: Secrets as a source
- **All Chapters**: Source priority affects all configuration methods
