---
title: Build Fullstack Apps with Midway Hooks
impact: MEDIUM
impactDescription: "Zero-API RPC with React/Vue + type-safe backend"
tags: architecture, hooks, fullstack, react, vue, rpc, zero-api
---

## Build Fullstack Apps with Midway Hooks

Midway Hooks (`@midwayjs/hooks` + `@midwayjs/hooks-kit`) is the function-style fullstack framework: a single codebase contains frontend (React/Vue) and Node.js backend. Backend API functions are imported and called directly on the frontend — the toolkit compiles this into an HTTP RPC call (`{ args: [...] }` wire format) with shared TypeScript types. Define APIs via `Api(Get(), async () => ...)` operators; use `useContext()` for request context. Validate with zod via `Validate(...)`. Use `hooks dev`/`hooks build`/`hooks start`.

> **Note:** The docs carry a deprecation caution — existing fullstack projects may continue, but evaluate carefully for new projects.

**Incorrect (writing separate API client + fetch calls, no shared types):**

```typescript
// ❌ frontend: untyped fetch, types drift from backend
const user = await fetch('/api/user').then(r => r.json()) as any;
```

**Correct (Api operators + useContext + Validate + typed import):**

```typescript
// backend: src/api/hello.ts
import { Api, Get, Post, Validate } from '@midwayjs/hooks';
import { z } from 'zod';

// auto-route: /api/hello (functionName + fileName, prefixed /api)
export default Api(Get(), async () => 'Hello World!');

// explicit path + zod validation (positional to handler args)
export const say = Api(
  Post('/say'),
  Validate(z.string()),
  async (name: string) => `Hello ${name}!`,
);

// useContext for request data
import { useContext } from '@midwayjs/hooks';
import { Context } from '@midwayjs/koa';
export const ip = Api(Get(), async () => {
  const ctx = useContext<Context>();
  return { ip: ctx.ip };
});

// frontend: import & call directly — full type inference
import say, { ip } from './api/hello';
const greeting = await say('Midway');   // → POST /api/say {args:['Midway']}
const { ip: clientIp } = await ip();    // → GET /api/ip
```

Middleware: function-style `(next) => { const ctx = useContext(); ... await next(); }`, scoped global/file/function. Test via `getApiTrigger(api)` + `createHttpRequest(app)`.

### Filesystem routing (simple mode)

Enable file-system routing in `midway.config.ts` — any async function exported from `.ts` files under the route `baseDir` auto-becomes an API. Functions returning without `Api()` default to GET (no args) or POST (with args). `index.ts` → root; nested files → nested paths; `[name]` → path param; `[...file]` → wildcard:

```typescript
// midway.config.ts — enable file routing
import { defineConfig } from '@midwayjs/hooks';
export default defineConfig({
  source: './src/apis',
  routes: [{ baseDir: 'lambda', basePath: '/api' }],
});
// lambda/index.ts → /api/         (export default = root)
// lambda/about.ts → /api/about    (export default)
// lambda/about/contact.ts → /api/about/contact (named export)
// lambda/[name]/project.ts → /api/:name/project (path param, needs Params<T>())
// lambda/[...index].ts → /api/*   (wildcard)
```

```typescript
// lambda/[name]/project.ts — path param with filesystem routing
import { Api, Get, Params, useContext } from '@midwayjs/hooks';
export default Api(Get(), Params<{ name: string }>(), async () => {
  const ctx = useContext();
  return { name: ctx.params.name };
});
```

### CORS, upload, and other sub-features

```typescript
// CORS — pass @koa/cors as middleware (global, file, or function level)
import cors from '@koa/cors';
export const config: ApiConfig = { middleware: [cors()] };

// File upload — handler args accept FormData on POST
export default Api(Post(), async (formData: FormData) => {
  const file = formData.get('file');
  /* process file */
});

// Input validation/security — Validate(zodSchema...) positional to handler args
import { Validate } from '@midwayjs/hooks';
import { z } from 'zod';
export const safe = Api(Post(), Validate(z.string().email()), async (email: string) => {
  return { ok: true };
});
```

Reference: [Midway Hooks Intro](https://midwayjs.org/docs/hooks/intro), [Hooks API](https://midwayjs.org/docs/hooks/api), [File Routing](https://midwayjs.org/docs/hooks/file-route), [Middleware](https://midwayjs.org/docs/hooks/middleware), [CORS](https://midwayjs.org/docs/hooks/cors), [Upload](https://midwayjs.org/docs/hooks/upload), [Safe](https://midwayjs.org/docs/hooks/safe)
