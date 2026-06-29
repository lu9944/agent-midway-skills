---
title: Enable the Async Context Manager for Request Context
impact: MEDIUM-HIGH
impactDescription: "Lets Singletons access request-scoped data without scope downgrade"
tags: performance, async-context, als, request-context
---

## Enable the Async Context Manager for Request Context

A common need: a Singleton service must know the current request/user without being Request-scoped (which causes scope-downgrade freezes). v4 enables `async_local_storage` by default (merged into core). Enable the async context manager in config so any code can read the current context via `AsyncContextManager` / `ASYNC_CONTEXT_KEY`, decoupling request data from DI scope. `cool-admin-midway` requires `asyncContextManager.enable: true` for its multi-tenant system.

**Incorrect (storing request state on a Singleton, or scope-downgrade everywhere):**

```typescript
// ❌ shared mutable state across concurrent requests
@Singleton()
export class TenantService {
  private currentTenantId: number;   // clobbered by every request!
  setTenant(id: number) { this.currentTenantId = id; }
}

// ❌ forcing every consumer into Request scope just to read ctx
@Scope(ScopeEnum.Request)
export class AuditService {
  @Inject() ctx: Context;
  log(msg: string) { console.log(this.ctx.tenantId, msg); }
}
```

**Correct (async context manager — read request data from anywhere):**

```typescript
// config.default.ts
export default {
  asyncContextManager: { enable: true },   // REQUIRED for ALS-based ctx
} as MidwayConfig;

// a Singleton can read the current request's tenant via the async context
import { Provide, Singleton } from '@midwayjs/core';
import { getCurrentAsyncContextManager } from '@midwayjs/core';

// define a symbol key for your context value
const TENANT_ID_KEY = Symbol('tenantId');

@Singleton()
export class TenantService {
  getTenantId(): number | undefined {
    // AsyncContextManager is an interface; use getCurrentAsyncContextManager() to get the instance
    const manager = getCurrentAsyncContextManager();
    if (!manager?.active()) return undefined;          // no active context (e.g. outside request)
    return manager.active().getValue(TENANT_ID_KEY);   // read by symbol key
  }
}

// set values early in a middleware (runs within the request chain)
@Middleware()
export class TenantMiddleware implements IMiddleware<Context, NextFunction> {
  resolve() {
    return async (ctx: Context, next: NextFunction) => {
      const manager = getCurrentAsyncContextManager();
      manager?.active()?.setValue(TENANT_ID_KEY, extractTenant(ctx));
      await next();
    };
  }
}
```

Reference: [Midway Context Definition](https://midwayjs.org/docs/context_definition)
