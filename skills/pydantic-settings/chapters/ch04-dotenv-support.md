# Chapter 4: Dotenv (.env) File Support

## Core Idea
Pydantic Settings loads configuration from `.env` files using python-dotenv, with environment variables always taking priority. Supports multiple files, custom encoding, and filtering.

## Frameworks Introduced
- **env_file**: Path(s) to dotenv files; loaded in order with later files overriding earlier
- **env_file_encoding**: Character encoding for dotenv files (default: OS default)
- **env_file_depth**: Number of parent directories to search for relative dotenv paths
- **dotenv_filtering**: Control which dotenv entries are passed to the model

## Key Concepts
- **Priority**: Environment variables always override dotenv file values
- **Multiple Files**: Pass tuple/list to `env_file`; loaded in order, later overrides earlier
- **Override at Instantiation**: `_env_file=None` disables dotenv loading entirely
- **Named Pipes**: Supported for tools like 1Password Environments
- **Extra Values**: By default, dotenv entries for non-existent fields raise `ValidationError` (with `extra='forbid'`)

## Mental Models
- Think of dotenv files as "environment variable templates" — they're loaded first, then real env vars override
- Use `env_file_depth` when your project has a nested directory structure
- `dotenv_filtering` controls scoping: `'match_prefix'` limits to env_prefix, `'only_existing'` ignores extra fields

## Anti-patterns
- **Storing secrets in dotenv files committed to git**: Use secrets management instead
- **Ignoring priority order**: Remember, real env vars always win over dotenv values
- **Not handling missing files**: Set `env_file_depth` or use absolute paths for reliability

## Code Examples
```python
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    model_config = SettingsConfigDict(
        env_file=('.env', '.env.prod'),  # .env.prod overrides .env
        env_file_encoding='utf-8',
        env_file_depth=2,  # Search 2 levels up for .env
    )
    
    database_url: str
    debug: bool = False

# Override at instantiation
settings = Settings(_env_file='staging.env')
settings = Settings(_env_file=None)  # Skip dotenv entirely
```

## Reference Tables
| Config/Arg | Default | Description |
|---|---|---|
| `env_file` | `None` | Path(s) to dotenv file(s) |
| `env_file_encoding` | OS default | Character encoding |
| `env_file_depth` | `None` | Parent directories to search |
| `dotenv_filtering` | `None` | `'match_prefix'`, `'only_existing'`, or `None` |
| `_env_file` kwarg | `None` | Override `env_file` at instantiation |

## Worked Example
```python
import os
from pydantic_settings import BaseSettings, SettingsConfigDict

# .env file contains:
# DATABASE_URL=postgres://localhost/mydb
# DEBUG=true
# APP_SECRET=secret123

class Settings(BaseSettings):
    model_config = SettingsConfigDict(
        env_file='.env',
        env_prefix='APP_',
        dotenv_filtering='match_prefix',  # Only load APP_* from dotenv
    )
    
    database_url: str
    debug: bool = False
    secret: str

# Real env var overrides dotenv
os.environ['APP_DEBUG'] = 'false'
config = Settings()
print(config.debug)  # False (env var wins)
```

## Key Takeaways
1. Environment variables always take priority over dotenv file values
2. Multiple dotenv files loaded in order; later files override earlier ones
3. Use `_env_file` kwarg to override or disable dotenv loading at instantiation
4. `env_file_depth` helps find `.env` files in nested project structures
5. `dotenv_filtering` scopes which entries are loaded (useful with `env_prefix`)

## Connects To
- **Ch 1**: Basic BaseSettings configuration
- **Ch 2**: Environment variable parsing rules apply to dotenv too
- **Ch 6**: Secrets management as alternative to dotenv
- **Ch 8**: Custom source priority (dotenv is one of the sources)
