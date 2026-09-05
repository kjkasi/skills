# Chapter 8: Experimental Features

## Core Idea
Zoo Code's experimental features—background editing, concurrent file edits, and custom tools—extend the tool's capabilities beyond stable releases. They optimize workflow for uninterrupted coding, batch operations, and project-specific tool creation, but may be unstable or change significantly.

## Frameworks Introduced
- **Background Editing Model**: Decouples file edits from editor focus. Zoo modifies files silently in the background while you continue coding, eliminating context switches from automatic diff views.
- **Batch File Operations**: Edit multiple files in a single request with unified batch approval. Replaces sequential edit-approve cycles with a single review step.
- **Custom Tool Definition**: Define TypeScript/JavaScript tools using `defineCustomTool` with Zod schema validation. Tools are auto-transpiled with esbuild and loaded from `.roo/tools/` (project) or `~/.roo/tools/` (global).

## Key Concepts
- **Background Editing**: Experimental setting that disables automatic diff view displays. Files are modified on disk, changes appear in source control, diagnostics continue working, but no editor tabs open or diff views appear. Ideal for large refactoring and batch operations.
- **Concurrent File Edits (Multi-File Edits)**: Now enabled by default (graduated from experimental). Edit multiple files in one request via a "Batch Diff Approval" interface. Leverages `apply_diff` tool's multi-file capabilities.
- **Custom Tools**: TypeScript/JavaScript files in `.roo/tools/` (project) or `~/.roo/tools/` (global) that Zoo calls like built-in tools. Tools are **auto-approved** when enabled—security trade-off for convenience. Tools must return strings; no interactive input.
- **Custom Tool Environment Variables**: Zoo copies `.env` and `.env.*` files from tool directories to cache folders. Tools must load them manually using `dotenv` and `__dirname`—Zoo does not auto-inject into `process.env`.
- **Custom Tool npm Dependencies**: Install packages in the tool directory (`npm init -y && npm install <pkg>`). Imports resolve normally from the tool's location.
- **Experimental Feature Warning**: These features may have unexpected behavior, including potential data loss or security vulnerabilities. Enable at your own risk.
- **Custom Tools vs MCP**: MCP is for external services (search, APIs). Custom tools are for in-repo logic you control directly. MCP is more extensible; custom tools are lighter weight.

## Mental Models
- **Background Editing as Stealth Mode**: Think of it as "silent file operations"—Zoo works while you work. Trade visual confirmation for uninterrupted flow. Requires discipline with version control.
- **Batch Approval as Atomic Commit**: Concurrent file edits present all changes as a single atomic unit—review and approve everything at once, or reject all. Like a commit with full diff preview.
- **Custom Tools as Reusable Macros**: Codify repetitive workflows into named tools. Instead of re-prompting the same steps, teammates use the tool directly. Ship tool schemas with the repo.
- **Auto-Approval as Trust Boundary**: Enabling custom tools auto-approves their execution. Only enable if you trust your tool code—this is the security-convenience trade-off.
- **Tool Override Hierarchy**: Project tools (`.roo/tools/`) override global tools (`~/.roo/tools/`) when names match. This allows project-specific customizations of shared tools.

