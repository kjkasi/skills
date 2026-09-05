# Chapter 5: Prompts

## Core Idea
Prompts are reusable interaction templates. They help a user or a host invoke a proven workflow with the right context, tools, and constraints. They do not replace the model; they structure the interaction around a repeatable pattern.

## Frameworks Introduced
- **Prompt-as-workflow**: package a sequence or template into a single named interaction.
  - When to use: when repeated user tasks need consistent structure
  - How: provide a prompt name, arguments, and expected behavior in a reusable form
- **Prompt composition with resources and tools**: combine context and actions into one workflow.
  - When to use: for tasks such as summarization, triage, or guided analysis
  - How: bind resources and tool selection to a prompt template to standardize execution

## Key Concepts
- **prompt template**: a reusable instruction or format for the model
- **user-controlled selection**: users choose which prompt to run, or the host surfaces them in UI
- **workflow normalization**: the same task becomes repeatable instead of ad hoc

## Mental Models
Prompts are the protocol’s way of standardizing how an AI assistant handles a task. Think of them as reusable playbooks: “analyze my PR,” “write a release note,” or “summarize incidents from these resources.” They reduce ambiguity while keeping the model’s reasoning intact.

## Anti-patterns
- **Prompting as a dump of raw instructions**: becomes brittle and hard to maintain
- **Embedding policy decisions in every prompt**: leaks governance into each workflow instead of using tooling and policies properly

## Key Takeaways
1. Prompts standardize workflow structure, not business logic.
2. Combine prompts with tools and resources for stronger operational patterns.
3. The host can surface prompts in a UI for user-led task selection.

## Connects To
- **Chapter 3**: tool use often starts from a prompt-driven workflow
- **Chapter 4**: resources supply the context prompts rely on
- **Chapter 6**: policy and auth still govern actions triggered by a prompt
