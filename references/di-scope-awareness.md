---
title: Understand Singleton, Request, and Prototype Scopes
impact: CRITICAL
impactDescription: "Wrong scope causes data races, memory leaks, and undefined injections"
tags: dependency-injection, scope, singleton, request, prototype
---

## Understand Singleton, Request, and Prototype Scopes

Midway has three scopes via `ScopeEnum` (imported from `@midwayjs/core`): `Singleton` (one instance per process), `Request` (the **default**; one instance per request chain, destroyed after), and `Prototype` (new instance every time). Controllers are always Request-scoped and cannot be changed. The critical rule: if a Singleton injects a Request-scoped dependency, that dependency gets **frozen** to a single instance — usually a bug. Use the `@Singleton()` shorthand for stateless, cross-request services (matches `cool-admin-midway`'s `Utils`, `PluginCenterService`, swagger builders).

**Incorrect (wrong scope, mutable shared state in singleton, injecting ctx into singleton):**

```typescript
// ❌ Mutable per-request state on a Singleton — shared across all concurrent requests
@Provide()
@Scope(ScopeEnum.Singleton)
export class RequestContextService {
  private userId: string;          // overwritten by every concurrent request!

  setUserId(id: string) { this.userId = id; }
  getUserId() { return this.userId; }   // returns WRONG user
}

// ❌ Injecting ctx (Request scope) into a Singleton — ctx is undefined
@Provide()
@Scope(ScopeEnum.Singleton)
export class AuditService {
  @Inject() ctx: any;              // undefined — no request in a Singleton
}
```

**Correct (appropriate scope per use case, @Singleton shorthand):**

```typescript
import { Provide, Scope, ScopeEnum, Singleton } from '@midwayjs/core';

// Stateless, cross-request service → use the @Singleton() shorthand
@Singleton()                        // equivalent to @Provide() @Scope(ScopeEnum.Singleton)
export class Utils {
  formatDate(d: Date) { return d.toISOString(); }
}

// Request-scoped service holding per-request state (the DEFAULT)
@Provide()
export class UserService {
  @Inject() ctx: any;              // ✓ valid: this is Request scope
  async getCurrentUser() { return this.ctx.user; }
}

// A Singleton that MUST depend on a Request-scoped service:
// explicitly allow downgrade so v4 doesn't throw MidwaySingletonInjectRequestError
@Provide()
@Scope(ScopeEnum.Request, { allowDowngrade: true })
export class ReportService {
  @Inject() userService: UserService;   // degrades to singleton when no request context
}
```

By default v4 **throws** `MidwaySingletonInjectRequestError` (code `MIDWAY_10010`) when a Singleton tries to inject a Request-scoped object. Use `{ allowDowngrade: true }` only when you understand the freezing behavior.

Reference: [Midway Injection Scopes](https://midwayjs.org/docs/container)
