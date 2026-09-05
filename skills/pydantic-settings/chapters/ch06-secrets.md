# Chapter 6: Secrets Management

## Core Idea
Pydantic Settings supports loading secrets from files, AWS Secrets Manager, AWS Systems Manager Parameter Store, Azure Key Vault, and Google Cloud Secret Manager — with nested model support and flexible configuration.

## Frameworks Introduced
- **SecretsSettingsSource**: Load secrets from files where filename = key
- **NestedSecretsSettingsSource**: Drop-in replacement supporting nested directories
- **AWSSecretsManagerSettingsSource**: Load from AWS Secrets Manager
- **AWSSystemsManagerSettingsSource**: Load from AWS Systems Manager Parameter Store
- **AzureKeyVaultSettingsSource**: Load from Azure Key Vault
- **GoogleSecretManagerSettingsSource**: Load from Google Cloud Secret Manager

## Key Concepts
- **Secret Files**: File content = value, filename = key (e.g., `/var/run/db_password` contains password)
- **Priority**: Env vars and dotenv files always override secrets
- **Multiple Directories**: Pass tuple/list to `secrets_dir`; last match wins
- **Docker Secrets**: Use `/run/secrets` as default mount point
- **Nested Secrets**: `NestedSecretsSettingsSource` supports subdirectories mirroring model hierarchy

## Mental Models
- Secrets are the lowest-priority source; env vars and dotenv always win
- Use `NestedSecretsSettingsSource` when you need directory-based secret organization
- Cloud secret managers are just specialized sources with their own auth/config

## Anti-patterns
- **Storing secrets in code**: Use secrets management instead
- **Ignoring secrets_dir_missing**: Set to `'ok'` or `'error'` if missing dirs should fail silently or loudly
- **Not limiting secrets_dir_max_size**: Prevents memory issues with large mounted secrets

## Code Examples
```python
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    model_config = SettingsConfigDict(secrets_dir='/run/secrets')
    
    database_password: str
    api_key: str

# File structure:
# /run/secrets/database_password  (contains: super_secret_pw)
# /run/secrets/api_key            (contains: abc123)
```

## Reference Tables
| Source | Config | Key Requirement |
|---|---|---|
| Secret Files | `secrets_dir` | Filename = field name |
| AWS Secrets Manager | `secret_id` | Secret ID in AWS |
| AWS Systems Manager | `ssm_path` | Parameter path (default: `/`) |
| Azure Key Vault | `url`, `credential` | Vault URL + Azure credentials |
| Google Secret Manager | `project_id` | GCP project ID |

### NestedSecretsSettingsSource Options
| Option | Default | Description |
|---|---|---|
| `secrets_dir_missing` | `'warn'` | `'ok'`, `'warn'`, `'error'` |
| `secrets_dir_max_size` | 16 MiB | Limit total secrets size |
| `secrets_case_sensitive` | `case_sensitive` | Override case sensitivity |
| `secrets_nested_delimiter` | `env_nested_delimiter` | Delimiter for nested secrets |
| `secrets_nested_subdir` | `False` | Use subdirectories for nesting |
| `secrets_prefix` | `env_prefix` | Prefix for secret paths |

## Worked Example
```python
import os
from pydantic import BaseModel, SecretStr
from pydantic_settings import (
    BaseSettings,
    NestedSecretsSettingsSource,
    SettingsConfigDict,
)

class DbSettings(BaseModel):
    user: str
    passwd: SecretStr

class Settings(BaseSettings):
    db: DbSettings
    
    model_config = SettingsConfigDict(
        env_prefix='MY_',
        env_nested_delimiter='__',
        secrets_dir='secrets',
        secrets_nested_delimiter='_',
    )
    
    @classmethod
    def settings_customise_sources(cls, settings_cls, init_settings, 
                                    env_settings, dotenv_settings, 
                                    file_secret_settings):
        return (
            init_settings,
            env_settings,
            dotenv_settings,
            NestedSecretsSettingsSource(file_secret_settings),
        )

# File structure:
# secrets/db_user     (contains: admin)
# secrets/db_passwd   (contains: secret123)
```

## Key Takeaways
1. Secrets are lowest priority; env vars and dotenv always override them
2. Use `NestedSecretsSettingsSource` for directory-based secret organization
3. Set `secrets_dir_missing` to control behavior when secrets dir doesn't exist
4. Cloud secret managers require their own auth configuration and dependencies
5. Always set `secrets_dir_max_size` to prevent memory issues with large secrets

## Connects To
- **Ch 1**: Basic BaseSettings configuration
- **Ch 3**: Nested model support in secrets
- **Ch 4**: Dotenv files (higher priority than secrets)
- **Ch 8**: Custom source priority for secrets
