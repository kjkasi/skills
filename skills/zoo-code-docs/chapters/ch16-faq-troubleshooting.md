# Chapter 16: FAQ & Troubleshooting

## Core Idea
Common issues with Zoo Code—API connectivity, context errors, markdown writing failures, and unwanted changes—have straightforward solutions, and proper error reporting accelerates resolution when self-service debugging isn't enough.

## Key Concepts
- **Modes**: Different personas (Code, Architect, Ask, Debug, Custom) with specific capabilities. Switch via dropdown or `/` command. Architect mode is read-only, making it safe for analysis.
- **Context Mentions**: `@` followed by file paths, folders, or `problems` to provide Zoo Code with specific project context.
- **Auto-Approval Settings**: Configure which actions Zoo Code can perform without explicit approval. Use with extreme caution for terminal commands.
- **Checkpoints**: Experimental feature enabling Zoo Code to revert file changes to a previous state.
- **Error Details Modal**: Click "Details" next to error messages to access diagnostic export options—basic (clipboard) or detailed (full task history + config).
- **`.roorules` Files**: Project-level configuration files that provide additional guidelines to Zoo Code, similar to `.editorconfig` for AI behavior.
- **Codebase Indexing**: Semantic search index using AI embeddings (requires OpenAI API key + Qdrant vector database) for navigating large codebases by meaning rather than keywords.
- **VS Code Language Model API**: Experimental integration allowing Zoo Code to use models from GitHub Copilot and other VS Code extensions.

## Mental Models
- **Undo-first recovery**: When Zoo Code makes unwanted changes, use Ctrl/Cmd+Z (VS Code's built-in undo) or checkpoints before investigating root cause.
- **Error triage**: Check API key → Check internet → Check provider status → Restart VS Code → Report issue. This sequence resolves most connectivity problems.
- **Extension conflict diagnosis**: Markdown writing failures often stem from VS Code extensions (format-on-save, preview mode), not Zoo Code itself.
- **Context pollution isolation**: Use Debug mode in a separate task to avoid contaminating your main conversation with debugging context.

## Anti-patterns
- **Auto-approving terminal commands without review**: Zoo Code can execute arbitrary shell commands; auto-approval removes the safety checkpoint.
- **Ignoring markdown extension conflicts**: Format-on-save and preview-mode extensions can silently break Zoo Code's file writing—disable them to test.
- **Sharing exported settings files**: The Roo Code export file may contain API keys; store it securely and never share it.
- **Using the wrong model ID for Bedrock**: Bedrock requires model IDs (e.g., `anthropic.claude-3-sonnet-20240229-v1:0`), not friendly names—this causes "Model Not Found" errors.

## Code Examples
```
# Starting a new Debug task to isolate context
"Start a new task in Debug mode with all of the necessary context needed to figure out X"

# Context mention patterns
@/src/utils.ts          # Specific file
@problems               # Current errors/warnings
@/commit/abc123         # Specific git commit

# Disabling markdown preview settings (VS Code settings.json)
{
  "markdown.preview.openMarkdownLinks": "inPreview",
  "workbench.editorAssociations": {
    "*.md": "vscode.markdown.preview.editor"
  }
}
# Remove these settings if they interfere with Zoo Code writing .md files

# Error reporting workflow
1. Click "Details" next to error message
2. Choose "Copy basic error info" for quick Discord sharing
3. Choose "Get detailed error info" for GitHub issues (includes full task history)
4. Share via email (support@zoocode.dev), Discord (#support), or GitHub Issues
```

## Worked Example
**Scenario**: Zoo Code fails to write to markdown files with "Failed to open diff editor" error.

1. Check VS Code extensions—disable any format-on-save extensions temporarily.
2. Remove markdown preview settings from `settings.json` (see code example above).
3. Restart VS Code after changes.
4. Test with a simple markdown write operation.
5. If the issue persists, click "Details" → "Get detailed error info" → file a GitHub issue with the diagnostic report.

## Key Takeaways
1. Most "Zoo Code isn't responding" issues resolve by checking API key validity, internet connection, and provider status.
2. Markdown writing failures are usually VS Code extension conflicts, not Zoo Code bugs—disable format-on-save and preview extensions.
3. Use the Error Details modal to export diagnostic information for faster support resolution.
4. Isolate debugging tasks using Debug mode to prevent context pollution in your main conversation.

## Connects To
- **Ch 13**: Context poisoning is a common cause of degraded output quality—start a new session to recover.
- **Ch 14**: Token limits and context window overflow cause "input length and max tokens exceed context limit" errors.
- **Ch 17**: Many troubleshooting issues are avoided by following tips like disabling MCP and using sticky models.
