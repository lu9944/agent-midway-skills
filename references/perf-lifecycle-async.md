---
title: Use Async Lifecycle Hooks Correctly
impact: HIGH
impactDescription: "Misuse blocks startup or causes race conditions"
tags: performance, lifecycle, onReady, onConfigLoad, init
---

## Use Async Lifecycle Hooks Correctly

Midway lifecycle hooks (`onConfigLoad`, `onReady`, `onServerReady`, `onStop`, `onHealthCheck`) are async and awaited by the framework. Use the right hook for the right job: `onConfigLoad` to merge remote config, `onReady` for container setup (DB connect, register middleware), `onServerReady` to access the running server/framework, `onStop` for cleanup. v4 adds built-in timeouts (`core.readyTimeout` etc.) and an `AbortController` to prevent startup hangs. Never do heavy synchronous work in a constructor — move it to `@Init` or `onReady`.

**Incorrect (fire-and-forget async, blocking constructor, wrong hook):**

```typescript
@Provide()
export class DatabaseService {
  constructor() {
    // ❌ BLOCKS module instantiation synchronously
    this.config = fs.readFileSync('config.json');
  }
}

@Configuration({})
export class MainConfiguration {
  async onReady() {
    connectDatabase();   // ❌ not awaited — app starts before DB is ready
  }
}
```

**Correct (awaited hooks, async @Init, v4 AbortController awareness):**

```typescript
@Provide()
export class DatabaseService {
  @Config('mysql') mysqlConfig;
  private pool: any;

  @Init()
  async init() {                       // ✓ async, awaited by container
    this.pool = await mysql.createPool(this.mysqlConfig);
  }

  @Destroy()
  async destroy() { await this.pool.end(); }
}

@Configuration({})
export class MainConfiguration implements ILifeCycle {
  // onConfigLoad's RETURN VALUE is merged into global config
  async onConfigLoad(container: IMidwayContainer) {
    const remote = await fetchRemoteConfig();
    return { mysql: remote.mysql };
  }

  // onReady: container ready, register middleware/filters
  async onReady(container: IMidwayContainer, app: IMidwayApplication) {
    app.useMiddleware(JwtMiddleware);
    app.useFilter([DefaultErrorFilter]);
  }

  // onServerReady: server is listening — safe to get framework/server
  async onServerReady(container: IMidwayContainer) {
    const framework = await container.getAsync(koa.Framework);
    const server = framework.getServer();
  }

  // onStop: cleanup (v4 timeout via core.stopTimeout, unlimited by default)
  async onStop(container: IMidwayContainer) {
    await closeAllConnections();
  }

  // onHealthCheck: v4 timeout 1s (core.healthCheckTimeout)
  async onHealthCheck(container: IMidwayContainer): Promise<HealthResult> {
    return { status: true, reason: 'ok' };
  }
}
```

Reference: [Midway Lifecycle](https://midwayjs.org/docs/lifecycle)
