---
title: Implement Multi-Tenancy with @midwayjs/tenant
impact: MEDIUM-HIGH
impactDescription: "Request-scoped tenant isolation visible to Singletons"
tags: api, tenant, multi-tenancy, isolation, middleware
---

## Implement Multi-Tenancy with @midwayjs/tenant

`@midwayjs/tenant` provides request-scoped tenant isolation via `TenantManager`. Set the current tenant in middleware (`setCurrentTenant`), then read it anywhere — including from `@Singleton` services — via `getCurrentTenant()`. This avoids the scope-downgrade problem: Singletons stay Singleton while still seeing per-request tenant data. Define a typed `TenantInfo` interface for your tenant shape.

**Incorrect (passing tenantId through every method, or shared mutable state):**

```typescript
// ❌ tenantId threaded through every signature — invasive, error-prone
async createOrder(userId: number, tenantId: number, dto: any) { /* ... */ }
async findOrder(id: number, tenantId: number) { /* ... */ }

// ❌ shared mutable tenant on a Singleton — clobbered by concurrent requests
@Singleton()
export class TenantHolder { private id: number; set(id) { this.id = id; } }
```

**Correct (middleware sets, any scope reads via TenantManager):**

```typescript
import * as tenant from '@midwayjs/tenant';
import { TenantManager } from '@midwayjs/tenant';
@Configuration({ imports: [tenant] })

// typed tenant info
interface TenantInfo { id: string; name: string; plan: string; }

// middleware extracts tenant from the request and sets it for the chain
@Middleware()
export class TenantMiddleware implements IMiddleware<Context, NextFunction> {
  @Inject() tenantManager: TenantManager;

  resolve() {
    return async (ctx: Context, next: NextFunction) => {
      // extract from header/subdomain/JWT — your strategy
      const info: TenantInfo = { id: ctx.headers['x-tenant-id'], name: '...', plan: 'pro' };
      await this.tenantManager.setCurrentTenant(info);
      return next();
    };
  }
}

// a Singleton reads the current tenant without being Request-scoped
@Singleton()
export class OrderService {
  @Inject() tenantManager: TenantManager;

  async create(dto: any) {
    const tenant = await this.tenantManager.getCurrentTenant<TenantInfo>();
    return this.repo.save({ ...dto, tenantId: tenant.id });
  }
}

// register globally
async onReady(_, app) { app.useMiddleware(TenantMiddleware); }
```

Tenant data is always tied to a request — set it inside middleware per request. Both Request and Singleton scopes see only the current request's tenant.

Reference: [Midway Tenant](https://midwayjs.org/docs/extensions/tenant)
