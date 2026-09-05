# Chapter 7: MCP Integration

## Core Idea
The Model Context Protocol (MCP) is a standardized communication protocol that enables AI systems to interact with external tools and services via a universal client-server architecture. It operates at a higher abstraction layer than REST APIs, providing stateful, contextual interactions with runtime tool discovery.

## Frameworks Introduced
- **MCP Client-Server Architecture**: The AI assistant (client) connects to MCP servers that provide specific capabilities (file access, database queries, API integrations). Communication occurs via JSON-RPC 2.0 messages.
- **Transport Layer Abstraction**: Three transport mechanisms—STDIO (local), Streamable HTTP (modern remote), and SSE (legacy remote)—each with distinct deployment and performance characteristics.
- **MCP vs REST Layer Model**: REST is a low-level web communication pattern; MCP is a high-level AI protocol that orchestrates tool usage and maintains context. MCP often uses REST internally but abstracts it away.

## Key Concepts
- **Model Context Protocol (MCP)**: A standardized protocol for LLM systems to interact with external tools. Like a USB-C port—any compatible LLM connects to any MCP server for functionality.
- **Stateful Connections**: MCP maintains context across interactions (unlike REST's stateless requests). One conversation context persists across multiple tool uses.
- **Runtime Tool Discovery**: AI discovers available tools at runtime with metadata (name, description, parameters). New tools can be added without redeploying or modifying the AI.
- **STDIO Transport**: Local, process-based communication via stdin/stdout. Low latency, no network configuration, one-to-one client-server relationship. Ideal for local tools and security-sensitive operations.
- **Streamable HTTP Transport**: Modern standard for remote MCP servers. Single HTTP endpoint supporting POST and GET. Supports multiple clients, standard HTTP auth, streaming via SSE over the same connection.
- **SSE Transport (Legacy)**: Older remote transport using two channels—GET for server-to-client events, POST for client-to-server requests. Being replaced by Streamable HTTP.
- **Context7**: Recommended first-choice MCP server. One-command install, cross-platform, rich toolset (database access, web search, text utilities), MIT licensed.
- **MCP Configuration**: Global (available in every workspace) or project-level (`.roo/mcp.json`, checked into version control). Project config wins when both define the same server name.

## Mental Models
- **MCP as Middleware Layer**: Think of MCP as middleware between the AI and external services. REST APIs provide discrete services; MCP orchestrates those services for AI agents with context preservation.
- **USB-C Analogy**: Any compatible LLM can connect to any MCP server through the standardized protocol, eliminating custom integrations for each tool/service pair.
- **Transport Selection Matrix**: STDIO for local/secure/single-client. Streamable HTTP for remote/multi-client/modern. SSE for legacy compatibility. Match transport to deployment needs.
- **Tool Discovery as Plugin Architecture**: MCP servers are like plugins—register them once, and the AI discovers and uses their tools at runtime without code changes.
- **Hybrid Deployment**: STDIO servers can proxy to remote services; remote servers can trigger local operations via callbacks. Gateway patterns combine both.

## Anti-patterns
- **Comparing MCP to REST as competitors**: They operate at different layers. MCP builds upon REST; it doesn't replace it. REST excels at discrete services; MCP excels at orchestrating them for AI.
- **Using SSE for new remote servers**: Streamable HTTP is the modern standard. SSE is legacy—only use for compatibility with older servers.
- **Ignoring transport security**: STDIO is inherently secure (no network exposure). Remote transports (Streamable HTTP, SSE) require explicit security measures—authentication, authorization, network security.
- **Not verifying MCP installations**: Always verify server is running in the MCP settings panel and approve tool invocations on first use.

## Code Examples
### Context7 Global Configuration
```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"]
    }
  }
}
```

### Context7 Windows (cmd.exe) Variant
```json
{
  "mcpServers": {
    "context7": {
      "type": "stdio",
      "command": "cmd",
      "args": ["/c", "npx", "-y", "@upstash/context7-mcp@latest"]
    }
  }
}
```

### Project-Level MCP Configuration
Create `.roo/mcp.json` at project root:
```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"]
    }
  }
}
```

### Streamable HTTP Configuration
```json
{
  "mcpServers": {
    "StreamableHTTPMCPName": {
      "type": "streamable-http",
      "url": "http://localhost:8080/mcp"
    }
  }
}
```

### STDIO Server Implementation (TypeScript)
```typescript
import { Server } from '@modelcontextprotocol/sdk/server/index.js';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';

const server = new Server({name: 'local-server', version: '1.0.0'});
// Register tools...
const transport = new StdioServerTransport(server);
transport.listen();
```

### SSE Server Implementation (TypeScript)
```typescript
import { Server } from '@modelcontextprotocol/sdk/server/index.js';
import { SSEServerTransport } from '@modelcontextprotocol/sdk/server/sse.js';
import express from 'express';

const app = express();
const server = new Server({name: 'remote-server', version: '1.0.0'});
// Register tools...
const transport = new SSEServerTransport(server);
app.use('/mcp', transport.requestHandler());
app.listen(3000, () => {
  console.log('MCP server listening on port 3000');
});
```

### Tool Discovery Response
```json
{
  "tools": [
    {
      "name": "readFile",
      "description": "Reads content from a file",
      "parameters": {
        "path": { "type": "string", "description": "File path" }
      }
    }
  ]
}
```

### Transport Comparison
| Feature | STDIO | Streamable HTTP | SSE (Legacy) |
|---------|-------|-----------------|--------------|
| Location | Local only | Local or remote | Local or remote |
| Clients | Single | Multiple | Multiple |
| Performance | Lower latency | Higher latency | Higher latency |
| Security | Inherently secure | Requires measures | Requires measures |
| Recommendation | Local tools | Modern standard | Legacy only |

## Worked Example
A developer needs to add database query and Slack notification capabilities to their workflow. They install Context7 via global MCP settings (one JSON block). Zoo Code discovers Context7's tools at runtime. The developer creates a project-level `.roo/mcp.json` for a custom internal API server deployed via Streamable HTTP. Both configurations coexist—project config wins for name conflicts. The developer's workflow: ask Zoo to query the database (Context7 STDIO), then post results to Slack (custom Streamable HTTP server). MCP maintains context across both tool uses, and new tools can be added to either server without modifying Zoo Code.

## Key Takeaways
1. MCP is a high-level AI protocol that orchestrates external tools with context preservation—it builds on REST, not replaces it.
2. Three transports: STDIO (local, low-latency, secure), Streamable HTTP (modern remote, multi-client), SSE (legacy remote).
3. Runtime tool discovery means new capabilities can be added without redeploying the AI.
4. Context7 is the recommended first-choice MCP server—install with one command, cross-platform, rich toolset.
5. Configuration can be global (every workspace) or project-level (`.roo/mcp.json`); project wins on conflicts.
6. Always verify MCP server installation and approve first tool invocations.

## Connects To
- **Ch 5: Modes System**: The `mcp` tool group controls access to MCP server capabilities across modes.
- **Ch 6: Core Features**: MCP tools integrate with diagnostics and custom instructions; `.rooignore` affects MCP file operations.
- **Ch 8: Experimental Features**: Custom tools are lighter-weight alternatives to MCP for in-repo logic; MCP is for external services.
