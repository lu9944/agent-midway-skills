---
title: Resolve Circular Dependencies with @LazyInject
impact: HIGH
impactDescription: "v4 auto-detects cycles; @LazyInject is the safe escape hatch"
tags: dependency-injection, circular, lazyinject, v4
---

## Resolve Circular Dependencies with @LazyInject

Circular dependencies (A → B → C → A) indicate an architectural smell and should ideally be resolved by extracting shared logic or using events. However, when unavoidable, v4 **auto-detects** cycles and throws a clear error (`Circular dependency detected: A -> B -> C -> A`). The correct escape hatch is `@LazyInject(() => Clazz)`, which defers resolution until first access. Use it **only** for genuine circular dependencies, not as a general pattern.

**Incorrect (mutual @Inject creates a cycle that crashes startup):**

```typescript
@Provide()
export class OrderService {
  @Inject() userService: UserService;   // Order → User
}

@Provide()
export class UserService {
  @Inject() orderService: OrderService;  // User → Order → cycle! startup crash
}
```

**Correct (break the cycle with @LazyInject on one side):**

```typescript
import { Provide, Inject, LazyInject } from '@midwayjs/core';

@Provide()
export class OrderService {
  @Inject() userService: UserService;   // eager — fine
}

@Provide()
export class UserService {
  // @LazyInject defers resolution to first property access, breaking the cycle
  @LazyInject(() => OrderService)
  orderService: OrderService;
}
```

**Prefer these structural fixes over `@LazyInject`:**

```typescript
// Fix 1: extract shared logic into a third, dependency-free module
@Provide()
export class SharedNotificationService { /* ... */ }
// Both OrderService and UserService depend on SharedNotificationService

// Fix 2: use the event emitter for fire-and-forget decoupling
import { OnEvent, EventEmitterService } from '@midwayjs/event-emitter';
@Singleton()
export class UserService {
  @Inject() eventEmitter: EventEmitterService;
  async createUser() {
    this.eventEmitter.emitAsync('user.created', user);  // no direct dependency
  }
}
```

Reference: [Midway Container — Circular Dependency](https://midwayjs.org/docs/container)
