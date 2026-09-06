# Chapter 1: Storage Model

## Core Idea
Storage knows nothing about agents, lanes, or conversations — it stores entries and usage rows, updates bound values/lists, and answers a small fixed query set. Parts 2–4 are built entirely on this foundation.

## Frameworks Introduced
- **Bound Typed Address**: `value<T>(namespace, key?)` for replaceable values, `list<T>(namespace, key?)` for append-only lists — namespace/key bound once, every later operation receives only the address
- **Write Vocabulary**: six operations — entry insert, usage insert, value set/delete, list append/delete — carried through erased storage records

## Key Concepts
- **UUIDv7**: every id is time-sortable with first 48 bits as mint time; tool-result ids inherit their assistant id's timestamp
- **Storage Version**: format version tracked per-session; JSONL header field, SQLite catalog column
- **CommitResult**: returns `firstSeq`, `seqs[]`, `timestamp`, and `stats` (session totals immediately after commit)
- **Scan Prefix**: five exported constructors for lane inventory and operation-cleanup grammar

## Mental Models
- Think of bound addresses as typed pointers — constructing one gives you a compile-time type-safe handle to a durable location
- Use the write vocabulary as your mental checklist: entry insert, usage insert, value set/delete, list append/delete

## Anti-patterns
- **Cross-kind collision**: value and list addresses may not share one (namespace, key) — trusted-programming defect
- **Raw key repetition**: don't pass a second unexplained key on every read/write — construct the address once
- **Unbounded reads**: list reads always use cursor + limit; there is no "read the whole list" helper

## Reference Tables

### Built-in Address Constructors
| Constructor | Kind | Namespace | Value | Meaning |
|---|---|---|---|---|
| `branchTip(lane)` | value | `pi.branch.tip` | entry id or null | where lane appends next |
| `laneConfig(lane)` | value | `pi.lane.config` | LaneConfiguration | total lane configuration |
| `laneState(lane)` | value | `pi.lane.state` | LaneState | current/last operation ids and inbox |
| `operationResult(opId)` | value | `pi.result` | OperationResultRecord | immutable terminal observation |
| `operationMeta(opId)` | value | `pi.op.meta` | OperationMeta | acceptance data; written once |
| `operationState(opId)` | value | `pi.op.state` | OperationState | total durable restart point |
| `pendingEntry(entryId)` | value | `pi.pending.entry` | PendingEntry | complete content awaiting placement |
| `pendingToolOutput(opId, invId)` | value | `pi.pending.tool_output` | AgentToolResult | latest bounded progress checkpoint |
| `pendingAssistantFrames(opId, respId)` | list | `pi.pending.assistant_frame` | AssistantMessageFrame | committed stream-frame prefix |

### Lifetime Rules
| Address Family | Lifetime |
|---|---|
| `pi.lane.*`, `pi.session.*`, `pi.entry.*` | Session-lived semantic values |
| `pi.result` | Immutable lane-lived records, one per terminal operation |
| `pi.op.*` | Operation-lived; deleted no later than terminal transaction |
| `pi.pending.entry` | Until placement, cancellation, or owning-operation cleanup |
| `pi.pending.tool_output` | Only while its invocation is effect-pending |
| `pi.pending.assistant_frame` | Only while its response is effect-pending |

## Code Examples
```ts
// Define a typed address
const state = value<ApplicationState>("my-app.state");
const events = list<ApplicationEvent>("my-app.events");

// Use it with session methods
await session.getValue(state, context);
await session.setValue(state, nextState, context);
await session.readList(events, { limit: 100 }, context);
await session.appendList(events, event, context);

// Keyed instances
const workspaceEvents = (workspaceId: string) =>
  list<ApplicationEvent>("my-app.events", workspaceId);
await session.readList(workspaceEvents("pi"), { limit: 100 }, context);
```

## Key Takeaways
1. Bound addresses are typed pointers — construct once, use everywhere
2. Every id is UUIDv7 with mint time in first 48 bits
3. Six write operations cover all state changes: entry insert, usage insert, value set/delete, list append/delete
4. Lifetime is determined by address family — operation-owned values die at terminal transaction
5. Lists are append-only with whole-key deletion; no per-element mutation exists

## Connects To
- **Ch 0**: Three Stores model
- **Ch 2**: Entries and conversation tree
- **Ch 6**: Assistant durability uses frame lists
- **Ch 7**: Tool durability uses pendingEntry and tool memos
- **Ch 8**: Values specification in detail
