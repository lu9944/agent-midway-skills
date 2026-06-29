---
title: Never Access Injected Dependencies in the Constructor
impact: CRITICAL
impactDescription: "The #1 cause of undefined property bugs in Midway"
tags: dependency-injection, init, constructor, lifecycle
---

## Never Access Injected Dependencies in the Constructor

Injected properties (`@Inject`, `@Config`, `@Logger`) are populated by the container **after** construction. Reading them inside the `constructor` returns `undefined`. Perform all initialization that depends on injected values in an `@Init()` method instead — it is always invoked asynchronously after injection is complete. This is the most common mistake for developers coming from constructor-injection frameworks.

**Incorrect (reading injected deps in constructor):**

```typescript
@Provide()
export class DatabaseService {
  @Config('mysql.host')
  host: string;

  constructor() {
    // ❌ this.host is undefined here — injected AFTER construction
    this.pool = mysql.createPool({ host: this.host });
  }
}
```

**Correct (use @Init for async post-injection setup):**

```typescript
import { Provide, Init, Config, Destroy } from '@midwayjs/core';

@Provide()
export class DatabaseService {
  @Config('mysql.host')
  host: string;

  private pool: any;

  @Init()
  async init() {
    // ✓ this.host is now populated; @Init runs after injection
    this.pool = mysql.createPool({ host: this.host });
  }

  @Destroy()
  async destroy() {
    // cleanup when the instance is disposed
    await this.pool.end();
  }

  async query(sql: string) {
    return this.pool.query(sql);
  }
}
```

The same rule applies to `@Config(...)`, `@Logger(...)`, and `@App()` — none are available in the constructor. Use `@Init()` (only one per class, always async) and pair cleanup with `@Destroy()`.

Reference: [Midway Lifecycle](https://midwayjs.org/docs/lifecycle)
