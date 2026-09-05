# Chapter 1: Getting Started

## Core Idea
Zoo Code is a VS Code extension that uses AI agents to help you code. It requires connecting an LLM provider (API key), then you interact via natural language in a chat panel with an approval-based workflow.

## Frameworks Introduced
- **Provider Abstraction**: Zoo Code decouples the AI agent from specific LLM providers—swap between OpenRouter, Anthropic, or others by changing one API key.
- **Iterative Approval Loop**: Every action the AI proposes requires explicit user confirmation before execution, keeping you in control.

## Key Concepts
- **LLM Provider**: The backend inference service (OpenRouter, Anthropic, etc.) that powers Zoo Code's intelligence. You need an API key to connect.
- **API Key**: A credential from your chosen provider that authorizes Zoo Code to call LLM models. Paste it into the VS Code panel during setup.
- **Claude Sonnet 4.5**: The recommended starting model—balances power and cost, and works reliably out of the box for most tasks.
- **Tool Proposal**: When Zoo Code wants to act, it presents a specific tool (e.g., `write_to_file`) with parameters. You approve or reject before execution.
- **Iterative Workflow**: Zoo Code breaks tasks into steps, proposing one action at a time, waiting for your feedback, then continuing.

## Mental Models
- **Human-in-the-loop**: Think of Zoo Code as a junior developer who asks permission before every change. You review diffs, approve file writes, and confirm commands. This is not autonomous—it's collaborative.
- **Conversational coding**: Treat the chat input like you're talking to a colleague. Use plain English, reference files explicitly, and give context. No special syntax needed.
- **Progressive complexity**: Start with Claude Sonnet 4.5. Other models introduce variability in how they follow instructions—try them only after you're comfortable.

## Anti-patterns
- **Skipping provider setup**: Trying to use Zoo Code without a valid API key will fail. Always complete provider configuration first.
- **Using unfamiliar models early**: Different models vary in tool instruction compliance, format parsing, and multi-step context retention. Start simple, experiment later.
- **Ignoring approval prompts**: Auto-approval exists for trusted operations, but for new tasks, always review what Zoo Code proposes before approving.

## Code Examples
```bash
# Provider selection (done via UI, not CLI):
# 1. Open Zoo Code panel → click Zoo Code icon in Activity Bar
# 2. Choose provider (OpenRouter or Anthropic)
# 3. Paste API key
# 4. Select model: claude-sonnet-4-5 or anthropic/claude-sonnet-4-5
```

## Worked Example
**Task**: Create a file named `hello.txt` containing "Hello, world!"

1. Click the Zoo Code icon in the VS Code Activity Bar to open the chat panel.
2. Type: `Create a file named hello.txt containing 'Hello, world!'.`
3. Press Enter to send.
4. Zoo Code proposes a `write_to_file` tool with the path and content.
5. Click "Approve" (or "Save") to execute.
6. Zoo Code confirms the file is created and awaits your next instruction.

## Key Takeaways
1. Zoo Code needs an LLM provider with an API key before it can function—setup takes under 2 minutes.
2. The approval workflow means you retain full control over every file change, command execution, and code modification.
3. Start with the recommended model (Claude Sonnet 4.5) to avoid unnecessary complexity from model variability.

## Connects To
- **Ch 2**: After setup, the chat interface is your primary interaction surface.
- **Ch 3**: Context mentions (`@file`, `@folder`) give Zoo Code precise project information for better task execution.
- **Ch 4**: The tools Zoo Code proposes during the approval workflow are the mechanism behind every action.
