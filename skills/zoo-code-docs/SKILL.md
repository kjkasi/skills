---
name: zoo-code-docs
description: "Knowledge base from \"Zoo Code Documentation\" by Zoo Code Org. Use when configuring Zoo Code VS Code extension, selecting modes, integrating MCP servers, setting up providers, troubleshooting issues, or applying Zoo Code's AI coding workflows."
---

<!-- argument-hint: [topic, feature name, tool name, or chapter number] -->

# Zoo Code Documentation
**Author**: Zoo Code Org | **Sources**: 91 doc files | **Chapters**: 18 | **Generated**: 2026-09-04

## How to Use This Skill

- **Without arguments** — load core frameworks for reference
- **With a topic** — ask about `modes`, `MCP`, `context-mentions`, or another indexed topic; I find and read the relevant chapter
- **With chapter** — ask for `ch05`; I load that specific chapter
- **With tool name** — ask about `read_file`, `execute_command`, or any tool; I find the reference
- **Browse** — ask "what chapters do you have?" to see the full index

When you ask about a topic not covered in Core Frameworks below, I will read
the relevant chapter file before answering.

---

## Core Frameworks & Mental Models

### Modes System (Ch 5)
Zoo Code uses specialized AI **modes** that constrain model behavior to specific task types:
- **Code Mode**: Default. Reads/writes files, executes commands, handles implementation tasks.
- **Architect Mode**: Plans architecture, designs systems, creates specs. Does NOT write code directly.
- **Ask Mode**: Answers questions about code without making changes. Read-only exploration.
- **Orchestrator Mode** (Boomerang): Delegates subtasks to other modes. Coordinates complex multi-step projects.

Use the right mode for the task. Don't ask Code mode to plan architecture — switch to Architect first.

### Iterative Approval Workflow (Ch 4)
All interactions follow: **Model proposes action → User approves/rejects → Model adapts**.
- Each tool call (file read, write, command exec) requires explicit approval
- Auto-approval can be enabled per tool type for faster workflows
- The approval boundary is your safety net — understand it before disabling

### MCP — Model Context Protocol (Ch 7)
Standardized protocol for connecting AI to external tools/data:
- **MCP Servers** expose tools, resources, and prompts to the AI
- **STDIO transport**: Local process communication (most common)
- **Streamable HTTP**: Remote servers with streaming support
- **SSE**: Legacy remote transport, being replaced by Streamable HTTP
- MCP servers run alongside Zoo Code in VS Code, providing capabilities beyond file editing

### Context Mentions — Targeted @ References (Ch 3)
Inject specific context into prompts using `@` syntax:
- `@/path/to/file` — include file contents in context
- `@/path/to/folder` — include folder structure
- `@problems` — include current VS Code problems/diagnostics
- `@terminal` — include terminal output
- `@git-commit-hash` — include git commit diff
- `@clipboard` — include clipboard contents

Use targeted mentions instead of loading entire files — precision beats volume.

### Custom Instructions Layered Rules (Ch 6)
Rules are loaded in priority order, highest wins:
1. **Global rules** — `~/.zoo-code/rules/` or `~/.roo/rules/`
2. **Project rules** — `.zoo-code/rules/` or `.roo/rules/` in repo root
3. **Workspace rules** — `.zoo-code/rules-{mode}/` or `.roo/rules-{mode}/`
4. **AGENTS.md** — Agent Rules Standard (community convention)
5. **`.rooignore`** — Files/patterns the model must NOT access

### Prompt Engineering for Zoo Code (Ch 13)
- Use **checklists** for multi-step tasks (model follows numbered lists reliably)
- Use **XML tags** to structure complex instructions
- Assign **roles** — "You are a senior TypeScript developer..."
- Include **constraints** — what NOT to do is as important as what to do
- Reference **specific files** with `@mentions` instead of describing them

### Context Poisoning Recovery (Ch 13)
When the model goes off-track or produces garbage:
1. **Don't continue** — the model won't self-correct from corrupted context
2. **Use `/clear`** to reset the conversation
3. **Re-state the task** with fresh context
4. Prevention: keep tasks focused, use modes appropriately, break large tasks into subtasks

### Orchestrator / Boomerang Pattern (Ch 5)
For complex projects:
1. Start in **Orchestrator mode**
2. Describe the full project goal
3. Orchestrator breaks it into subtasks
4. Each subtask is delegated to the appropriate mode (Code, Architect, Ask)
5. Modes complete their work and "boomerang" back results
6. Orchestrator coordinates and continues

### Auto-Approval Trust Levels (Ch 4)
Configure per tool type:
- **Read operations** (read_file, search_files): Low risk, safe to auto-approve
- **Edit operations** (write_to_file, apply_diff): Medium risk, auto-approve when confident
- **Command execution** (execute_command): High risk, auto-approve only for known-safe commands
- Start conservative, increase as you learn the model's behavior

