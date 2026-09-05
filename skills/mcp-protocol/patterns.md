# MCP Patterns

## Capability-first server design
**When to use**: when building a new server or integration layer.
**How**: advertise supported versions and capabilities before exposing feature-specific behavior. Make capability negotiation part of the design, not a post-hoc assumption.
**Trade-offs**: improves compatibility and version safety, but requires a bit more protocol discipline.

## Tool-as-action boundary
**When to use**: when the model needs to trigger side effects.
**How**: define a narrow tool schema, write a clear human description, and separate side-effecting actions from read-only information access.
**Trade-offs**: safer and easier to audit, but more tooling is needed for every action.

## Resource-as-context boundary
**When to use**: when the app needs evidence or working memory.
**How**: expose URIs, MIME types, and templates that can be read in controlled chunks rather than sending broad context to the model.
**Trade-offs**: lower-risk context handling, but requires careful retrieval and filtering logic.

## Prompt-as-workflow template
**When to use**: when the same interaction pattern should be repeated consistently.
**How**: package the model instructions and tool/resource expectations into a reusable prompt with explicit inputs.
**Trade-offs**: stronger reproducibility, but less improvisation in ad hoc tasks.

## Approval-aware execution
**When to use**: for anything destructive, sensitive, or externally impactful.
**How**: put a user or host approval gate in front of sensitive operations and keep logs of what was executed.
**Trade-offs**: better trust and safety, but adds user friction.

## Transport abstraction
**When to use**: when a server may run locally or remotely.
**How**: keep the JSON-RPC semantics stable and decide between stdio or Streamable HTTP based on deployment constraints.
**Trade-offs**: portable design, but needs explicit handling for auth and lifecycle differences.
