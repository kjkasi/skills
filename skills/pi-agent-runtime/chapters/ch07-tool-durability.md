# Chapter 7: Tool Durability

## Core Idea
Separates two orders: outcome durability (actual completion order) and entry materialization (assistant source order). A complete finalized result becomes durable immediately in `pi.pending.entry`.

## Frameworks Introduced
- **Outcome Ready State**: durable state between external-effect settlement and source-ordered placement
- **Invocation Memo**: operation-owned durable memoization for tool-local state
- **Checkpoint Policy**: tool-owned snapshot bounding, cadence, and duplicate suppression

## Key Concepts
- **outcome_ready**: execution, error normalization, and after_tool finished; result staged in pendingEntry
- **Invocation ID**: equals resultEntryId; session-unique, survives safe replay
- **Tool Memo**: `operationToolMemo(opId, invId, name)` — invocation-scoped durable key-value
- **Checkpoint**: tool requests persistence of complete bounded snapshot via `checkpoint: true`

## Mental Models
- Use outcome_ready as "durable completion" — result is safe, waiting for source-ordered placement
- Think of checkpoints as "tool-owned snapshots" — harness owns persistence, tool owns cadence

## Anti-patterns
- **Rerunning after outcome_ready**: never rerun a tool after its complete finalized outcome is durable
- **Raw Session access**: tools never get raw Session or bound storage-address access
- **Infinite checkpoints**: tool owns cadence; bash policy bounds to every 2 seconds

## Code Examples
```ts
// Tool update callback with checkpoint option
export type AgentHarnessToolUpdateCallback<TDetails> = (
  partialResult: AgentToolResult<TDetails>,
  options?: { checkpoint?: true }
) => void;

// Invocation memo address
export const operationToolMemo = (
  operationId: string,
  invocationId: string,
  memoName: string,
) => value<JsonValue>(
  "pi.op.tool_memo",
  `${operationId}:${invocationId}:${memoName}`,
);
```

## Reference Tables

### Tool Call State Transitions
| Status | Meaning | Next |
|---|---|---|
| planned | result ID reserved | effect_pending |
| effect_pending | tool executing | outcome_ready |
| outcome_ready | result staged | completed |
| completed | entry materialized | — |

### Bash Checkpoint Policy
| Parameter | Value |
|---|---|
| Live update cadence | 100 ms |
| Checkpoint interval | 2 seconds |
| Snapshot size | last 2,000 lines or 50 KiB |

## Key Takeaways
1. outcome_ready separates completion order from placement order
2. Never rerun a tool after outcome_ready
3. Invocation ID equals resultEntryId — session-unique, survives replay
4. Tools own checkpoint cadence; harness owns persistence
5. Tool memos are invocation-scoped durable key-value storage

## Connects To
- **Ch 0**: Intent-settlement pattern
- **Ch 1**: Storage values and pendingEntry
- **Ch 3**: Operation state machine
- **Ch 4**: Recovery and fault handling
- **Ch 6**: Assistant durability (parallel pattern)
