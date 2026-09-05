# Chapter 1: Getting Started with Pydantic Settings

## Core Idea
Pydantic Settings extends Pydantic's `BaseSettings` to automatically load configuration from environment variables, dotenv files, secrets, and CLI arguments — with full type validation and IDE support.

## Frameworks Introduced
- **BaseSettings Pattern**: Inherit from `BaseSettings` instead of `BaseModel` to enable automatic environment variable loading
- **SettingsConfigDict**: Configure behavior (prefixes, files, validation) through a typed config dictionary
- **Field Aliases**: Use `alias` or `validation_alias` to map environment variable names to Python field names

## Key Concepts
- **BaseSettings**: Core class that reads values from environment variables when not provided as kwargs
- **env_prefix**: Prefix applied to all environment variable names (default: empty string)
- **AliasChoices**: Allows multiple environment variable names for a single field; first found wins
- **ImportString**: Type that imports an object from a string path (e.g., `'math.cos'`)
- **validate_default**: Whether default values are validated (default: True in BaseSettings, unlike BaseModel)

## Mental Models
- Use `BaseSettings` when your config should come from environment variables
- Think of `env_prefix` as a namespace for all your settings (e.g., `MYAPP_`)
- Use `AliasChoices` when you need backward compatibility or multiple naming conventions

## Anti-patterns
- **Using BaseModel for config**: Won't automatically load from environment variables
- **Hardcoding env var names**: Use aliases instead to keep field names Pythonic
- **Ignoring validate_default**: Default values are validated in BaseSettings; disable explicitly if needed

## Code Examples
```python
from pydantic_settings import BaseSettings, SettingsConfigDict
from pydantic import Field, AliasChoices

class Settings(BaseSettings):
    auth_key: str = Field(validation_alias='my_auth_key')
    api_key: str = Field(alias='my_api_key')
    redis_dsn: str = Field(
        'redis://localhost:6379/1',
        validation_alias=AliasChoices('service_redis_dsn', 'redis_url'),
    )
    
    model_config = SettingsConfigDict(env_prefix='my_prefix_')

# Environment variables read: my_prefix_auth_key, my_prefix_api_key, etc.
settings = Settings()
```

## Reference Tables
| Config Option | Default | Description |
|---|---|---|
| `env_prefix` | `''` | Prefix for all env var names |
| `validate_default` | `True` | Validate default values on instantiation |
| `case_sensitive` | `False` | Env var names are case-insensitive by default |

## Worked Example
```python
import os
from pydantic_settings import BaseSettings, SettingsConfigDict

# Set environment variables
os.environ['APP_NAME'] = 'MyApp'
os.environ['APP_DEBUG'] = 'true'

class AppConfig(BaseSettings):
    name: str
    debug: bool = False
    
    model_config = SettingsConfigDict(env_prefix='APP_')

# Automatically reads from environment
config = AppConfig()
print(config.name)   # 'MyApp'
print(config.debug)  # True

# Can still override via kwargs
config = AppConfig(name='Override', debug=False)
print(config.name)   # 'Override'
```

## Key Takeaways
1. Inherit from `BaseSettings` for automatic environment variable loading
2. Use `env_prefix` to namespace your settings and avoid collisions
3. Prefer `validation_alias` over `alias` when you want env var lookup without affecting serialization
4. `AliasChoices` provides fallback mechanism for multiple naming conventions
5. Default values are validated in BaseSettings — set `validate_default=False` to disable

## Connects To
- **Ch 2**: Environment variable naming and parsing rules
- **Ch 3**: Nested model configuration
- **Ch 4**: Dotenv file support
- **Ch 8**: Custom source priority and debugging
