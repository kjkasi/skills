# Chapter 5: Modes System

## Core Idea
Zoo Code's modes are specialized personas that tailor the assistant's behavior to your current task. Each mode offers different capabilities, expertise, and access levels, enabling task specialization, safety controls, focused interactions, and workflow optimization.

## Frameworks Introduced
- **Task Specialization via Modes**: Assign different AI personas (Code, Ask, Architect, Debug, Orchestrator) to different stages of your workflow, each with tailored tool access and behavior instructions.
- **Sticky Models & Mode Persistence**: Each mode remembers your last-used model. When switching modes, Zoo automatically selects that model—no manual selection needed. Mode also persists between sessions.

## Key Concepts
- **Code Mode (Default)**: Full access to all tool groups (`read`, `edit`, `command`, `mcp`). A skilled software engineer for writing code, implementing features, debugging, and general development.
- **Ask Mode**: Limited access: `read` and `mcp` only. A knowledgeable technical assistant focused on providing thorough answers, often using diagrams, without modifying your project.
- **Architect Mode**: Access to `read`, `mcp`, and restricted `edit` (markdown files only). An experienced technical leader for system design, high-level planning, and architecture discussions.
- **Debug Mode**: Full access to all tools. An expert problem solver using a methodical approach—analyzing, narrowing possibilities, and confirming before fixing.
- **Orchestrator Mode (Boomerang)**: No direct tool access. A strategic workflow orchestrator that breaks down complex tasks and delegates subtasks to specialized modes using the `new_task` tool.
- **Tool Groups**: Four capability categories—`read` (file reading/listing/searching), `edit` (file modification/creation), `command` (terminal execution), `mcp` (MCP server interactions).

## Mental Models
- **Tool Access Escalation**: Modes are ordered by tool access scope—Ask (read-only) → Architect (read + markdown edit) → Code/Debug (full). Match mode to task risk level.
- **Sticky Model Assignment**: Assign different models to different modes (e.g., Gemini 2.5 Preview for Architect, Claude Sonnet 3.7 for Code) and Zoo switches automatically. Think of this as role-based model routing.
- **Mode Cycling via Keyboard**: Use Ctrl+. (Windows/Linux) or Cmd+. (macOS) to cycle through modes sequentially. Useful for rapid context switching during multi-phase work.

## Anti-patterns
- **Using Code mode for planning**: Architect mode restricts editing to markdown, preventing accidental code changes during design phases. Use it for system design.
- **Using Ask mode for implementation**: Ask mode has no edit or command access—it cannot modify files or run commands. Switch to Code mode for actual implementation.
- **Ignoring Orchestrator for complex tasks**: For multi-step projects, Orchestrator delegates intelligently. Using Code mode for everything leads to monolithic, error-prone workflows.

## Code Examples
### Switching Modes via Slash Command
Type at the beginning of your message:
```
/architect
/ask
/debug
/code
/orchestrator
```

### Mode Tool Access Matrix
| Mode | read | edit | command | mcp |
|------|------|------|---------|-----|
| Code | ✅ | ✅ | ✅ | ✅ |
| Ask | ✅ | ❌ | ❌ | ✅ |
| Architect | ✅ | ✅ (markdown only) | ❌ | ✅ |
| Debug | ✅ | ✅ | ✅ | ✅ |
| Orchestrator | ❌ | ❌ | ❌ | ❌ (uses new_task) |

### Keyboard Shortcuts for Mode Cycling
| OS | Shortcut |
|----|----------|
| macOS | Cmd + . |
| Windows | Ctrl + . |
| Linux | Ctrl + . |

## Worked Example
A typical multi-phase workflow: Start in **Architect mode** to design the system and create implementation plans (restricted to markdown editing). Switch to **Code mode** (via dropdown, slash command, or Ctrl+.) to implement features with full tool access. Switch to **Debug mode** when issues arise—it uses the same full tool access but adds custom instructions for systematic troubleshooting. For a complex project spanning multiple files and services, switch to **Orchestrator mode** to break down the task and delegate subtasks to specialized modes automatically.

## Key Takeaways
1. Match modes to task risk levels: Ask for exploration, Architect for planning, Code for implementation, Debug for troubleshooting, Orchestrator for delegation.
2. Modes persist between sessions and remember your model selection, so set up model assignments once and forget.
3. Four switching methods exist: dropdown, slash commands, keyboard shortcuts (Ctrl+.), and accepting Zoo's mode-switch suggestions.
4. Custom modes can extend this system with tailored tool access, file permissions, and behavior instructions.

## Connects To
- **Ch 6: Core Features**: Custom instructions can be mode-specific (`.roo/rules-{modeSlug}/`), and tools like diagnostics integrate differently per mode.
- **Ch 7: MCP Integration**: The `mcp` tool group in each mode controls access to MCP server capabilities.
- **Ch 8: Experimental Features**: Background editing and concurrent edits work across all modes with edit access.
