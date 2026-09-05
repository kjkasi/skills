# Cheatsheet

## Quick Reference Tables

### Essential Keyboard Shortcuts

| Action | Windows/Linux | macOS |
|--------|---------------|-------|
| Command Palette | `Ctrl+Shift+P` | `Cmd+Shift+P` |
| Quick Open | `Ctrl+P` | `Cmd+P` |
| Settings | `Ctrl+,` | `Cmd+,` |
| Extensions | `Ctrl+Shift+X` | `Cmd+Shift+X` |
| Source Control | `Ctrl+Shift+G` | `Cmd+Shift+G` |
| Terminal | `Ctrl+`` | `Cmd+`` |
| Debug | `F5` | `F5` |
| Find in Files | `Ctrl+Shift+F` | `Cmd+Shift+F` |
| Toggle Sidebar | `Ctrl+B` | `Cmd+B` |
| Zen Mode | `Ctrl+K Z` | `Cmd+K Z` |

### Debugging Actions

| Action | Shortcut | Description |
|--------|----------|-------------|
| Start/Continue | `F5` | Start debugging or continue |
| Step Over | `F10` | Execute next line |
| Step Into | `F11` | Enter function |
| Step Out | `Shift+F11` | Exit current function |
| Restart | `Ctrl+Shift+F5` | Restart debugging |
| Stop | `Shift+F5` | Stop debugging |

### Editor Actions

| Action | Shortcut | Description |
|--------|----------|-------------|
| Multi-cursor | `Alt+Click` | Add cursor at position |
| Column Selection | `Shift+Alt+Drag` | Select rectangular block |
| Duplicate Line | `Shift+Alt+Down` | Copy line below |
| Move Line | `Alt+Down/Up` | Move line up/down |
| Delete Line | `Ctrl+Shift+K` | Delete current line |
| Comment | `Ctrl+/` | Toggle line comment |

## Decision Rules

### When to use which Remote Development method?
- **SSH required** → Remote - SSH (traditional remote servers)
- **Windows + Linux needed** → Remote - WSL (native Linux on Windows)
- **Environment consistency** → Remote - Containers (Docker-based)
- **Quick access, no SSH setup** → Remote - Tunnels (VS Code tunneling)

### When to use Workspace vs User Settings?
- **Team-shared config** → Workspace settings (`.vscode/settings.json`)
- **Personal preference** → User settings (`settings.json`)
- **Project override** → Folder settings (`.vscode/settings.json` in subfolder)

### When to use which debugging approach?
- **Interactive debugging** → Launch configuration with breakpoints
- **Non-invasive logging** → Logpoints (no code changes needed)
- **Production debugging** → Remote Attach (connect to running process)
- **Quick inspection** → Debug Console REPL

### When to use which testing approach?
- **Visual test management** → Test Explorer
- **CI/CD integration** → Command-line test runner
- **Coverage analysis** → Test Explorer with coverage enabled
- **Watch mode** → Continuous testing during development

### When to use which AI assistance?
- **Quick code suggestion** → Inline Chat (`Ctrl+Shift+I`)
- **Exploratory question** → Chat view (Ask mode)
- **Complex task execution** → Agent mode (Edit mode)
- **Code review** → Agent with specific instructions

## Trade-off Matrices

### Extension vs Built-in Feature

| Factor | Extension | Built-in |
|--------|-----------|----------|
| Functionality | Rich, specialized | Basic, general |
| Performance | May impact startup | Optimized |
| Maintenance | Developer-dependent | VS Code team |
| Consistency | Varies | Uniform |

### Workspace vs User Settings

| Factor | Workspace | User |
|--------|-----------|------|
| Scope | Current project | All projects |
| Sharing | Via version control | Personal only |
| Override | Yes (higher priority) | Default |
| Team use | Recommended | Avoid for team settings |

### Debug Configuration Approaches

| Approach | Setup Time | Flexibility | Best For |
|----------|------------|-------------|----------|
| Launch Config | Medium | High | Standard debugging |
| Attach Config | Low | High | Running processes |
| Compound Config | High | Very High | Multi-process debugging |
| Auto Attach | None | Low | Quick Node.js debugging |

## Thresholds & Defaults

### Performance Thresholds
- **Extension count**: Keep under 20-30 for optimal startup
- **Workspace size**: Over 10,000 files may slow down IntelliSense
- **Debug session memory**: Monitor for large applications

### Security Defaults
- **Workspace Trust**: Enabled by default; restricts features for untrusted folders
- **Extension permissions**: Review before installing
- **Telemetry**: Can be disabled in settings

### Configuration Defaults
- **Tab size**: 4 spaces (configurable per language)
- **Font size**: 14px (adjust for screen size)
- **Auto save**: Off by default (enable for convenience)

## Tells & Smells

### Good Signs
- IntelliSense responds quickly
- Debugging launches without configuration errors
- Tests run automatically in Test Explorer
- Extensions don't conflict with each other

### Warning Signs
- Slow startup with many extensions
- IntelliSense not working for a language
- Debugging fails to launch
- Extension conflicts causing errors

### Red Flags
- Storing secrets in settings.json
- Ignoring workspace trust warnings
- Debugging production without proper security
- Using outdated extensions with known vulnerabilities

## Quick Setup Checklist

### New Project Setup
1. Initialize version control
2. Create `.vscode/settings.json` with project-specific config
3. Create `.vscode/extensions.json` with recommended extensions
4. Create `.vscode/launch.json` if debugging needed
5. Create `.vscode/tasks.json` if build automation needed

### Language Setup
1. Install language extension from marketplace
2. Configure interpreter/compiler path if needed
3. Set up linting and formatting
4. Configure debugging if applicable
5. Set up testing framework

### Team Setup
1. Commit `.vscode/` folder to version control
2. Define recommended extensions in extensions.json
3. Use workspace settings for shared config
4. Avoid user-specific settings in workspace
5. Document custom keybindings or tasks
