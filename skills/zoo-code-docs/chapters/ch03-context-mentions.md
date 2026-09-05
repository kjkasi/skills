# Chapter 3: Context Mentions

## Core Idea
Context mentions (`@` prefix) let you inject precise project information—files, folders, diagnostics, terminal output, git history, and URLs—directly into your requests so Zoo Code can act with full context.

## Frameworks Introduced
- **Mention-Driven Context Injection**: Instead of describing what you want Zoo Code to look at, you reference it directly with `@`. This eliminates ambiguity and reduces back-and-forth.
- **Selective Content Filtering**: Mentions respect `.rooignore` for dropdown suggestions but bypass ignore rules when you explicitly reference a file—giving you control over what's visible vs. what's accessible.

## Key Concepts
- **`@/path/to/file.ts` (File Mention)**: Includes the complete file contents with line numbers. Works for text files, PDFs, and DOCX. Always use `/` from workspace root.
- **`@/path/to/image.png` (Image Mention)**: Sends images as inline visual content (requires vision-capable model). Supports PNG, JPG, GIF, SVG, WEBP, and more.
- **`@/path/to/folder/` (Folder Mention)**: Includes contents of all non-binary files directly in the folder (non-recursive). Trailing slash is required to distinguish from file mentions.
- **`@problems`**: Imports all errors and warnings from VS Code's Problems panel, grouped by file with line numbers and diagnostic messages.
- **`@terminal`**: Captures the last command and its complete output without clearing the terminal. Limited to visible buffer content.
- **`@a1b2c3d` (Git Commit)**: References a specific commit by hash—includes message, author, date, and complete diff. Only works in Git repos.
- **`@git-changes`**: Shows `git status` output and diff of all uncommitted changes.
- **`@https://example.com` (URL Mention)**: Fetches website content via headless browser, strips scripts/styles/navigation, converts to Markdown.
- **`/<command-name>` (Slash Command)**: Executes predefined commands (e.g., `/test`, `/init`). Uses `/` prefix, not `@`.

## Mental Models
- **Mention as pointer**: Think of `@` as a pointer that tells Zoo Code "look exactly here." It's not a search—it's a direct reference. This is the difference between "look at the code" and "look at line 42 of `src/utils.ts`."
- **Context budget**: Every mention adds content to the conversation. Large files or directories consume context window space. Be strategic—mention only what's relevant.
- **Ignore rules are suggestions, not walls**: `.rooignore` hides files from the dropdown but doesn't prevent you from mentioning them directly. This is intentional—you might need to reference an ignored file for debugging.

## Anti-patterns
- **Mentioning entire large directories**: `@/src/` on a big project will blow your context window. Mention specific files or narrow subdirectories instead.
- **Forgetting the trailing slash for folders**: `@/path/to/folder` without `/` tries to match a file, not a folder. Always use `@/path/to/folder/`.
- **Assuming git mentions work everywhere**: `@git-changes` and `@a1b2c3d` only work inside Git repositories. Outside a repo, they'll fail.
- **Using non-vision models for image mentions**: If your model doesn't support vision, `@image.png` will be treated as text (file path) rather than visual content.
- **Overloading a single request with many mentions**: Combine mentions sparingly. "Fix @problems in @/src/component.ts" is good; stuffing 10 mentions into one request dilutes focus.

## Code Examples
```
# File mention with code explanation
explain the function `calculateTotal` in @/src/utils.ts

# Folder mention for batch analysis
Analyze the code in @/src/components/

# Problems-driven fix
@problems address all detected problems

# Terminal debugging
Fix the errors shown in @terminal

# Git context
What changed in commit @a1b2c3d?

# Uncommitted changes
Suggest a commit message for @git-changes

# URL import
Summarize @https://docusaurus.io/

# Slash command
/test Run all tests

# Combining mentions
Fix @problems in @/src/component.ts
```

## Worked Example
**Task**: Fix all TypeScript errors in your project.

1. Type: `@problems address all detected problems`
2. Zoo Code imports the VS Code Problems panel—all errors and warnings with file paths and line numbers.
3. It proposes fixes for each error, one at a time.
4. You approve each fix; Zoo Code applies the edit and moves to the next error.
5. Continue until the Problems panel is clear.

## Key Takeaways
1. `@` mentions are the most powerful way to give Zoo Code precise context—use them instead of describing locations.
2. Folder mentions require a trailing slash (`@/folder/`) and are non-recursive (only direct children).
3. `.rooignore` hides files from suggestions but doesn't prevent direct mentions—you can always reference ignored files explicitly.
4. Git mentions (`@git-changes`, `@a1b2c3d`) only work inside Git repositories.
5. Image mentions require a vision-capable model; without one, image paths are treated as text.

## Connects To
- **Ch 2**: Context mentions are typed into the chat interface's input field.
- **Ch 4**: When you use `@problems`, Zoo Code may propose tool calls (like `apply_diff`) to fix the issues it found.
- **Ch 1**: Effective use of mentions requires a configured provider—Zoo Code needs an LLM to process the context.
