---
title: Use Defined Injection Identifiers, Not Magic Strings
impact: HIGH
impactDescription: "Reduces magic variables and makes DI intent explicit"
tags: dependency-injection, identifiers, tokens, type-safety
---

## Use Defined Injection Identifiers, Not Magic Strings

TypeScript interfaces are erased at runtime, so they cannot serve as injection keys. When you need to inject an abstraction (e.g. swap implementations per environment), define the identifier as a **constant symbol/string** in one place and reuse it — never scatter raw string literals. For multi-instance service factories, use `@InjectClient(Factory, 'name')` with a named constant. This keeps DI intent explicit and grep-able, eliminating magic variables.

**Incorrect (raw string identifiers scattered as magic variables):**

```typescript
// ❌ magic string 'payment' duplicated and typo-prone across files
@Configuration({ imports: [PaymentModule] })
export class AppModule {}

@Provide('payment')
export class StripeService implements PaymentGateway {}

@Controller('/pay')
export class PayController {
  @Inject('payment') payment: PaymentGateway;   // ❌ 'payment' literal — no single source of truth
}

// ❌ magic instance name for a service factory
@InjectClient(RedisServiceFactory, 'instance1') redis1: RedisService;
```

**Correct (centralized constants + registerObject + typed config keys):**

```typescript
// src/constants.ts — single source of truth for identifiers
export const PAYMENT_GATEWAY = Symbol('PAYMENT_GATEWAY');

// provider
@Provide()
export class StripeService implements PaymentGateway { /* ... */ }

// register an implementation by token (e.g. in onReady or a factory)
container.registerObject(PAYMENT_GATEWAY, new StripeService());

// consumer — uses the constant, not a literal
@Provide()
export class OrderService {
  @Inject(PAYMENT_GATEWAY) payment: PaymentGateway;
}

// service-factory instances: name them via typed config, not inline literals
// config.default.ts → redis.clients.cache / redis.clients.session
@InjectClient(RedisServiceFactory, 'cache') cacheRedis: RedisService;

// register a global object once and inject by a named constant
export const LODASH_KEY = 'lodash';
async onReady(container: IMidwayContainer) {
  container.registerObject(LODASH_KEY, _);
}
```

Reference: [Midway Container — Identifiers](https://midwayjs.org/docs/container), [Service Factory](https://midwayjs.org/docs/service_factory)
