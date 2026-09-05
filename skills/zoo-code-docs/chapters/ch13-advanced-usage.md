# Chapter 13: Advanced Usage

## Core Idea
Advanced usage techniques help you manage context effectively, recover from corrupted sessions, work with local models, and craft prompts that produce reliable results from Zoo Code.

## Key Concepts
- **Context Poisoning**: When inaccurate or irrelevant data contaminates the language model's active context, causing it to draw incorrect conclusions and progressively deviate from the intended task. Once a session is poisoned, it is effectively disposable.
- **Context Window**: The maximum amount of text (measured in tokens) the LLM can process at once, including system prompt, conversation history, file contents, and tool outputs.
- **Prompt Caching**: A feature supported by providers like Anthropic, OpenAI, OpenRouter, and Requesty that caches prompts for reuse in future tasks, reducing cost and latency.
- **Local Models**: LLMs run on your own machine via Ollama or LM Studio, offering privacy, offline access, and cost savings at the expense of resource requirements and potentially lower performance.
- **Custom Instructions**: Persistent guidance added to the system prompt—either global (all modes) or mode-specific—that enforces coding style, preferred libraries, or project conventions.
- **System Prompt**: The foundation of Zoo Code's behavior, dynamically generated each interaction, containing role definition, tool descriptions, capabilities, operational rules, and custom instructions.
- **Support Prompts**: Specialized templates for specific code actions (Explain, Fix, Improve) that operate independently of the main chat history.

## Mental Models
- **Think-then-do workflow**: Guide Zoo Code through Analyze → Plan → Execute → Review for each task step, preventing hasty actions.
- **Session hygiene**: Treat each chat session as a disposable workspace—start fresh when context degrades rather than trying to patch corrupted context.
- **Context as a resource**: The context window is finite; allocate it deliberately by being specific with file references, breaking tasks into sub-tasks, and summarizing large code sections.
- **Layered prompt architecture**: System prompt + user messages + assistant responses + tool results form the complete context—understanding this helps you avoid redundancy.

## Anti-patterns
- **Wake-up prompt after poisoning**: Attempting to "fix" a poisoned session by re-injecting tool definitions or directives. The corrupted data persists in the session history and will resurface.
- **Vague file references**: Using phrases like "the main file" instead of specific paths and function names, wasting context on ambiguity.
- **Dumping entire logs into chat**: Pasting large logs or text with hidden control characters contaminates context; be selective about what you include.
- **Ignoring context window overflow**: As sessions grow, older useful information is pushed out, letting poisoned data dominate. Break large tasks into focused sessions.

## Code Examples
```
# Good prompt with context mention
@/src/components/Button.tsx Refactor the `Button` component to use the `useState` hook instead of `useReducer`.

# Progressive task breakdown for large file refactoring
@/src/components/MyComponent.tsx List the functions and classes in this file.
@/src/components/MyComponent.tsx Refactor the `processData` function to use async/await instead of Promises.

# Ollama local model setup
ollama pull qwen2.5-coder:32b
ollama run qwen2.5-coder:32b
/set parameter num_ctx 32768
/save my-qwen-32k

# Modelfile approach (recommended for reproducibility)
FROM qwen2.5-coder:32b
PARAMETER num_ctx 32768
PARAMETER temperature 0.7
```

## Worked Example
**Scenario**: You need to refactor a 500-line TypeScript file but keep hitting context limits.

1. Start with an overview: `@/src/components/MyComponent.tsx List the functions and classes in this file.`
2. Target specific functions: `@/src/components/MyComponent.tsx Refactor the processData function to use async/await.`
3. Make small incremental changes, reviewing each step before proceeding.
4. If the session degrades (nonsensical suggestions, tool misalignment), start a new session rather than trying to patch it.

## Key Takeaways
1. Context poisoning is permanent within a session—start fresh rather than trying to fix corrupted context.
2. Be specific with file paths and function names; break large tasks into focused sub-tasks.
3. Local models offer privacy and offline access but require careful resource management and may lack advanced features.
4. The system prompt is dynamically generated and includes your custom instructions—use it to enforce project conventions.

## Connects To
- **Ch 14**: Token usage optimization directly impacts cost when working with advanced prompting techniques.
- **Ch 15**: Local model providers (Ollama, LM Studio) are configured through the provider system.
- **Ch 17**: Tips like disabling MCP and using sticky models complement advanced usage patterns.
