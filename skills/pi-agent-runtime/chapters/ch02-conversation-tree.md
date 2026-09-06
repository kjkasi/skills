# Chapter 2: The Conversation Tree

## Core Idea
Entries are the complete stored row (placement + payload), created complete when placement happens. Branches are named paths through the tree; AgentLanes add agent state to branches.

## Frameworks Introduced
- **Born Placed**: assistant responses and direct appends arrive in one transaction
- **Content First**: queued input and tool outcomes stage in pendingEntry before placement
- **Branch as Data Path**: a Branch owns only its tip, queries, and direct append — no model, queues, or execution policy

## Key Concepts
- **Entry Types**: message, compaction, branch_summary, custom — each with specific payload rules
- **Placement**: content durable before placement waits in `pendingEntry(id)`; placement writes entry and deletes pending value atomically
- **AgentLane**: Branch + LaneConfiguration + LaneState + operation results
- **Context Projection**: how provider requests are built — scan branch newest-first, reverse, drop error responses, run through projectors

## Mental Models
- Think of placement as "birth" — an entry is created complete, never modified after
- Use Content First when designing any deferred write — stage in pending, place atomically

## Anti-patterns
- **Post-placement modification**: entries are write-once; never modify after creation
- **Reading past compaction**: context never reads past a compaction — it's a self-contained checkpoint
- **Raw Branch mutation while Harness owns lane**: trusted-programming defect

## Code Examples
```ts
// Entry types
interface MessageEntry extends EntryBase {
  type: "message"; message: AgentMessage; terminate?: true;
}
interface CompactionEntry extends EntryBase {
  type: "compaction"; summary: string; retainedTail: AgentMessage[];
  tokensBefore: number; details?: JsonValue; usage?: Usage; fromHook: boolean;
}

// Branch query
const entries = await storage.scanBranch({
  start: tip,
  order: "newestFirst",
  stopAtType: "compaction",
  limit: 100
}, context);
```

## Worked Example
Queued input flow:
1. `t0`: TX[ upsert pi.pending.entry/e_q1 = message, S(next){ inbox.steer += "e_q1" } ]
2. `t1`: TX[ insert e_q1 (parent), delete pi.pending.entry/e_q1, upsert pi.branch.tip = "e_q1" ]

Crash before t1: still queued. After t1: placed, pending value gone. Until placement, exactly one of pending value and entry exists.

## Key Takeaways
1. Entries are born placed — never modified after creation
2. Content First stages in pendingEntry, places atomically
3. Branch = data path only; AgentLane = Branch + agent state
4. Context projection stops at compaction — never reads past it
5. Compaction is a self-contained checkpoint, not a pointer into history

## Connects To
- **Ch 0**: Three Stores model
- **Ch 1**: Storage primitives
- **Ch 3**: Operation state machine
- **Ch 5**: Public surface and snapshots
