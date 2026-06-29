---
title: Extend ServiceFactory for Multi-Instance Services
impact: MEDIUM-HIGH
impactDescription: "The core pattern behind redis/oss/cos/kafka multi-instance components"
tags: architecture, service-factory, multi-instance, service, core
---

## Extend ServiceFactory for Multi-Instance Services

`ServiceFactory<T>` (from `@midwayjs/core`) is the base class for components that manage multiple named instances of an SDK client (e.g. Redis, OSS, COS, Kafka producer). Extend it as a `@Singleton`, implement `createClient(config)` and `getName()`, call `initClients(config)` in `@Init`. Consumers get instances via `factory.get('name')` or `@InjectClient(FactoryClass, 'name')`. This is distinct from `DataSourceManager` (which also binds entities).

**Incorrect (manual client management, no lifecycle, no multi-instance support):**

```typescript
@Provide()
export class MyClientService {
  private client: MyClient;
  @Init() init() { this.client = new MyClient(config); }  // ❌ single instance, no cleanup, no config-driven
}
```

**Correct (extend ServiceFactory + config-driven multi-instance + InjectClient):**

```typescript
import { ServiceFactory, Provide, Scope, ScopeEnum, Init, Config, InjectClient } from '@midwayjs/core';

@Provide()
@Scope(ScopeEnum.Singleton)
export class MyClientServiceFactory extends ServiceFactory<MyClient> {
  @Config('myClient') myClientConfig;

  @Init()
  async init() {
    await this.initClients(this.myClientConfig, { concurrent: true });  // v4 parallel option
  }

  protected createClient(config: any, clientName?: string): MyClient {
    return new MyClient(config);     // create one instance from config (clientName optional)
  }

  getName(): string { return 'myClient'; }

  async destroyClient(client: MyClient, clientName: string) {
    await client.close();            // optional cleanup
  }
}

// config.default.ts — 3 config forms
export default {
  myClient: {
    default: { timeout: 5000 },                  // shared defaults
    client: { baseUrl: 'https://a.com' },        // single instance → becomes clients.default
    // OR multiple:
    // clients: { aaa: { baseUrl: '...' }, bbb: { baseUrl: '...' } },
    // defaultClientName: 'aaa',
  },
} as MidwayConfig;

// consumer — default instance
@Provide()
export class MyService {
  @Inject() factory: MyClientServiceFactory;
  async call() { return this.factory.get().request(); }        // default
  async callNamed() { return this.factory.get('bbb').request(); }
}

// consumer — named instance via decorator
@Provide()
export class AnotherService {
  @InjectClient(MyClientServiceFactory, 'aaa') aaaClient: MyClient;
}

// dynamic creation at runtime
const dyn = await this.factory.createInstance({ baseUrl: '...' }, 'dynamicName');
```

Config type augmentation + default proxy pattern:
```typescript
import { ServiceFactoryConfigOption } from '@midwayjs/core';
declare module '@midwayjs/core' {
  interface MidwayConfig {
    myClient?: ServiceFactoryConfigOption<MyClientConfig>;
  }
}
// optional: delegate all prototype methods to the default instance
// delegateTargetAllPrototypeMethod(MyClientWrapperService, MyClient);
```

Reference: [Midway Service Factory](https://midwayjs.org/docs/service_factory)
