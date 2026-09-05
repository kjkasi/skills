---
name: mcp-protocol
description: "Knowledge base from \"Model Context Protocol (MCP)\" by the MCP community. Use when designing protocol-compliant servers, reasoning about tool/resource/prompt boundaries, applying capability negotiation, or studying the architecture and security model of MCP."
---

<!-- argument-hint: [tool, resource, prompt, transport, auth, capability, chapter, server] -->

# Model Context Protocol (MCP)
**Author**: Model Context Protocol community | **Pages**: ~N/A | **Chapters**: 7 | **Generated**: 2026-09-03

## How to Use This Skill

- **Without arguments** — load the core protocol model and decision rules
- **With a topic** — ask about `tools`, `resources`, `prompts`, `auth`, `stdio`, `streamable-http`, or `sampling`
- **With chapter** — ask for `ch03` or `ch06` to read the relevant section
- **Browse** — ask "what chapters do you have?" to inspect the index

When you ask about a topic not covered below, I will read the matching chapter and answer from the MCP spec and docs.

---

## Core Frameworks & Mental Models

### 1. MCP is a protocol for context exchange, not a model policy engine
MCP separates the AI app from the underlying LLM behavior. It standardizes how a host asks a server for capabilities, how a server exposes context, and how the app decides when to invoke tools or fetch resources. The protocol is about structured context exchange and control, not about choosing the model or deciding what the model should do with context.

Use MCP when you need a portable, well-specified interface between an AI client and a toolchain. Prefer it over bespoke ad hoc integrations when the same server needs to work across hosts and SDKs.

### 2. The protocol is split between data and transport layers
The data layer defines the JSON-RPC message model, capability negotiation, protocol metadata, and the primitives: tools, resources, prompts, and notifications. The transport layer handles the actual connection: stdio for local processes and streamable HTTP for remote services. The same MCP semantics can be reused across transports, which keeps the protocol stable while communication details vary.

Prefer designing the server around the semantic primitives first, then pick the transport second.

### 3. Capability negotiation is the contract you negotiate before use
MCP clients and servers advertise protocol versions and supported features. The server does not assume every client supports every capability. This prevents incompatible combinations and allows graceful evolution as the protocol matures.

Use discovery/negotiation before assuming a feature exists. An application should treat tools, resources, prompts, and authorization as negotiated capabilities rather than a universal guarantee.

### 4. Tools are model-controlled actions; resources are app-controlled context
Tools are designed for actions: they are invoked by the model when it decides it needs to do something. Resources are passive data sources retrieved by the application or user to inform the model. Prompts are reusable interaction templates that nudge the model toward a pattern or workflow. The boundary matters: model calls to tools should be explicit, contextual, and reviewable; resource reads are often lower-risk and more directly user-controlled.

Use tools for side effects and state changes; use resources for evidence, files, schemas, and state snapshots; use prompts to standardize interaction patterns.

### 5. Human oversight is part of the architecture
MCP is not a permissionless tool-execution model. It supports approval flows, logs, and user-visible control surfaces. The protocol assumes that deployments may require explicit consent before destructive or sensitive operations are executed.

When building a server, think in terms of user trust and action boundaries: small, well-described tools; clear descriptions; and approval points for anything high-impact.

### 6. Security is protocol-level, not just application-level
The docs treat auth, transport security, origin validation, and scope boundaries as first-class concerns. Remote servers should use standard HTTP auth patterns, OAuth where appropriate, and explicit identity handling. For local stdio servers, avoid writing to stdout because it corrupts the JSON-RPC stream. Use stderr or the logging module.

Treat auth, trust boundaries, and output channeling as engineering requirements, not afterthoughts.

### 7. Versioning is date-based and compatibility-aware
The repo uses date-based spec versioning such as `2025-11-25` and `2026-07-28`. That gives a clear evolution path without pretending semantic-version compatibility is enough for an ecosystem of clients and servers. When a server supports multiple versions, it announces them and negotiates accordingly.

Do not assume a feature exists across all versions; always check the negotiated protocol version and capability set.

---

## Chapter Index

| # | Title | Key Frameworks |
|---|-------|----------------|
| [ch01](chapters/ch01-overview.md) | Overview | protocol scope, host-client-server model |
| [ch02](chapters/ch02-architecture.md) | Architecture | data layer, transport layer, capability negotiation |
| [ch03](chapters/ch03-tools.md) | Tools | tool schema, actions, approval and execution |
| [ch04](chapters/ch04-resources.md) | Resources | contextual reads, URIs, templates, subscriptions |
| [ch05](chapters/ch05-prompts.md) | Prompts | reusable workflows, model guidance, user intent |
| [ch06](chapters/ch06-security-auth.md) | Security & Auth | OAuth, HTTP auth, trust boundaries, local stdio rules |
| [ch07](chapters/ch07-transports-versioning.md) | Transports & Versioning | stdio, streamable HTTP, compatibility strategy |

## Topic Index

- **authorization** → ch06, ch07
- **capability negotiation** → ch02, ch07
- **JSON-RPC** → ch02
- **prompts** → ch05
- **resources** → ch04
- **sampling** → ch02, ch06
- **server discovery** → ch02, ch07
- **stdio** → ch02, ch06, ch07
- **streamable HTTP** → ch02, ch06, ch07
- **tools** → ch03
- **transport** → ch02, ch07
- **user approval** → ch03, ch06

## Supporting Files

- [glossary.md](glossary.md) — key definitions across the MCP ecosystem
- [patterns.md](patterns.md) — protocol patterns and design rules
- [cheatsheet.md](cheatsheet.md) — decision rules and quick trade-offs

---

## Scope & Limits

This skill covers the MCP specification, official docs, and repo guidance in this project. For hands-on implementation in a specific SDK or runtime, combine it with those language-specific SDKs and host tooling. For broader AI systems design, check related skills or ask the agent directly.
