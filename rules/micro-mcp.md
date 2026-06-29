---
title: Build MCP Servers with @midwayjs/mcp
impact: MEDIUM
impactDescription: "Expose Tools/Prompts/Resources to AI clients"
tags: microservices, mcp, ai, tool, llm
---

## Build MCP Servers with @midwayjs/mcp

`@midwayjs/mcp` implements the Model Context Protocol for exposing Tools, Prompts, and Resources to AI clients (Cursor, Claude Desktop, etc.). Choose a transport: `stdio` (CLI, no HTTP needed), `stream-http` (recommended for web), or `sse` (deprecated). For HTTP transports you **must** also import a web framework (`@midwayjs/koa`/`express`). Define capabilities via `@Tool`/`@Prompt`/`@Resource` classes with `zod` schemas. Return `{ content, isError: true }` instead of throwing to surface errors to the AI client.

**Incorrect (incorrect transport config, throwing instead of structured error):**

```typescript
// ❌ stream-http without a web framework — nothing serves the endpoint
@Configuration({ imports: [mcp] })

@Tool('fetch', { inputSchema: { url: z.string() } })
export class FetchTool implements IMcpTool {
  async execute(args) {
    throw new Error('failed');   // ❌ crashes; AI client gets no structured error
  }
}
```

**Correct (HTTP transport + zod schemas + structured results + JWT auth):**

```typescript
import * as mcp from '@midwayjs/mcp';
import * as koa from '@midwayjs/koa';
@Configuration({ imports: [koa, mcp] })

// config.default.ts
export default {
  koa: { port: 3000 },
  mcp: {
    serverInfo: { name: 'my-mcp-server', version: '1.0.0' },
    transportType: 'stream-http',        // recommended; endpoints default to /mcp
    enableJwtAuthHelper: true,           // built-in JWT auth → ctx.authInfo
  },
} as MidwayConfig;

// Tool — zod input schema, CallToolResult output
import { Tool, IMcpTool, ToolConfig } from '@midwayjs/mcp';
import { CallToolResult } from '@modelcontextprotocol/sdk/types.js';
import { z } from 'zod';

const cfg: ToolConfig<{ city: z.ZodString }> = {
  description: 'Get weather for a city',
  inputSchema: { city: z.string().describe('City name') },
};
@Tool('get_weather', cfg)
export class WeatherTool implements IMcpTool {
  async execute(args: { city: string }): Promise<CallToolResult> {
    try {
      const temp = await this.weatherApi(args.city);
      return { content: [{ type: 'text', text: `${args.city}: ${temp}°C` }] };
    } catch (err) {
      return { content: [{ type: 'text', text: err.message }], isError: true };  // ✓ structured error
    }
  }
}

// Prompt — returns messages; Resource — uri template + mimeType
// @Prompt('greet', { argsSchema: { name: z.string() } }) → GetPromptResult
// @Resource('db://users/{id}', { uri: '...', mimeType: 'application/json' })

// dynamic registration at runtime
@Inject() mcpFramework: mcp.MidwayMCPFramework;
async onReady() {
  const server = this.mcpFramework.getServer();
  server.registerTool('dynamic_tool', { /* config */ }, async (args) => ({ /* result */ }));
}
```

One transport type per app instance. `ctx.authInfo` holds JWT-derived fields (`clientId`, `scopes`, `expiresAt`) when the JWT helper is enabled.

Reference: [Midway MCP](https://midwayjs.org/docs/extensions/mcp)
