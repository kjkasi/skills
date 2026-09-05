# Chapter 10: Tools Reference - Search & Edit

## Core Idea
Zoo Code offers layered search and edit tools: `search_files` for regex-based codebase discovery via Ripgrep, and three text-replacement tools (`edit`, `edit_file`, `search_replace`) that differ in replacement scope and safety guarantees.

## Key Concepts
- **search_files**: Regex search across files using Ripgrep. Parameters: `path` (directory), `regex` (Rust regex syntax), `file_pattern` (glob filter), `respect_gitignore` (default true). Returns matches with 1-line context, capped at 300 results. Merges nearby matches into contiguous blocks.
- **edit**: Search-and-replace replacing **first occurrence only** by default. Optional `replace_all: true` for global replacement. Exact string matching (case/whitespace sensitive, no regex). Parameters: `file_path`, `old_string`, `new_string`, `replace_all`.
- **edit_file**: Replaces **exactly one** uniquely-identified occurrence. Errors if multiple matches found unless `expected_replacements` is set. Supports `old_string=""` for file creation/appending. Parameters: `file_path`, `old_string`, `new_string`, `expected_replacements`.
- **search_replace**: Simplest interface—replaces **exactly one** uniquely-identified occurrence. Three parameters only: `file_path`, `old_string`, `new_string`. Errors on multiple matches as a safety design.

## Code Examples

Regex search for TODOs in TypeScript:
```xml
<search_files>
<path>src</path>
<regex>TODO|FIXME</regex>
<file_pattern>*.ts</file_pattern>
</search_files>
```

Search including gitignored files:
```xml
<search_files>
<path>.</path>
<regex>console\.log</regex>
<respect_gitignore>false</respect_gitignore>
</search_files>
```

Replace first occurrence only:
```xml
<edit>
<file_path>src/config.ts</file_path>
<old_string>const API_URL = "http://old.api.com"</old_string>
<new_string>const API_URL = "https://new.api.com"</new_string>
</edit>
```

Replace all occurrences globally:
```xml
<edit>
<file_path>src/config.ts</file_path>
<old_string>oldFunction</old_string>
<new_string>newFunction</new_string>
<replace_all>true</replace_all>
</edit>
```

Unique replacement with `edit_file`:
```xml
<edit_file>
<file_path>src/app.ts</file_path>
<old_string>import { LegacyAuth } from './legacy-auth'</old_string>
<new_string>import { ModernAuth } from './auth'</new_string>
</edit_file>
```

Create a new file via `edit_file`:
```xml
<edit_file>
<file_path>src/utils.ts</file_path>
<old_string></old_string>
<new_string>export const greet = (name: string) => `Hello, ${name}!`</new_string>
</edit_file>
```

## Worked Example
A developer needs to find and update all API endpoint references. First, `search_files` with regex `api\.example\.com` locates 12 occurrences across 5 files. To change only the base URL in the config file, `edit_file` targets the uniquely-identified config line. For a global rename of a utility function across a single file, `edit` with `replace_all: true` handles all 8 occurrences at once.

## Key Takeaways
1. `search_files` respects `.gitignore` by default—use `respect_gitignore: false` only for debugging or auditing.
2. The three replacement tools differ in safety: `edit` defaults to first match; `edit_file` and `search_replace` error on ambiguous matches.
3. `edit_file` doubles as a file creation tool when `old_string` is empty—no need to switch to `write_to_file` for simple files.
4. None of the text-replacement tools support regex—use `apply_diff` with fuzzy matching or `search_files` to locate, then `edit_file` to replace.

## Connects To
- **Ch 9**: `apply_diff` provides context-aware replacement when exact-string tools are too rigid.
- **Ch 11**: `execute_command` can run project-specific grep/linting; `search_files` is the built-in alternative.
- **Ch 12**: `skill` tool can load specialized workflows that orchestrate search-then-edit patterns.