## Anti-patterns
- **Enabling background editing without version control discipline**: Without regular `git status` checks, you can lose track of silent changes. Always monitor source control.
- **Auto-approving untrusted custom tools**: Tools run without permission prompts. Only enable custom tools if you control and trust all tool code.
- **Using custom tools for external service integration**: Custom tools are for in-repo logic. For external APIs, databases, or services, use MCP servers instead.
- **Ignoring tool string-only return constraint**: Custom tools must return strings (Zoo's protocol constraint). Don't attempt to return complex objects.
- **Forgetting .env loading in custom tools**: Zoo copies `.env` files but doesn't inject variables. Tools must load them manually using `dotenv` and `__dirname`.

## Code Examples
### Custom Tool Basic Structure
```typescript
import { parametersSchema as z, defineCustomTool } from "@roo-code/types"

export default defineCustomTool({
  name: "tool_name",
  description: "What the tool does (shown to AI)",
  parameters: z.object({
    param1: z.string().describe("Parameter description"),
    param2: z.number().describe("Another parameter"),
  }),
  async execute(args, context) {
    // args are type-safe and validated
    // context provides: mode, task
    return "Result string shown to AI"
  }
})
```

### Custom Tool with npm Dependencies
```typescript
import { parametersSchema as z, defineCustomTool } from "@roo-code/types"
import axios from "axios"

export default defineCustomTool({
  name: "fetch_api",
  description: "Fetch data from an API endpoint",
  parameters: z.object({
    url: z.string().describe("API endpoint URL"),
  }),
  async execute({ url }) {
    const response = await axios.get(url)
    return JSON.stringify(response.data, null, 2)
  }
})
```

### Custom Tool with Environment Variables
```typescript
import { parametersSchema as z, defineCustomTool } from "@roo-code/types"
import dotenv from "dotenv"
import path from "path"

dotenv.config({ path: path.join(__dirname, ".env") })

export default defineCustomTool({
  name: "notify_slack",
  description: "Send a notification to Slack",
  parameters: z.object({
    message: z.string().describe("Message to send"),
  }),
  async execute({ message }) {
    const webhookUrl = process.env.SLACK_WEBHOOK_URL
    if (!webhookUrl) {
      return "Error: SLACK_WEBHOOK_URL not set in .env"
    }
    const response = await fetch(webhookUrl, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ text: message }),
    })
    return response.ok ? "Message sent" : `Failed: ${response.status}`
  }
})
```

### Tool Directory Setup with Dependencies
```bash
cd .roo/tools/
npm init -y
npm install axios lodash
```

### Environment File Structure
```
.roo/tools/
├── my-tool.ts
├── .env          # Copied to cache dir at load time
└── package.json
```

### .env File Contents
```bash
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/XXX
API_SECRET=your-secret-key
```

### Enabling Experimental Features
1. Open Zoo Code settings (gear icon)
2. Navigate to "Experimental" tab
3. Toggle "Background editing" or "Enable custom tools"

### Concurrent File Edits Workflow
1. Ask Zoo to "Update all API endpoints to use the new authentication method"
2. Zoo identifies all affected files
3. Batch approval shows changes across multiple files
4. Review all changes in unified diff view
5. Approve to apply all simultaneously

## Worked Example
A team enables background editing and custom tools. The lead developer creates a `deploy_check` custom tool in `.roo/tools/deploy_check.ts` that verifies deployment prerequisites (checks for uncommitted changes, runs tests, validates config). Teammates install dependencies via `npm install` in the tool directory. When any team member runs the tool, Zoo auto-approves it (since custom tools are auto-approved). The team also enables background editing for large refactoring tasks—Zoo silently updates 20+ files while the developer continues working, then they review all changes in a single batch approval. Both features trade visual confirmation for speed, so the team establishes a discipline of checking `git status` regularly.

## Key Takeaways
1. Background editing eliminates context switches but requires version control discipline—check `git status` regularly.
2. Concurrent file edits are now enabled by default—review all proposed changes in the batch approval before confirming.
3. Custom tools are auto-approved when enabled—only enable if you trust your tool code (security-convenience trade-off).
4. Custom tools must return strings and cannot prompt users interactively—they're one-shot execution functions.
5. Environment variables require manual loading via `dotenv` and `__dirname`—Zoo copies `.env` files but doesn't inject them.
6. Custom tools are for in-repo logic; MCP is for external services. Choose based on the integration type.
7. Tool loading: project tools override global tools of the same name; use "Refresh Custom Tools" command to pick up changes.
8. All experimental features may be unstable—enable at your own risk and report issues on GitHub.

## Connects To
- **Ch 5: Modes System**: Background editing and concurrent edits work across all modes with edit access; custom tools appear in the tool list.
- **Ch 6: Core Features**: Custom tools integrate with `.rooignore` restrictions; concurrent file edits build on the core `apply_diff` tool.
- **Ch 7: MCP Integration**: Custom tools are lighter-weight alternatives to MCP for in-repo logic; MCP is more extensible for external services.
