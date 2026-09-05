# Zoo Code Patterns

## Iterative Approval Workflow
**When to use**: Every task that involves file changes, command execution, or code modifications.
**How**: Zoo Code proposes one tool at a time as XML. You review parameters, click "Save" to approve or "Reject" to decline. Loop continues until task completes.
**Trade-offs**: Maximum control but slower execution. Use message queueing to auto-approve when confidence is high.

## Context Mention Injection
**When to use**: You need Zoo Code to operate on specific files, diagnostics, or external resources.
**How**: Prefix references with `@` — `@/path/file.ts`, `@/folder/`, `@problems`, `@terminal`, `@git-changes`, `@<commit-hash>`, `@https://url`. Use `/` from workspace root for file paths.
**Trade-offs**: Direct and unambiguous but consumes context window tokens. Avoid mentioning large directories; be selective.

## Progressive Task Breakdown
**When to use**: Large files, complex refactoring, or tasks exceeding context limits.
**How**: Start with overview (`List functions in @/file.ts`), then target specific pieces one at a time. Make small incremental changes, reviewing each before proceeding.
**Trade-offs**: More approval cycles but prevents context poisoning and produces higher-quality results.

## Mode Selection by Task Risk
**When to use**: Matching operational mode to task type and risk level.
**How**: Ask (read-only) → Architect (read + markdown) → Code (full) → Debug (full + troubleshooting). Orchestrator for complex multi-step delegation.
**Trade-offs**: Higher-risk tasks benefit from restricted modes; switching adds friction but prevents accidental damage.

## Sticky Model Assignment
**When to use**: Different models excel at different tasks (reasoning vs. coding).
**How**: Assign reasoning model to Architect/Debug, non-reasoning to Code. Each mode remembers its last model automatically via Ctrl+. cycling.
**Trade-offs**: Eliminates manual switching but requires initial setup per mode.

## Layered Custom Instructions
**When to use**: Enforcing coding standards, project conventions, or team preferences.
**How**: Create `.roo/rules/` for workspace, `.roo/rules-{mode}/` for mode-specific. Global rules at `~/.roo/rules/`. Precedence: global → workspace → mode-specific.
**Trade-offs**: Powerful but rules can conflict. Workspace overrides global; mode-specific complements but doesn't replace general rules.

## Diagnostics-Driven Development
**When to use**: Fixing linting errors, TypeScript errors, or runtime issues detected by VS Code.
**How**: Use `@problems` to import all errors. Zoo Code proposes fixes one at a time. Diagnostics auto-capture before/after edits to detect new errors.
**Trade-offs**: Efficient for systematic fixes but may miss issues not in Problems panel.

## MCP Server Setup Pattern
**When to use**: Adding external tool capabilities (databases, APIs, search).
**How**: Add config to global settings or `.roo/mcp.json`. Start with Context7 (one-command install). Verify server in MCP settings panel. Approve first tool invocation.
**Trade-offs**: STDIO for local (low latency, secure), Streamable HTTP for remote (multi-client), SSE for legacy only.

## Custom Tool Creation
**When to use**: Repetitive in-repo workflows that teammates need to reuse.
**How**: Create TypeScript files in `.roo/tools/` with `defineCustomTool`, Zod schema validation. Tools auto-approve. Load `.env` manually via dotenv.
**Trade-offs**: Lightweight but auto-approved (security trade-off). Use MCP for external services instead.

## Background Editing Mode
**When to use**: Large batch refactoring where visual confirmation is unnecessary.
**How**: Enable in Experimental settings. Zoo modifies files silently; changes appear in source control. Review via git status.
**Trade-offs**: Eliminates context switches but requires strict version control discipline.

## Message Queueing
**When to use**: Sending follow-up corrections while Zoo processes, skipping manual review.
**How**: Type and send messages while Zoo works. Queued messages auto-approve the next pending action.
**Trade-offs**: Faster iteration but removes safety checkpoint. Don't queue for tasks requiring manual review.

## Session Hygiene
**When to use**: Context degrades (nonsensical suggestions, tool misalignment, context poisoning).
**How**: Start a new chat session. Use Debug mode in separate task to isolate debugging context. Never try to "wake up" a poisoned session.
**Trade-offs**: Loses conversation history but prevents cascading errors.

## Parallel Development Pattern
**When to use**: Complex features requiring multiple concurrent approaches.
**How**: Clone repo multiple times, run separate Zoo Code instances on each. Merge with git when ready.
**Trade-offs**: Like having multiple human developers; git handles integration but adds merge complexity.

## Temperature Tuning
**When to use**: Matching output randomness to task type.
**How**: 0.0–0.3 for code/debugging (deterministic), 0.4–0.7 for architecture (balanced), 0.7–1.0 for brainstorming (creative). Set via API config profiles.
**Trade-offs**: Higher temperature increases creativity but also potential errors in code.

## Prompt Optimization
**When to use**: Reducing token consumption and API costs.
**How**: Be concise, use specific file references not entire directories, break large tasks into sub-tasks, disable unused MCP servers, use Architect mode for analysis-only phases.
**Trade-offs**: Less context = lower cost but may miss relevant information if over-reduced.
