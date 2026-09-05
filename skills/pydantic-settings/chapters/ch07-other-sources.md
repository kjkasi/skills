# Chapter 7: Other Settings Sources (JSON, TOML, YAML)

## Core Idea
Pydantic Settings supports loading configuration from JSON, TOML, YAML files, and pyproject.toml — with deep merge support, table headers, and multiple file loading.

## Frameworks Introduced
- **JsonConfigSettingsSource**: Load from JSON files
- **TomlConfigSettingsSource**: Load from TOML files
- **YamlConfigSettingsSource**: Load from YAML files
- **PyprojectTomlConfigSettingsSource**: Load from pyproject.toml `[tool.pydantic-settings]` table

## Key Concepts
- **Multiple Files**: Pass list of paths; files merged shallowly in order (later overrides earlier)
- **Deep Merge**: Set `deep_merge=True` on source to merge nested dicts instead of replacing
- **Table Headers**: Load from specific table in TOML/pyproject.toml files
- **Traversable Support**: Load from packaged resources (zip/wheel) via `importlib.resources`
- **Deep Merge Warning**: Not available through `SettingsConfigDict`; must set on source directly

## Mental Models
- Config files are sources with lower priority than env vars and CLI
- Use `deep_merge=True` when you want nested dicts to merge rather than replace
- `toml_table_header` and `pyproject_toml_table_header` scope which part of the file is read

## Anti-patterns
- **Assuming deep merge by default**: It's shallow; set `deep_merge=True` explicitly
- **Not handling missing files**: Use try/except or validate file existence
- **Confusing priority order**: Config files < env vars < CLI (by default)

## Code Examples
```python
from pydantic_settings import (
    BaseSettings,
    SettingsConfigDict,
    TomlConfigSettingsSource,
)

class Settings(BaseSettings):
    model_config = SettingsConfigDict(toml_file='config.toml')
    
    field: str
    
    @classmethod
    def settings_customise_sources(cls, settings_cls, init_settings,
                                    env_settings, dotenv_settings,
                                    file_secret_settings):
        return (TomlConfigSettingsSource(settings_cls),)
```

## Reference Tables
| Source | Config Key | File Format |
|---|---|---|
| JSON | `json_file` | `.json` |
| TOML | `toml_file` | `.toml` |
| YAML | `yaml_file` | `.yaml`, `.yml` |
| pyproject.toml | (uses `PyprojectTomlConfigSettingsSource`) | `pyproject.toml` |

### Configuration Options
| Option | Applies To | Description |
|---|---|---|
| `deep_merge` | All config sources | Merge nested dicts instead of replacing |
| `toml_table_header` | TOML | Load from specific table (tuple of keys) |
| `pyproject_toml_table_header` | pyproject.toml | Default: `('tool', 'pydantic-settings')` |
| `json_file_encoding` | JSON | Character encoding |
| `yaml_file_encoding` | YAML | Character encoding |

## Worked Example
```python
from pydantic import BaseModel
from pydantic_settings import (
    BaseSettings,
    SettingsConfigDict,
    TomlConfigSettingsSource,
)

class Nested(BaseModel):
    foo: int
    bar: int = 0

class Settings(BaseSettings):
    hello: str
    nested: Nested
    
    model_config = SettingsConfigDict(
        toml_file=['config.default.toml', 'config.custom.toml']
    )
    
    @classmethod
    def settings_customise_sources(cls, settings_cls, init_settings,
                                    env_settings, dotenv_settings,
                                    file_secret_settings):
        return (TomlConfigSettingsSource(settings_cls, deep_merge=True),)

# config.default.toml:
# hello = "World"
# [nested]
# foo = 1
# bar = 2

# config.custom.toml:
# [nested]
# foo = 3

# Result: hello="world", nested.foo=3, nested.bar=2 (bar preserved with deep merge)
```

## Key Takeaways
1. Config files are lower priority than env vars and CLI by default
2. Use `deep_merge=True` to merge nested dicts instead of replacing them
3. `toml_table_header` scopes which table is read from TOML files
4. Multiple config files are merged in order; later files override earlier ones
5. Config sources support `Traversable` objects for packaged resources

## Connects To
- **Ch 1**: Basic BaseSettings configuration
- **Ch 2**: Environment variables (higher priority)
- **Ch 4**: Dotenv files (another config source)
- **Ch 8**: Custom source priority and ordering
