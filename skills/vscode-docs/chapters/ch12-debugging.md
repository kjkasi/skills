# Chapter 12: Debugging & Testing

## Core Idea
VS Code provides integrated debugging and testing capabilities across multiple languages with breakpoints, watch expressions, and test explorers.

## Frameworks Introduced
- **Debug Adapter Protocol (DAP)**: Language-agnostic debugging interface
- **Launch Configurations**: Define debug scenarios in `launch.json`
- **Test Explorer**: Visual interface for running and managing tests
- **Inline Test Results**: See test status directly in the editor
- **Conditional Breakpoints**: Break only when conditions are met

## Key Concepts
- **Breakpoints**: Mark lines to pause execution; support conditions, hit counts, logpoints
- **Watch Expressions**: Monitor variable values during debugging
- **Debug Console**: REPL for evaluating expressions during debug sessions
- **Call Stack**: View and navigate the execution chain
- **Test Discovery**: Automatic detection of tests from project configuration
- **Code Coverage**: Visualize which lines are covered by tests

## Mental Models
- Use **Logpoints** instead of `console.log` — they don't require code changes
- Use **Conditional Breakpoints** for loops or when you need specific states
- Use **Test Explorer** for visual test management and coverage reports

## Anti-patterns
- **Debugging without source maps**: Always enable source maps for TypeScript/JavaScript
- **Ignoring error handling**: Add exception breakpoints for unhandled errors
- **Over-using print statements**: Use debugger for complex state inspection

## Code Examples
```json
// .vscode/launch.json (multi-language debugging)
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Python: Remote Attach",
      "type": "python",
      "request": "attach",
      "connect": { "host": "localhost", "port": 5678 },
      "pathMappings": [
        { "localRoot": "${workspaceFolder}", "remoteRoot": "/app" }
      ]
    },
    {
      "name": "Node.js: Attach",
      "type": "node",
      "request": "attach",
      "port": 9229,
      "restart": true
    },
    {
      "name": "Chrome: Attach",
      "type": "chrome",
      "request": "attach",
      "port": 9222,
      "webRoot": "${workspaceFolder}/src"
    }
  ]
}
```

## Key Takeaways
1. Use Logpoints for non-invasive logging during debugging
2. Conditional breakpoints save time in loops and complex scenarios
3. Test Explorer provides visual test management across frameworks
4. Code coverage helps identify untested code paths
5. Debug configurations can be shared via `.vscode/launch.json`

## Connects To
- **Ch 5-9**: Language-specific debugging details
- **Ch 10-11**: Debugging in containers and remotely
- **Ch 3**: AI assistance for debugging
