# Chapter 4: How Tools Work

## Core Idea
Zoo Code uses specialized tools (read, edit, execute, workflow) to interact with your codebase. Every tool call is proposed with full parameters, requires your explicit approval, and executes only after you click "Save"—giving you granular control over all AI-driven changes.

## Frameworks Introduced
- **Tool-as-Proposal Pattern**: Zoo Code doesn't execute actions directly. It proposes a tool with its parameters (displayed as XML), you review the exact operation, then approve or reject. This separates intention from execution.
- **Category-Based Tool Organization**: Tools are grouped into Read, Edit, Execute, Image, and Workflow categories. Understanding which category a task falls into helps you predict what Zoo Code will propose.

## Key Concepts
- **Tool Proposal**: The XML-formatted representation of what Zoo Code wants to do—includes the tool name, parameters (file paths, content, commands), and metadata like line counts.
- **Approval Workflow**: Every tool use requires explicit user approval via "Save" (approve) or "Reject" (decline) buttons. Auto-approval is available for trusted operations.
- **Read Tools**: `read_file`, `search_files`, `list_files`, `codebase_search`, `read_command_output`—for accessing information without modifying anything.
- **Edit Tools**: `write_to_file`, `apply_diff`, `apply_patch`, `edit`, `edit_file`, `search_replace`—for creating or modifying files. Multiple variants handle different granularity levels.
- **Execute Tools**: `execute_command`—runs commands in the VS Code terminal.
- **Workflow Tools**: `ask_followup_question`, `attempt_completion`, `switch_mode`, `new_task`, `skill`—for managing task flow and context.
- **Auto-approve**: Optional setting to skip manual approval for trusted operations. Use cautiously.

## Mental Models
- **Tool as contract**: Each tool proposal is a contract: "I will do X with these exact parameters." Review it like you'd review a pull request. If the parameters are wrong, reject and provide feedback.
- **Read-before-write**: Zoo Code typically reads files before editing them. This is a safety pattern—understand the current state before proposing changes. If you see Zoo Code skipping reads, flag it.
- **Granularity awareness**: `apply_diff` is surgical (changes specific lines), `write_to_file` is wholesale (replaces entire file), `edit` is search-and-replace. Know which tool is being proposed to understand the scope of change.
- **Iterative refinement**: Tools execute one at a time. Zoo Code proposes, you approve, it executes, then proposes the next step. This loop continues until the task is complete.

## Anti-patterns
- **Blindly approving tool proposals**: Always review the XML parameters—check file paths, content, and command strings before clicking Save. A misplaced `write_to_file` can overwrite critical code.
- **Overusing auto-approve**: Auto-approval is convenient but removes your safety net. Only enable it for operations you fully trust (e.g., reading files in a known directory).
- **Ignoring tool category**: If Zoo Code proposes `execute_command` when you expected a file edit, something may be wrong with its understanding. Reject and clarify.
- **Expecting batch execution**: Zoo Code executes tools sequentially, one approval at a time. Don't expect it to "just do everything"—each step needs your sign-off.

## Code Examples
```xml
<!-- Tool proposal example: write_to_file -->
<write_to_file>
<path>greeting.js</path>
<content>
function greet(name) {
  console.log(`Hello, ${name}!`);
}

greet('World');
</content>
<line_count>5</line_count>
</write_to_file>
```

```xml
<!-- Tool proposal example: execute_command -->
<execute_command>
<command>npm install</command>
</execute_command>
```

```xml
<!-- Tool proposal example: apply_diff -->
<apply_diff>
<path>src/utils.ts</path>
<diff>
<<<<<<< SEARCH
:old line of code
=======
:new line of code
>>>>>>> REPLACE
</diff>
</apply_diff>
```

## Worked Example
**Task**: Create a greeting script.

1. You type: `Create a file named greeting.js that logs a greeting message`
2. Zoo Code proposes `write_to_file` with path `greeting.js` and the function content (shown as XML above).
3. The tool approval interface appears with "Save" and "Reject" buttons, plus an "Auto-approve" checkbox.
4. You review the proposed content—confirm it's correct.
5. Click "Save" to approve.
6. Zoo Code executes the tool, creates the file, and confirms success.
7. It then proposes `attempt_completion` to signal the task is done.

## Key Takeaways
1. Every tool call is a proposal—you review XML parameters before anything executes.
2. Tools are categorized (Read, Edit, Execute, Image, Workflow) and understanding categories helps predict Zoo Code's behavior.
3. `write_to_file` overwrites entire files; `apply_diff` and `edit` make surgical changes—know which is being proposed.
4. Auto-approval exists but should be used sparingly; the approval workflow is your primary safety mechanism.
5. Tools execute sequentially in an iterative loop: propose → approve → execute → repeat until complete.

## Connects To
- **Ch 1**: The approval workflow introduced in "Your First Task" is the same mechanism used for all tool calls.
- **Ch 2**: Tool proposals appear in the chat interface; action buttons above the input field are for approving/rejecting them.
- **Ch 3**: Context mentions provide the information Zoo Code uses to select appropriate tools and parameters.
