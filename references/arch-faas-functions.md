---
title: Write FaaS Functions with @ServerlessTrigger
impact: CRITICAL
impactDescription: "The core entry pattern for @midwayjs/faas projects"
tags: architecture, serverless, faas, trigger, function, lambda
---

## Write FaaS Functions with @ServerlessTrigger

Midway FaaS (`@midwayjs/faas`) is the Serverless project mode. Functions are `@Provide()` classes with methods decorated by `@ServerlessTrigger(type, options)` (from `@midwayjs/core`). Each trigger binds an invocation source: `HTTP`, `API_GATEWAY`, `TIMER`, `OS` (OSS), `MQ` (MNS), `EVENT`, `KAFKA`, `HSF`, `MTOP`, `SSR`, `CDN`, or `LOG`. In v4 the old `@Func`/`@FuncHook` decorators are **gone** — use `@ServerlessTrigger` + `@ServerlessFunction`. Inject the FaaS `Context` from `@midwayjs/faas`. For Aliyun, register starter types via `import type {} from '@midwayjs/fc-starter'` in `interface.ts`.

**Incorrect (v1 @Func decorator, missing @Provide, no trigger):**

```typescript
// ❌ @Func/@FuncHook removed in v4
import { Func } from '@midwayjs/core';
@Func()
export class HelloService {
  async handler() { return 'hello'; }   // ❌ no trigger bound, never invoked
}
```

**Correct (v4 @ServerlessTrigger with typed events + Context injection):**

```typescript
// src/function/hello.ts
import {
  Provide, Inject, Query,
  ServerlessTrigger, ServerlessTriggerType,
} from '@midwayjs/core';
import { Context } from '@midwayjs/faas';
import type { TimerEvent, OSSEvent } from '@midwayjs/fc-starter';

@Provide()
export class HelloService {
  @Inject() ctx: Context;

  // HTTP trigger — auto-normalizes to a Koa-like context (ctx.query, ctx.request.body)
  @ServerlessTrigger(ServerlessTriggerType.HTTP, { path: '/', method: 'get' })
  async handleHTTP(@Query() name = 'midway') {
    return `hello ${name}`;
  }

  // Timer trigger — typed event from the starter package
  @ServerlessTrigger(ServerlessTriggerType.TIMER, { name: 'cleanupTimer' })
  async handleTimer(event: TimerEvent) {
    this.ctx.logger.info('timer fired at %s', event.triggerTime);
  }

  // OSS event trigger (file create/update)
  @ServerlessTrigger(ServerlessTriggerType.OS)
  async handleOSS(event: OSSEvent) { /* process file event */ }
}

// multiple triggers on one function → use @ServerlessFunction for the name
@ServerlessFunction({ functionName: 'abcde' })
@ServerlessTrigger(ServerlessTriggerType.EVENT)
async handleEvent(event: any) { return event; }
```

Project structure: `f.yml` declares the platform + starter (`provider.name: aliyun`, `provider.starter: '@midwayjs/fc-starter'`); function code lives under `src/function/`. **Pitfall:** Aliyun forbids mixing HTTP + non-HTTP triggers in the same function. FaaS is unsuitable for long connections (>5s), large uploads (>2MB gateway limit), or stateful workloads.

Reference: [Midway Serverless Intro](https://midwayjs.org/docs/serverless/serverless_intro), [Aliyun FC](https://midwayjs.org/docs/serverless/aliyun_faas)
