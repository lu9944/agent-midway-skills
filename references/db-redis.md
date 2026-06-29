---
title: Use the Redis Service Factory Correctly
impact: MEDIUM-HIGH
impactDescription: "Correct single/cluster/multi-instance patterns"
tags: database, redis, cache, service-factory
---

## Use the Redis Service Factory Correctly

The `@midwayjs/redis` component exposes a default `RedisService` and a `RedisServiceFactory` for multiple instances. Configure via `redis.client` (single), `redis.clients` (multi), or `redis.client.cluster` (cluster mode). Inject the default service with `@Inject()`, and named instances with `@InjectClient(RedisServiceFactory, 'name')`. Always set TTL explicitly and prefer Redis stores over per-process memory for cross-worker consistency.

**Incorrect (accessing redis directly, ignoring cluster mode, magic instance names):**

```typescript
// ❌ raw ioredis, bypassing the component's lifecycle/health checks
import Redis from 'ioredis';
const redis = new Redis('redis://localhost');   // ❌ not managed, not closed on shutdown

// ❌ magic string instance name
@InjectClient(RedisServiceFactory, 'instance1') redis1: RedisService;
```

**Correct (component-managed service + typed instance names + cluster config):**

```typescript
// config.default.ts
export default {
  redis: {
    // single client
    client: { port: 6379, host: '127.0.0.1', password: process.env.REDIS_PASSWORD, db: 0 },
    // OR cluster
    // client: { cluster: true, nodes: [{host,port}], redisOptions: { password: '...' } },
    // OR multiple named instances
    // clients: { cache: {...}, session: {...} },
  },
} as MidwayConfig;

// default service
import { Provide, Inject, InjectClient } from '@midwayjs/core';
import { RedisService, RedisServiceFactory } from '@midwayjs/redis';

@Provide()
export class CacheService {
  @Inject() redisService: RedisService;

  async getCached(key: string) {
    const cached = await this.redisService.get(key);
    if (cached) return JSON.parse(cached);
    const data = await this.fetch(key);
    await this.redisService.set(key, JSON.stringify(data), 'EX', 60);  // 60s TTL
    return data;
  }
}

// named instance via factory
@Provide()
export class SessionService {
  @InjectClient(RedisServiceFactory, 'session') sessionRedis: RedisService;
}
```

Reference: [Midway Redis](https://midwayjs.org/docs/extensions/redis)
