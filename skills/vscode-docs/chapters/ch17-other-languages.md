# Chapter 17: Other Languages

## Core Idea
VS Code supports many programming languages through extensions, each providing language-specific features like IntelliSense, debugging, and formatting.

## Frameworks Introduced
- **Language Server Protocol (LSP)**: Standard protocol for language intelligence features
- **Extension-based Language Support**: Each language gets features via extensions
- **Multi-language Workspaces**: Work with multiple languages in one project

## Key Concepts
- **Language Extensions**: Install from marketplace for language support
- **IntelliSense**: Language-specific code completions and hints
- **Language Configuration**: Customize brackets, comments, folding per language
- **Embedded Languages**: Support for languages within other languages (e.g., SQL in strings)

## Mental Models
- Search marketplace for your language + "VS Code" to find best extensions
- Most languages need at least: language support + linter + formatter
- Check extension reviews and install counts for quality indication

## Supported Languages (partial list)
- **Web**: HTML, CSS, SCSS, JavaScript, TypeScript, JSON, Markdown
- **Systems**: C, C++, Rust, Go
- **Managed**: Java, C#, F#, VB.NET
- **Scripting**: Python, Ruby, PHP, Perl, Lua
- **Functional**: Haskell, OCaml, Elixir, Clojure
- **Data**: R, Julia, MATLAB
- **Mobile**: Swift, Kotlin
- **Infrastructure**: Terraform, Docker, YAML, TOML

## Key Takeaways
1. Most languages have official or well-maintained extensions
2. Language support includes: syntax highlighting, IntelliSense, debugging, formatting
3. Some languages need additional tools installed on your system
4. Extension recommendations in `.vscode/extensions.json` help team consistency
5. Check extension compatibility with your VS Code version

## Connects To
- **Ch 1**: Installing language extensions
- **Ch 2**: Editing features work across languages
- **Ch 5-9**: Specific language guides
- **Ch 14**: Language-specific settings
