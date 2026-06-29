---
title: Use the Functional API for Declarative Routes
impact: MEDIUM-HIGH
impactDescription: "Function-style routes with end-to-end type safety (zod)"
tags: architecture, functional, defineApi, zod, frontend-integration, react, vue, migration
---

## Use the Functional API for Declarative Routes

Midway's Functional API (`@midwayjs/core/functional`) is an alternative to class-based decorators for defining routes. Use `defineApi()` with a builder chain (`api.get(path).input(schema).output(schema).handle(fn)`), and `useInject`/`useConfig`/`useContext` hooks inside handlers. It **coexists** with `@Controller` — use it when co-developing a frontend in the same repo (the typed client enables "zero API" RPC calls). Entry point is `defineConfiguration()` instead of `@Configuration`.

### Core pattern: defineApi + zod + hooks + typed client

**Incorrect (hand-written fetch, no type safety, no validation):**

```typescript
// ❌ hand-written fetch with no type safety
async function getUser(id: string) {
  const res = await fetch(`/api/users/${id}`);
  return res.json();   // ❌ untyped, no validation
}
```

**Correct:**

```typescript
// src/server/index.ts — functional entry
import { defineConfiguration } from '@midwayjs/core/functional';
import * as koa from '@midwayjs/koa';
export default defineConfiguration({ imports: [koa] });

// src/server/api/user.api.ts — declarative route with validation
import { defineApi, useInject, useContext, useConfig, useLogger } from '@midwayjs/core/functional';
import { z } from 'zod';
import { UserService } from '../service/user.service';

export const userApi = defineApi('/users', (api) => ({
  getUser: api
    .get('/:id')
    .input({ params: z.object({ id: z.string().min(1) }) })
    .output(z.object({ id: z.string(), name: z.string() }))
    .handle(async ({ input }) => {
      const ctx = useContext();                         // request context
      const logger = useLogger();                        // logger
      const cfg = useConfig('app');                      // config
      const userService = await useInject(UserService);  // async IoC
      return userService.findById(input.params.id);
    }),

  createUser: api
    .post('/')
    .input({ body: z.object({ name: z.string() }) })
    .handle(async ({ input }) => {
      const svc = await useInject(UserService);
      return svc.create(input.body);
    }),
}));

// src/web/api/client.ts — typed frontend client (zero-API RPC)
import { createClient } from '@midwayjs/react';   // or '@midwayjs/vue'
import { userApi } from '../../server/api/user.api';
export const api = createClient({ user: userApi }, { basePath: '/api' });
// usage: const u = await api.user.getUser({ params: { id: 'u-1' } });  // fully typed
```

Hooks (`useInject`, `useInjectSync`, `useConfig`, `useLogger`, `useContext`, `useApp`, `useMainApp`, `useInjectClient`, `useInjectDataSource`, `usePlugin`) must be called **inside** `.handle()`. `input(...)` validates before business logic; `output(...)` validates before the response is sent.

### Frontend integration (React / Vue + Vite)

Configure the Vite bridge so the frontend imports server API contracts and the dev server runs the backend:

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';        // or '@vitejs/plugin-vue'
import { devPlugin } from '@midwayjs/mock/vite';
import { apiPlugin } from '@midwayjs/web-bridge/vite';

export default defineConfig({
  plugins: [
    devPlugin({ appDir: process.cwd(), baseDir: 'src/server', basePath: '/api' }), // runs backend in dev
    react(),                        // or vue()
    apiPlugin({ root: process.cwd(), apiDir: 'src/server/api', target: 'both' }), // type bridge
  ],
});
// Rspack alternative: createApiRspackRule({ root: process.cwd(), apiDir: 'src/server/api' })
```

React usage: `api.user.getUser({ params: { id: 'u-1' } }).then(u => setName(u.name))`. Vue: `const user = await api.user.getUser({ params: { id: 'u-1' } })` in `setup()`.

### Workspace boundaries

```
src
├── server                    # serverDir (default: src/server)
│   ├── index.ts              # defineConfiguration entry
│   └── api                   # apiDir (default: src/server/api)
│       └── user.api.ts       # API definitions = route contracts + type source
└── web                       # webDir (default: src/web)
    ├── main.tsx              # frontend entry
    └── api
        └── client.ts         # createClient
```

**Frontend CAN import:** `src/server/api/*.api.ts` exports (API definitions, types, zod schemas).
**Frontend CANNOT import:** Node-only modules (`fs`/`path`/`net`), server runtime code, handler internals.
Directories are customizable — just keep `client.ts` import paths and the plugin's `apiDir` in sync.

### Gradual migration from @Controller

`@Controller` and `defineApi` coexist in the same project — no forced split. Midway detects conflicts by `method + fullPath`. Recommended order:

1. Keep existing Service layer untouched
2. New APIs use `defineApi`
3. Migrate old controllers module-by-module
4. Add `input`/`output` schemas to critical APIs first
5. Delete old controllers once stable

### Build & deploy (unified dev, separate deploy)

```json
{
  "scripts": {
    "dev": "vite",                              // runs both via devPlugin
    "build:server": "tsc -p tsconfig.server.json",
    "build:web": "vite build",
    "build": "npm run build:server && npm run build:web"
  }
}
```

Artifacts: `dist/server` (Node runtime) + `dist/web` (static → Nginx/CDN). Deploy: `node dist/server/bootstrap.js`, serve `dist/web` statically, proxy `/api/*` to server.

### Testing (3 layers)

```typescript
// Layer 1 — contract test (fast): call API directly, cover valid/invalid/boundary input
// Layer 2 — server integration test (primary, highest ROI):
import { close, createApp, createHttpRequest } from '@midwayjs/mock';

describe('functional api', () => {
  let app;
  beforeAll(async () => { app = await createApp(); });
  afterAll(async () => { await close(app); });

  it('GET /api/users/:id', async () => {
    const res = await createHttpRequest(app).get('/api/users/u-1');
    expect(res.status).toBe(200);
    expect(res.body.id).toBe('u-1');
  });

  it('rejects invalid input (validation failure)', async () => {
    const res = await createHttpRequest(app).post('/api/users').send({ /* invalid */ });
    expect(res.status).toBe(500);   // input() schema rejects
  });
});
// Layer 3 — frontend client test (as needed): mock client returns in unit tests,
//           verify one real end-to-end chain in E2E.
```

Priority: 1) server integration tests (highest ROI), 2) key contract tests, 3) frontend interaction tests.

Reference: [Functional Intro](https://midwayjs.org/docs/functional/intro), [API Reference](https://midwayjs.org/docs/functional/api-reference), [Testing](https://midwayjs.org/docs/functional/testing), [Migration](https://midwayjs.org/docs/functional/migration), [Workspace](https://midwayjs.org/docs/functional/workspace), [React](https://midwayjs.org/docs/functional/react), [Vue](https://midwayjs.org/docs/functional/vue)
