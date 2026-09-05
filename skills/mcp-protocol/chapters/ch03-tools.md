# Chapter 3: Tools

## Core Idea
MCP tools expose model-invocable actions. Tools are the protocol’s action primitive: a tool has a clear purpose, a typed input schema, and a result payload. The model chooses when to call a tool, but the host or user retains control over execution policy.

## Frameworks Introduced
- **Tool schema as an interface**: JSON Schema describes inputs and outputs explicitly.
  - When to use: when exposing external APIs or functions to the model
  - How: define a narrow, well-documented contract with required fields and expected semantics
- **Model-controlled execution with human oversight**: the model can discover and invoke tools, but applications can gate them.
  - When to use: for side-effecting operations such as database writes, API calls, or file changes
  - How: use approval dialogs, permission policies, or audit logs

## Key Concepts
- **`tools/list`**: discover available tools
- **`tools/call`**: run a specific tool with validated inputs
- **inputSchema**: the structured contract for arguments
- **approval boundary**: the user-visible control layer for side effects

## Mental Models
Treat tools like well-scoped actions rather than “the LLM can do anything.” The best tool is small, precise, and easy to explain in human terms. A clear description and schema let the model decide whether to use it without needing hidden business logic.

## Anti-patterns
- **Overbroad “do-everything” tools**: make model selection and approval difficult
- **Unclear safety boundaries**: create trust issues and user confusion
- **Passing raw free-form strings instead of schemas**: undermines reliability and validation

## Worked Example
A weather server exposes `get_forecast(latitude, longitude)` and `get_alerts(state)`. The model sees a clear schema and a human-readable description, chooses a tool based on the task, and performs a controlled action. The deployment may show a confirmation dialog before the tool makes an HTTP call or modifies a record.

## Key Takeaways
1. Tools represent actions, not rich context objects.
2. Schema quality is the main reliability lever.
3. UIs and permissions matter as much as protocol mechanics.

## Connects To
- **Chapter 4**: resources provide context, while tools perform actions
- **Chapter 6**: approval and auth controls are part of trust boundaries
