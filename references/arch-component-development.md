---
title: Develop Reusable Components and Custom Frameworks
impact: HIGH
impactDescription: "Components are the unit of reuse; frameworks are components with a server lifecycle"
tags: architecture, component, framework, namespace, bas-framework, reuse
---

## Develop Reusable Components and Custom Frameworks

A Midway component is a self-contained mini-application: a unique `namespace`, `@Configuration` entry, explicit exports, and typed config augmentation via `declare module`. Test it with `createLightApp({ imports: [custom] })`. Since v3, a **Framework** is part of a component — extend an existing one (inject `koa.Framework`, add middleware/filters, extend context) or build a custom protocol framework by extending `BaseFramework` and implementing `configure()`/`applicationInitialize()`/`run()`.

### Component basics: namespace, entry export, typed config

**Incorrect (no namespace, missing entry export, untyped config):**

```typescript
// ❌ no namespace — collides with host config keys
@Configuration({ importConfigs: [{ default: DefaultConfig }] })
export class BookConfiguration {}

// ❌ index.ts forgets to export Configuration — component never registers
// ❌ index.d.ts missing → @Config('book.*') has no type info
```

**Correct:**

```typescript
// src/configuration.ts
@Configuration({
  namespace: 'book',                                    // REQUIRED unique namespace
  importConfigs: [{ default: DefaultConfig }],
})
export class BookConfiguration implements ILifeCycle {
  async onReady(container: IMidwayContainer) {}
}

// src/index.ts — MANDATORY: only explicitly exported decorated classes are scanned
export { BookConfiguration as Configuration } from './configuration';
export * from './service/book.service';

// index.d.ts — typed config (declaration merging, eliminates magic strings)
import '@midwayjs/core';
// if depending on another component, import it so its types are visible:
// import '@midwayjs/axios';
export * from './dist/index';
declare module '@midwayjs/core' {
  interface MidwayConfig {
    book?: { pageSize: number; apiBase: string };
  }
}
```

```json
// package.json
{ "main": "dist/index.js", "typings": "index.d.ts",
  "files": ["dist/**/*.js", "dist/**/*.d.ts", "index.d.ts"] }
```

### Test components with createLightApp

```typescript
import { createLightApp, close } from '@midwayjs/mock';
import * as custom from '../src';

describe('book component', () => {
  it('service works', async () => {
    const app = await createLightApp('', { imports: [custom] });
    const svc = await app.getApplicationContext().getAsync(custom.BookService);
    expect(await svc.getBookById()).toEqual({ data: 'hello world' });
    await close(app);
  });
});

// If the component needs HTTP context, test with a full app + createHttpRequest:
// const app = await createApp(join(__dirname, 'fixtures/base-app'), { imports: [custom] });
// const res = await createHttpRequest(app).get('/');
```

### Extend an existing Framework

Inject the framework (e.g. `koa.Framework`) in `onReady` to add middleware/filters and extend context:

```typescript
@Configuration({ namespace: 'myKoa', imports: [koa] })
export class MyKoaConfiguration {
  @Inject() framework: koa.Framework;

  async onReady() {
    this.framework.useMiddleware(CustomMiddleware);    // app.useMiddleware proxies this
    this.framework.useFilter(CustomFilter);

    // extend koa context (pair with declare module in index.d.ts — see context_definition)
    const app = this.framework.getApplication();
    Object.defineProperty(app.context, 'user', {
      get() { return 'xxx'; },   // ctx.user now available everywhere
      enumerable: true,
    });
  }

  async onServerReady() {
    const server = this.framework.getServer();   // raw HTTP server
  }
}
```

### Write a custom Framework (custom protocol)

Extend `BaseFramework` and implement the three required methods. Export `Application`, `Context`, `Framework`, and the options interface:

```typescript
import { Framework, BaseFramework, IConfigurationOptions, IMidwayApplication, IMidwayContext } from '@midwayjs/core';
import * as http from 'http';

export interface Context extends IMidwayContext {}
export interface Application extends IMidwayApplication<Context> {}
export interface IMidwayCustomConfigurationOptions extends IConfigurationOptions {
  port: number;
}

@Framework()
export class MidwayCustomHTTPFramework extends BaseFramework<Application, Context, IMidwayCustomConfigurationOptions> {
  configure(): IMidwayCustomConfigurationOptions {
    return this.configService.getConfiguration('customKey');
  }

  async applicationInitialize() {
    this.app = http.createServer((req, res) => {
      const ctx = this.app.createAnonymousContext();    // request context with logger, DI scope
      ctx.requestContext.getAsync('xxx').then(ins => ins.doWork()).then(() => res.end());
    });
    this.defineApplicationProperties();   // bind getConfig, getLogger, etc. to app
  }

  async run() {
    if (this.configurationOptions.port) {
      await new Promise<void>(resolve => this.app.listen(this.configurationOptions.port, resolve));
    }
  }
}

// src/index.ts — export the framework
export { Application, Context, MidwayCustomHTTPFramework as Framework, IMidwayCustomConfigurationOptions } from './framework';
export { MyConfiguration as Configuration } from './configuration';
```

### Strong vs weak dependencies, publishing

- **Strong dependency** (always needed): declare in `imports` — e.g. `imports: [axios]`.
- **Weak dependency** (optional): check at runtime — `if (container.hasNamespace('axios')) { ... }`.
- Publish: `npm run build && npm publish` (standard Node.js package).
- Monorepo dev: use lerna with hoist, or place components under `src/components/` with `main` pointing to `src/index` (change back to `dist/index` before publish).

Reference: [Midway Component Development](https://midwayjs.org/docs/component_development), [Context Definition](https://midwayjs.org/docs/context_definition)
