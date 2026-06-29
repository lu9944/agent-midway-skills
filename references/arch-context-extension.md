---
title: Extend the Context with Typed Custom Properties
impact: MEDIUM-HIGH
impactDescription: "Type-safe custom ctx fields via declaration merging"
tags: architecture, context, ctx, declare-module, type-augmentation, interface
---

## Extend the Context with Typed Custom Properties

Midway discourages dynamically attaching properties to `ctx` because TypeScript can't type them. The correct way to add custom context fields (e.g. `ctx.userId`, `ctx.tenantId`) is via **TypeScript declaration merging** (`declare module`) so both the runtime assignment and type checking work. In a **project**, augment the Context interface in `src/interface.ts`. In a **component**, augment it in the component's `index.d.ts` — either globally (`@midwayjs/core`) or framework-specifically (`@midwayjs/koa/dist/interface`). Always `import` the module before augmenting it.

**Incorrect (dynamic untyped ctx properties — TS errors, no autocomplete):**

```typescript
@Middleware()
export class AuthMiddleware implements IMiddleware<Context, NextFunction> {
  resolve() {
    return async (ctx: Context, next: NextFunction) => {
      ctx.userId = user.id;   // ❌ TS2339: Property 'userId' does not exist on type 'Context'
      await next();
    };
  }
}

@Controller('/')
export class HomeController {
  @Inject() ctx: Context;
  @Get('/')
  async index() {
    return this.ctx.userId;   // ❌ TS2339 — no type info, no autocomplete
  }
}
```

**Correct (project-level augmentation + middleware assignment + typed access):**

```typescript
// src/interface.ts — augment the global Context interface
import '@midwayjs/core';     // ✓ must import before declare module

declare module '@midwayjs/core' {
  interface Context {
    userId: number;          // now typed everywhere ctx is used
    tenantId?: number;
    requestId: string;
  }
}

// src/middleware/auth.middleware.ts — set values in middleware
import { Middleware, IMiddleware } from '@midwayjs/core';
import { Context, NextFunction } from '@midwayjs/koa';

@Middleware()
export class AuthMiddleware implements IMiddleware<Context, NextFunction> {
  resolve() {
    return async (ctx: Context, next: NextFunction) => {
      const token = extractToken(ctx);
      if (token) {
        ctx.userId = verifyToken(token).sub;   // ✓ typed assignment
        ctx.requestId = ctx.headers['x-request-id'] || randomUUID();
      }
      await next();
    };
  }
  static getName(): string { return 'auth'; }
}

// src/controller/user.controller.ts — typed access in any handler
@Controller('/user')
export class UserController {
  @Inject() ctx: Context;

  @Get('/me')
  async me() {
    return { id: this.ctx.userId, requestId: this.ctx.requestId };  // ✓ typed read
  }
}
```

**Component-level augmentation (in the component's `index.d.ts`):**

```typescript
// index.d.ts — extend ALL frameworks' Context
declare module '@midwayjs/core' {
  interface Context {
    tenantId?: number;
  }
}

// OR extend only a specific framework's Context:
declare module '@midwayjs/koa/dist/interface' {
  interface Context { tenantId?: number; }
}
// @midwayjs/web → '@midwayjs/web/dist/interface'
// @midwayjs/faas → '@midwayjs/faas/dist/interface'
// @midwayjs/express → '@midwayjs/express/dist/interface'
```

> **Caveat:** `declare module` merges (not replaces) interface definitions, but you must `import` the target module first. In components, use the `index.d.ts` form (not `src/interface.ts`) to avoid cross-component augmentation conflicts.

Reference: [Midway Context Definition](https://midwayjs.org/docs/context_definition)