---

## Chapter Index

| # | Title | Key Frameworks |
|---|-------|----------------|
| [ch01](chapters/ch01-getting-started.md) | Getting Started | Provider setup, first task, installation |
| [ch02](chapters/ch02-chat-interface.md) | Chat Interface & Requests | UI layout, message sending, request tips |
| [ch03](chapters/ch03-context-mentions.md) | Context Mentions | @file, @folder, @problems, @terminal, @git |
| [ch04](chapters/ch04-how-tools-work.md) | How Tools Work | Approval workflow, execution flow, auto-approve |
| [ch05](chapters/ch05-modes-system.md) | Modes System | Code, Architect, Ask, Orchestrator modes |
| [ch06](chapters/ch06-core-features.md) | Core Features | Custom instructions, settings, diagnostics, temperature |
| [ch07](chapters/ch07-mcp-integration.md) | MCP Integration | Protocol, transports, servers, MCP vs APIs |
| [ch08](chapters/ch08-experimental-features.md) | Experimental Features | Background editing, concurrent edits, custom tools |
| [ch09](chapters/ch09-tools-file-ops.md) | Tools: File Operations | read_file, write_to_file, apply_diff, apply_patch |
| [ch10](chapters/ch10-tools-search-edit.md) | Tools: Search & Edit | search_files, edit, edit_file, search_replace |
| [ch11](chapters/ch11-tools-commands-tasks.md) | Tools: Commands & Tasks | execute_command, read_command_output, new_task, switch_mode |
| [ch12](chapters/ch12-tools-mcp-misc.md) | Tools: MCP & Misc | access_mcp_resource, use_mcp_tool, skill, update_todo_list |
| [ch13](chapters/ch13-advanced-usage.md) | Advanced Usage | Prompt engineering, context poisoning, large projects |
| [ch14](chapters/ch14-rate-limits-costs.md) | Rate Limits & Costs | Token usage, cost optimization, provider tiers |
| [ch15](chapters/ch15-providers.md) | Provider Setup | Anthropic, OpenAI, Gemini, Ollama, 25+ providers |
| [ch16](chapters/ch16-faq-troubleshooting.md) | FAQ & Troubleshooting | Common issues, error reporting, diagnostics |
| [ch17](chapters/ch17-tips-tricks.md) | Tips & Tricks | Power-user techniques, best practices |
| [ch18](chapters/ch18-migration.md) | Migration Guide | Roo Code → Zoo Code transition |

## Topic Index

- **Anthropic** → ch15
- **Architect Mode** → ch05
- **Auto-approval** → ch04, ch13
- **Background Editing** → ch08
- **Bedrock** → ch15
- **Boomerang Tasks** → ch05
- **Checkpoints** → ch06
- **Claude** → ch01, ch15
- **Code Mode** → ch05
- **Codebase Search** → ch12
- **Context Mentions** → ch03
- **Context Poisoning** → ch13
- **Custom Instructions** → ch06
- **Custom Tools** → ch08
- **Diagnostics** → ch06
- **Execute Command** → ch11
- **Gemini** → ch15
- **Image Generation** → ch08
- **Keyboard Shortcuts** → ch02
- **Large Projects** → ch13
- **LiteLLM** → ch15
- **Local Models** → ch13
- **MCP** → ch07
- **Message Queueing** → ch06
- **Mimo** → ch15
- **Model Temperature** → ch06
- **Modes** → ch05
- **Ollama** → ch15
- **OpenAI** → ch15
- **OpenRouter** → ch15
- **Orchestrator Mode** → ch05
- **Prompt Engineering** → ch13
- **Prompt Structure** → ch13
- **Rate Limits** → ch14
- **Read File** → ch09
- **Roo Ignore** → ch06
- **Roo Code Migration** → ch18
- **Search Files** → ch10
- **Settings Management** → ch06
- **Skills** → ch12
- **Slash Commands** → ch11
- **Switch Mode** → ch11
- **Terminal Output** → ch03
- **Tool Use** → ch04, ch09-12
- **Vertex** → ch15
- **Write to File** → ch09

## Supporting Files

- [glossary.md](glossary.md) — all key terms with definitions
- [patterns.md](patterns.md) — all techniques and configuration patterns
- [cheatsheet.md](cheatsheet.md) — quick reference tables and decision guides

---

## Scope & Limits

This skill covers Zoo Code VS Code extension documentation only. For hands-on implementation in your codebase, combine with project-specific tools. For topics beyond this documentation, check related skills or ask the agent directly.

Zoo Code is a rebrand of Roo Code — the migration guide (ch18) covers the transition. Most concepts apply to both.
