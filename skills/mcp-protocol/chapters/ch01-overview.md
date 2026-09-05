# Chapter 1: Overview

## Core Idea
MCP standardizes how AI applications (hosts) connect to external systems and context providers (servers) using a shared protocol. It is not a model-specific framework; it is an interoperability layer for tools, resources, prompts, and metadata.

## Frameworks Introduced
- **Host–Client–Server model**: A host manages one or more MCP clients; each client maintains a dedicated connection to a server.
  - When to use: when modeling integrations for local or remote services
  - How: treat the host as orchestration logic, the client as a connection wrapper, and the server as the capability provider
- **Data layer vs transport layer**: one layer defines semantics and messages; the other defines the connection mechanism.
  - When to use: when separating protocol design from networking details
  - How: keep protocol-language stable while swapping stdio or HTTP as needed

## Key Concepts
- **MCP host**: the AI application or editor that coordinates clients
- **MCP client**: the connection object used to reach an MCP server
- **MCP server**: the program that exposes context and actions
- **capability**: a declared feature such as tools, resources, prompts, or auth
- **protocol version**: a date-based spec identifier used for compatibility checks

## Mental Models
Use MCP when you need a stable, discoverable interface between an AI app and external systems. Think of the server as a context provider, not a hidden model wrapper. Treat the host as the policy boundary: it decides what context to request, what tools to expose, and how to handle user approval.

## Anti-patterns
- **Hardcoding a host-specific API contract**: breaks portability and forces every client to know custom semantics
- **Mixing transport concerns into tool design**: makes the server more brittle across stdio and HTTP variants

## Key Takeaways
1. MCP is about structured context exchange and actions, not model governance.
2. The host/client/server model is the core abstraction.
3. Capability discovery is mandatory for building robust, interoperable systems.

## Connects To
- **Chapter 2**: explains the architecture and negotiation flow
- **JSON-RPC**: the message format used by MCP requests and responses
