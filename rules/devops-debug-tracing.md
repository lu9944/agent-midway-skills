---
title: Debug with Request Tracing (@midwayjs/code-dye)
impact: LOW-MEDIUM
impactDescription: "Per-method timing and call-chain tracing — dev only"
tags: devops, debug, tracing, code-dye, profiling
---

## Debug with Request Tracing (@midwayjs/code-dye)

`@midwayjs/code-dye` traces per-method execution time, call chains, args, and return values for a single request. **Gate it to dev only** via `{ component: codeDye, enabledEnvironment: ['local'] }` — it instruments every method and has a strong performance cost. Trigger tracing per-request via a query param (`?codeDyeXXX=html`) or header; report types are `html` (inline report), `json`, or `log` (no response modification).

**Incorrect (enabling in production, or always-on instrumentation):**

```typescript
// ❌ enabled in ALL environments — production perf disaster
@Configuration({ imports: [codeDye] })
```

**Correct (env-gated + opt-in per-request trigger):**

```typescript
import * as codeDye from '@midwayjs/code-dye';

// configuration.ts — dev only
@Configuration({
  imports: [
    { component: codeDye, enabledEnvironment: ['local', 'qa'] },   // never prod
  ],
})

// config.local.ts
export default {
  codeDye: {
    matchQueryKey: 'codeDyeABC',    // trigger: ?codeDyeABC=<reportType>
    matchHeaderKey: 'codeDyeHeader',
  },
} as MidwayConfig;

// Request: GET /api/users?codeDyeABC=html → HTML trace report in response
//          GET /api/users?codeDyeABC=json → JSON trace in response
//          GET /api/users?codeDyeABC=log  → trace to logs, normal response
```

Only dyed requests (carrying the trigger param) pay the instrumentation cost.

Reference: [Midway Code Dye](https://midwayjs.org/docs/extensions/code_dye)
