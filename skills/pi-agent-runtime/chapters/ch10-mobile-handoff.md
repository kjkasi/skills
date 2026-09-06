# Chapter 10: Mobile Handoff

## Core Idea
Replaces per-frame durable writes and quadratic message_update replication with scoped storage and Chord delta batches — reducing durable amplification while preserving unknown-outcome recovery.

## Frameworks Introduced
- **Delta Tracking**: Chord-based change tracking for efficient replication
- **Scoped Storage**: pending assistant/tool output in ephemeral scopes, never becoming main-log history
- **Chord Batches**: replace full/per-frame replication with efficient delta batches

## Key Concepts
- **Delta**: change record tracking what changed, not full state
- **Scopes**: ephemeral storage for pending output; lifetime managed by scope, not main log
- **Handoff**: transition from one owner to another (e.g., session worker to server)

## Mental Models
- Use deltas for replication — only transmit what changed, not full state
- Think of scopes as "temporary storage" — pending output lives here, not in main log

## Anti-patterns
- **Per-frame durable writes**: eliminated by scoped storage
- **Quadratic message_update**: replaced by delta batches
- **Main-log pending output**: moved to ephemeral scopes

## Key Takeaways
1. Delta tracking replaces full-state replication
2. Scoped storage moves pending output out of main log
3. Chord batches replace per-frame writes
4. Preserves unknown-outcome recovery
5. Reduces durable amplification significantly

## Connects To
- **Ch 6**: Assistant durability (what handoff replaces)
- **Ch 7**: Tool durability (what handoff replaces)
- **Ch 9**: Plugins and Chord runtime
