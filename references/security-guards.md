---
title: Use @Guard for Route-Level Authorization
impact: HIGH
impactDescription: "Guards sit inside the route, know the matched handler, ideal for authz"
tags: security, guard, authorization, rbac
---

## Use @Guard for Route-Level Authorization

Guards (`@Guard`, since v3.6.0) run **after global middleware** but **before** the route method. Unlike middleware (which has no knowledge of the matched controller/method), a guard receives the exact `supplierClz` and `methodName`, making it the correct tool for authorization. Combine a custom metadata decorator with `getPropertyMetadata` for declarative role-based access control. Return `true` to proceed, `false` to throw a 403, or throw a custom error.

**Incorrect (manual auth checks repeated in every handler):**

```typescript
@Controller('/admin')
export class AdminController {
  @Get('/users')
  async getUsers(@Ctx() ctx: Context) {
    if (!ctx.user) throw new httpError.UnauthorizedError();   // ❌ repeated everywhere
    if (!ctx.user.roles.includes('admin')) throw new httpError.ForbiddenError();
    return this.adminService.getUsers();
  }
}
```

**Correct (declarative @UseGuard + custom @Role metadata):**

```typescript
import { Guard, IGuard, savePropertyMetadata, getPropertyMetadata, UseGuard } from '@midwayjs/core';
import { httpError } from '@midwayjs/core';
import { Context } from '@midwayjs/koa';

// 1. Custom metadata decorator carrying required roles
const ROLE_META_KEY = 'role:name';
export function Role(roleName: string | string[]): MethodDecorator {
  return (target, key, desc) => {
    savePropertyMetadata(ROLE_META_KEY, [].concat(roleName), target, key);
  };
}

// 2. The guard reads metadata and checks the user
@Guard()
export class AuthGuard implements IGuard<Context> {
  async canActivate(
    ctx: Context,
    supplierClz: any,
    methodName: string,
  ): Promise<boolean> {
    const roles = getPropertyMetadata<string[]>(ROLE_META_KEY, supplierClz, methodName);
    // no @Role metadata → public route
    if (!roles?.length) return true;
    if (!ctx.user?.role) throw new httpError.UnauthorizedError();
    if (!roles.includes(ctx.user.role)) throw new httpError.ForbiddenError();
    return true;
  }
}

// 3. Apply globally, then decorate routes
async onReady(_, app) { app.useGuard(AuthGuard); }

@Controller('/user')
export class UserController {
  @UseGuard(AuthGuard)
  @Role(['admin'])
  @Get('/roles')
  async getRoles() { return this.userService.getRoles(); }
}
```

Reference: [Midway Guards](https://midwayjs.org/docs/guard)
