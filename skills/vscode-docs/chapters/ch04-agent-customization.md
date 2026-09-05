# Chapter 4: Agent Customization & Plugins

## Core Idea
Extensively customize agent behavior through plugins, skills, custom agents, hooks, MCP servers, and prompt files that shape how AI interacts with your codebase.

## Frameworks Introduced
- **Agent Plugins**: Portable packages bundling skills, MCP servers, agents, and hooks (open standard at agent-plugins.org)
- **Skills**: On-demand instructions, scripts, and resources that load when triggered by topic or command
- **Custom Agents**: Specialized AI personas with specific tool configurations and behaviors
- **Hooks**: Shell commands that execute at agent lifecycle points (before/after tool execution, session start/end)
- **MCP Servers**: External tool integrations providing agents with additional capabilities
- **Prompt Files**: Reusable prompt templates stored in `.github/prompts/` or `.agents/prompts/`

## Key Concepts
- **Agent Plugin Standard**: Open standard for packaging agent customizations; supports portable skills and MCP servers plus client-specific components
- **Skill Files**: `SKILL.md` + supporting files in `skills/` directory; loaded on-demand when topic matches
- **Custom Agents**: Define in `.agent.md` files with persona, tools, and instructions
- **Hooks**: JSON configuration triggering shell commands at specific agent events
- **Tool Sets**: Named collections of tools that can be assigned to agents or skills
- **Language Models**: Configure which AI models agents use and how they're accessed
- **Workspace Context**: How agents discover and use project files and information

## Mental Models
- Use **Skills** for reusable knowledge that multiple agents can share
- Use **Custom Agents** when you need a specialist persona (e.g., code reviewer, test writer)
- Use **Hooks** to enforce policies or add custom behavior to agent workflows
- Use **Prompt Files** for complex, parameterized prompts you reuse frequently

## Anti-patterns
- **Over-customizing**: Start with defaults; customize only when you have a specific need
- **Hardcoding secrets in hooks**: Use environment variables; never commit API keys
- **Ignoring security**: Review plugin contents before installing; hooks run arbitrary code

## Code Examples
```json
// .agents/hooks/hooks.json
{
  "onToolCall": [
    {
      "pattern": "bash",
      "command": "echo 'Tool called: ${TOOL_NAME} at ${TIMESTAMP}' >> .agent-audit.log"
    }
  ]
}
```

```yaml
# .github/prompts/code-review.prompt.md
---
mode: agent
tools: [read_file, search]
---
Review the following code for:
1. Security vulnerabilities
2. Performance issues
3. Style violations
4. Missing error handling

Provide specific line-by-line feedback.
```

## Key Takeaways
1. Agent Plugins are the standard for portable customization across VS Code and other tools
2. Skills load on-demand — keep them focused on specific topics
3. Custom agents shine when you need consistent personas across team members
4. Hooks provide lifecycle control — use for auditing, validation, and automation
5. Prompt files make complex prompts reusable and version-controlled

## Connects To
- **Ch 3**: Core agent concepts and usage patterns
- **Ch 15**: Enterprise management of agent customizations
- **Ch 17**: Reference for all configuration options
