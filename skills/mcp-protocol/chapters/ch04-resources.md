# Chapter 4: Resources

## Core Idea
Resources turn external data into structured, readable context. They let an MCP host fetch files, database slices, calendars, docs, or API data via a resource URI and MIME type instead of exposing raw internal state directly to the model.

## Frameworks Introduced
- **Resource-as-context**: provide static or dynamic information in a discoverable, URI-based form.
  - When to use: when the model needs relevant evidence or current state
  - How: expose URIs, templates, and MIME metadata for retrieval and filtering
- **Resource templates**: parameterized URI patterns for dynamic access.
  - When to use: when data varies by user, time, or object identity
  - How: declare `uriTemplate`, metadata, and expected format in a reusable way

## Key Concepts
- **resource URI**: a stable identifier pointing at data
- **resource template**: parameterized pattern for generated URIs
- **`resources/list`** and ``resources/read``: discovery and retrieval methods
- **subscriptions**: optional change notifications for watched resources

## Mental Models
Use resources when you want to supply evidence or working memory to the app without a side effect. Tools change state; resources inform state. A good resource strategy keeps the app selective: fetch only what’s needed, normalize it to a useful form, and let the application decide what to pass to the model.

## Anti-patterns
- **Dumping unbounded context into every request**: increases cost and noise
- **Using tool calls for read-only fetches**: weakens separation of intent and context
- **Unstable or untyped resource URIs**: breaks cacheability and discovery

## Key Takeaways
1. Resources are data access, not actions.
2. URIs and templates make retrieval discoverable and composable.
3. Applications remain in control of which context reaches the model.

## Connects To
- **Chapter 3**: tools are action-oriented while resources are context-oriented
- **Chapter 5**: prompts often combine curated resources with tool calls
