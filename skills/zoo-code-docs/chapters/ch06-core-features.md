# Chapter 6: Core Features

## Core Idea
Zoo Code's core features optimize the development workflow through concurrent file reads, customizable instructions, diagnostics integration, prompt enhancement, message queueing, temperature control, file access restrictions, and settings management. Together they form the operational backbone of the tool.

## Frameworks Introduced
- **Multi-File Context Loading**: Read up to 100 files simultaneously in a single request, replacing sequential file reads and enabling faster, more accurate context building.
- **Layered Custom Instructions**: A hierarchy of instruction sources—global rules, workspace rules, mode-specific rules, and AGENTS.md—that combine into the system prompt with clear precedence.
- **Diagnostics-Driven Development**: Leverage VSCode's Problems panel integration to automatically detect, understand, and fix code errors introduced by edits.

## Key Concepts
- **Concurrent File Reads**: Read up to 100 files in a single request (default: 5, configurable 1-100). Reduces back-and-forth approvals and builds complete context faster.
- **Custom Instructions**: Define behaviors, preferences, and constraints via files. Global rules (`~/.roo/rules/`), workspace rules (`.roo/rules/`), mode-specific rules (`.roo/rules-{modeSlug}/`), and AGENTS.md files.
- **Rule Loading Order**: Global → Project rules (take precedence) → Legacy files. Mode-specific rules load before general rules within each level.
- **Diagnostics Integration**: Auto-captures diagnostics before/after edits, detects new errors introduced by changes, and reports only new issues (not pre-existing).
- **@problems Mention**: Include `@problems` in messages to get a complete list of workspace errors and warnings for debugging context.
- **Enhance Prompt**: Refine prompts before sending using a wand icon. Adds clarity, context, and better instructions. Supports context-aware enhancement using conversation history (last 10 messages).
- **Message Queueing**: Send multiple messages while Zoo is working. Messages process sequentially (FIFO). Queued messages implicitly approve the next pending action.
- **Model Temperature**: Controls output randomness (0.0-2.0). Default 0.0 for most models. Higher values increase creativity; lower values increase determinism.
- **.rooignore**: Gitignore-like file access control. Protects sensitive files, prevents accidental changes to build artifacts. Enforced across read, write, apply_diff, and command execution.
- **Settings Management**: Export/import/reset settings. Auto-import on startup via `roo-cline.autoImportSettingsPath`. Exported JSON contains API keys in plaintext—handle securely.

## Mental Models
- **Rule Precedence Hierarchy**: Think of instructions as layered from general to specific: Global rules → Workspace rules (override) → Mode-specific rules (scoped). This mirrors CSS specificity.
- **Temperature as Task Dial**: Low temperature (0.0-0.3) for code generation and debugging (deterministic). Medium (0.4-0.7) for architecture planning (balanced). High (0.7-1.0) for explanations and brainstorming (creative).
- **Queue as Approval Bypass**: Queueing a message tells Zoo to proceed without pausing for confirmations. If you need manual review, don't queue—wait for the approval prompt.
- **rooignore as Access Firewall**: Not a full sandbox, but a strong guardrail for file operations. Always check tool interactions against `.rooignore` rules.

## Anti-patterns
- **Ignoring concurrent read limits**: Setting the limit too high for many large files can consume excessive memory. Start with default (5) and increase based on task needs.
- **Mixing global and workspace rules without precedence awareness**: Workspace rules override global rules on conflict. Don't duplicate rules at both levels without understanding this.
- **Queueing messages for complex approvals**: Queued messages auto-approve the next action. For tasks requiring manual review, don't queue.
- **Using high temperature for code generation**: Temperature 0.0 is optimal for precise code. Higher temperatures increase randomness and potential errors.
- **Forgetting .rooignore scope**: Rules apply only within the VS Code workspace root. Files outside this scope are not affected.

## Code Examples
### Custom Instructions Directory Structure
```
.roo/
├── rules/              # Workspace-wide rules
│   ├── 01-general.md
│   └── 02-coding-style.txt
├── rules-code/         # Rules for Code mode
│   ├── 01-js-style.md
│   └── 02-ts-style.md
└── rules-architect/    # Rules for Architect mode
    └── architecture.md
```

### Global Rules Setup
```bash
# Linux/macOS
mkdir -p ~/.roo/rules
# Windows
mkdir %USERPROFILE%\.roo\rules
```

### .rooignore Patterns
```
node_modules/
*.log
config/secrets.json
!important.log    # Exception to *.log
build/
docs/**/*.md
```

### Temperature Settings by Mode
| Mode | Recommended Range | Purpose |
|------|------------------|---------|
| Code | 0.0-0.3 | Precise, deterministic code |
| Architect | 0.4-0.7 | Balanced creativity for design |
| Ask | 0.7-1.0 | Diverse, insightful responses |
| Debug | 0.0-0.3 | Consistent troubleshooting |

### Auto-Import Settings Configuration
```json
{
  "roo-cline.autoImportSettingsPath": "~/roo-code-settings.json"
}
```

### Diagnostics Usage
```
@problems Fix all TypeScript errors in my project
@problems Help me debug why my application is crashing
@problems Review my code and fix any linting issues
```

### Enhanced Prompt Template (Default)
```
Generate an enhanced version of this prompt (reply with only the enhanced prompt - no conversation, explanations, lead-in, bullet points, placeholders, or surrounding quotes):

${userInput}
```

## Worked Example
A team sets up workspace rules in `.roo/rules/` with coding standards, then adds mode-specific rules in `.roo/rules-code/` for TypeScript requirements. They configure `.rooignore` to exclude `node_modules/` and `config/secrets.json`. When a developer runs a task, Zoo reads custom instructions (global + workspace + mode-specific combined), respects file access restrictions, and auto-detects new errors after edits. The developer uses `@problems` to get full diagnostic context during debugging, and uses message queueing to send follow-up corrections while Zoo processes—each queued message auto-approving the next action. Temperature is set to 0.0 for Code mode via an API configuration profile.

## Key Takeaways
1. Concurrent file reads (default 5, max 100) dramatically speed up context gathering by replacing sequential read-approve cycles.
2. Custom instructions follow a clear precedence: workspace rules override global; mode-specific rules complement but don't replace general rules.
3. Diagnostics integration automatically detects new errors after edits—use `@problems` for full workspace diagnostic context.
4. Enhance Prompt refines your prompts with context from conversation history; always review before sending.
5. Message queueing auto-approves the next action—only queue when you're ready to skip manual review.
6. Temperature 0.0 is optimal for code; use API configuration profiles to set per-mode temperatures.
7. `.rooignore` protects sensitive files but doesn't create a system-level sandbox—it's a workspace-scoped guardrail.
8. Exported settings contain API keys in plaintext—treat exported files as highly sensitive.

## Connects To
- **Ch 5: Modes System**: Custom instructions can be mode-specific; temperature varies by mode; tool access per mode determines feature availability.
- **Ch 7: MCP Integration**: MCP tools are part of the `mcp` tool group; custom tools (Ch 8) complement MCP for in-repo logic.
- **Ch 8: Experimental Features**: Background editing and concurrent file edits build on the core file operations described here.
