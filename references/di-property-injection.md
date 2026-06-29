---
title: Use Property Injection with @Provide/@Inject Pairing
impact: CRITICAL
impactDescription: "The fundamental DI contract of Midway"
tags: dependency-injection, provide, inject, property-injection
---

## Use Property Injection with @Provide/@Inject Pairing

Midway's primary DI style is **property injection**: a class decorated with `@Provide()` (or implicitly via `@Controller`/`@Singleton`) registers itself with the IoC container, and consumers inject it with `@Inject()`. The two are paired — `@Inject` resolves by the TypeScript class type, so the dependency must be `@Provide`-d. Prefer this over reaching into the container manually. v4 also restores constructor injection, but property injection remains the idiomatic, zero-arg-constructable default used throughout `cool-admin-midway`.

**Incorrect (manual container access, missing @Provide, unpaired injection):**

```typescript
// ❌ Service missing @Provide — cannot be injected
export class UserService {
  async getUser(id: number) { return { id }; }
}

@Controller('/api')
export class UserController {
  // ❌ UserService was never registered, so this is undefined
  @Inject() userService: UserService;

  // ❌ Service locator anti-pattern — hides dependencies
  async list(@Query() q, @ApplicationContext() ctx) {
    const svc = await ctx.requestContext.getAsync(UserService);
  }
}
```

**Correct (@Provide/@Inject pairing, idiomatic property injection):**

```typescript
import { Provide, Inject, Controller, Get, Query } from '@midwayjs/core';

// @Provide() registers the class with the container (makes it injectable)
@Provide()
export class UserService {
  async getUser(id: number) {
    return { id, name: 'Harry' };
  }
}

@Controller('/api/user')   // @Controller implicitly includes @Provide
export class UserController {
  // @Inject resolves by the UserService type — paired with @Provide above
  @Inject()
  userService: UserService;

  @Get('/')
  async getUser(@Query('id') uid: number) {
    const user = await this.userService.getUser(uid);
    return { success: true, message: 'OK', data: user };
  }
}

// v4 also supports constructor injection (only @Inject works in constructors):
@Provide()
export class OrderService {
  constructor(@Inject() private userService: UserService) {}
}
```

When you need an explicit identifier (multiple implementations, or a non-class dependency), pair string identifiers explicitly: `@Provide('cache')` + `@Inject('cache')`.

Reference: [Midway Container & DI](https://midwayjs.org/docs/container)
