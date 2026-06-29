---
title: Declare the v4 File Detector Explicitly
impact: CRITICAL
impactDescription: "v4 removed implicit auto-scan — code silently fails to load without it"
tags: architecture, detector, scanning, v4
---

## Declare the v4 File Detector Explicitly

The single most common v4 migration breakage: the framework **removed implicit automatic scanning**. Without an explicit `detector`, decorated classes are never registered with the IoC container and injection silently fails. Always declare a `CommonJSFileDetector` (CommonJS) or `ESModuleFileDetector` (ESM) in `@Configuration`. The `conflictCheck` and `detectorOptions` that previously lived on `@Configuration` are now properties of the detector.

**Incorrect (relying on removed implicit scanning):**

```typescript
import { Configuration } from '@midwayjs/core';

@Configuration({
  imports: [koa],
  // ❌ No detector — NOTHING is scanned in v4. @Provide classes never register.
  conflictCheck: true,   // ❌ ignored here in v4
})
export class MainConfiguration {}
```

**Correct (explicit detector with conflict checking):**

```typescript
import { Configuration, CommonJSFileDetector } from '@midwayjs/core';

@Configuration({
  imports: [koa],
  detector: new CommonJSFileDetector({
    // optional: pattern (default '**/*.{ts,tsx,js,mts,mjs,cts,cjs}')
    // optional: ignore (default logs, run, public, node_modules, *.test.*, __test__, *.d.ts)
    ignore: [
      '**/logs/**',
      '**/run/**',
    ],
    conflictCheck: true,   // throws on duplicate class names across the codebase
  }),
})
export class MainConfiguration {}

// ESM projects use the ESM detector instead:
// import { ESModuleFileDetector } from '@midwayjs/core';
// detector: new ESModuleFileDetector()
```

Reference: [v4 Upgrade — Detector](https://midwayjs.org/docs/upgrade_v4)
