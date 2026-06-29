---
title: Use @OnEvent for Decoupled Event-Driven Processing
impact: MEDIUM-HIGH
impactDescription: "Decouples modules without direct service dependencies"
tags: microservices, events, event-emitter, decoupling
---

## Use @OnEvent for Decoupled Event-Driven Processing

Use the `@midwayjs/event-emitter` component (built on eventemitter2) for in-process pub/sub. Emitters fire events; listeners react without a direct dependency. **Critical:** `@OnEvent` handlers must be on a `@Singleton()` class. Emit synchronously with `emit()` or asynchronously with `emitAsync()`. **Best practice:** put `@OnEvent` handlers on a `@Singleton()` class (listeners lack request context; Singleton avoids scope issues)

**Incorrect (tight coupling — service knows every downstream consumer):**

```typescript
@Provide()
export class UserService {
  @Inject() emailService: EmailService;
  @Inject() analyticsService: AnalyticsService;
  @Inject() loyaltyService: LoyaltyService;

  async createUser(dto: CreateUserDTO) {
    const user = await this.repo.save(dto);
    // ❌ every new behavior requires editing this service
    await this.emailService.sendWelcome(user);
    await this.analyticsService.track('signup', user);
    await this.loyaltyService.addPoints(user.id, 100);
    return user;
  }
}
```

**Correct (emit event; listeners in separate modules react):**

```typescript
import { Provide, Singleton, Inject } from '@midwayjs/core';
import { OnEvent, EventEmitterService } from '@midwayjs/event-emitter';

@Provide()
export class UserService {
  @Inject() eventEmitter: EventEmitterService;

  async createUser(dto: CreateUserDTO) {
    const user = await this.repo.save(dto);
    // emit — no knowledge of consumers
    await this.eventEmitter.emitAsync('user.created', user);
    return user;
  }
}

// each listener in its own module — @Singleton is REQUIRED for @OnEvent
@Singleton()
export class EmailListener {
  @OnEvent('user.created', { suppressErrors: false })
  async sendWelcome(user: User) { await this.mailer.send(user.email); }
}

@Singleton()
export class AnalyticsListener {
  @OnEvent('user.created')   // wildcard 'user.*' also supported when config.wildcard=true
  async track(user: User) { await this.analytics.track('signup', user); }
}

// config.default.ts
export default {
  eventEmitter: { wildcard: true, delimiter: '.', maxListeners: 10 },
} as MidwayConfig;
```

Reference: [Midway Events](https://midwayjs.org/docs/extensions/events)
