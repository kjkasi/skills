# Chapter 11: Telemetry

## Core Idea
Telemetry provides span vocabulary for observability — declared spans include pi.ai.request, operation, checkpoint, turn, step, tool, hook, sleep, event-handler, and session-write.

## Frameworks Introduced
- **Span Vocabulary**: declared span types for different operations
- **TelemetryContext**: carries telemetry parentage through async calls
- **Trace Injection/Extraction**: specified but not implemented (T1)

## Key Concepts
- **TelemetryContext**: process-local telemetry parentage; never durable
- **Declared Spans**: pi.ai.request, operation, checkpoint, turn, step, tool, hook, sleep, event-handler, session-write
- **Production Status**: only pi.harness.hook span is production-ready

## Mental Models
- Use TelemetryContext for async parentage — it carries trace context through calls
- Think of spans as "operation markers" — they group related work for observability

## Anti-patterns
- **Mixing with Context/RPC cancellation**: telemetry is separate from request-ID cancellation
- **Assuming all spans implemented**: only tool-hook span is production-ready

## Key Takeaways
1. TelemetryContext carries telemetry parentage through async calls
2. Declared spans provide vocabulary for observability
3. Only tool-hook span is production-ready
4. Trace injection/extraction specified but not implemented
5. Telemetry is separate from Context/RPC cancellation

## Connects To
- **Ch 0**: Context model
- **Ch 5**: Public surface and hooks
- **Ch 9**: Plugins and facet lifecycle
