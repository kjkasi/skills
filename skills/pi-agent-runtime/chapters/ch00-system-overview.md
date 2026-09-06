# Chapter 0: System Overview & Orientation

## Core Idea
Pi is a durable runtime for agent conversations that persists conversation and operation state so interrupted work resumes without repeating settled effects. Everything follows from three stores and four rules.

## Frameworks Introduced
- **Three Stores Model**: entries (conversation tree), values/lists (mutable state), usage ledger (cost history) — every payload lives in exactly one of these three places
- **Intent-Settlement Pattern**: provider requests and tool calls wrapped in two commits — intent ("about to do X; output will use ids R and U"), then settlement (complete output + next state)
- **Atomic Transactions**: all-or-none commits with strictly increasing sequence numbers; no crash state exists inside a transaction

## Key Concepts
- **Session**: four parts — immutable entry tree, mutable values/lists, Branches/AgentLanes, usage ledger
- **Harness**: drives lanes through `accept`, `drive`, `requestAbort`, `inspectExecution` primitives
- **Operation**: one accepted unit of lane work (run, compaction, or navigation)
- **Context**: explicit trailing parameter for every async public method; process-local invocation authority, never durable
- **Lane**: one named data path with a movable tip; AgentLane adds model configuration, queues, and operation state

## Mental Models
- Use the Three Stores when designing any new state — if it doesn't fit entries, values/lists, or the ledger, it shouldn't exist
- Think of intent-settlement as a two-phase commit for agent actions — intent reserves IDs and declares intent, settlement commits the actual result
- Treat the Session as a conversation tree where branches enable parallel work while preserving history

## Anti-patterns
- **Third place storage**: never store state outside entries, values/lists, or the ledger — there is no third place
- **Partial transactions**: a failed commit must fault the harness; partial application is never tolerated
- **Inferred recovery state**: recovery reads the complete total current state, never replaying a journal or inferring from absence

## Worked Example
A user posts in a channel with 400 entries. The application creates a lane and calls `lane.prompt(...)`. The normative write order:
1. Acceptance: insert user message, create lane tip, set operation meta/state
2. Intent: reserve response/usage IDs before anything sent
3. Stream: append compact frames without blocking the stream
4. Settlement: commit response, usage, next state, and frame-list deletion together
5. Tool calls follow intent → effect → outcome settlement, materializing in source order
6. Terminal transaction deletes operation values/lists and writes result

## Key Takeaways
1. Every payload lives in exactly one of three stores — entries, values/lists, or the ledger
2. Atomic transactions with strictly increasing sequences are the only write primitive
3. Recovery reads the complete total current state, never replaying journals
4. Intent-settlement wraps every external effect in two commits for crash safety
5. Context is process-local authority, never durable data

## Connects To
- **Ch 1**: Storage model in detail
- **Ch 2**: Conversation tree and branches
- **Ch 3**: Operation state machine
- **Ch 6**: Assistant durability builds on frame persistence
- **Ch 7**: Tool durability builds on outcome_ready state
