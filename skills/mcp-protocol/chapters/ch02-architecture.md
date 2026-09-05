# Chapter 2: Architecture

## Core Idea
MCP is built around a stateless, JSON-RPC-based protocol paired with transport-specific connection handling. Architectural clarity comes from separating capability negotiation, message semantics, and communication channels.

## Frameworks Introduced
- **Stateless request model**: every request carries the metadata needed to process it.
  - When to use: when designing resilient request handling and version-aware integrations
  - How: include protocol version and capability metadata, and avoid assumptions from prior requests
- **Capability negotiation**: a server advertises support for features before clients depend on them.
  - When to use: when building cross-version or multi-server deployments
  - How: discover versions and supported features before making feature-specific calls

## Key Concepts
- **JSON-RPC 2.0**: message envelope for requests, notifications, and responses
- **`server/discover`**: the capability advertisement mechanism
- **capability metadata**: per-request `_meta` fields identifying the client/server context
- **transport abstraction**: stdio vs HTTP as different communication channels with the same protocol semantics

## Mental Models
Think of MCP as a layered contract: the app says “what do you support and how do you version it?” and the server replies with explicit support. Once the connection is negotiated, the protocol is mostly message-driven rather than stateful. That makes remote and local deployments feel similar while still allowing native features per transport.

## Anti-patterns
- **Assuming features without discovery**: fails in mixed-version ecosystems and hidden compatibility gaps
- **Treating stdio and HTTP as interchangeable at implementation level**: they differ in lifecycle and auth behavior

## Key Takeaways
1. Discovery is the first handshake in a robust MCP deployment.
2. Statelessness reduces implicit coupling between requests.
3. Data semantics should be transport-independent; transport rules remain outside the protocol core.

## Connects To
- **Chapter 1**: provides the overall model
- **Chapter 7**: covers the concrete transport and compatibility paths
- **Security**: protocol identity and auth assumptions are inseparable from negotiation
