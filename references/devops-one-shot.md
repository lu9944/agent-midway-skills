---
title: Use @midwayjs/one-shot for IoC-Powered One-Time Scripts
impact: MEDIUM
impactDescription: "Run one-time admin scripts with full DI without a web server"
tags: devops, one-shot, scripts, cli, lifecycle
---

## Use @midwayjs/one-shot for IoC-Powered One-Time Scripts

`@midwayjs/one-shot` (v4) is a standalone framework for one-time scripts (data migration, backfill, batch import/export). It boots the IoC container so scripts get `@Inject` DI, config, and components (ORM, Redis) exactly like a real app — no HTTP server is started. Two styles: run logic directly in `onServerReady` for container-wide tasks, or implement the `OneShotRunner<T, R>` contract and execute via `framework.runScript(RunnerClass, payload)` when you need request-scoped dependencies (the framework creates a context and walks the middleware/filter chain). The component registers its own `oneShotLogger`.

**Incorrect (standalone script re-implements config/DI, or blocks in a web app):**

```typescript
// ❌ script duplicates bootstrap logic — no config, no DI, no logger
import { createConnection } from 'typeorm';
async function main() {
  const conn = await createConnection({ /* duplicated credentials */ });
  const rows = await conn.query('SELECT * FROM user');   // ❌ raw SQL, no service reuse
}
main();

// ❌ one-time work squeezed into a HTTP controller
@Post('/fix-data')
async fixData() { /* ❌ one-time migration exposed as an API endpoint */ }
```

**Correct (one-shot with DI + custom logger):**

```typescript
// src/configuration.ts — import the component and run in onServerReady
import { Configuration, Inject } from '@midwayjs/core';
import * as oneShot from '@midwayjs/one-shot';
import * as orm from '@midwayjs/typeorm';
import { UserService } from './service/user.service';

@Configuration({
  imports: [oneShot, orm],           // ORM/Redis/etc. available to scripts
})
export class MainConfiguration {
  @Inject()
  userService: UserService;          // full DI inside the script

  async onServerReady() {
    await this.userService.backfillNickname();   // one-time task, then process exits
  }
}
```

**Correct (request-scoped runner via framework.runScript):**

```typescript
// src/script/syncUser.script.ts — OneShotRunner contract (payload/ctx injected)
import { Provide } from '@midwayjs/core';
import { OneShotRunner, Context } from '@midwayjs/one-shot';

@Provide()
export class SyncUserScript implements OneShotRunner<{ tenantId: number }, { done: number }> {
  async run(payload?: { tenantId: number }, ctx?: Context) {
    // ctx provides a request-like context — request-scoped services work here
    return { done: 1 };
  }
}

// src/configuration.ts
import { Framework } from '@midwayjs/one-shot';
@Configuration({ imports: [oneShot] })
export class MainConfiguration {
  @Inject()
  framework: Framework;

  async onServerReady() {
    await this.framework.runScript(SyncUserScript, { tenantId: 42 });
  }
}

// Script services get the dedicated logger (default file: midway-one-shot.log)
// @Logger('oneShotLogger') logger: ILogger;
// config.default.ts may override: midwayLogger.clients.oneShotLogger = { fileLogName: 'sync.log', level: 'info' }
```

**When to use which:**

| Task | Choice |
|------|--------|
| Simple container-wide one-time task (config + singleton services) | `onServerReady` directly |
| Script needing request scope / middleware / filters | `framework.runScript(RunnerClass, payload)` |
| Repeated admin maintenance scripts | `@midwayjs/commander` CLI instead (interactive, argument-driven) |

Reference: [Midway One-Shot](https://midwayjs.org/docs/extensions/one-shot) (package: `@midwayjs/one-shot`)
