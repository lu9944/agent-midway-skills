---
title: Test FaaS Functions with createFunctionApp
impact: MEDIUM-HIGH
impactDescription: "Correct HTTP + non-HTTP trigger testing patterns"
tags: testing, serverless, faas, mock, createFunctionApp
---

## Test FaaS Functions with createFunctionApp

Use `createFunctionApp<Framework>()` from `@midwayjs/mock` (the FaaS-specific variant of `createApp`) to boot a function app for tests. HTTP/API-Gateway triggers are tested with `createHttpRequest(app)` (supertest-style). Non-HTTP triggers (Timer/OS/MQ/Event) **cannot** be triggered via `npm run dev` — test them by getting the instance via `app.getServerlessInstance(Class)` and calling the method directly with mocked events from `@midwayjs/fc-starter`. Always pair with `close(app)` for teardown.

**Incorrect (mocking process.argv, only testing happy-path HTTP):**

```typescript
// ❌ mocking argv instead of booting the real function app
process.argv = ['node', 'bootstrap', '--trigger', 'timer'];
// ❌ non-HTTP triggers never tested at all
```

**Correct (createFunctionApp + HTTP supertest + non-HTTP instance calls + mock events):**

```typescript
import { createFunctionApp, close, createHttpRequest } from '@midwayjs/mock';
import { Framework, Application } from '@midwayjs/faas';
import { mockContext, mockTimerEvent, mockOSSEvent } from '@midwayjs/fc-starter';
import { HelloService } from '../src/function/hello';

describe('FaaS function', () => {
  let app: Application;

  beforeAll(async () => {
    app = await createFunctionApp<Framework>(join(__dirname, '../'), {
      initContext: mockContext(),   // simulate the Aliyun FC Context struct
    });
  });
  afterAll(async () => { await close(app); });

  // HTTP trigger — real HTTP request
  it('GET / returns greeting', async () => {
    const res = await createHttpRequest(app).get('/').query({ name: 'zhang' });
    expect(res.text).toEqual('hello zhang');
  });

  // Timer trigger — get instance + call with mocked event
  it('handles timer event', async () => {
    const instance = await app.getServerlessInstance<HelloService>(HelloService);
    await instance.handleTimer(mockTimerEvent());
    // assert side effects
  });

  // OSS trigger — mocked event
  it('handles OSS event', async () => {
    const instance = await app.getServerlessInstance<HelloService>(HelloService);
    await instance.handleOSS(mockOSSEvent());
  });

  // override mock context fields
  it('with custom function metadata', async () => {
    const customApp = await createFunctionApp<Framework>(join(__dirname, '../'), {
      initContext: Object.assign(mockContext(), { function: { name: 'myFn', handler: 'x.y' } }),
    });
    await close(customApp);
  });
});
```

Reference: [Midway Serverless Testing](https://midwayjs.org/docs/serverless/serverless_testing)
