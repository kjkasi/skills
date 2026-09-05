# Chapter 3: Nested Models and Partial Updates

## Core Idea
Pydantic Settings supports hierarchical configuration through nested BaseModel classes, with fine-grained control over how partial updates from environment variables merge with default values.

## Frameworks Introduced
- **nested_model_default_partial_update**: Controls whether env vars partially update nested model defaults or replace them entirely
- **NestedSecretsSettingsSource**: Drop-in replacement for `SecretsSettingsSource` with nested directory support
- **env_nested_delimiter**: Flattens nested models into environment variables

## Key Concepts
- **Partial Update (default=False)**: New SubModel instance created; only explicitly set fields change
- **Partial Update (True)**: Existing SubModel instance updated; unset fields keep defaults
- **Sub-model Inheritance**: Nested models must inherit from `pydantic.BaseModel`, not `BaseSettings`
- **Nested Secrets**: Secret files can mirror model hierarchy via directory structure

## Mental Models
- Think of `nested_model_default_partial_update=False` as "replace the whole object if any env var touches it"
- Think of `nested_model_default_partial_update=True` as "merge env vars into the existing object"
- Use `env_nested_delimiter` to flatten hierarchical config: `APP_DB__HOST=localhost` → `Settings(db=DbSettings(host='localhost'))`

## Anti-patterns
- **Using BaseSettings for nested models**: Only `BaseModel` works correctly for nested config
- **Forgetting JSON encoding**: Nested models from env vars must be valid JSON
- **Mismatched delimiters**: `env_nested_delimiter` must match your env var naming convention

## Code Examples
```python
import os
from pydantic import BaseModel
from pydantic_settings import BaseSettings, SettingsConfigDict

class SubModel(BaseModel):
    val: int = 0
    flag: bool = False

class SettingsPartialUpdate(BaseSettings):
    model_config = SettingsConfigDict(
        env_nested_delimiter='__',
        nested_model_default_partial_update=True,
    )
    nested_model: SubModel = SubModel(val=1)

# NESTED_MODEL__FLAG=True
# Result: {'nested_model': {'val': 1, 'flag': True}}  # val preserved

class SettingsNoPartialUpdate(BaseSettings):
    model_config = SettingsConfigDict(
        env_nested_delimiter='__',
        nested_model_default_partial_update=False,
    )
    nested_model: SubModel = SubModel(val=1)

# NESTED_MODEL__FLAG=True
# Result: {'nested_model': {'val': 0, 'flag': True}}  # val reset to default
```

## Reference Tables
| Config | Effect |
|---|---|
| `nested_model_default_partial_update=True` | Env vars merge into existing nested model defaults |
| `nested_model_default_partial_update=False` | New nested model created; only env-set fields change |

## Worked Example
```python
import os
from pydantic import BaseModel
from pydantic_settings import BaseSettings, SettingsConfigDict

class DatabaseConfig(BaseModel):
    host: str = 'localhost'
    port: int = 5432
    name: str = 'mydb'

class Settings(BaseSettings):
    model_config = SettingsConfigDict(
        env_nested_delimiter='__',
        nested_model_default_partial_update=True,
    )
    database: DatabaseConfig = DatabaseConfig()

# Only set one field via env var
os.environ['DATABASE__PORT'] = '3306'

config = Settings()
print(config.database)
# DatabaseConfig(host='localhost', port=3306, name='mydb')
# host and name kept defaults; port overridden
```

## Key Takeaways
1. `nested_model_default_partial_update=True` preserves existing defaults when only some fields are set
2. `nested_model_default_partial_update=False` creates a fresh instance with only env-set fields
3. Nested models must inherit from `BaseModel`, not `BaseSettings`
4. Use `env_nested_delimiter` to flatten hierarchy: `PARENT__CHILD__FIELD=VALUE`
5. JSON encoding required for complex nested types from env vars

## Connects To
- **Ch 1**: Basic BaseSettings configuration
- **Ch 2**: Environment variable naming and parsing
- **Ch 6**: Secrets with nested directory layouts
- **Ch 8**: Custom source priority for nested models
