---
name: vscode-docs
description: "Knowledge base from \"VS Code Documentation\" by Microsoft. Use when working with VS Code features, configuring the editor, debugging, remote development, AI agents, or referencing language-specific setups."
---

<!-- argument-hint: [feature name, configuration topic, or chapter number] -->

# VS Code Documentation
**Author**: Microsoft | **Sources**: 358 files | **Generated**: 2026-09-04

## How to Use This Skill

- **Without arguments** — load core VS Code concepts and quick reference
- **With a topic** — ask about `debugging`, `extensions`, `remote development`, or another indexed topic; I find and read the relevant chapter
- **With chapter** — ask for `ch01`; I load that specific chapter
- **Browse** — ask "what chapters do you have?" to see the full index

When you ask about a topic not covered in Core Frameworks below, I will read
the relevant chapter file before answering.

---

## Core Frameworks & Mental Models

### VS Code Architecture
- **Extension-based**: Core is minimal; features come from extensions
- **Language Server Protocol**: Standardized interface for language intelligence
- **Debug Adapter Protocol**: Standardized interface for debugging

### Editor Concepts
- **Command Palette** (`Ctrl+Shift+P`): Access all features when you don't know where something lives
- **Quick Open** (`Ctrl+P`): Fast file navigation without touching explorer
- **Settings Hierarchy**: Default → User → Workspace → Folder (higher overrides lower)
- **Profiles**: Separate configurations for different workflows

### AI Agent Framework
- **Agent**: Autonomous AI that plans, edits, runs commands, manages context
- **Skills**: On-demand instructions loaded when topic matches
- **Hooks**: Shell commands at agent lifecycle points (before/after tool execution)
- **Custom Agents**: Specialized personas with specific tools and instructions
- **MCP Servers**: External tool integrations for agents

### Remote Development
- **Remote - SSH**: Linux server development via SSH
- **Remote - WSL**: Linux development on Windows
- **Remote - Containers**: Docker-based environments
- **Remote - Tunnels**: Quick access without SSH setup

### Debugging
- **Launch Configs**: Define debug scenarios in launch.json
- **Logpoints**: Non-invasive logging without code changes
- **Conditional Breakpoints**: Break only when conditions met
- **Test Explorer**: Visual test management across frameworks

### Configuration
- **settings.json**: All editor and extension settings
- **keybindings.json**: Custom keyboard shortcuts
- **tasks.json**: Build and automation commands
- **launch.json**: Debug configurations

---

## Chapter Index

| # | Title | Key Topics |
|---|-------|------------|
| [ch01](chapters/ch01-getting-started.md) | Getting Started & Setup | Installation, Settings Sync, Profiles |
| [ch02](chapters/ch02-editing.md) | Code Editing & Customization | IntelliSense, Snippets, Zen Mode |
| [ch03](chapters/ch03-ai-agents.md) | AI Agents & Copilot | Copilot Chat, Agent Sessions, Context |
| [ch04](chapters/ch04-agent-customization.md) | Agent Customization & Plugins | Skills, Hooks, MCP, Prompt Files |
| [ch05](chapters/ch05-python.md) | Python Development | Interpreter, Testing, Jupyter |
| [ch06](chapters/ch06-javascript-typescript.md) | JavaScript & TypeScript | jsconfig/tsconfig, npm, Debugging |
| [ch07](chapters/ch07-java.md) | Java Development | Extension Pack, Maven, Spring Boot |
| [ch08](chapters/ch08-csharp.md) | C# & .NET Development | Dev Kit, OmniSharp, Solution Explorer |
| [ch09](chapters/ch09-cpp.md) | C++ Development | IntelliSense Modes, CMake, Cross-compile |
| [ch10](chapters/ch10-containers.md) | Containers & DevOps | Dev Containers, Docker, Kubernetes |
| [ch11](chapters/ch11-remote.md) | Remote Development | SSH, WSL, Containers, Tunnels |
| [ch12](chapters/ch12-debugging.md) | Debugging & Testing | DAP, Launch Configs, Test Explorer |
| [ch13](chapters/ch13-source-control.md) | Version Control | Git, Branches, Merge Conflicts |
| [ch14](chapters/ch14-reference.md) | Reference & Configuration | Settings, Keybindings, Tasks |
| [ch15](chapters/ch15-data-science.md) | Data Science | Jupyter, Data Wrangler, Notebooks |
| [ch16](chapters/ch16-enterprise.md) | Enterprise & Extensions | Policies, Security, Telemetry |
| [ch17](chapters/ch17-other-languages.md) | Other Languages | LSP, Multi-language, Extensions |

## Topic Index

- **Agent** → ch03, ch04
- **Agent Plugin** → ch04
- **Breakpoint** → ch12
- **Build Task** → ch14
- **C/C++** → ch09
- **C#** → ch08
- **Chat** → ch03
- **Code Action** → ch2
- **Command Palette** → ch01
- **Container** → ch10
- **Custom Agent** → ch04
- **Debug** → ch12
- **Dev Container** → ch10
- **Extension** → ch01, ch16
- **Git** → ch13
- **Hook** → ch04
- **IntelliSense** → ch02
- **Java** → ch07
- **JavaScript** → ch06
- **Jupyter** → ch15
- **Keybinding** → ch14
- **Launch Configuration** → ch12
- **MCP Server** → ch04
- **Merge Conflict** → ch13
- **Multi-cursor** → ch02
- **Python** → ch05
- **Remote Development** → ch11
- **Settings** → ch14
- **Settings Sync** → ch01
- **Skill** → ch04
- **SSH** → ch11
- **Task** → ch14
- **Terminal** → ch06
- **Test Explorer** → ch12
- **TypeScript** → ch06
- **Virtual Environment** → ch05
- **Workspace Trust** → ch11
- **WSL** → ch11
- **Zen Mode** → ch02

## Supporting Files

- [glossary.md](glossary.md) — all key terms with definitions
- [patterns.md](patterns.md) — all techniques and design patterns
- [cheatsheet.md](cheatsheet.md) — quick reference tables and decision guides

---

## Scope & Limits

This skill covers VS Code features and configuration only. For specific language implementations or frameworks beyond VS Code's built-in support, combine with language-specific documentation. For VS Code internals or extension development, consult the VS Code API documentation directly.
