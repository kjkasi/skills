# Chapter 11: Tools Reference - Commands & Tasks

## Core Idea
Zoo Code bridges terminal execution and workflow orchestration: `execute_command` runs CLI commands with VS Code shell integration, `read_command_output` retrieves truncated output, `new_task` spawns subtasks with specialized modes, and `switch_mode` transitions between operational contexts.

## Key Concepts
- **execute_command**: Runs CLI commands via VS Code shell API with terminal reuse, real-time output capture, and security validation (blocks subshell patterns, checks RooIgnore). Parameters: `command` (required), `cwd` (optional). Detects "hot" processes like compilers for special handling. Output throttled at 100ms intervals.
- **read_command_output**: Retrieves full output from truncated `execute_command` artifacts. Two modes: **read mode** (pagination via `offset`/`limit` in bytes) and **search mode** (`search` parameter for grep-like filtering, case-insensitive). Parameters: `artifact_id`, `search`, `offset`, `limit`.
- **new_task**: Creates subtasks with parent-child relationships. The parent pauses while the subtask runs in its own conversation context, then results transfer back on completion. Parameters: `mode` (required), `message` (required), `todos` (optional checklist).
- **switch_mode**: Changes operational mode (Code, Architect, Ask, Debug, or custom). Maintains context continuity. Requires user approval. Enforces mode-specific tool groups and file restrictions (e.g., Architect can only edit `.md`). Parameters: `mode_slug` (required), `reason` (optional). Applies 500ms delay after switch.

## Code Examples

Run a build command:
```xml
<execute_command>
<command>npm run build</command>
</execute_command>
```

Run in a specific directory:
```xml
<execute_command>
<command>git status</command>
<cwd>./my-project</cwd>
</execute_command>
```

Retrieve truncated build output and search for errors:
```xml
<read_command_output>
<artifact_id>cmd-1706119234567.txt</artifact_id>
<search>error|failed|Error</search>
</read_command_output>
```

Create a code subtask with initial todos:
```xml
<new_task>
<mode>code</mode>
<message>Implement user authentication service</message>
<todos>
[ ] Set up Express server
[ ] Create user model
[ ] Implement CRUD endpoints
[ ] Add authentication middleware
[ ] Write API tests
</todos>
</new_task>
```

Switch to Debug mode:
```xml
<switch_mode>
<mode_slug>debug</mode_slug>
<reason>Need to systematically diagnose the authentication error</reason>
</switch_mode>
```

## Worked Example
A developer runs `npm test` via `execute_command`. The output is truncated to an artifact file. `read_command_output` with `search=FAIL` pinpoints 3 failing tests. The developer creates a `new_task` in `code` mode to fix the failures. After fixing, `switch_mode` transitions to `ask` mode to verify understanding of the test expectations, then back to `code` mode for final cleanup. `attempt_completion` summarizes the work.

## Key Takeaways
1. `execute_command` reuses terminal instances—avoid creating new terminals for sequential commands; chain with `&&` or `;`.
2. When output is truncated, look for the artifact ID in the `[OUTPUT TRUNCATED]` message and use `read_command_output` with search mode to filter efficiently.
3. `new_task` pauses the parent context—use it to isolate different phases (design, implement, test) without context pollution.
4. `switch_mode` is always available regardless of the current mode's tool group, enabling mid-workflow transitions.

## Connects To
- **Ch 9**: `read_file` examines files; `execute_command` runs tools like linters or formatters on them.
- **Ch 10**: `search_files` locates patterns; `execute_command` can run project-specific search tools for deeper analysis.
- **Ch 12**: `update_todo_list` provides in-chat tracking for tasks spawned via `new_task`.
