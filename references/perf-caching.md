---
title: Use @midwayjs/cache-manager for Strategic Caching
impact: HIGH
impactDescription: "Dramatically reduces DB load and response times"
tags: performance, caching, cache-manager, redis
---

## Use @midwayjs/cache-manager for Strategic Caching

In v4, prefer `@midwayjs/cache-manager` (based on cache-manager v5) over the legacy `@midwayjs/cache`. It supports multi-instance stores (`clients`), Redis-backed stores (reusing the redis component), multi-level caching, and method-level caching via the `@Caching(instanceName, ttl?)` decorator. Note the TTL unit difference: cache-manager v5 uses **milliseconds**. For cross-worker consistency, use a Redis store — in-memory cache is per-process.

**Incorrect (no caching for expensive queries, or per-process memory cache shared across workers):**

```typescript
@Provide()
export class ProductService {
  async getPopular() {
    // ❌ runs a heavy aggregation on EVERY request
    return this.repo.createQueryBuilder('p')
      .leftJoin('p.orders', 'o')
      .groupBy('p.id').orderBy('COUNT(o.id)', 'DESC').limit(20).getMany();
  }
}
```

**Correct (Redis-backed store + @Caching decorator + invalidation):**

```typescript
// config.default.ts — Redis store reuses the redis component instance
import { createRedisStore } from '@midwayjs/cache-manager';
export default {
  cacheManager: {
    clients: {
      default: { store: createRedisStore('default'), options: { ttl: 60000 } }, // ms
    },
  },
  redis: { clients: { default: { port: 6379, host: '127.0.0.1' } } },
} as MidwayConfig;

// service — imperative access for granular control + invalidation
import { CachingFactory, MidwayCache, Caching } from '@midwayjs/cache-manager';
import { InjectClient, Provide } from '@midwayjs/core';

@Provide()
export class ProductService {
  @InjectClient(CachingFactory, 'default')
  cache: MidwayCache;

  async getPopular() {
    const cached = await this.cache.get('products:popular');
    if (cached) return cached;
    const products = await this.fetchPopular();
    await this.cache.set('products:popular', products, 5 * 60 * 1000); // 5 min
    return products;
  }

  async update(id: number, dto: any) {
    const product = await this.repo.save({ id, ...dto });
    await this.cache.del('products:popular');   // invalidate on change
    return product;
  }
}

// method-level caching via decorator — simplest form
@Provide()
export class ConfigService {
  @Caching('default', 10000)          // cache result for 10s
  async getAppConfig() { return this.repo.find(); }

  // dynamic key via callback: return null to skip caching
  @Caching('default', (ctx) => ctx.methodArgs[0]?.id ?? null, 30000)
  async getUser(id: number) { return this.repo.findOne({ where: { id } }); }
}
```

Reference: [Midway Cache Manager](https://midwayjs.org/docs/extensions/caching)
