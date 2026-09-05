# Chapter 3: AI Agents & Copilot

## Core Idea
VS Code integrates AI-powered coding assistance through GitHub Copilot and agent-based workflows that can autonomously plan, edit, and run code.

## Frameworks Introduced
- **Agent Architecture**: Agents are specialized AI assistants with configurable tools, context, and behaviors
- **Agent Harnesses**: Framework for running agents with specific toolsets and constraints
- **Context Engineering**: Strategically providing relevant code, files, and instructions to AI
- **Subagents**: Agents that delegate tasks to other specialized agents
- **Approval Workflows**: Human-in-the-loop controls for agent actions

## Key Concepts
- **Copilot Chat**: AI assistant accessible via Chat view or inline (`Ctrl+Shift+I`)
- **Agent**: Autonomous AI that can plan, edit files, run commands, and manage context
- **Agent Session**: Persistent conversation with an agent that maintains context
- **Tools**: Capabilities agents can use (file read/write, terminal, search, browser)
- **Subagents**: Specialized agents spawned by a primary agent for specific tasks
- **Memory**: Agent ability to persist information across sessions
- **Planning**: Agent's ability to break down tasks into steps before executing

## Mental Models
- Use **agents** for complex, multi-file tasks that require planning and iteration
- Use **inline chat** for quick code suggestions without leaving your workflow
- Use **custom instructions** to shape how agents behave in your project

## Anti-patterns
- **Vague prompts**: "Make it better" wastes agent cycles; be specific about what and why
- **Over-reliance on agents**: Agents work best when you verify their output
- **Ignoring context**: Agents perform better when given relevant files and clear scope

## Code Examples
```yaml
# .github/copilot-instructions.md (project-level custom instructions)
You are a Python developer working on a FastAPI project.
- Always use type hints
- Follow PEP 8 style
- Add docstrings to all public functions
- Use pytest for testing
```

## Key Takeaways
1. Start with Copilot Chat for exploratory questions; use agents for execution tasks
2. Provide context: mention files, functions, or paste relevant code in prompts
3. Review agent edits before accepting — use the diff view to inspect changes
4. Use custom instructions (`.github/copilot-instructions.md`) for project-specific guidance
5. Break complex tasks into smaller agent invocations for better results

## Connects To
- **Ch 4**: Customize agents with plugins, skills, and hooks
- **Ch 12**: Use agents for debugging assistance
- **Ch 2**: Inline chat enhances editing workflow
