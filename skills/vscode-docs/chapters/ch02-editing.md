# Chapter 2: Code Editing & Customization

## Core Idea
VS Code provides intelligent code editing features including IntelliSense, snippets, refactoring, and extensive customization options.

## Frameworks Introduced
- **IntelliSense**: Context-aware code completions, parameter hints, quick info, and member lists
- **Multi-cursor Editing**: Edit multiple locations simultaneously with `Alt+Click` or `Ctrl+Alt+Down/Up`
- **Zen Mode**: Distraction-free editing with full screen, no sidebar, status bar, or activity bar
- **Focus Mode**: Even more minimal than Zen Mode — only the editor is visible

## Key Concepts
- **IntelliSense**: Powered by language servers; provides completions, signatures, hover info
- **Snippets**: Predefined or custom code templates triggered by prefix + Tab
- **Emmet**: Built-in HTML/CSS abbreviation expansion (e.g., `div.container>ul>li*3` → full HTML)
- **Code Actions**: Quick fixes and refactoring suggestions (lightbulb icon)
- **Foldings**: Collapse code blocks for better overview (click arrows or `Ctrl+Shift+[`)
- **Minimap**:缩略图overview of entire file on the right side

## Mental Models
- Use **Snippets** for repetitive code patterns — create custom ones in `snippets/` folder
- Use **Multi-cursor** for simultaneous edits across multiple locations
- Use **Zen Mode** for focused coding sessions without distractions

## Code Examples
```javascript
// Custom snippet example (Python)
{
  "Print with Debug": {
    "prefix": "pdb",
    "body": [
      "print(f\"DEBUG: {$1} = {{$1}}\")",
      "$2"
    ],
    "description": "Print debug statement"
  }
}
```

## Key Takeaways
1. IntelliSense adapts to each language — install language extensions for best results
2. Create custom snippets for your team's common patterns
3. Use Zen Mode for focused work; Quick Switch Scheme (`Ctrl+K Ctrl+T`) for fast theme changes
4. Code Actions (lightbulb) provide context-aware fixes and refactorings
5. Emmet abbreviations work in HTML, CSS, and many template languages

## Connects To
- **Ch 1**: Getting started and basic navigation
- **Ch 3**: AI features enhance editing with suggestions
- **Ch 5-9**: Language-specific editing features
