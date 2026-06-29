---
title: Offload CPU Work to Thread Pools (@midwayjs/piscina)
impact: MEDIUM
impactDescription: "Keep the event loop free for heavy CPU tasks"
tags: performance, piscina, thread-pool, worker, cpu
---

## Offload CPU Work to Thread Pools (@midwayjs/piscina)

`@midwayjs/piscina` offloads CPU-intensive work to worker threads via the `piscina` pool. Two modes: **plain worker** (`run({ handler, payload })`) for low-overhead function execution, and **container mode** (`runInContainer('taskName', payload)`) where the worker runs a separate Midway app with DI. Configure pool size via `piscina.client`; the main app **must ignore the worker directory** in its detector to avoid double-loading. Data is serialized (structured clone) — use `transferList` for large buffers.

**Incorrect (CPU-heavy work on the event loop):**

```typescript
@Get('/hash')
async hash(@Query('input') input: string) {
  // ❌ blocks the event loop for all requests while hashing
  return heavyHash(input);
}
```

**Correct (offload to a thread pool with cancellation):**

```typescript
import * as piscina from '@midwayjs/piscina';
@Configuration({
  imports: [piscina],
  detector: new CommonJSFileDetector({ ignore: ['**/worker/**'] }),  // ✓ ignore worker dir
})

// config.default.ts
export default {
  piscina: {
    client: {
      workerFile: join(__dirname, '../worker/index'),   // no extension; searches .js→.ts
      minThreads: 1, maxThreads: 4, idleTimeout: 60000,
    },
    // OR multi-pool: clients: { compute: {...}, image: {...} }
  },
} as MidwayConfig;

// worker/task (container mode) — src/worker/index.ts uses functional API
import { defineConfiguration } from '@midwayjs/core/functional';
export default defineConfiguration({ namespace: 'worker', detector: new CommonJSFileDetector(), imports: [piscina] });

// src/worker/calculate.task.ts
import { PiscinaTask, IPiscinaTask } from '@midwayjs/piscina';
@PiscinaTask('calculate')
export class CalculateTask implements IPiscinaTask {
  async execute(payload: { a: number; b: number }) { return payload.a + payload.b; }
}

// main app — run with cancellation
import { PiscinaService } from '@midwayjs/piscina';
@Provide()
export class ComputeService {
  @Inject() piscinaService: PiscinaService;

  async compute() {
    return this.piscinaService.runInContainer('calculate', { a: 5, b: 6 });
  }
  async cancelable() {
    const ac = new AbortController();
    setTimeout(() => ac.abort(), 3000);
    return this.piscinaService.run({ handler: 'longTask', payload: { ms: 10000 } }, { signal: ac.signal });
  }
}
```

No functions/class instances across the thread boundary (structured clone only). Use `transferList` for large `ArrayBuffer`/`Buffer`.

Reference: [Midway Piscina](https://midwayjs.org/docs/extensions/piscina)
