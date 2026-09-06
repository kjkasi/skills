# Chapter 3: The Operation State Machine

## Core Idea
An operation is one accepted unit of lane work with a total current state that records phase, control, and recovery data. The state machine governs transitions through acceptance, execution, and settlement.

## Frameworks Introduced
- **Operation State Machine**: total state replaces after each transition with complete current state — never depending on previous state
- **Effect Gate**: controls admission of external effects (provider requests, tool calls)
- **Mutation Line**: FIFO queue for Session mutations — one writer, one queue

## Key Concepts
- **OperationMeta**: immutable identity, intent, starting point — written once at acceptance
- **OperationState**: total durable restart point — replaced after each transition
- **LaneState**: current/last operation ids and inbox
- **ToolCall Status**: planned → effect_pending → outcome_ready → completed

## Mental Models
- Think of operation state as a snapshot — every transition writes the complete current state
- Use the mutation line as a FIFO queue — completion of latest promise implies all earlier writes completed

## Anti-patterns
- **Partial state**: never depend on previous state; each transition writes the complete current state
- **Out-of-order mutations**: the mutation line is FIFO; never skip or reorder
- **State inference**: recovery reads the complete total current state, never inferring from absence

## Reference Tables

### Operation State Transitions
| From | To | Trigger |
|---|---|---|
| starting | checkpoint | first drive |
| checkpoint | assistant_ready | model selected |
| assistant_ready | effect_pending | intent committed |
| effect_pending | tools | response settled |
| tools | call N effect_pending | tool call admitted |
| call N effect_pending | call N outcome_ready | tool result staged |
| call N outcome_ready | call N completed | entry materialized |
| call N completed | checkpoint | next turn ready |
| any | terminal | completion/abort |

### Tool Call Lifecycle
| Status | Meaning |
|---|---|
| planned | result entry ID reserved, not yet admitted |
| effect_pending | tool executing; replay policy set |
| outcome_ready | complete finalized result staged in pendingEntry |
| completed | entry materialized in conversation tree |

## Key Takeaways
1. OperationState is the total durable restart point — never depend on previous state
2. Mutation line is FIFO — completion of latest promise implies all earlier writes
3. Tool calls flow: planned → effect_pending → outcome_ready → completed
4. Effect gate controls admission of external effects
5. Recovery reads complete total current state, never replaying journals

## Connects To
- **Ch 0**: Intent-settlement pattern
- **Ch 1**: Storage primitives
- **Ch 2**: Entry placement
- **Ch 4**: Execution and recovery
- **Ch 7**: Tool durability uses outcome_ready
