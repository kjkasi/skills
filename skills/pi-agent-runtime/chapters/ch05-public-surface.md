# Chapter 5: Public Surface

## Core Idea
The public surface exposes lane, harness, Session/Branch, snapshots, events, hooks, and telemetry — all through explicit Context parameters.

## Frameworks Introduced
- **Snapshot Model**: LaneSnapshot carries latest partial message, operation state, and fault status
- **Event System**: passive events for lifecycle observation (message_start, message_update, message_end, entry_added, usage)
- **Hook System**: before_drive, before_run, after_response, before_tool, after_tool — replay contract

## Key Concepts
- **LaneSnapshot**: latest observed partial assistant message, not proof of live stream
- **Events**: passive observation; handlers receive Context when invoked
- **Hooks**: replay contract — result becomes durable in transaction that consumes it
- **Execution Blocks**: groups of operations for external observation

## Mental Models
- Use snapshots for observation, not authority — they show latest state, not proof of liveness
- Think of hooks as "before/after" hooks with replay semantics — crash before consumption may rerun

## Anti-patterns
- **Snapshot as authority**: snapshot is observation, not proof of completion
- **Hook side effects**: hooks must be idempotent; crash may rerun before consumption
- **Missing Context**: every async public method takes explicit trailing Context

## Key Takeaways
1. Snapshots are observation, not authority — they show latest state
2. Events are passive observation; handlers receive Context
3. Hooks follow replay contract — crash may rerun before consumption
4. Every async public method takes explicit trailing Context
5. Execution blocks group operations for external observation

## Connects To
- **Ch 0**: System model
- **Ch 2**: Branch queries and context
- **Ch 3**: Operation state machine
- **Ch 6**: Assistant durability events
- **Ch 11**: Telemetry spans
