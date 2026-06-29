---
title: Use @Autoload for Self-Initializing Classes
impact: MEDIUM
impactDescription: "Keeps onReady lean by auto-initializing independent background logic"
tags: architecture, autoload, init, lifecycle, startup
---

## Use @Autoload for Self-Initializing Classes

When a class contains independent background logic (event listeners, data sync, health monitors) that isn't part of the main request flow, use `@Autoload()` instead of manually resolving it in `onReady`. The framework auto-creates the instance and runs its `@Init()` method at startup — no `container.getAsync(Class)` needed in `@Configuration`. Always pair `@Autoload` with `@Scope(ScopeEnum.Singleton)` and clean up resources in `@Destroy()`. This keeps `onReady` focused on main-flow wiring.

**Incorrect (cluttering onReady with unrelated initialization):**

```typescript
@Configuration({})
export class MainConfiguration {
  async onReady(container) {
    await container.getAsync(RedisErrorListener);   // ❌ unrelated listeners bloating onReady
    await container.getAsync(DataSyncListener);
    await container.getAsync(MetricsCollector);
    await container.getAsync(HealthChecker);
  }
}
```

**Correct (@Autoload self-initializes + @Init + @Destroy cleanup):**

```typescript
import { Autoload, Scope, ScopeEnum, Init, Destroy } from '@midwayjs/core';

@Autoload()
@Scope(ScopeEnum.Singleton)
export class RedisErrorListener {
  private redis: Redis;

  @Init()
  async init() {
    this.redis = new Redis();
    this.redis.on('error', (err) => this.handleError(err));
  }

  @Destroy()
  async destroy() {
    this.redis.disconnect();   // ✓ clean up on shutdown
  }

  private handleError(err: Error) { /* ... */ }
}

// no need to touch onReady — the class starts itself
@Configuration({})
export class MainConfiguration {
  async onReady(container) {
    // only main-flow wiring here (middleware, filters, DB connect)
  }
}
```

`@Autoload` is how many framework components (like RabbitMQ producers) self-register without manual instantiation. The class is scanned, instantiated as Singleton, and `@Init()` runs before `onReady`.

Reference: [Midway Auto Run (@Autoload)](https://midwayjs.org/docs/auto_run)
