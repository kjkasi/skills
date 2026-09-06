# Chapter 12: RPC

## Core Idea
RPC provides service transport semantics — request-ID cancellation, multiplexed connections, and server routing between presentations and session workers.

## Frameworks Introduced
- **Request-ID Cancellation**: client maps signal to cancel(requestId); server derives request Context with AbortController
- **Multiplexed Connection**: one connection per presentation/session worker to server
- **Server Routing**: routes service calls between presentations and session workers

## Key Concepts
- **Request-ID**: maps client abort signal to server-side cancellation
- **Multiplexing**: one connection carries multiple logical streams
- **Server Routing**: server owns session records, worker management, authentication, attachment

## Mental Models
- Use request-ID cancellation as "cooperative abort" — client signals, server respects
- Think of server as "traffic cop" — routes calls between presentations and workers

## Anti-patterns
- **Direct presentation→worker connection**: all routing goes through server
- **Private sockets**: facets never open private sockets
- **Request ID leaking**: cancellation is per-request, not per-connection

## Key Takeaways
1. Request-ID cancellation maps client signal to server abort
2. One multiplexed connection per presentation/worker to server
3. Server routes all calls between presentations and workers
4. Facets never open private sockets
5. Cancellation is per-request, not per-connection

## Connects To
- **Ch 0**: Context model and abort signals
- **Ch 5**: Public surface and hooks
- **Ch 9**: Plugins and service binding
- **Ch 11**: Telemetry trace propagation
