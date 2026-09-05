# Chapter 17: Tips & Tricks

## Core Idea
Power-user techniques for Zoo Code focus on optimizing your workspace layout, reducing context waste, leveraging parallel execution, and using specialized modes and models strategically.

## Key Concepts
- **Secondary Sidebar**: Moving Zoo Code to VS Code's secondary sidebar lets you see Explorer, Search, and Source Control simultaneously—critical for multi-file workflows.
- **File Drag-and-Drop**: With Zoo Code in a separate sidebar, drag files from Explorer into the chat window (hold Shift after starting drag) for quick context addition.
- **Sticky Models**: Assign specialized AI models to different modes—reasoning model for Architect/Debug, non-reasoning model for Code. Zoo auto-switches to each mode's last-used model.
- **Max Tokens Tuning**: Every token allocated to thinking takes away from conversation history storage. Use high Max Tokens only for Architect/Debug; keep Code mode at 16K or less.
- **Parallel Repository Copies**: Run Zoo Code on multiple copies of your repository in parallel, using git to resolve conflicts—like having multiple human developers.
- **Custom Mode File Restrictions**: Limit the types of files custom modes can edit to keep them on-track and prevent unintended modifications.
- **`roo.acceptInput` Keyboard Shortcut**: Set up a keyboard shortcut to accept suggestions or submit text without using the mouse, reducing hand strain.
- **File Read Auto-Truncate Threshold**: Adjust in Advanced Settings to control how many lines are read from a file in one batch—lower values improve performance with very large files.

## Mental Models
- **Workspace partitioning**: Secondary sidebar + separate Debug tasks = clean context separation between exploration, coding, and debugging.
- **Model-task matching**: Use reasoning models for planning (Architect), non-reasoning for implementation (Code), and long-context models for overflow recovery.
- **Parallel development**: Multiple repository copies with separate Zoo Code instances = human-like team development with git as the integration layer.
- **Context budget allocation**: Max Tokens is a shared budget between thinking and conversation history—allocate deliberately per mode.

## Anti-patterns
- **Using high Max Tokens in Code mode**: Wastes token budget on thinking when you need conversation history for multi-step coding tasks.
- **Not using secondary sidebar**: Keeping Zoo Code in the primary sidebar forces constant context switching between file explorer and chat.
- **Ignoring context overflow errors**: The `input length and max tokens exceed context limit` error is recoverable—delete a message, rollback to checkpoint, or switch to a long-context model temporarily.
- **Creating custom modes without file restrictions**: Unrestricted custom modes can modify files outside their intended scope.

## Code Examples
```
# Keyboard shortcut for roo.acceptInput
# Add to VS Code keybindings.json:
{
  "key": "ctrl+enter",
  "command": "roo.acceptInput"
}

# Creating a custom mode from a job posting
"Create a custom mode based on the job posting at @[url]"

# File read auto-truncate threshold (Settings → Advanced Settings)
# Lower values = fewer lines per batch = better performance for large files
# Higher values = more context per read = fewer operations

# Debug mode context isolation
"Start a new task in Debug mode with all of the necessary context needed to figure out X"
# This keeps debugging context separate from your main task

# Parallel development pattern
git clone repo repo-copy-1
git clone repo repo-copy-2
# Run Zoo Code on each copy independently
# Merge with git when ready
```

## Worked Example
**Scenario**: You're working on a large codebase and hitting context limits frequently.

1. Move Zoo Code to the secondary sidebar for simultaneous file explorer access.
2. Set Code mode Max Tokens to 16K to preserve conversation history.
3. Set Architect mode Max Tokens to 32K for deep planning with reasoning.
4. Enable Sticky Models so each mode remembers its optimal model.
5. When hitting context limits, switch to Gemini (1M context) for one message to recover.
6. For complex features, create a custom mode restricted to specific file types.
7. For parallel work, clone the repo and run separate Zoo Code instances, merging with git.

## Key Takeaways
1. Secondary sidebar + file drag-and-drop is the highest-impact workspace optimization.
2. Max Tokens should be tuned per mode—high for planning (Architect/Debug), low for coding (Code).
3. Sticky Models eliminate manual model switching between modes.
4. Parallel repository copies enable human-like team development workflows.

## Connects To
- **Ch 13**: Context management and prompt structure directly impact token consumption and Max Tokens allocation.
- **Ch 14**: Token optimization tips (disable MCP, be concise) reduce cost alongside the workspace optimizations here.
- **Ch 16**: Many troubleshooting scenarios (context overflow, mode-specific issues) are avoided by following these tips.
