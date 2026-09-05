# Chapter 1: Getting Started & Setup

## Core Idea
VS Code is a lightweight, extensible code editor that supports multiple languages and workflows through extensions and built-in features.

## Frameworks Introduced
- **Extension-based Architecture**: Install extensions from the marketplace to add language support, themes, debuggers, and tools
- **Settings Sync**: Synchronize settings, keybindings, extensions across machines via GitHub account
- **Profiles**: Create separate configurations for different workflows (e.g., Python dev vs. web dev)

## Key Concepts
- **Workspace**: Project folder opened in VS Code; contains `.vscode/` directory for workspace-specific settings
- **User Settings**: Global configuration (`settings.json` in user directory)
- **Workspace Settings**: Project-specific overrides (`.vscode/settings.json`)
- **Keybindings**: Customizable keyboard shortcuts (`keybindings.json`)
- **Extensions**: Install from marketplace; manage via Extensions view (`Ctrl+Shift+X`)
- **Command Palette**: `Ctrl+Shift+P` — access all commands and features
- **Quick Open**: `Ctrl+P` — open files by name

## Mental Models
- Use **Command Palette** when you don't know where a feature lives
- Use **Quick Open** for fast file navigation without touching the explorer
- Use **Profiles** when you need different extension sets for different projects

## Code Examples
```json
// .vscode/settings.json (workspace settings)
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.tabSize": 2,
  "editor.wordWrap": "on"
}
```

## Key Takeaways
1. Start with Command Palette (`Ctrl+Shift+P`) to discover features
2. Use Settings Sync to maintain consistent setup across machines
3. Create Profiles for different development contexts
4. Keep workspace settings in `.vscode/` for team consistency
5. Extensions are the primary way to add functionality

## Connects To
- **Ch 2**: Editing features build on these foundations
- **Ch 3**: AI agents integrate into the editor workflow
- **Ch 14**: Reference for all settings and keybindings
