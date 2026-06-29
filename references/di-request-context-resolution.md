---
title: Resolve Request-Scoped Services in Singleton Middleware
impact: HIGH
impactDescription: "Middleware/Aspect are always Singleton — direct @Inject of Request scope fails"
tags: dependency-injection, middleware, requestContext, singleton
---

## Resolve Request-Scoped Services in Singleton Middleware

Middleware classes (`@Middleware()`) and AOP aspects (`@Aspect()`) are **always Singleton scope**. You cannot `@Inject()` a Request-scoped service into them — the injected instance has no request context (`ctx` is undefined). To call Request-scoped logic from a Singleton middleware, resolve the service per-request via `ctx.requestContext.getAsync(Service)` inside the `resolve()` handler.

**Incorrect (injecting Request-scoped service into Singleton middleware):**

```typescript
@Middleware()
export class AuthMiddleware implements IMiddleware<Context, NextFunction> {
  @Inject() userService: UserService;   // ❌ not bound to the current request/ctx
  resolve() {
    return async (ctx: Context, next: NextFunction) => {
      // ❌ this.userService has no ctx — behaves like a stale/frozen instance
      const user = await this.userService.getCurrentUser();
      ctx.user = user;
      await next();
    };
  }
}
```

**Correct (resolve per-request via requestContext.getAsync):**

```typescript
import { Middleware, IMiddleware } from '@midwayjs/core';
import { Context, NextFunction } from '@midwayjs/koa';

@Middleware()
export class AuthMiddleware implements IMiddleware<Context, NextFunction> {
  resolve() {
    return async (ctx: Context, next: NextFunction) => {
      // ✓ resolve a fresh Request-scoped instance bound to THIS ctx
      const userService = await ctx.requestContext.getAsync<UserService>(UserService);
      ctx.user = await userService.getCurrentUser();
      await next();
    };
  }
  static getName(): string { return 'auth'; }
}
```

The same applies to AOP aspects: inside `before`/`around`, access the request context via `joinPoint.target[REQUEST_OBJ_CTX_KEY]` (imported from `@midwayjs/core`) rather than injected `ctx`.

Reference: [Midway Middleware](https://midwayjs.org/docs/middleware)
