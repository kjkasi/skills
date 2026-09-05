# Chapter 12: Tools Reference - MCP & Misc

## Core Idea
Zoo Code integrates with Model Context Protocol (MCP) servers for external data (`access_mcp_resource`) and tool execution (`use_mcp_tool`), plus provides utility tools: `ask_followup_question` for interactive clarification, `attempt_completion` for task finalization, `skill` for loading specialized instruction sets, and `update_todo_list` for workflow tracking.

## Key Concepts
- **access_mcp_resource**: Fetches data from MCP server resources (files, API responses, docs). Uses URI-based addressing. Two resource types: **Standard** (fixed URIs) and **Resource Templates** (parameterized URIs). Parameters: `server_name`, `uri`. Requires user approval. Handles text and image content.
- **use_mcp_tool**: Executes tools on MCP servers with full argument passing. Supports multiple transports: StdioClientTransport (local processes), SSEClientTransport (HTTP SSE), StreamableHTTPClientTransport. Parameters: `server_name`, `tool_name`, `arguments` (JSON). Validates against Zod schemas. Configurable timeouts (1-3600s, default 60s). "Always allow" mechanism for trusted tools.
- **ask_followup_question**: Structured clarification tool. Parameters: `question` (required), `follow_up` with `<suggest>` tags (2-4 suggested answers). Responses wrapped in `<answer>` tags. Available in all modes. Resets error counter on success.
- **attempt_completion**: Signals task completion. Parameters: `result` (summary), `command` (optional demo command). Displays in special UI format. Supports subtask completion with parent resumption. Waits for user feedback for iterative refinement.
- **skill**: Loads specialized instruction sets (e.g., `create-mcp-server`, `create-mode`). Resolved by priority: project `.roo/skills` > project `.agents/skills` > global equivalents. Mode-aware. Parameters: `skill` (required), `args` (optional). Linked files not auto-loaded.
- **update_todo_list**: Replaces entire todo list with updated markdown checklist. Status markers: `[ ]` pending, `[-]` in progress, `[x]` completed. Single-level structure, no nesting. Interactive UI with add/edit capabilities.

## Code Examples

Access an MCP resource:
```xml
<access_mcp_resource>
<server_name>weather-server</server_name>
<uri>weather://san-francisco/current</uri>
</access_mcp_resource>
```

Use an MCP tool:
```xml
<use_mcp_tool>
<server_name>code-analysis</server_name>
<tool_name>complexity_metrics</tool_name>
<arguments>{"language": "typescript", "file_path": "src/app.ts", "metrics": ["cyclomatic"]}</arguments>
</use_mcp_tool>
```

Ask a follow-up with suggestions:
```xml
<ask_followup_question>
<question>Which styling approach would you prefer?</question>
<follow_up>
<suggest>Bootstrap for rapid development</suggest>
<suggest>Tailwind CSS for utility-first flexibility</suggest>
<suggest>Vanilla CSS for minimal dependencies</suggest>
</follow_up>
</ask_followup_question>
```

Complete a task with a demo command:
```xml
<attempt_completion>
<result>Created a React app with responsive landing page and contact form.</result>
<command>npm start</command>
</attempt_completion>
```

Load a skill:
```xml
<skill>
<skill>create-mcp-server</skill>
<args>weather API integration</args>
</skill>
```

Track progress with todo list:
```xml
<update_todo_list>
<todos>
[x] Analyze requirements
[-] Design architecture
[ ] Implement core logic
[ ] Write tests
</todos>
</update_todo_list>
```

## Worked Example
An architect loads the `create-mcp-server` skill to scaffold a new server. During setup, `ask_followup_question` clarifies the desired transport (stdio vs HTTP). The skill instructions guide the developer to configure the server. After creation, `access_mcp_resource` tests the server's health endpoint, and `use_mcp_tool` invokes the server's custom analysis tool. Progress is tracked via `update_todo_list`, and `attempt_completion` presents the running server with `npm start`.

## Key Takeaways
1. MCP tools require explicit user approval by default—configure "always allow" for trusted, read-only tools to reduce friction.
2. `skill` loads instructions but doesn't auto-read linked files—explicitly `read_file` any referenced resources.
3. `ask_followup_question` is available in all modes—use it whenever ambiguity could lead to incorrect implementation.
4. `attempt_completion` enables an iterative refinement cycle, not just termination—user feedback loops back for improvements.
5. `update_todo_list` replaces the entire list each call—always include all items (pending + in-progress + completed) in every update.

## Connects To
- **Ch 9**: MCP tools may provide data written to files; `attempt_completion` can demo results via CLI.
- **Ch 10**: `search_files` locates code that MCP tools analyze; `edit_file` applies fixes discovered by MCP analysis.
- **Ch 11**: `new_task` spawns subtasks; `update_todo_list` tracks their progress; `switch_mode` transitions between MCP-related contexts.
