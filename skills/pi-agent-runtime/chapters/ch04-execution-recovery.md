# Chapter 4: Execution, Recovery, Abort, Close

## Core Idea
Drive advances an expected operation through the effect gate, mutation line, and recovery procedures. Recovery is index-driven and bounded — never inferring state from absent values.

## Frameworks Introduced
- **Drive Primitive**: advances an expected operation through execution
- **Recovery Procedure**: reads complete total current state, constructs synthetic results for orphaned effects
- **Abort Request**: durably requests cancellation through requestAbort

## Key Concepts
- **Effect Gate**: controls admission of provider requests and tool calls
- **Mutation Line**: FIFO queue for Session mutations
- **Attachment**: publishes small lane/operation projection for external observation
- **Fault**: storage failure faults the harness — all effects stop, all calls reject

## Mental Models
- Think of drive as "advance the state machine" — it reads current state, decides next action, commits transition
- Use recovery as "resume from snapshot" — read complete state, continue from responsible procedure

## Anti-patterns
- **Journal replay**: recovery never replays journals or infers from absence
- **Partial fault tolerance**: a failed commit faults the harness; partial application is never tolerated
- **Out-of-order mutations**: mutation line is FIFO; never skip or reorder

## Worked Example
Crash mid-tool execution:
1. Model returns two tool calls
2. Harness commits batch plan, then intent for call 0 with `replay: "never"`
3. Tool deletes files, emits bounded progress, requests checkpoint
4. Process dies after one checkpoint commits
5. On restart, `pi.op.state` says `calls[0].status = "effect_pending", replay = "never"`
6. Later drive reconciles: latest durable checkpoint + explicit interruption warning
7. Synthetic error staged under reserved ID, materialized normally

## Key Takeaways
1. Drive advances operations through the effect gate and mutation line
2. Recovery is index-driven — reads complete state, never infers from absence
3. A failed commit faults the harness — all effects stop
4. Mutation line is FIFO — completion of latest promise implies all earlier writes
5. Orphaned effects get synthetic results with explicit warnings

## Connects To
- **Ch 0**: Intent-settlement pattern
- **Ch 1**: Storage primitives
- **Ch 3**: Operation state machine
- **Ch 6**: Assistant durability recovery
- **Ch 7**: Tool durability recovery
