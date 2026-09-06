# Chapter 13: Runtime Simplification

## Core Idea
Runtime simplification removes complexity while preserving correctness — eliminating dead code, simplifying state machines, and reducing surface area.

## Frameworks Introduced
- **Simplification Pattern**: remove complexity while preserving invariants
- **Dead Code Elimination**: remove unused paths, types, and dependencies

## Key Concepts
- **Dead Code**: unused types, paths, and dependencies
- **State Machine Simplification**: reduce transition complexity while preserving correctness
- **Surface Area Reduction**: fewer public APIs, simpler contracts

## Mental Models
- Use simplification as "subtraction" — remove complexity, keep correctness
- Think of dead code as "liability" — it must be maintained even if unused

## Anti-patterns
- **Removing without understanding**: simplification must preserve invariants
- **Over-simplification**: don't remove necessary complexity

## Key Takeaways
1. Simplification removes complexity while preserving correctness
2. Dead code is liability — remove unused paths and dependencies
3. State machine simplification reduces transition complexity
4. Surface area reduction means fewer public APIs
5. Always verify invariants after simplification

## Connects To
- **Ch 0**: System model and invariants
- **Ch 3**: Operation state machine
- **Ch 4**: Execution and recovery
