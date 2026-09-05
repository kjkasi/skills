# Chapter 9: Tools Reference - File Operations

## Core Idea
Zoo Code provides a comprehensive file operations toolkit: `read_file` for reading content (including images, PDFs, DOCX), `write_to_file` for creating or replacing files with diff-view approval, `apply_diff` for surgical single-file edits via fuzzy matching, and `apply_patch` for multi-file unified diff patches.

## Key Concepts
- **read_file**: Reads files with two modes—`slice` (offset/limit) and `indentation` (extract complete code blocks around an anchor line). Supports multi-file concurrent reads via `args` format, token budget auto-truncation, and images (PNG, JPG, GIF, WebP, SVG, BMP, ICO, TIFF, AVIF) as base64 data URLs.
- **write_to_file**: Creates new files or completely replaces content. Requires `path`, `content`, and `line_count`. Content is preprocessed to strip AI artifacts; a diff view lets users review and edit before approval. Slower than `apply_diff` for modifying existing files.
- **apply_diff**: Makes targeted search-and-replace changes using fuzzy matching (Levenshtein distance) guided by `:start_line:` hints within a context window (default 40 lines). Supports multiple diff blocks in one request. Uses `<<<<<<< SEARCH` / `=======` / `>>>>>>> REPLACE` format.
- **apply_patch**: Applies unified diff patches across multiple files in one operation. Uses custom headers: `*** Add File:`, `*** Update File:`, `*** Delete File:`. Ideal for multi-file refactoring or applying VCS-generated patches.

## Code Examples

Reading a line range (slice mode):
```xml
<read_file>
<path>src/app.js</path>
<offset>46</offset>
<limit>23</limit>
</read_file>
```

Reading multiple files concurrently:
```xml
<read_file>
<args>
  <file><path>src/app.ts</path></file>
  <file><path>src/utils.ts</path><line_range>10-25</line_range></file>
</args>
</read_file>
```

Applying a surgical diff:
```xml
<apply_diff>
<path>src/utils.js</path>
<diff>
<<<<<<< SEARCH
:start_line:10
:end_line:12
-------
    const result = value * 0.9;
    return result;
=======
    console.log(`Calculating for value: ${value}`);
    const result = value * 0.95;
    return result;
>>>>>>> REPLACE
</diff>
</apply_diff>
```

Multi-file patch:
```xml
<apply_patch>
<patch>
*** Add File: src/utils/newHelper.ts
--- /dev/null
+++ b/src/utils/newHelper.ts
@@ -0,0 +1,3 @@
+export function helper(value: string): string {
+  return value.toUpperCase();
+}

*** Update File: src/main.ts
--- a/src/main.ts
+++ b/src/main.ts
@@ -10,3 +10,3 @@
-const timeout = 5000;
+const timeout = 10000;
</patch>
</apply_patch>
```

Writing a new file:
```xml
<write_to_file>
<path>config/settings.json</path>
<content>{"apiUrl": "https://api.example.com", "version": "1.0.0"}</content>
<line_count>3</line_count>
</write_to_file>
```

## Worked Example
A developer needs to refactor a utility module. First, `read_file` with `line_range` loads the relevant sections of `utils.js` (lines 1-50 and 80-120). After analysis, `apply_diff` replaces a specific function at line 42 using fuzzy matching. A new helper is added to `utils/helpers.js` using `apply_patch` alongside an import update in `main.ts`. Finally, `write_to_file` creates the updated `package.json` with a new dependency.

## Key Takeaways
1. Prefer `apply_diff` over `write_to_file` for modifying existing files—it's faster and preserves context.
2. Use `read_file`'s indentation mode to extract complete functions/classes around a target line rather than guessing line ranges.
3. `apply_patch` is the only tool for atomic multi-file operations (add, delete, update) in a single request.
4. All write tools require user approval via diff view before changes are applied, ensuring safety.

## Connects To
- **Ch 10**: Search/edit tools locate content before `apply_diff` targets it.
- **Ch 11**: `execute_command` can generate patches externally, then `apply_patch` applies them.
- **Ch 12**: MCP tools may provide external data that gets written to files.
