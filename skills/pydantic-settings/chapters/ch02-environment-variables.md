# Chapter 2: Environment Variable Configuration

## Core Idea
Environment variables are the primary source for pydantic-settings configuration. The library provides fine-grained control over naming, parsing, case sensitivity, and nested variable handling.

## Frameworks Introduced
- **env_prefix_target**: Control whether prefix applies to variable names, aliases, or both
- **env_nested_delimiter**: Explode flat env vars into nested models (e.g., `FOO__BAR=1` → `Foo(bar=1)`)
- **env_nested_max_split**: Limit nesting depth when delimiter appears in field names
- **NoDecode/ForceDecode**: Override JSON parsing behavior per-field

## Key Concepts
- **Environment Variable Names**: By default, field name = env var name (with optional prefix)
- **Alias Mapping**: Use `alias` or `validation_alias` to change env var names per field
- **Case Sensitivity**: Default is case-insensitive; set `case_sensitive=True` for exact matching
- **JSON Parsing**: Complex types (list, dict, nested models) are parsed as JSON strings
- **env_ignore_empty**: Skip empty env vars, using default values instead

## Mental Models
- Think of `env_prefix_target` as controlling where the prefix is applied: `variable` (default), `alias`, or `all`
- Use `env_nested_delimiter` to flatten hierarchical config into env vars (e.g., `APP_DB__HOST=localhost`)
- `env_nested_max_split` prevents delimiter collisions in field names (e.g., `GENERATION_LLM_API_KEY`)

## Anti-patterns
- **Forgetting JSON encoding for complex types**: `NUMBERS=1,2,3` fails; use `NUMBERS=[1,2,3]`
- **Not considering case on Windows**: Windows env vars are case-insensitive; `case_sensitive` has no effect
- **Using nested delimiter with non-declared fields**: Only declared fields get nested parsing

## Code Examples
```python
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    model_config = SettingsConfigDict(
        env_prefix='APP_',
        env_prefix_target='all',  # Apply prefix to both vars and aliases
        env_nested_delimiter='__',
        env_nested_max_split=1,
        case_sensitive=False,
    )
    
    host: str = 'localhost'
    port: int = 8000

# Reads from: APP_HOST, APP_PORT (variable names)
# Also applies prefix to aliases if set
```

## Reference Tables
| Config Option | Default | Values |
|---|---|---|
| `env_prefix_target` | `'variable'` | `'variable'`, `'alias'`, `'all'` |
| `env_nested_delimiter` | `None` | Any string (e.g., `'__'`, `'_'`) |
| `env_nested_max_split` | `None` | Integer (max nesting depth) |
| `case_sensitive` | `False` | `True`, `False` |
| `env_ignore_empty` | `False` | `True`, `False` |
| `enable_decoding` | `True` | `True`, `False` (global JSON parsing) |

## Worked Example
```python
import os
from pydantic import BaseModel
from pydantic_settings import BaseSettings, SettingsConfigDict

class LLMConfig(BaseModel):
    provider: str = 'openai'
    api_key: str
    api_version: str = '2024-03-15'

class GenerationConfig(BaseSettings):
    model_config = SettingsConfigDict(
        env_nested_delimiter='_',
        env_nested_max_split=1,
        env_prefix='GENERATION_',
    )
    llm: LLMConfig

# Environment:
# GENERATION_LLM_PROVIDER=anthropic
# GENERATION_LLM_API_KEY=your-key
# GENERATION_LLM_API_VERSION=2024-03-15

config = GenerationConfig()
print(config.llm.provider)  # 'anthropic'
print(config.llm.api_key)   # 'your-key'
```

## Key Takeaways
1. Use `env_prefix` for namespacing; `env_prefix_target` controls where it applies
2. `env_nested_delimiter` flattens nested models into env vars (`FOO__BAR=1`)
3. Set `env_nested_max_split` when field names contain the delimiter
4. Complex types must be valid JSON in env vars (`[1,2,3]` not `1,2,3`)
5. Windows ignores `case_sensitive` for env vars; use for dotenv files only

## Connects To
- **Ch 1**: Basic BaseSettings usage
- **Ch 3**: Nested model configuration details
- **Ch 4**: Dotenv file parsing (similar rules)
- **Ch 8**: Debugging which source provides values
