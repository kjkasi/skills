# Chapter 6: JavaScript & TypeScript Development

## Core Idea
VS Code has built-in JavaScript/TypeScript support with IntelliSense, debugging, and testing; Node.js development is a core strength.

## Frameworks Introduced
- **jsconfig.json/tsconfig.json**: Project configuration files that enable better IntelliSense and project-wide features
- **Integrated Terminal**: Run npm/yarn commands directly in VS Code
- **Node.js Debugging**: Built-in debugging with breakpoints, step-through, and variable inspection
- **Task Automation**: `tasks.json` for build, test, and other repeatable workflows

## Key Concepts
- **IntelliSense**: Automatic completions based on TypeScript definitions, JSDoc, and node_modules
- **TypeScript**: JavaScript superset; VS Code uses it internally for better JS support
- **npm/yarn/pnpm**: Package manager integration; run scripts from package.json
- **ESLint/Prettier**: Linting and formatting configuration
- **Webpack/Vite**: Build tool integration through tasks and extensions
- **Debugging**: Attach to Node.js processes, debug browser code, React Native debugging

## Mental Models
- Use **jsconfig.json** in JS projects for project-aware IntelliSense
- Use **tsconfig.json** in TS projects for full type checking and compiler options
- Use **tasks.json** to define reusable build/test commands
- Use **launch.json** for debugging configurations

## Code Examples
```json
// jsconfig.json (JavaScript project)
{
  "compilerOptions": {
    "module": "commonjs",
    "target": "es6",
    "checkJs": true,
    "baseUrl": "./src",
    "paths": {
      "@/*": ["./*"]
    }
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}
```

```json
// .vscode/launch.json (Node.js debugging)
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Launch Node.js",
      "type": "node",
      "request": "launch",
      "program": "${workspaceFolder}/src/index.js",
      "outFiles": ["${workspaceFolder}/dist/**/*.js"]
    },
    {
      "name": "Attach to Node.js",
      "type": "node",
      "request": "attach",
      "port": 9229
    }
  ]
}
```

## Key Takeaways
1. Always include `jsconfig.json` or `tsconfig.json` for project-aware IntelliSense
2. Use integrated terminal for npm/yarn commands instead of external terminal
3. Debug console (`Ctrl+Shift+Y`) provides REPL for inspecting variables during debugging
4. ESLint + Prettier combination handles both linting and formatting
5. TypeScript strict mode catches more bugs; enable it gradually with `// @ts-check`

## Connects To
- **Ch 1**: Basic setup and extensions
- **Ch 2**: Editing features and IntelliSense
- **Ch 3**: AI assistance for code suggestions
- **Ch 11**: Remote development with containers
