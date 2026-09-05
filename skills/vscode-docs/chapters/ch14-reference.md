# Chapter 14: Reference & Configuration

## Core Idea
VS Code provides extensive configuration options through settings, keybindings, and extensions, with defaults, user overrides, and workspace-specific settings.

## Frameworks Introduced
- **Settings Hierarchy**: Default → User → Workspace → Folder settings
- **Keybinding System**: Customizable keyboard shortcuts with When clauses
- **Extension API**: How extensions extend VS Code functionality

## Key Concepts
- **settings.json**: All configuration options; searchable via Settings editor
- **keybindings.json**: Custom keyboard shortcuts; when-clause conditionals
- **Default Settings**: Built-in defaults; search "default" to see all
- **Workspace Settings**: `.vscode/settings.json` for project-specific config
- **Task Configuration**: `tasks.json` for build and automation tasks
- **Launch Configuration**: `launch.json` for debug scenarios

## Mental Models
- Use **Settings editor** (`Ctrl+,`) for visual configuration
- Use **keybindings.json** for advanced keybinding customization
- Use **Workspace Settings** to share configuration with team via version control

## Anti-patterns
- **Overriding defaults blindly**: Understand what default does before changing
- **Hardcoding paths**: Use `${workspaceFolder}`, `${env:VAR}` variables
- **Ignoring setting scope**: Understand user vs. workspace vs. folder settings

## Code Examples
```json
// keybindings.json (custom keybinding with when clause)
[
  {
    "key": "ctrl+shift+f",
    "command": "workbench.action.findInFiles",
    "when": "editorTextFocus"
  },
  {
    "key": "ctrl+k ctrl+d",
    "command": "editor.action.duplicateSelection"
  }
]
```

```json
// tasks.json (build task)
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Build",
      "type": "shell",
      "command": "npm run build",
      "group": {
        "kind": "build",
        "isDefault": true
      },
      "problemMatcher": ["$tsc"]
    }
  ]
}
```

## Key Takeaways
1. Settings editor (`Ctrl+,`) is easiest way to find and change settings
2. Workspace settings override user settings and are shareable via version control
3. Keybinding when-clauses control context-sensitive shortcuts
4. Task definitions enable one-click build and automation
5. Use `"id"` in problem matchers to capture compiler output

## Connects To
- **Ch 1**: Basic setup and configuration
- **Ch 2**: Editing customization
- **Ch 4**: Agent and extension customization
- **Ch 5-9**: Language-specific configurations
