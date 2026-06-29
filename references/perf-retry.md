---
title: Retry Fallible Operations with retryWithAsync
impact: MEDIUM
impactDescription: "Built-in retry for transient failures without external libs"
tags: performance, retry, resilience, retryWithAsync
---

## Retry Fallible Operations with retryWithAsync

Midway provides `retryWithAsync(fn, retryTimes, options)` / `retryWith(fn, retryTimes)` from `@midwayjs/core` for retrying transient failures. `retryTimes` is the **number of extra retries** (total executions = 1 + retryTimes). Use the `receiver` option to bind `this` for class methods (avoids manual `.bind()`), `retryInterval` for backoff delay, and `throwOriginError` to surface the original error instead of `MidwayRetryExceededMaxTimesError`.

**Incorrect (manual retry loops, no backoff, wrong error handling):**

```typescript
async function callWithRetry() {
  for (let i = 0; i < 3; i++) {          // ❌ manual loop, no delay, no error chaining
    try { return await flakyApi(); }
    catch (e) { if (i === 2) throw e; }
  }
}
```

**Correct (retryWithAsync + receiver binding + interval + typed errors):**

```typescript
import { retryWithAsync, MidwayRetryExceededMaxTimesError } from '@midwayjs/core';

@Provide()
export class PaymentService {
  async charge(orderId: string) {
    // wrap a class method: receiver binds `this`, 2 retries with 500ms backoff
    const charge = retryWithAsync(this.callPaymentGateway, 2, {
      receiver: this,
      retryInterval: 500,
    });
    try {
      return await charge(orderId);
    } catch (err) {
      if (err instanceof MidwayRetryExceededMaxTimesError) {
        this.logger.error('payment failed after retries; cause:', err.cause);
        throw new httpError.BadGatewayError('payment gateway unavailable');
      }
      throw err;
    }
  }

  private async callPaymentGateway(orderId: string) {
    return this.httpService.post('/charge', { orderId });
  }
}

// sync variant (no retryInterval)
import { retryWith } from '@midwayjs/core';
const parseSafe = retryWith(JSON.parse, 1);
```

Use `throwOriginError: true` when you want the underlying error (not the wrapper) to propagate.

Reference: [Midway Retry](https://midwayjs.org/docs/retry)
