---
title: Use @Configuration with Explicit Component Composition
impact: CRITICAL
impactDescription: "The entry point that wires the entire application together"
tags: architecture, configuration, components, v4
---

## Use @Configuration with Explicit Component Composition

In Midway v4, the `@Configuration` class is the single entry point that wires components, loads config, and runs lifecycle hooks. Components are always imported as namespace objects (`import * as koa from '@midwayjs/koa'`) and listed in `imports`. Each component is a self-contained mini-application that registers its own framework, services, and config namespace.

**Incorrect (mismatched imports, manual wiring, missing detector):**

```typescript
// configuration.ts — v3 implicit auto-scan (BROKEN in v4)
import { Configuration } from '@midwayjs/core';
import * as koa from '@midwayjs/koa';

@Configuration({
  imports: [koa],
  // ❌ v4 REMOVED implicit scanning — nothing loads without a detector
  conflictCheck: true,           // ❌ moved to detector in v4
  detectorOptions: {},            // ❌ moved to detector in v4
})
export class MainConfiguration {}
```

**Correct (v4 explicit detector + namespace component imports + typed lifecycle):**

```typescript
// src/configuration.ts
import { Configuration, MainApp, CommonJSFileDetector } from '@midwayjs/core';
import { ILifeCycle, IMidwayContainer, IMidwayApplication } from '@midwayjs/core';
import * as koa from '@midwayjs/koa';
import * as orm from '@midwayjs/typeorm';
import * as validation from '@midwayjs/validation';
import { join } from 'path';
import DefaultConfig from './config/config.default';
import LocalConfig from './config/config.local';

@Configuration({
  imports: [
    koa,          // web server framework
    orm,          // typeorm (each is a namespace object)
    validation,   // v4 validation component
    // environment-gated component — only loaded in dev/prod (real pattern from cool-admin-midway)
    {
      component: info,                    // import * as info from '@midwayjs/info'
      enabledEnvironment: ['local', 'prod'],
    },
  ],
  // v4: explicit file detector replaces implicit auto-scan
  detector: new CommonJSFileDetector({
    ignore: ['**/logs/**'],
    conflictCheck: true,   // detect duplicate class names
  }),
  importConfigs: [
    {
      default: DefaultConfig,
      local: LocalConfig,
    },
  ],
})
export class MainConfiguration implements ILifeCycle {
  @MainApp() app: IMidwayApplication;   // v4: @MainApp() replaces the empty @App() form

  async onReady(container: IMidwayContainer, app: IMidwayApplication) {
    // container is ready; register global middleware/guards/filters here
  }

  async onStop?(container: IMidwayContainer, app: IMidwayApplication) {
    // cleanup resources
  }
}
```

**Environment-gated components (`@midwayjs/info`):** import the `@midwayjs/info` component with an `enabledEnvironment` list to expose its inspection endpoint (default path `/_info`, configurable via `info.infoPath`; shows package versions, modules, and env) only where you actually need it — e.g. `['local', 'prod']` in cool-admin-midway. Gating avoids exposing version/module internals in staging. The same object form applies to any component: `{ component: someComp, enabledEnvironment: [...] }`.
```

Reference: [Midway Component Development](https://midwayjs.org/docs/component_development), [v4 Upgrade Guide](https://midwayjs.org/docs/upgrade_v4)
