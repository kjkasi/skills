---
name: pi-agent-runtime
description: "Knowledge base from \"Pi Agent Runtime Documentation\" by earendil-works. Use when applying Pi's durable runtime frameworks for agent conversations, studying the harness specification, or referencing its storage, operation, and recovery concepts."
---

<!-- argument-hint: [topic, framework name, or chapter number] -->

# Pi Agent Runtime
**Author**: earendil-works | **Sources**: 33 docs | **Chapters**: 15 | **Generated**: 2026-09-06

## How to Use This Skill

- **Without arguments** — load core frameworks for reference
- **With a topic** — ask about `storage`, `operations`, `recovery`, or another indexed topic; I find and read the relevant chapter
- **With chapter** — ask for `ch03`; I load that specific chapter
- **Browse** — ask "what chapters do you have?" to see the full index

When you ask about a topic not covered in Core Frameworks below, I will read
the relevant chapter file before answering.

---

## Core Frameworks & Mental Models

### Three Stores Model
Every payload lives in exactly one of three places: **entries** (conversation tree — write-once, append-only), **values/lists** (mutable state — replaceable values; append-only lists), or **usage ledger** (cost history — append-only rows). There is no third place.

### Intent-Settlement Pattern
Provider requests and tool calls wrapped in two commits: **intent** ("about to do X; output will use ids R and U"), then **settlement** (complete output + next state). This makes every external effect crash-safe without journal replay.

### Bound Typed Address
`value<T>(namespace, key?)` names one replaceable durable value; `list<T>(namespace, key?)` one append-only durable list. Namespace/key bound once; every later operation receives only the address. No global value-type map, token catalog, or separate application-state mechanism.

### Operation State Machine
`operationState(operationId)` is the **total durable restart point** — replaced after each transition with complete current state. Recovery reads it and starts at the responsible procedure, never replaying a journal.

### Mutation Line FIFO
Session mutations queue as one writer, one queue. Completion of `latestFrameWrite` implies every earlier append completed. No array of promises, timer, batcher, or flush method.

### Frame Persistence (Assistant Durability)
Compact replayable stream frames stored in `pi.pending.assistant_frame` list. Frames are auxiliary — they never prove request completion; only settlement does. Frame list deleted atomically with response settlement.

### Outcome Ready State (Tool Durability)
Separates two orders: **outcome durability** (actual completion order) and **entry materialization** (assistant source order). Complete finalized result staged in `pendingEntry`; call becomes `outcome_ready`; placement when earlier sources complete.

### Recovery from Snapshot
Recovery reads complete total current state and continues from the responsible procedure. Never replays journals or infers state from absent values. Orphaned effects get synthetic results with explicit warnings.

---

## Chapter Index

| # | Title | Key Frameworks |
|---|-------|----------------|
| [ch00](chapters/ch00-system-overview.md) | System Overview & Orientation | Three Stores, Intent-Settlement, Atomic Transactions |
| [ch01](chapters/ch01-storage.md) | Storage Model | Bound Addresses, UUIDv7, Write Vocabulary |
| [ch02](chapters/ch02-conversation-tree.md) | The Conversation Tree | Born Placed, Content First, Branch as Data Path |
| [ch03](chapters/ch03-operation-state.md) | Operation State Machine | State Transitions, Effect Gate, Mutation Line |
| [ch04](chapters/ch04-execution-recovery.md) | Execution, Recovery, Abort, Close | Drive, Recovery Procedure, Fault Handling |
| [ch05](chapters/ch05-public-surface.md) | Public Surface | Snapshots, Events, Hooks |
| [ch06](chapters/ch06-assistant-durability.md) | Assistant Durability | Frame Persistence, Frame Encoder, Frame Reduction |
| [ch07](chapters/ch07-tool-durability.md) | Tool Durability | Outcome Ready, Invocation Memo, Checkpoint Policy |
| [ch08](chapters/ch08-values.md) | Typed Values and Lists | Bound Address Model, Phantom Types |
| [ch09](chapters/ch09-plugins.md) | Plugins and Facets | Facet Kernel, Service Tokens, Host Model |
| [ch10](chapters/ch10-mobile-handoff.md) | Mobile Handoff | Delta Tracking, Scoped Storage, Chord Batches |
| [ch11](chapters/ch11-telemetry.md) | Telemetry | Span Vocabulary, TelemetryContext |
| [ch12](chapters/ch12-rpc.md) | RPC | Request-ID Cancellation, Multiplexing |
| [ch13](chapters/ch13-runtime-simplification.md) | Runtime Simplification | Simplification Pattern, Dead Code Elimination |
| [ch14](chapters/ch14-work-packages.md) | Work Packages | WP00-WP09 Milestones |

## Topic Index

- **entries** → ch02
- **values/lists** → ch01, ch08
- **usage ledger** → ch01
- **atomic transactions** → ch01
- **bound addresses** → ch01, ch08
- **UUIDv7** → ch01
- **placement** → ch02
- **pendingEntry** → ch02
- **Branch** → ch02
- **AgentLane** → ch02
- **compaction** → ch02
- **operation state** → ch03
- **effect gate** → ch03, ch04
- **mutation line** → ch03, ch04
- **recovery** → ch04
- **fault** → ch04
- **snapshots** → ch05
- **events** → ch05
- **hooks** → ch05
- **frames** → ch06
- **frame encoder** → ch06
- **frame reduction** → ch06
- **outcome_ready** → ch07
- **invocation memo** → ch07
- **checkpoint** → ch07
- **facets** → ch09
- **services** → ch09
- **Chord** → ch09, ch10
- **delta** → ch10
- **scopes** → ch10
- **telemetry** → ch11
- **RPC** → ch12
- **work packages** → ch14

## Supporting Files

- [glossary.md](glossary.md) — all key terms with definitions
- [patterns.md](patterns.md) — all techniques and design patterns
- [cheatsheet.md](cheatsheet.md) — quick reference tables and decision guides

---

## Scope & Limits

This skill covers the Pi agent runtime documentation only. For hands-on implementation in your codebase,
combine with project-specific tools. For topics beyond this runtime, check related skills
or ask the agent directly.
