# Chapter 7: Transports & Versioning

## Core Idea
MCP’s transport layer carries the same protocol semantics across different communication mechanisms, while versioning keeps the ecosystem interoperable. The repo uses date-based versions and multiple transports, each with different operational trade-offs.

## Frameworks Introduced
- **Transport abstraction**: use the same protocol semantics over stdio or HTTP.
  - When to use: when moving from local development to remote deployments
  - How: keep message semantics stable while changing connection mechanics
- **Date-based versioning**: versions like `2026-07-28` explicitly identify protocol evolution.
  - When to use: when managing compatibility over time
  - How: advertise supported versions and negotiate appropriately

## Key Concepts
- **stdio transport**: local process-to-process communication
- **Streamable HTTP**: remote, HTTP-based transport with optional streaming support
- **OAuth**: recommended auth mechanism for remote servers
- **version negotiation**: the client/server handshake for supported protocol versions

## Mental Models
Choose the transport based on where the server runs and how much networked trust you need. Local stdio is simplest and fast; remote HTTP adds network boundaries, auth, and more operational concerns. The architecture is safe when you keep the protocol semantics stable while choosing the right transport for the context.

## Anti-patterns
- **Assuming local transport behavior is valid for remote services**: misses auth, network, and reliability constraints
- **Hardcoding protocol versions without negotiation**: makes the ecosystem fragile over time

## Key Takeaways
1. Transport choice is operational, not semantic.
2. Version negotiation keeps the ecosystem compatible.
3. Remote server deployments need explicit identity and auth design.

## Connects To
- **Chapter 2**: architecture and negotiation flow
- **Chapter 6**: auth and security under remote transport conditions
