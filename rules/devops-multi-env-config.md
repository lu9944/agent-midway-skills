---
title: Use Multi-Environment Configuration with Validation
impact: HIGH
impactDescription: "Proper configuration prevents deployment failures"
tags: devops, configuration, environment, config, type-safety
---

## Use Multi-Environment Configuration with Validation

Midway loads config by environment: `config.default.ts` (all environments) + `config.{env}.ts` (env-specific), merged via `extend2` (arrays are overwritten, not merged). Map configs explicitly by environment key in `importConfigs`. Type every config file as `MidwayConfig` (and component-specific via declaration merging). Inject values with `@Config('dotted.path')` or `@AllConfig()` (v4 replaces `@Config(ALL)`). Never read `@Config` in a constructor — use `@Init`. For runtime/remote config, return it from `onConfigLoad`.

**Incorrect (raw process.env scattered, untyped config, accessing config in constructor):**

```typescript
@Provide()
export class DatabaseService {
  constructor() {
    // ❌ process.env scattered, NaN on missing, untyped
    this.pool = mysql.createPool({
      host: process.env.DB_HOST,
      port: parseInt(process.env.DB_PORT),    // NaN if missing
    });
  }
}

// ❌ untyped config object
export default { koa: { port: 7001 } };
```

**Correct (typed MidwayConfig + explicit env mapping + @Config injection):**

```typescript
// config/config.default.ts — loaded in ALL environments
import { MidwayConfig } from '@midwayjs/core';
export default {
  keys: process.env.APP_KEYS,
  koa: { port: 7001, globalPrefix: '/api' },
  typeorm: { dataSource: { default: {
    type: 'mysql', host: process.env.DB_HOST, port: Number(process.env.DB_PORT),
  } } },
} as MidwayConfig;

// config/config.local.ts — dev only
import { MidwayConfig } from '@midwayjs/core';
export default {
  typeorm: { dataSource: { default: { synchronize: true } } },  // dev-only sync
} as MidwayConfig;

// config/config.prod.ts — prod only
import { MidwayConfig } from '@midwayjs/core';
export default {
  typeorm: { dataSource: { default: { synchronize: false } } }, // NEVER sync in prod
} as MidwayConfig;

// configuration.ts — explicit environment mapping
@Configuration({
  importConfigs: [{ default: DefaultConfig, local: LocalConfig, prod: ProdConfig }],
})

// service — inject typed config (NOT in constructor)
import { Provide, Config, Init } from '@midwayjs/core';
@Provide()
export class DatabaseService {
  @Config('typeorm.dataSource.default') dbConfig: any;

  @Init()
  async init() { this.pool = mysql.createPool(this.dbConfig); }  // ✓ available in @Init
}

// remote/dynamic config via onConfigLoad (return value is merged)
async onConfigLoad(container: IMidwayContainer) {
  return await fetchRemoteConfig();
}
```

Config load order: component `default` → app `default` → component `{env}` → app `{env}`. The function form `(appInfo: MidwayAppInfo) => MidwayConfig` gives access to `appDir`, `baseDir`, `HOME`, etc. for path resolution.

Reference: [Midway Environment Config](https://midwayjs.org/docs/env_config)
