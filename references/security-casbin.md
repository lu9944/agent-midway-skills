---
title: Use Casbin for Fine-Grained RBAC/ABAC Authorization
impact: MEDIUM
impactDescription: "Policy-driven access control with externalized rules"
tags: security, casbin, rbac, abac, authorization
---

## Use Casbin for Fine-Grained RBAC/ABAC Authorization

`@midwayjs/casbin` provides policy-driven authorization (RBAC, ABAC). Casbin does **authorization only** — pair it with `@midwayjs/passport` for authentication. Define a model file + policy, configure `usernameFromContext` to read the authenticated user, and apply the built-in `AuthGuard` + `@UsePermission` decorator. For multi-instance, use the Redis or TypeORM adapter with a watcher for policy sync.

**Incorrect (hardcoded role checks, no policy model):**

```typescript
@Get('/users')
async findAll(@Ctx() ctx) {
  if (ctx.user.role !== 'admin') throw new httpError.ForbiddenError();  // ❌ hardcoded, not scalable
  return this.userService.findAll();
}
```

**Correct (declarative @UsePermission + AuthGuard + policy files):**

```typescript
import * as casbin from '@midwayjs/casbin';
@Configuration({ imports: [passport, casbin] })

// config.default.ts
export default {
  casbin: {
    modelPath: join(appInfo.appDir, 'basic_model.conf'),
    policyAdapter: join(appInfo.appDir, 'basic_policy.csv'),
    usernameFromContext: (ctx) => ctx.user,   // read from passport auth
  },
};

// controller — declarative permission
import { AuthGuard, UsePermission, AuthActionVerb, AuthPossession } from '@midwayjs/casbin';

@Controller('/users')
@UseGuard(AuthGuard)
export class UserController {
  @UsePermission({ action: AuthActionVerb.READ, resource: 'users', possession: AuthPossession.ANY })
  @Get('/')
  async findAll() { return this.userService.findAll(); }

  @UsePermission({ action: AuthActionVerb.UPDATE, resource: 'users', possession: AuthPossession.OWN })
  @Patch('/:id')
  async update() { /* ... */ }
}

// imperative check (custom guard)
@Guard()
export class CustomGuard implements IGuard {
  @Inject() casbinEnforcerService: CasbinEnforcerService;
  async canActivate(ctx, clz, method) {
    return this.casbinEnforcerService.enforce(ctx.user, 'users', 'read');
  }
}
```

For distributed deployments use adapters: `@midwayjs/casbin-redis-adapter` (`createAdapter({ clientName })`) or `@midwayjs/casbin-typeorm-adapter` (`createAdapter({ dataSourceName })`, register `CasbinRule` entity). Add a watcher (`createWatcher`) for pub/sub policy sync — note the subscriber needs a **dedicated** Redis client.

Reference: [Midway Casbin](https://midwayjs.org/docs/extensions/casbin)
