# Chapter 8: C# & .NET Development

## Core Idea
VS Code provides C# development through the C# Dev Kit extension, including IntelliSense, debugging, testing, and project management for .NET applications.

## Frameworks Introduced
- **C# Dev Kit**: Microsoft's extension for C# development with solution/explorer views
- **OmniSharp**: Language server providing IntelliSense and code navigation
- **.NET Debugging**: Integrated debugging for .NET applications
- **Solution Explorer**: Visual Studio-like project and solution management

## Key Concepts
- **C# Dev Kit**: Provides solution view, test explorer, and project management
- **IntelliSense**: Code completions, signatures, hover info powered by OmniSharp
- **Debugging**: Breakpoints, step-through, variable inspection for .NET applications
- **Testing**: xUnit, NUnit, MSTest support with test explorer
- **Code Analysis**: Built-in analyzers for code quality and style
- **NuGet**: Package management integration

## Mental Models
- Use **C# Dev Kit** for full Visual Studio-like experience in VS Code
- Use **Solution Explorer** to manage multi-project solutions
- Use **Test Explorer** for visual test management and coverage

## Code Examples
```json
// .vscode/settings.json (C# project)
{
  "dotnet.completion.useCompletionItemsFromWorkspace": true,
  "dotnetalyzer.severity": "warning",
  "omnisharp.enableEditorConfigSupport": true,
  "omnisharp.enableRoslynAnalyzers": true
}
```

## Key Takeaways
1. Install C# Dev Kit for complete C# development experience
2. Solution Explorer provides Visual Studio-like project management
3. Roslyn analyzers provide code quality feedback inline
4. OmniSharp provides IntelliSense; restart language server if issues occur
5. Use .NET CLI from integrated terminal for build/run commands

## Connects To
- **Ch 1**: Setting up C# development environment
- **Ch 2**: Code editing and refactoring
- **Ch 12**: Debugging and testing workflows
- **Ch 11**: Remote .NET development
