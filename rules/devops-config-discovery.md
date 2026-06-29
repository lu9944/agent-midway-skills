---
title: Use Config Centers and Service Discovery (Consul/etcd)
impact: MEDIUM
impactDescription: "Centralized config and distributed service registration"
tags: devops, consul, etcd, config-center, service-discovery
---

## Use Config Centers and Service Discovery (Consul/etcd)

`@midwayjs/consul` and `@midwayjs/etcd` both use the Service Factory pattern (`client`/`clients`) and provide a unified v4 **Service Discovery** abstraction (`XxxServiceDiscovery`). For config, poll the KV store in a `@Singleton` service with `@Init()` + `setInterval` — never call `kv.get` per request (it hammers the server). For service registration, call `createClient()` in `onServerReady`, then `register()` (idempotent) and `getInstances()`.

**Incorrect (per-request KV lookup, no registration caching):**

```typescript
// ❌ KV read on every request — overwhelms Consul
@Provide()
export class ConfigService {
  @Inject() consul: ConsulService;
  async getValue(key: string) {
    return (await this.consul.kv.get(key)).Value;   // ❌ network call per request
  }
}
```

**Correct (singleton-cached config + service registration + discovery):**

```typescript
import * as consul from '@midwayjs/consul';
@Configuration({ imports: [consul] })

// config.default.ts
export default {
  consul: {
    client: { host: '127.0.0.1', port: 8500 },
    serviceDiscovery: {},
  },
} as MidwayConfig;

// config polling — Singleton caches the value
import { Provide, Scope, ScopeEnum, Init, Inject } from '@midwayjs/core';
import { ConsulService } from '@midwayjs/consul';
@Singleton()
export class DynamicConfigService {
  @Inject() consul: ConsulService;
  private config: Record<string, any>;

  @Init()
  async init() {
    await this.refresh();
    setInterval(() => this.refresh(), 5000);   // poll every 5s, not per request
  }
  private async refresh() { this.config = (await this.consul.kv.get('app/config')).Value; }
  get<T>(key: string): T { return this.config?.[key]; }
}

// service registration in onServerReady
import { ConsulServiceDiscovery } from '@midwayjs/consul';
@Configuration({})
export class MainConfiguration {
  @Inject() discovery: ConsulServiceDiscovery;
  async onServerReady() {
    const client = this.discovery.createClient();
    await client.register({                    // idempotent
      id: 'order-1', name: 'order-service',
      address: '10.0.0.1', port: 7001,
      check: { ttl: '10s' },                   // health-check TTL
      meta: { version: '1.0.0' },
    });
  }
}

// discover instances
const instances = await this.discovery.getInstances({ service: 'order-service' });
```

etcd follows the same pattern (`ETCDService`, `EtcdServiceDiscovery`, `etcd.client.host: [...]`). Consul `getInstance` takes `{ service }` (object); etcd takes a bare string — check each type.

### The General ServiceDiscovery Abstraction

Midway provides a unified abstraction in `@midwayjs/core` for implementing custom service discovery against any registry. Extend `ServiceDiscoveryClient` (instance lifecycle: `register`/`deregister`/`online`/`offline`) and `ServiceDiscovery` (entry: `createClient`/`getInstances`/`getInstance` with built-in load balancing). Built-in load balancers: `LoadBalancerType.RANDOM`, `ROUND_ROBIN`. Health checks via `ServiceDiscoveryHealthCheckFactory.create('ttl'|'http'|'tcp', opts)`.

```typescript
import { ServiceDiscovery, ServiceDiscoveryClient, LoadBalancerType } from '@midwayjs/core';

// 1. implement the client (talks to your registry)
class MySDClient extends ServiceDiscoveryClient<any, MyOptions, MyInstance> {
  async register(inst: MyInstance) { /* write to registry */ }
  async deregister() { /* remove */ }
  async online() { /* mark available */ }
  async offline() { /* mark unavailable */ }
  async beforeStop() { /* stop heartbeat/timers */ }
}

// 2. implement the entry class
class MyServiceDiscovery extends ServiceDiscovery<any, MyOptions, MyInstance, MyInstance, string> {
  protected getServiceClient() { return {}; }
  protected createServiceDiscoverClientImpl(opts: MyOptions) { return new MySDClient(this.getServiceClient(), opts); }
  protected getDefaultServiceDiscoveryOptions(): MyOptions { return { ttl: 30 }; }
  public async getInstances(serviceName: string): Promise<MyInstance[]> { /* query registry */ return []; }
}

// 3. use
const discovery = await container.getAsync(MyServiceDiscovery);
discovery.setLoadBalancer(LoadBalancerType.ROUND_ROBIN);
const client = discovery.createClient({ ttl: 20 });
await client.register({ serviceName: 'order', id: 'order-1' });
const instances = await discovery.getInstances('order');
const one = await discovery.getInstance('order');   // load-balanced pick
// on shutdown: ServiceDiscovery.destroy() calls stop() on all clients → deregister
```

Reference: [Midway Service Discovery](https://midwayjs.org/docs/service_discovery), [Consul](https://midwayjs.org/docs/extensions/consul), [etcd](https://midwayjs.org/docs/extensions/etcd)
