# Chapter 6: Security & Auth

## Core Idea
MCP treats security as a first-class protocol concern, not an optional layer. It includes guidance for authorization, HTTP security, user approval, and local process safety, especially for stdio transports.

## Frameworks Introduced
- **Principle of least trust**: servers should only be trusted with the permissions and scopes they are explicitly granted.
  - When to use: for remote deployments, OAuth, or multi-tenant integrations
  - How: use bearer tokens, OAuth flows, and explicit scope boundaries
- **Explicit output channeling**: for local stdio servers, do not write to stdout.
  - When to use: when implementing process-based MCP servers
  - How: use stderr or logging instead of `print()` for status output

## Key Concepts
- **OAuth / bearer tokens**: common auth mechanisms for remote HTTP-based MCP servers
- **origin and trust validation**: ensure the remote service is the one the client expects
- **approval boundary**: UI or host-level consent for high-impact tool calls
- **local process safety**: avoid corrupting JSON-RPC streams

## Mental Models
If a tool can do work or expose sensitive context, it must be governed by identity, trust rules, and clear user intent. The protocol assumes applications may be multi-user, multi-server, or multi-tenant; therefore security boundaries must be explicit and auditable.

## Anti-patterns
- **Writing logs to stdout in a stdio server**: breaks the message stream
- **Using remote auth as an afterthought**: creates trust and replay problems
- **Allowing tool calls without a visible approval flow**: undermines user control

## Key Takeaways
1. Auth and approval are part of the protocol architecture, not side concerns.
2. Local servers must respect stdout/stderr separation.
3. Trust must be explicit, traceable, and scoped.

## Connects To
- **Chapter 2**: capability negotiation and service identity matter during setup
- **Chapter 3**: tool execution is where approval and auth become operational
- **Chapter 7**: secure transport choices are the concrete enforcement point
