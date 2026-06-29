---
title: Extend MidwayError and MidwayHttpError
impact: HIGH
impactDescription: "Structured errors carry status codes and codes for reliable handling"
tags: error-handling, midwayerror, exceptions, http
---

## Extend MidwayError and MidwayHttpError

Midway provides a structured error hierarchy: `MidwayError` (base, carries `code` and `cause`) and `MidwayHttpError` (adds an HTTP status code). Throw these instead of generic `Error` so filters and logging can act on them. For predefined HTTP statuses, use the `httpError.*` helpers (`BadRequestError`, `UnauthorizedError`, `NotFoundError`, etc.). Custom business errors extend `MidwayError` with a stable error code; HTTP-facing errors extend `MidwayHttpError`.

**Incorrect (generic Error, manual status code plumbing):**

```typescript
@Provide()
export class UserService {
  async findById(id: number) {
    const user = await this.repo.findOne({ where: { id } });
    if (!user) {
      throw new Error('User not found');   // ❌ no status, no code, unstructured
    }
    return user;
  }
}
```

**Correct (structured Midway errors with codes and HTTP status):**

```typescript
import { MidwayError, MidwayHttpError, httpError, HttpStatus } from '@midwayjs/core';

// Throw predefined HTTP errors for common cases
@Provide()
export class UserService {
  async findById(id: number) {
    const user = await this.repo.findOne({ where: { id } });
    if (!user) {
      throw new httpError.NotFoundError(`User #${id} not found`);  // 404, structured
    }
    return user;
  }
}

// Custom business error — extends MidwayError, carries a stable code
export class InsufficientBalanceError extends MidwayError {
  constructor(accountId: number, needed: number) {
    super(
      `Account ${accountId} needs ${needed}`,
      'BIZ_INSUFFICIENT_BALANCE',     // stable, non-colliding code
    );
  }
}

// Custom HTTP error — extends MidwayHttpError, carries status
export class UserNotFoundException extends MidwayHttpError {
  constructor(userId: number) {
    super(`User ${userId} not found`, HttpStatus.NOT_FOUND);
  }
}
```

### Register Error Codes (non-colliding)

Use `registerErrorCode` to generate stable, prefixed error codes for a module/component. This prevents collisions across packages:

```typescript
import { registerErrorCode } from '@midwayjs/core';

// generates codes like 'ORDER_10000', 'ORDER_10001'
export const OrderErrorEnum = registerErrorCode('order', {
  INSUFFICIENT_STOCK: 10000,
  PAYMENT_FAILED: 10001,
  ADDRESS_INVALID: 10002,
} as const);

// use in custom errors
export class InsufficientStockError extends MidwayError {
  constructor(productId: string) {
    super(`Product ${productId} out of stock`, OrderErrorEnum.INSUFFICIENT_STOCK);
  }
}
```

The framework's own codes are `MIDWAY_10000`–`MIDWAY_10022` (e.g. `MIDWAY_10003` = DI definition not found, `MIDWAY_10010` = Singleton injecting Request scope).

Reference: [Midway Custom Errors](https://midwayjs.org/docs/custom_error), [Error Codes](https://midwayjs.org/docs/error_code)
