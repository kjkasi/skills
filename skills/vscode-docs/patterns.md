# Patterns & Techniques

## Extension-Based Architecture Pattern
**When to use**: Adding new functionality to VS Code
**How**: Install extensions from marketplace; manage via Extensions view
**Trade-offs**: Extensions add features but can impact startup time and memory

## Settings Hierarchy Pattern
**When to use**: Configuring VS Code for different contexts
**How**: Default → User → Workspace → Folder settings; each level overrides previous
**Trade-offs**: Flexibility vs. complexity; workspace settings are shareable via version control

## Launch Configuration Pattern
**When to use**: Debugging applications in VS Code
**How**: Define scenarios in launch.json with type, request, and configuration-specific options
**Trade-offs**: Predefined configs save time but may need customization for edge cases

## Test Discovery Pattern
**When to use**: Running automated tests in VS Code
**How**: Test explorer automatically discovers tests from project configuration (pytest, jest, etc.)
**Trade-offs**: Automatic discovery is convenient but may include unwanted tests

## Remote Development Pattern
**When to use**: Developing on remote machines or in containers
**How**: Use Remote extensions (SSH, WSL, Containers, Tunnels) to connect
**Trade-offs**: Full IDE experience on remote vs. network latency and setup complexity

## Dev Container Pattern
**When to use**: Creating consistent, reproducible development environments
**How**: Define in devcontainer.json with image, features, and post-create commands
**Trade-offs**: Environment consistency vs. Docker dependency and build time

## Agent Plugin Pattern
**When to use**: Creating portable agent customizations across VS Code and other tools
**How**: Package skills, MCP servers, agents, and hooks following agent-plugins.org standard
**Trade-offs**: Portability across tools vs. client-specific feature limitations

## Custom Agent Pattern
**When to use**: Creating specialized AI personas for specific tasks
**How**: Define in .agent.md files with persona, tools, and instructions
**Trade-offs**: Specialized behavior vs. maintenance overhead

## Hook Pattern
**When to use**: Executing commands at agent lifecycle points
**How**: Configure in hooks.json with event triggers and shell commands
**Trade-offs**: Automation vs. security risks; always review hook code

## Prompt File Pattern
**When to use**: Creating reusable, parameterized prompts for agents
**How**: Store in .github/prompts/ with frontmatter for mode and tools
**Trade-offs**: Reusability vs. complexity of parameter management

## Multi-cursor Editing Pattern
**When to use**: Editing multiple locations simultaneously
**How**: Alt+Click for multiple cursors; Ctrl+Alt+Up/Down for column selection
**Trade-offs**: Speed vs. potential for accidental edits

## Conditional Breakpoint Pattern
**When to use**: Debugging loops or complex state conditions
**How**: Set breakpoint with JavaScript expression that evaluates to true/pause
**Trade-offs**: Precision vs. evaluation overhead on each execution

## Logpoint Pattern
**When to use**: Non-invasive logging during debugging
**How**: Set logpoint instead of breakpoint; logs message without pausing
**Trade-offs**: No code changes needed vs. less control than print statements

## Code Lens Pattern
**When to use**: Showing contextual information above code
**How**: Extensions provide Code Lens for references, test counts, etc.
**Trade-offs**: Useful information vs. visual clutter

## Workspace Trust Pattern
**When to use**: Opening untrusted remote folders
**How**: VS Code prompts for trust; restricts features for untrusted folders
**Trade-offs**: Security vs. functionality limitations

## Inline Diff Pattern
**When to use**: Reviewing changes before committing
**How**: Click file in Source Control to see side-by-side or inline diff
**Trade-offs**: Visual review vs. large diffs being hard to navigate

## Merge Editor Pattern
**When to use**: Resolving merge conflicts
**How**: Open conflicted file; use visual merge editor to choose changes
**Trade-offs**: Visual clarity vs. complex conflicts still requiring manual resolution

## Stash Pattern
**When to use**: Temporarily saving changes without committing
**How**: Use Source Control panel to stash changes; restore later
**Trade-offs**: Quick save vs. potential confusion with multiple stashes

## Branch Management Pattern
**When to use**: Parallel development workflows
**How**: Create, switch, merge branches from Source Control panel
**Trade-offs**: Isolation vs. merge complexity

## Profile Pattern
**When to use**: Different configurations for different workflows
**How**: Create profiles via Manage Profiles; switch via status bar
**Trade-offs**: Clean separation vs. profile management overhead

## Settings Sync Pattern
**When to use**: Consistent setup across multiple machines
**How**: Enable via Accounts menu; syncs settings, keybindings, extensions
**Trade-offs**: Convenience vs. potential conflicts between machines

## Debug Configuration Sharing Pattern
**When to use**: Team debugging consistency
**How**: Commit launch.json to version control; use workspace-relative paths
**Trade-offs**: Shared configs vs. machine-specific debugging needs

## Extension Recommendations Pattern
**When to use**: Guiding team extension adoption
**How**: Define in .vscode/extensions.json with recommendations and unwanted lists
**Trade-offs**: Consistent tooling vs. potential unwanted notifications

## Enterprise Policy Pattern
**When to use**: Centralized VS Code management
**How**: Use Group Policy or MDM to control settings and extensions
**Trade-offs**: Control vs. user flexibility

## Telemetry Control Pattern
**When to use**: Managing data collection in enterprise
**How**: Configure telemetry settings; use policy templates for enforcement
**Trade-offs**: Privacy vs. product improvement data
