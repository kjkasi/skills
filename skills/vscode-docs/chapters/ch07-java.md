# Chapter 7: Java Development

## Core Idea
VS Code provides comprehensive Java support through the Extension Pack for Java, including compilation, debugging, testing, and Spring Boot integration.

## Frameworks Introduced
- **Extension Pack for Java**: Collection of extensions for Java development (IntelliSense, debugging, testing, Maven/Gradle)
- **Language Server Protocol (LSP)**: Java language features powered by Eclipse JDT Language Server
- **Spring Boot Integration**: Support for Spring Boot applications with dashboard and code completion

## Key Concepts
- **Java Extension Pack**: Microsoft's bundled extensions for Java development
- **Build Tools**: Maven and Gradle integration for dependency management and build automation
- **Debugging**: Breakpoints, step-through, variable inspection, hot code replacement
- **Testing**: JUnit and TestNG support with test explorer integration
- **Code Navigation**: Go to definition, find references, call hierarchy
- **Refactoring**: Rename, extract method, inline, and other Java-specific refactorings

## Mental Models
- Use **Extension Pack for Java** for one-click setup of all Java tools
- Use **Maven/Gradle integration** for dependency management and build tasks
- Use **Spring Boot Dashboard** to run and debug Spring applications

## Code Examples
```json
// .vscode/settings.json (Java project)
{
  "java.configuration.updateBuildConfiguration": "automatic",
  "java.compile.nullAnalysis.mode": "automatic",
  "java.debug.settings.hotCodeReplace": "auto",
  "java.debug.settings.enableRunDebugCodeLens": true
}
```

## Key Takeaways
1. Install the Extension Pack for Java for complete Java support
2. Maven/Gradle projects auto-configure; manually add via "Java: Clean Java Language Server Workspace"
3. Hot code replacement lets you change code while debugging without restart
4. Code Lens shows references and test counts above methods
5. Use Java projects view for project-level operations

## Connects To
- **Ch 1**: Setting up Java development environment
- **Ch 2**: Code editing and refactoring
- **Ch 12**: Debugging and testing workflows
- **Ch 11**: Remote Java development
