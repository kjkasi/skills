# MCP Glossary

**Authorization** — the process of proving identity and granting permission to a remote or hosted MCP server. In MCP, OAuth and bearer-token patterns are common for HTTP-based services.

**Capability** — a declared feature a client or server supports, such as tools, resources, prompts, sampling, or discovery.

**Client** — the MCP endpoint that connects to a server on behalf of a host, maintaining a dedicated channel for context exchange.

**Host** — the AI application or runtime that coordinates one or more MCP clients and decides what gets exposed to the model.

**JSON-RPC** — the request/response message format used by MCP for structured communication between peers.

**Prompt** — a reusable instruction or workflow template that helps the model act consistently with a task pattern.

**Resource** — read-only context a server exposes through URIs, often including MIME metadata and template-based access.

**Resource template** — a URI pattern parameterized by values such as date, city, or document ID.

**Sampling** — a client-side or server-side ability to request model-generated output; this capability is treated as versioned and may be deprecated in newer protocol versions.

**Server** — the component that exposes MCP capabilities to a client, such as tools, resources, prompts, or notifications.

**Server discovery** — the process of querying a server for supported versions and capabilities before feature-specific calls.

**Stdio** — a local transport that uses standard input/output streams for process-based communication.

**Streamable HTTP** — the remote transport pattern in which clients send HTTP messages, often with optional streaming responses.

**Tool** — a model-invocable action with a typed interface and an expected side effect.

**Transport layer** — the network or process mechanism that carries JSON-RPC messages between peers.

**Version negotiation** — the compatibility process by which clients and servers agree on a supported protocol version.
