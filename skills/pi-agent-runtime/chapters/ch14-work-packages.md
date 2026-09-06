# Chapter 14: Work Packages

## Core Idea
Work packages (WP00-WP09) are incremental implementation milestones — each adds specific functionality while maintaining backwards compatibility.

## Frameworks Introduced
- **Work Package Pattern**: incremental milestones with specific scope
- **Backwards Compatibility**: each WP maintains compatibility with previous

## Key Concepts
- **WP00**: Runtime v1 removal
- **WP01**: Bound values and lists
- **WP02**: Atomic run acceptance
- **WP03**: Remove drive deadlines
- **WP04**: Mutation publication
- **WP05**: Direct durable drive
- **WP06**: Session/branch/lane separation
- **WP07**: SQLite host ownership and live forks
- **WP08**: Named branch and streaming forks
- **WP09**: Lane snapshot settled tools

## Mental Models
- Use work packages as "building blocks" — each adds specific functionality
- Think of backwards compatibility as "contract" — each WP must not break previous

## Anti-patterns
- **Breaking changes**: each WP must maintain backwards compatibility
- **Scope creep**: each WP has specific scope; don't add unrelated changes

## Key Takeaways
1. Work packages are incremental milestones with specific scope
2. Each WP maintains backwards compatibility
3. WP00-WP07 are complete
4. WP08 (named branch/streaming forks) is in progress
5. WP09 (lane snapshot settled tools) is in progress

## Connects To
- **Ch 0**: System model
- **Ch 1**: Storage model
- **Ch 3**: Operation state machine
- **Ch 4**: Execution and recovery
