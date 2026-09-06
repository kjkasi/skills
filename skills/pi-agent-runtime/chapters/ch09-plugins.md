# Chapter 9: Plugins and Facets

## Core Idea
The application-neutral facet, service, and replicated-state runtime is provided by Chord. Pi composes that runtime with Pi-owned service contracts, process roles, routing, and lifecycle policy.

## Frameworks Introduced
- **Facet Kernel**: owns service-aware lifecycle mechanics — setup, dependency assembly, activation, disposal
- **Service Token**: `defineService<T>(id, options?)` creates service contract identity
- **Host Model**: session host (owns Harness), presentation host (owns UI), server host (owns routing)

## Key Concepts
- **Facet**: in-process unit; `setup(env)` is synchronous declaration; async init in `onActivate()`
- **Service**: remotely publishable by default; `provide(service, impl)` registers singleton
- **ServiceSpawner**: `provideMany(service)` returns spawner for keyed instances
- **Dependency Assembly**: API calls during setup declare dependencies; kernel validates graph

## Mental Models
- Use facets as "host-native code" — each process loads only facets built for that host
- Think of services as "cross-process contracts" — token identity, not implementation sharing

## Anti-patterns
- **Aggregate extension object**: no CodingAgentPlugin runtime interface — each host loads independently
- **Late dependency declaration**: all dependencies must be acquired during setup
- **Mixed provide modes**: within one facet generation, token must stay in one mode

## Code Examples
```ts
// Service definition
export interface Models {
  readonly state: ReplicatedState<ModelsState>;
  cycleThinking(context: Context): Promise<void>;
  refresh(context: Context): Promise<void>;
  select(model: ModelRef, context: Context): Promise<void>;
}
export const Models = defineService<Models>("pi.models");

// Facet setup
export const providersBuiltinSessionFacet = defineFacet({
  id: "@pi/providers-builtin",
  setup(env) {
    env.provide(Models, new ModelsImpl());
    env.use(ToolRegistry);
  },
});
```

## Worked Example
Question extension flow:
1. Model calls question tool (session facet)
2. Session facet adds one invocation-keyed dialog service (session authority)
3. Every connected TUI/web facet observes the service instance (keyed service)
4. First accepted answer settles it for everyone
5. Session facet returns durable tool result
6. Closing instance closes every presentation's dialog

## Key Takeaways
1. Facets are host-native code — each process loads independently
2. Services are cross-process contracts via token identity
3. Dependencies declared during setup; kernel validates graph
4. Session host owns Harness; presentation owns UI; server owns routing
5. One feature may ship multiple facets (session, TUI, web) connected by services

## Connects To
- **Ch 0**: System model and Context
- **Ch 5**: Public surface and hooks
- **Ch 12**: RPC transport
- **Ch 11**: Telemetry integration
