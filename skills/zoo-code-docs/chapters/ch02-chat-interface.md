# Chapter 2: Chat Interface & Requests

## Core Idea
The Zoo Code chat interface is your primary workspace—consisting of a chat history, input field, action buttons, and mode selector. Communicating effectively means being specific, providing context, and breaking tasks into focused steps.

## Frameworks Introduced
- **Contextual Communication Model**: Zoo Code understands natural language, but accuracy improves dramatically when you reference specific files, functions, and expected outcomes rather than speaking in abstractions.
- **Theme-Aware UI Architecture**: The chat composer, controls, and all interactive elements follow your active VS Code theme (light, dark, high-contrast) for visual consistency.

## Key Concepts
- **Chat History**: Displays the full conversation—your requests, Zoo Code's responses, and all actions taken (file edits, command executions).
- **Input Field**: Where you type tasks in plain English. No special commands or syntax required.
- **Action Buttons**: Context-sensitive buttons above the input for approving or rejecting proposed actions. Change based on what Zoo Code is proposing.
- **Mode Selector**: Dropdown to the left of the input field that switches Zoo Code's operational mode (e.g., different workflows for coding vs. debugging).
- **Secondary Sidebar**: Drag Zoo Code's icon to the right side of the editor to keep it visible alongside Explorer, Search, and Source Control panels.
- **Status Indicators**: Loading spinner (processing), red error messages (failures), green messages (success).

## Mental Models
- **Conversation as interface**: Think of the chat not as a command line but as a dialogue. Zoo Code proposes actions, you approve or reject, and it adapts based on your feedback. Each exchange builds context.
- **Focused requests > broad requests**: One specific request per message yields better results than bundling multiple unrelated tasks. Break complex work into steps.
- **Context is king**: The more precisely you describe what you want and where it lives (file paths, function names, expected behavior), the more accurate Zoo Code's output.

## Anti-patterns
- **Vague requests**: "Fix the code" gives Zoo Code no direction. "Fix the bug in `calculateTotal` that returns incorrect results" is actionable.
- **Assuming implicit context**: Zoo Code doesn't automatically know which file or function you mean. Use `@file` mentions or specify paths explicitly.
- **Bundling unrelated tasks**: "Create a file and also fix that bug and deploy"—submit one focused request at a time.
- **Excessive jargon**: Use clear, straightforward language. Zoo Code understands plain English better than obscure technical shorthand.
- **Skipping confirmation**: Always review Zoo Code's output before moving on. Check that files are complete and correct.

## Code Examples
```
# Effective request examples:

# Specific file creation
create a new file named `utils.py` and add a function called `add` 
that takes two numbers as arguments and returns their sum

# Context-aware edit with file mention
in the file @src/components/Button.tsx, change the color of the button to blue

# Search and replace across a file
find all instances of the variable `oldValue` in @/src/App.js 
and replace them with `newValue`

# Command execution
run the command `npm install` in the terminal

# Code explanation with context
explain the function `calculateTotal` in @/src/utils.ts

# Diagnostics-driven fix
@problems address all detected problems
```

## Worked Example
**Task**: Fix a button color in a React component.

1. Open Zoo Code panel (click the icon in Activity Bar).
2. Type: `in the file @src/components/Button.tsx, change the color of the button to blue`
3. Press Enter.
4. Zoo Code reads the file, proposes a diff (green for additions, red for removals).
5. Review the diff—confirm it only changes the color property.
6. Click "Approve" to apply the change.
7. Zoo Code confirms and awaits next instruction.

## Key Takeaways
1. The chat interface is theme-aware and adapts to your VS Code appearance (light, dark, high-contrast).
2. Drag Zoo Code to the Secondary Sidebar to keep it visible while using other panels.
3. Effective requests are specific, include file/function references, and focus on one task at a time.
4. Mermaid diagrams render directly in chat and update automatically when you change themes.

## Connects To
- **Ch 1**: Setup must be complete before using the chat interface.
- **Ch 3**: Context mentions (`@file`, `@folder`, `@problems`) are the primary way to give Zoo Code precise context.
- **Ch 4**: Every action Zoo Code proposes in the chat is a tool call that requires your approval.
