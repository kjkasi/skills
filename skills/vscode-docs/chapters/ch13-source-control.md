# Chapter 13: Version Control

## Core Idea
VS Code has built-in Git integration with visual diff, merge conflict resolution, and branch management, plus support for other version control systems.

## Frameworks Introduced
- **Source Control Manager (SCM)**: Built-in Git integration with visual interface
- **Inline Diff**: See changes side-by-side or inline within the editor
- **Merge Editor**: Resolve merge conflicts with visual tools
- **Branch Management**: Switch, create, and manage branches from the sidebar

## Key Concepts
- **Staging**: Select specific changes to commit (partial commits)
- **Interactive Rebase**: Visual interface for squashing, reordering commits
- **Stash**: Temporarily save changes without committing
- **Push/Pull**: Synchronize with remote repositories
- **Clone**: Download repository from remote URL
- **Branches**: Parallel development lines; switch between them easily

## Mental Models
- Use **Source Control panel** (`Ctrl+Shift+G`) for all Git operations
- Use **Inline Diff** to review changes before committing
- Use **Merge Editor** for complex conflict resolution
- Use **Timeline view** to see file history across commits

## Anti-patterns
- **Committing secrets**: Always review changes before committing
- **Force pushing shared branches**: Use force-with-lease instead of force
- **Ignoring .gitignore**: Always maintain a proper .gitignore file

## Code Examples
```json
// .vscode/settings.json (Git configuration)
{
  "git.autofetch": true,
  "git.confirmSync": false,
  "git.enableSmartCommit": true,
  "git.openRepositoryInParentFolders": "always",
  "gitlens.currentLine.enabled": false
}
```

## Key Takeaways
1. Source Control panel shows all changes; stage individual lines with +/- buttons
2. Merge Editor provides visual conflict resolution
3. GitLens extension adds blame, history, and advanced Git features
4. Use keyboard shortcuts: `Ctrl+Shift+G` for Source Control, `Ctrl+Enter` to commit
5. Enable auto-fetch for timely remote updates

## Connects To
- **Ch 1**: Basic setup and source control integration
- **Ch 3**: AI can help resolve merge conflicts
- **Ch 10**: Container workflows with version control
- **Ch 15**: Enterprise source control policies
