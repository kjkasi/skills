# Glossary

**abort signal** — cooperative cancellation mechanism; client maps to cancel(requestId), server derives AbortController (Ch 0, Ch 12)

**accept** — durable creation of an operation; hook-free, starts no task or effect (Ch 3)

**AgentLane** — Branch plus total agent state: LaneConfiguration, LaneState, and operation results (Ch 2)

**attachment** — publishes small lane/operation projection for external observation (Ch 4)

**atomic transaction** — all-or-none commit with strictly increasing sequence numbers; no crash state inside (Ch 0, Ch 1)

**born placed** — assistant responses and direct appends arrive in one transaction (Ch 2)

**Branch** — named data path through the conversation tree; owns only tip, queries, and direct append (Ch 2)

**BranchSummaryEntry** — entry type summarizing a branch for navigation; carries fromId, summary, usage (Ch 2)

**bound typed address** — `value<T>(namespace, key?)` or `list<T>(namespace, key?)` — namespace/key bound once (Ch 1, Ch 8)

**cache invalidation** — compaction is the one deliberate cache invalidation, traded for smaller context (Ch 2)

**checkpoint** — tool-owned snapshot of bounded progress; requests persistence via `checkpoint: true` (Ch 7)

**Chord** — application-neutral facet, service, and replicated-state runtime (Ch 9)

**compaction** — summarization of old messages; context never reads past a compaction (Ch 2)

**CompactionEntry** — entry type with summary, retainedTail, tokensBefore, usage (Ch 2)

**Context** — explicit trailing parameter for every async public method; process-local invocation authority (Ch 0)

**content first** — staged in pendingEntry before placement; used for queued input and tool outcomes (Ch 2)

**custom entry** — application-defined entry type with optional data payload (Ch 2)

**delta** — change record tracking what changed, not full state (Ch 10)

**drive** — advances an expected operation through execution (Ch 4)

**effect gate** — controls admission of provider requests and tool calls (Ch 3, Ch 4)

**effect_pending** — operation state indicating external effect in progress (Ch 3)

**entry** — complete stored row: placement fields and payload together; write-once (Ch 2)

**EntryBase** — base entry interface: id, parentId, seq, timestamp, type, customType (Ch 1)

**entryAdded** — event emitted when entry commits to conversation tree (Ch 5)

**facet** — in-process unit; `setup(env)` is synchronous declaration (Ch 9)

**facet kernel** — owns service-aware lifecycle mechanics: setup, dependency assembly, activation, disposal (Ch 9)

**fault** — storage failure that stops all effects and rejects all calls (Ch 4)

**frame** — compact replayable stream frame for assistant durability (Ch 6)

**frame encoder** — per-stream encoder handling shared live partials without duplicate content (Ch 6)

**frame list** — `pi.pending.assistant_frame` list; deleted atomically with response settlement (Ch 6)

**fork** — repository operation over one coherent source-storage boundary (Ch 2)

**hook** — before/after callback with replay contract; result becomes durable in consuming transaction (Ch 5)

**intent** — first phase of two-phase commit; reserves IDs and declares intent (Ch 0)

**invocation ID** — equals resultEntryId; session-unique, survives safe replay (Ch 7)

**JSONL** — file-based storage; one physical line per commit; format 4 (Ch 1)

**Lane** — named data path with movable tip (Ch 0)

**LaneConfiguration** — model configuration: provider, modelId, thinkingLevel, activeToolNames (Ch 2)

**LaneSnapshot** — latest observed partial assistant message; observation, not authority (Ch 5)

**LaneState** — current/last operation ids and inbox (Ch 2)

**latestFrameWrite** — promise reference for most recent frame append (Ch 6)

**list** — append-only durable list; elements ordered by write seq; whole-key deletion only (Ch 1, Ch 8)

**Memory** — in-memory storage backend; maps for entries, values, lists, usage (Ch 1)

**message** — entry type containing AgentMessage payload (Ch 2)

**mutation line** — FIFO queue for Session mutations; one writer, one queue (Ch 3, Ch 4)

**operation** — one accepted unit of lane work: run, compaction, or navigation (Ch 0)

**operationState** — total durable restart point; replaced after each transition (Ch 3)

**outcome_ready** — durable state between external-effect settlement and source-ordered placement (Ch 7)

**pendingEntry** — `pi.pending.entry` value; complete content awaiting placement (Ch 1)

**pendingToolOutput** — `pi.pending.tool_output` value; latest bounded progress checkpoint (Ch 1)

**parentSessionId** — source session id recorded in fork destination metadata (Ch 2)

**placement** — transaction that writes entry and deletes pending value atomically (Ch 2)

**provider** — external AI model service; requests wrapped in intent-settlement (Ch 0)

**query bound** — limit on scan results; must be positive safe integer (Ch 1)

**recovery** — reads complete total current state; never replays journals or infers from absence (Ch 4)

**replay contract** — hook result becomes durable in transaction that consumes it; crash may rerun (Ch 5)

**reserved namespace** — `pi` and `pi.*` reserved for built-ins by contract (Ch 8)

**result** — `pi.result` value; immutable terminal observation; written once, never updated or deleted (Ch 1)

**scan prefix** — five exported constructors for lane inventory and operation-cleanup grammar (Ch 1)

**scope** — ephemeral storage for pending output; lifetime managed by scope, not main log (Ch 10)

**service** — cross-process contract via token identity; remotely publishable by default (Ch 9)

**ServiceSpawner** — `provideMany(service)` returns spawner for keyed instances (Ch 9)

**settlement** — second phase of two-phase commit; commits complete output + next state (Ch 0)

**Session** — four parts: entry tree, values/lists, Branches/AgentLanes, usage ledger (Ch 0)

**SessionMutation** — atomic write operation on Session state (Ch 3)

**snapshot** — observation of latest state; not proof of completion (Ch 5)

**storage version** — format version tracked per-session; JSONL header, SQLite catalog column (Ch 1)

**ToolCall** — state union: planned → effect_pending → outcome_ready → completed (Ch 3, Ch 7)

**tool memo** — `operationToolMemo(opId, invId, name)` — invocation-scoped durable key-value (Ch 7)

**usage ledger** — append-only cost history rows; never modified, never deleted (Ch 1)

**UUIDv7** — time-sortable id; first 48 bits are mint time (Ch 1)

**value** — replaceable durable value at bound typed address (Ch 1, Ch 8)

**write vocabulary** — six operations: entry insert, usage insert, value set/delete, list append/delete (Ch 1)

**write-once** — entries created complete, never modified after placement (Ch 2)
