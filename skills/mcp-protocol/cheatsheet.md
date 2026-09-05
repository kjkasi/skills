# MCP Cheatsheet

## Decision rules

- **When the model needs to act**, prefer a tool over a direct data dump.
- **When the app needs to provide context**, prefer a resource over a tool action.
- **When a workflow should be repeatable**, prefer a prompt template.
- **When the server is remote**, assume auth, trust boundaries, and origin checks matter.
- **When the deployment is local**, keep stdio rules strict: do not write to stdout.

## Trade-off matrix

| Situation | Prefer | Why |
|---|---|---|
| Side effects or API calls | Tool | action-oriented, schema-validated, auditable |
| Structured evidence or documents | Resource | read-only, contextual, app-controlled |
| Repeatable operational workflow | Prompt | consistent, user-friendly, reusable |
| Local service | stdio | simple and fast |
| Remote service | Streamable HTTP | networked, auth-capable, scalable |

## Decision tree

- Is the need primarily about action?
  - Yes → use a tool with typed inputs and approval boundaries.
  - No → continue.
- Is the need primarily about context?
  - Yes → expose a resource or resource template.
  - No → continue.
- Should this workflow be repeated by users or hosts?
  - Yes → express it as a prompt template.
  - No → keep it ad hoc.

## Tells & smells

- **If you see “unstructured prompt-injection”**, move to a prompt template and explicit tools.
- **If the model keeps calling tools to fetch data**, convert the data into resources.
- **If your server works locally but fails remotely**, review auth and transport assumptions.
- **If you cannot explain which part is the action and which part is the context**, your design is too mixed.

## Default safety checks

- Validate input schemas before executing a tool.
- Log side-effecting operations for user review.
- Verify remote auth and issuer trust before accepting requests.
- Keep the protocol version and capabilities explicit.
