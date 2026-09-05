# Zoo Code Cheatsheet

## Mode Selection Decision Tree

| Task Type | Recommended Mode | Why |
|---|---|---|
| Explore / understand code | **Ask** | Read-only, no accidental changes |
| System design / planning | **Architect** | Restricted to markdown, prevents code edits |
| Write / modify code | **Code** | Full tool access, default mode |
| Diagnose bugs | **Debug** | Full access + troubleshooting instructions |
| Multi-file complex project | **Orchestrator** | Delegates subtasks, prevents monolithic flow |
| Approve design then implement | Architect → Code | Ctrl+. to cycle between modes |

## Tool Selection Guide

| Need | Tool | Notes |
|---|---|---|
| Read file content | `read_file` | Slice mode (offset/limit) or indentation mode |
| Read many files | `read_file` (args) | Up to 100 concurrent reads |
| Create new file | `write_to_file` | Requires path, content, line_count |
| Replace entire file | `write_to_file` | Overwrites all content |
| Surgical single-file edit | `apply_diff` | Fuzzy matching, SEARCH/REPLACE format |
| Multi-file atomic patch | `apply_patch` | Unified diff, add/update/delete |
| Simple string replacement | `edit_file` or `search_replace` | `edit_file` also creates files (empty old_string) |
| Replace all occurrences | `edit` with `replace_all` | First-occurrence-only by default |
| Regex search | `search_files` | Ripgrep-based, respects .gitignore |
| Run terminal command | `execute_command` | Security validation, terminal reuse |
| Get truncated output | `read_command_output` | Search mode or pagination mode |
| Spawn subtask | `new_task` | Pauses parent, isolated context |
| Switch mode | `switch_mode` | Available in all modes |
| Load instructions | `skill` | Priority: project .roo/skills > global |
| Track progress | `update_todo_list` | Replace entire list each call |
| Ask for clarification | `ask_followup_question` | 2–4 suggested answers |
| Signal completion | `attempt_completion` | Optional demo command |
| Access MCP data | `access_mcp_resource` | URI-based, requires approval |
| Execute MCP tool | `use_mcp_tool` | JSON arguments, configurable timeout |

## Provider Selection Guide

| Priority | Provider Type | Examples |
|---|---|---|
| Cost | Local models | Ollama, LM Studio |
| Capability | Direct API | Anthropic, OpenAI, Google Gemini |
| Flexibility | Unified gateway | OpenRouter, Requesty, LiteLLM |
| Enterprise | Cloud managed | AWS Bedrock, GCP Vertex AI, Azure OpenAI |
| No API key needed | OAuth | ChatGPT Plus/Pro, Kimi Code |

**Key rule**: Models must support native OpenAI-compatible tool calling. Verify before selecting from OpenAI-compatible providers.

## Auto-Approval Thresholds

| Operation | Safe to auto-approve? | Setting |
|---|---|---|
| Read files | Yes | Enable read auto-approve |
| Write/edit files | Use with caution | Set Max Requests limit |
| Terminal commands | **No** (default) | Never auto-approve without review |
| Custom tools | Always auto-approved | Security trade-off by design |
| MCP tools | Configurable | "Always allow" for trusted read-only tools |

**Circuit breaker**: Set Max Requests (e.g., 5) to prevent runaway costs.

## Context Poisoning Recovery

1. **Identify**: Nonsensical suggestions, tool misalignment, repeated errors
2. **Don't**: Try "wake-up prompts" or re-injecting directives — corrupted data persists
3. **Do**: Start a new chat session immediately
4. **Isolate**: Use Debug mode in separate task for debugging
5. **Prevent**: Be specific with file paths, avoid dumping large logs, break tasks into focused sessions

## Token Optimization Checklist

- [ ] Use specific `@file` references, not entire directories
- [ ] Break large tasks into focused sub-tasks
- [ ] Set Code mode Max Tokens to 16K (save budget for history)
- [ ] Set Architect/Debug Max Tokens higher for reasoning
- [ ] Disable MCP if not in use (reduces system prompt)
- [ ] Use temperature 0.0 for code generation
- [ ] Enable prompt caching if provider supports it
- [ ] Use Architect mode for analysis phases (no expensive edits)

## Context Mention Quick Reference

| Mention | What it includes |
|---|---|
| `@/path/file.ts` | Full file contents with line numbers |
| `@/path/folder/` | Direct children only (trailing slash required) |
| `@problems` | All VS Code errors and warnings |
| `@terminal` | Last command output |
| `@git-changes` | Git status + uncommitted diff |
| `@a1b2c3d` | Specific commit (message, author, diff) |
| `@https://url` | Fetched webpage as Markdown |

## Troubleshooting Flowchart

```
Zoo Code not responding?
├─ Check API key validity → Re-enter if needed
├─ Check internet connection
├─ Check provider status page
├─ Restart VS Code
└─ Report issue with Error Details modal

Markdown files not writing?
├─ Disable format-on-save extensions
├─ Remove markdown preview settings from settings.json
├─ Restart VS Code
└─ Test with simple write operation

Context overflow error?
├─ Delete a message from history
├─ Roll back to checkpoint
├─ Switch to long-context model temporarily
└─ Start new session
```
