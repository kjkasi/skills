# Cheatsheet

## Decision Rules

| When | Do | Because |
|---|---|---|
| New state needed | Classify into entries, values/lists, or ledger | Three Stores — no third place |
| External effect | Wrap in intent-settlement | Crash safety without journal replay |
| Deferred write | Stage in pendingEntry, place atomically | Atomic placement |
| Parallel tool calls | Use outcome_ready state | Prevent replay of completed effects |
| Process loss | Read complete state, continue from procedure | Recovery never replays journals |
| Cross-process service | Use facet + service tokens | Independent host loading |

## Decision Trees

### Where does this state belong?
- Is it a conversation record? → **Entry** (write-once, append-only)
- Is it mutable current state? → **Value** (replaceable) or **List** (append-only)
- Is it cost history? → **Usage ledger** (append-only rows)
- Is it pending content? → **pendingEntry** (until placement)
- Is it tool progress? → **pendingToolOutput** (until effect settles)
- Is it assistant frames? → **pendingAssistantFrames** (until response settles)

### How do I persist a tool result?
- Is it the final result? → Stage in `pendingEntry`, set `outcome_ready`
- Is it progress? → Use `pendingToolOutput` with `checkpoint: true`
- Is it invocation state? → Use `operationToolMemo`

### How do I recover after crash?
- Read `operationState(operationId)` — complete total state
- Check tool call statuses — any `effect_pending` with `replay: "never"` → synthetic error
- Check assistant frames — reduce for partial content
- Continue from responsible procedure

## Trade-off Matrices

### Storage Backend Selection
| Backend | Durability | Performance | Complexity |
|---|---|---|---|
| Memory | Process-level | Fastest | Lowest |
| JSONL | Process-crash | Fast writes | Medium |
| SQLite | Process-crash | Fast reads | Highest |

### Fork Strategy
| Strategy | Scope | Performance | Use Case |
|---|---|---|---|
| Source-sized snapshot | Full source | Slowest | Default |
| Divergent branch | Segment chain | Fast | Named branches |
| Shared container | Cross-session | Medium | Multi-session |

## Thresholds & Defaults

- **List read limit**: 1,000 default, 10,000 max
- **Bash live update cadence**: 100 ms
- **Bash checkpoint interval**: 2 seconds
- **Bash snapshot size**: 2,000 lines or 50 KiB
- **UUIDv7 mint time**: first 48 bits
- **Sequence gaps**: legal within and between transactions

## Tells & Smells

| Signal | Likely Issue |
|---|---|
| Third-place storage | Violates Three Stores |
| Partial transaction visible | Fault tolerance violation |
| Recovery replaying journals | Wrong recovery pattern |
| Hook with side effects | May rerun on crash |
| Snapshot used as authority | Observation ≠ completion |
| Per-frame durable writes | Amplification problem |
| Raw Session access by tool | Trust boundary violation |
