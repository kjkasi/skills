# Chapter 5: Command Line Interface Support

## Core Idea
Pydantic Settings provides integrated CLI support via argparse, enabling you to override settings from command-line arguments. Supports subcommands, positional arguments, mutual exclusion, and extensive customization.

## Frameworks Introduced
- **CliSubCommand**: Define subcommands (like `git clone`, `git init`)
- **CliPositionalArg**: Define positional arguments
- **CliApp**: Utility class for running settings as CLI applications
- **CliMutuallyExclusiveGroup**: Group fields that cannot be used together
- **CliImplicitFlag/CliExplicitFlag**: Control boolean flag behavior

## Key Concepts
- **cli_parse_args=True**: Enable CLI argument parsing
- **CLI is Topmost Source**: CLI args override env vars by default (unless priority customized)
- **Subcommands**: Single subparser per model; all subcommands grouped together
- **Async Commands**: `cli_cmd` methods can be async; only use at leaf subcommands
- **cli_cmd Method**: Define custom logic that runs after parsing

## Mental Models
- CLI is just another settings source with highest priority by default
- Use `CliSubCommand` for verb-like operations (clone, init, deploy)
- Use `CliPositionalArg` for required inputs that don't need flags
- `CliApp.run` orchestrates parsing → instantiation → cli_cmd execution

## Anti-patterns
- **Defining async on parent commands**: Only use async at leaf subcommands to avoid extra threads
- **Forgetting cli_enforce_required**: Required fields aren't strictly required at CLI unless this is set
- **Using aliases on union subcommands**: Can cause complications; use separate fields instead

## Code Examples
```python
import sys
from pydantic import BaseModel
from pydantic_settings import (
    BaseSettings,
    CliPositionalArg,
    CliSubCommand,
    CliApp,
    get_subcommand,
)

class Init(BaseModel):
    directory: CliPositionalArg[str]
    
    def cli_cmd(self) -> None:
        print(f'git init "{self.directory}"')

class Clone(BaseModel):
    repository: CliPositionalArg[str]
    directory: CliPositionalArg[str]
    
    def cli_cmd(self) -> None:
        print(f'Cloning {self.repository} into {self.directory}')

class Git(BaseSettings):
    clone: CliSubCommand[Clone]
    init: CliSubCommand[Init]
    
    def cli_cmd(self) -> None:
        CliApp.run_subcommand(self)

# Run: python app.py clone repo dest
cmd = CliApp.run(Git, cli_args=['clone', 'repo', 'dest'])
```

## Reference Tables
| Config/Annotation | Purpose |
|---|---|
| `cli_parse_args` | Enable CLI parsing (bool or argparse settings) |
| `cli_enforce_required` | Make required fields strictly required at CLI |
| `cli_avoid_json` | Avoid JSON strings for complex fields |
| `cli_kebab_case` | Convert field names to kebab-case on CLI |
| `cli_implicit_flags` | Boolean flags: `'toggle'`, `'dual'`, or `True` |
| `cli_show_env_vars` | Show env var names in help text |
| `cli_exit_on_error` | Exit on parse error (default: True) |
| `CliSubCommand` | Define a subcommand field |
| `CliPositionalArg` | Define a positional argument field |
| `CliApp.run()` | Run settings as CLI app |
| `get_subcommand()` | Retrieve parsed subcommand instance |

## Worked Example
```python
import sys
from pydantic import BaseModel
from pydantic_settings import BaseSettings, CliApp, CliPositionalArg, CliSubCommand

class Deploy(BaseModel):
    environment: CliPositionalArg[str]
    tag: str = 'latest'
    
    def cli_cmd(self) -> None:
        print(f'Deploying {self.tag} to {self.environment}')

class App(BaseSettings, cli_parse_args=True, cli_enforce_required=True):
    verbose: bool = False
    deploy: CliSubCommand[Deploy]
    
    def cli_cmd(self) -> None:
        CliApp.run_subcommand(self)

# python app.py deploy production --tag=v1.2.3
cmd = CliApp.run(App, cli_args=['deploy', 'production', '--tag=v1.2.3'])
print(cmd.model_dump())
# {'verbose': False, 'deploy': {'environment': 'production', 'tag': 'v1.2.3'}}
```

## Key Takeaways
1. Enable CLI parsing with `cli_parse_args=True` or by inheriting the setting
2. CLI is the topmost source by default; use `settings_customise_sources` to change priority
3. Use `CliSubCommand` for subcommands and `CliPositionalArg` for positional args
4. Only define `async cli_cmd` at leaf subcommands to avoid unnecessary threads
5. `CliApp.run` handles parsing → instantiation → cli_cmd execution in one call

## Connects To
- **Ch 1**: Basic BaseSettings configuration
- **Ch 2**: Environment variable sources (CLI overrides env by default)
- **Ch 8**: Custom source priority and settings_customise_sources
- **Ch 4**: Dotenv files (another source with lower priority)
