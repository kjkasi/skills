# Chapter 8: Typed Values and Lists

## Core Idea
The public abstraction is a bound typed address — one exact durable address with one compile-time value type. Applications define scalar values and lists without declaration merging or editing a core type map.

## Frameworks Introduced
- **Bound Address Model**: `value<T>(namespace, key?)` and `list<T>(namespace, key?)` — namespace/key bound once
- **Phantom Type Invariance**: `T` is invariant through phantom function; address for one type cannot widen to another

## Key Concepts
- **Address Construction**: validate once, use everywhere — no global value-type map
- **Reserved Namespace**: `pi` and `pi.*` are reserved for built-ins by contract
- **Empty Key**: legal; addresses one session-wide value or list
- **Object Identity**: no durable meaning — equal (kind, namespace, key) triples name same location

## Mental Models
- Use addresses as typed pointers — construct once, get compile-time type safety
- Think of namespace reservation as "API boundary" — `pi.*` is internal, application code uses collision-resistant prefixes

## Anti-patterns
- **Global value map**: delete RegisterValues/RegisterNamespace — type belongs to address constructor
- **Runtime privilege checks**: no runtime privilege split; constructor tests enforce convention
- **Cross-kind collision**: value and list may not share (namespace, key) — trusted-programming defect

## Code Examples
```ts
// Address constructors
export function value<T>(namespace: string, key = ""): Value<T> {
  validateAddress(namespace, key);
  return Object.freeze({ namespace, key, kind: "value" });
}

export function list<T>(namespace: string, key = ""): ValueList<T> {
  validateAddress(namespace, key);
  return Object.freeze({ namespace, key, kind: "list" });
}

// Application usage
const state = value<ApplicationState>("my-app.state");
const events = list<ApplicationEvent>("my-app.events");

// Built-in usage
export const branchTip = (lane: string) =>
  value<string | null>("pi.branch.tip", lane);
```

## Key Takeaways
1. Bound addresses are typed pointers — construct once, use everywhere
2. `pi` and `pi.*` namespaces reserved for built-ins by contract
3. Empty key is legal — addresses one session-wide value or list
4. Object identity has no durable meaning
5. Applications use collision-resistant namespace prefixes

## Connects To
- **Ch 1**: Storage model and built-in addresses
- **Ch 6**: Assistant durability uses frame lists
- **Ch 7**: Tool durability uses tool memos
