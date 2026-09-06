# Patterns

## Intent-Settlement Pattern
**When to use**: Every external effect (provider request, tool call) that must survive crashes
**How**: Two-phase commit: intent reserves IDs and declares intent, then settlement commits complete output + next state
**Trade-offs**: Extra write per effect; guarantees crash safety without journal replay

## Three Stores Model
**When to use**: Any new state in the system
**How**: Classify as entries (conversation tree), values/lists (mutable state), or usage ledger (cost history)
**Trade-offs**: Strict classification prevents third-place storage; may require staging for deferred writes

## Bound Typed Address
**When to use**: Any durable state access
**How**: Construct `value<T>(namespace, key?)` or `list<T>(namespace, key?)` once; use everywhere
**Trade-offs**: Compile-time type safety; no runtime validation; namespace reservation prevents collisions

## Content First Staging
**When to use**: Deferred writes (queued input, tool outcomes)
**How**: Stage in `pendingEntry(id)`, then place atomically in placement transaction
**Trade-offs**: Double-write for deferred content; guarantees atomic placement

## Frame Persistence
**When to use**: Assistant streaming durability
**How**: Append compact frames to `pi.pending.assistant_frame` list; delete atomically with settlement
**Trade-offs**: Preserves partial on crash; frames never prove completion

## Outcome Ready State
**When to use**: Parallel tool calls with source-ordered placement
**How**: Complete result staged in `pendingEntry`; call becomes `outcome_ready`; placement when earlier sources complete
**Trade-offs**: Extra staging write; prevents replay of completed effects after crash

## Mutation Line FIFO
**When to use**: Session mutations requiring ordering
**How**: One writer, one queue; completion of latest promise implies all earlier writes completed
**Trade-offs**: Simple ordering; no parallelism within one mutation line

## Recovery from Snapshot
**When to use**: After process loss
**How**: Read complete total current state; continue from responsible procedure; never replay journals
**Trade-offs**: Requires complete state at every transition; no inference from absence

## Facet Assembly
**When to use**: Cross-process service composition
**How**: Each host loads independent facets; services connect via tokens; kernel validates dependency graph
**Trade-offs**: No aggregate extension object; each process loads independently

## Delta Replication
**When to use**: Efficient state synchronization
**How**: Track changes (deltas), not full state; transmit only what changed
**Trade-offs**: Requires delta tracking infrastructure; eliminates quadratic replication

## Scoped Storage
**When to use**: Pending output with ephemeral lifetime
**How**: Store in scoped addresses; lifetime managed by scope, not main log
**Trade-offs**: Eliminates main-log amplification; requires scope lifecycle management

## Atomic Fork
**When to use**: Repository operations over source-storage boundary
**How**: Capture coherent snapshot; stage in temporary database; commit in one transaction
**Trade-offs**: Requires staging; guarantees atomic fork without blocking source
