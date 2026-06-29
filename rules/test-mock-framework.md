---
title: Use @midwayjs/mock for App Bootstrap Testing
impact: HIGH
impactDescription: "Proper DI-isolated test environments"
tags: testing, mock, createapp, bootstrap, jest
---

## Use @midwayjs/mock for App Bootstrap Testing

Use `@midwayjs/mock` to create isolated test apps with real DI wiring. `createApp<Framework>()` boots the full app (matching production), `createLightApp` boots a minimal container, and `close(app)` tears it down (auto-restoring all mocks). Get services from the container with `getApplicationContext().getAsync(Class)`. For request-scoped services that read `ctx`, resolve them via `app.createAnonymousContext().requestContext.getAsync(Class)`. v4 dropped the trailing array param — pass extra components via `options.imports`.

**Incorrect (manual instantiation bypassing DI, no teardown):**

```typescript
describe('UserService', () => {
  it('creates user', async () => {
    const service = new UserService(new UserRepository()); // ❌ bypasses DI, hits real DB
    const user = await service.create({ name: 'x' });
    // ❌ no app teardown, no isolation, mocks never restored
  });
});
```

**Correct (createApp + container resolution + mock groups + teardown):**

```typescript
import { createApp, close } from '@midwayjs/mock';
import { Framework, Application } from '@midwayjs/koa';
import { UserService } from '../src/service/user.service';

describe('UserService', () => {
  let app: Application;

  beforeAll(async () => {
    // boots the real app (reads configuration.ts); v4 extra components via options.imports
    app = await createApp<Framework>(process.cwd(), { imports: [] });
  });

  afterAll(async () => {
    await close(app, { sleep: 50 });   // closes app + restores all mocks
  });

  it('finds user by id', async () => {
    // resolve from container — full DI graph
    const userService = await app
      .getApplicationContext()
      .getAsync<UserService>(UserService);

    // for a Request-scoped service that needs ctx:
    // const svc = await app.createAnonymousContext().requestContext.getAsync(UserService);

    const user = await userService.findById(1);
    expect(user).toBeDefined();
  });
});

// Light app for fast component-only tests
import { createLightApp } from '@midwayjs/mock';
it('tests a component in isolation', async () => {
  const app = await createLightApp('', { imports: [myComponent] });
  const svc = await app.getApplicationContext().getAsync(MyService);
  await close(app);
});
```

Reference: [Midway Testing](https://midwayjs.org/docs/testing), [Mock](https://midwayjs.org/docs/mock)
