# MCP Python SDK Glossary

**Adapter** — TypeAdapter for validating union types (no longer RootModel). (Ch 8)

**Assertion** — ID-JAG: JWT signed by enterprise IdP granting MCP server access. (Ch 3)

**Authorization Server** — MCP server minting access tokens. (Ch 3)

**Back-channel** — Pre-2026 server-to-client request channel. Retired. (Ch 9)

**Bus** — `SubscriptionBus` carrying typed events; in-memory default. (Ch 7)

**Callback** — Functions passed to `Client(...)` for server-to-client requests. (Ch 3)

**CacheConfig** — Client cache: store, partition, default_ttl_ms, share_public. (Ch 3)

**CacheHint** — Server per-method: `ttl_ms` and `cache_scope`. (Ch 3)

**CacheMode** — Per-call: "use", "refresh", "bypass". (Ch 3)

**CacheScope** — "public" (shared) or "private" (one auth context). (Ch 3)

**Capability** — Server declaration: tools, resources, prompts, completions, extensions. (Ch 1)

**Client** — Python object connecting to MCP server via any transport. (Ch 3)

**ClientCredentialsOAuthProvider** — httpx2.Auth for M2M OAuth. (Ch 3)

**ClientExtension** — Client-side: settings, claims, notification bindings. (Ch 2)

**ClientSessionGroup** — Multiple connections merged into unified dicts. (Ch 3)

**ComponentNameHook** — Function rewriting names for uniqueness in groups. (Ch 3)

**Context** — Injected `ctx`: request_id, session, headers, lifespan_context. (Ch 5)

**Cursor** — Opaque pagination token; client passes back verbatim. (Ch 2)

**Dependencies** — `Annotated[T, Resolve(fn)]`: invisible to model. (Ch 5)

**Discover** — `server/discover` (2026-07-28) replacing legacy `initialize`. (Ch 2)

**Elicit** — Resolver marker asking user mid-call. (Ch 5)

**Elicitation** — Tool mechanism to ask users for input mid-call. (Ch 5)

**ElicitationCallback** — Client handler for form and URL elicitation. (Ch 3)

**EventStore** — ABC for persisting Streamable HTTP events. (Ch 6)

**Extension** — Opt-in behavior bundle behind `vendor-prefix/name`. (Ch 2)

**FastMCP** — Deprecated name for `MCPServer`. (Ch 1)

**Handler** — Function responding to protocol method. (Ch 4)

**Host** — LLM application running MCP clients. (Ch 1)

**ID-JAG** — Identity Assertion JWT Authorization Grant. (Ch 3)

**IdentityAssertionOAuthProvider** — httpx2.Auth for enterprise auth. (Ch 3)

**InputRequiredResult** — 2026-07-28 multi-round-trip result. (Ch 5)

**InputSchema** — JSON Schema 2020-12 for tool arguments. (Ch 4)

**Interceptor** — `intercept_tool_call` on extensions. (Ch 2)

**JSON-RPC** — Wire format: method, params, id, result/error. (Ch 1)

**Lifespan** — `@asynccontextmanager` yielding startup state. (Ch 5)

**ListenHandler** — Server handler for `subscriptions/listen`. (Ch 2)

**Low-level Server** — `Server` with `on_*` constructor params. (Ch 2)

**MCPServer** — High-level server; decorators, type-hint schemas. (Ch 1)

**MethodBinding** — Extension custom method bound to versions. (Ch 2)

**Middleware** — `async (ctx, call_next)` wrapping inbound messages. (Ch 2)

**Mode** — Client negotiation: "auto" or "legacy". (Ch 3)

**Multi-round-trip** — 2026-07-28: InputRequiredResult + client retry. (Ch 5)

**OAuthClientProvider** — httpx2.Auth: discovery, registration, PKCE. (Ch 3)

**OAuthFlowError** — OAuth failure exception. (Ch 3)

**OutputSchema** — JSON Schema for tool structured output. (Ch 4)

**Pagination** — Cursor-based paging; opt-in on low-level Server. (Ch 2)

**Partition** — CacheConfig label for private entries. (Ch 3)

**PKCE** — Proof Key for Code Exchange. (Ch 3)

**Progress** — `ctx.report_progress()` streaming to client. (Ch 5)

**Prompt** — User-controlled: returns `PromptMessage` objects. (Ch 1)

**RequestState** — Opaque token sealed/verified across retries. (Ch 5)

**RequestStateSecurity** — Config for sealing/verifying requestState. (Ch 5)

**Resolve** — `Annotated[T, Resolve(fn)]`: runs fn before tool. (Ch 5)

**Resource** — Application-controlled: data for model's context. (Ch 1)

**ResourceTemplate** — Resource with parameterized URI. (Ch 1)

**ResultClaim** — Client extension: wire tag, model, resolver. (Ch 2)

**Roots** — Deprecated (SEP-2577). (Ch 9)

**Sample** — Resolver marker for LLM completion. (Ch 5)

**Sampling** — Deprecated (SEP-2577). (Ch 9)

**Schema** — JSON Schema 2020-12 default dialect. (Ch 4)

**Server** — Low-level class with `on_*` handlers. (Ch 2)

**Session** — `ClientSession` or `ServerRequestContext.session`. (Ch 1)

**SSE** — Legacy transport; superseded by Streamable HTTP. (Ch 3)

**Stdio** — Transport: subprocess via stdin/stdout. (Ch 3)

**StdioServerParameters** — Subprocess description. (Ch 3)

**Streamable HTTP** — Production HTTP transport. (Ch 3)

**Structured Content** — Tool return as JSON matching schema. (Ch 4)

**SubscriptionBus** — Protocol: publish + subscribe. (Ch 7)

**SubscriptionFilter** — Client filter for listen stream. (Ch 7)

**Subscriptions** — 2026-07-28 change notifications. (Ch 7)

**SubscriptionLost** — Exception on stream drop. (Ch 3)

**TTL** — Time-to-live for cached results; max 24h. (Ch 3)

**Tool** — Model-controlled: function the model calls. (Ch 1)

**ToolBinding** — Extension tool definition. (Ch 2)

**ToolError** — Tool exception → is_error=True. (Ch 4)

**Transport** — `async with x as (read, write)`. (Ch 3)

**TransportSecuritySettings** — DNS rebinding protection. (Ch 6)

**Vendor-prefix/name** — Extension identifier grammar. (Ch 2)

**Worker Thread** — v2 runs sync handlers on worker thread. (Ch 4)
