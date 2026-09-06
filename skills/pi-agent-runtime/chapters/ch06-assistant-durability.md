# Chapter 6: Assistant Durability

## Core Idea
Durable partial assistant messages for ordinary generation and deferred-response polling — persists compact replayable stream frames without making them operation-state authority.

## Frameworks Introduced
- **Frame Persistence**: compact replayable stream frames stored in `pi.pending.assistant_frame` list
- **Frame Encoder**: per-stream encoder handles shared live partials without duplicate content
- **Frame Reduction**: `reduceAssistantMessageFrames()` reconstructs partial from committed frames

## Key Concepts
- **AssistantMessageFrame**: one frame per non-terminal event; covered queued deltas produce no frame
- **Frame Address**: `pendingAssistantFrames(operationId, responseEntryId)` — one exact address per effect-pending response
- **Frame Lifecycle**: append frames → await latest frame write → settle → delete list atomically
- **Unknown-Outcome Recovery**: orphaned generation reduces frames, constructs synthetic error with partial content

## Mental Models
- Use frames as "stream checkpoints" — they preserve provider-stream observation without proving completion
- Think of frame reduction as "replay from committed prefix" — never infer completion from frames

## Anti-patterns
- **Frame as authority**: frames never prove request completion; only settlement does
- **Second frame codec**: do not define a second harness frame codec or reducer
- **Terminal events as frames**: `done`/`error` produce no frames; settlement is separate

## Worked Example
Live event ordering:
1. message_start
2. message_update* (each listener delivery awaited by provider loop)
3. await latest frame write
4. after_response
5. message_end
6. atomic response settlement + frame-list delete
7. entry_added
8. usage

A crash after live update but before frame commit → reconnect shows latest committed frame prefix, which may be older than last live event.

## Key Takeaways
1. Frames are auxiliary — they never prove completion; only settlement does
2. One frame encoder per provider stream; do not define a second codec
3. Frame list deleted atomically with response settlement
4. Unknown-outcome recovery reduces frames, constructs synthetic error
5. Frames preserve provider-stream observation without proving how request ended

## Connects To
- **Ch 0**: Intent-settlement pattern
- **Ch 1**: Storage lists and addresses
- **Ch 3**: Operation state machine
- **Ch 4**: Recovery procedures
- **Ch 10**: Mobile handoff replaces per-frame writes
