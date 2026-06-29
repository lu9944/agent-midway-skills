---
title: Use @Aspect for Cross-Cutting AOP Interception
impact: MEDIUM-HIGH
impactDescription: "Clean separation for logging, timing, and cross-cutting logic"
tags: api, aspect, aop, interceptor
---

## Use @Aspect for Cross-Cutting AOP Interception

AOP aspects (`@Aspect` + `IMethodAspect`) intercept method calls for cross-cutting concerns (logging, timing, validation, caching) without polluting business logic. Aspects are always **Singleton scope** and live in `src/aspect/`. They target a class (or array of classes) and optionally a picomatch method pattern. Use the `IMethodAspect` lifecycle (`before`, `around`, `afterReturn`, `afterThrow`, `after`) — `around` gives full control. Remember: aspects do **not** apply to inherited (parent class) methods.

**Incorrect (duplicating logging/timing logic in every method):**

```typescript
@Provide()
export class OrderService {
  async createOrder(dto: any) {
    const start = Date.now();
    this.logger.info('createOrder called');        // ❌ repeated boilerplate
    const result = await this.repo.save(dto);
    this.logger.info(`createOrder done in ${Date.now() - start}ms`);
    return result;
  }
  async cancelOrder(id: number) {
    const start = Date.now();                       // ❌ same boilerplate again
    this.logger.info('cancelOrder called');
    /* ... */
  }
}
```

**Correct (a single @Aspect handles timing for the whole class):**

```typescript
import { Aspect, IMethodAspect, JoinPoint } from '@midwayjs/core';
import { ILogger, Logger } from '@midwayjs/core';
import { OrderService } from '../service/order.service';

// src/aspect/timing.aspect.ts
@Aspect(OrderService)                       // intercept ALL methods of OrderService
export class TimingAspect implements IMethodAspect {
  @Logger() logger: ILogger;

  async around(point: JoinPoint) {          // around = full control
    const start = Date.now();
    const name = point.methodName;
    try {
      const result = await point.proceed(...point.args);  // call original
      this.logger.info(`${name} ok in ${Date.now() - start}ms`);
      return result;                         // can modify the return value
    } catch (err) {
      this.logger.error(`${name} failed in ${Date.now() - start}ms`, err);
      throw err;
    }
  }
}

// target multiple classes + filter by method name (picomatch glob)
@Aspect([OrderService, PaymentService], '*By*')   // only methods containing "By"
export class AuditAspect implements IMethodAspect {
  async before(point: JoinPoint) { /* runs before each matched method */ }
}
```

`@Aspect(target, matchPattern?, priority?)` — higher priority registers first and is called later (onion model).

Reference: [Midway Aspect (AOP)](https://midwayjs.org/docs/aspect)
