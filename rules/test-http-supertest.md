---
title: Use Supertest for HTTP E2E Testing
impact: HIGH
impactDescription: "Validates the full request/response cycle"
tags: testing, e2e, supertest, http, controller
---

## Use Supertest for HTTP E2E Testing

End-to-end tests use `createHttpRequest(app)` (wrapping supertest) to make real HTTP requests against the booted app — exercising routes, middleware, guards, pipes, and filters together. Always set up the app in `beforeAll`, tear it down in `afterAll`, and assert both status codes and response bodies. Use `mockClassProperty`/`mockContext` to stub external dependencies without hitting real services.

**Incorrect (only unit-testing controllers, no route/filter coverage):**

```typescript
describe('UserController', () => {
  it('returns users', async () => {
    const ctrl = new UserController({ findAll: async () => [] } as any);
    const result = await ctrl.findAll();
    expect(result).toEqual([]);
    // ❌ never tests routing, validation pipe, or error filter
  });
});
```

**Correct (real HTTP requests with supertest + dependency mocking):**

```typescript
import { createApp, close, createHttpRequest, mockClassProperty } from '@midwayjs/mock';
import { Framework, Application } from '@midwayjs/koa';
import { UserService } from '../src/service/user.service';

describe('UserController (e2e)', () => {
  let app: Application;

  beforeAll(async () => { app = await createApp<Framework>(); });
  afterAll(async () => { await close(app); });

  describe('POST /user', () => {
    it('creates a user (201)', async () => {
      const res = await createHttpRequest(app)
        .post('/user')
        .send({ name: 'John', email: 'john@test.com', age: 30 });
      expect(res.status).toBe(200);
      expect(res.body.data).toHaveProperty('id');
    });

    it('rejects invalid email (validation error)', async () => {
      const res = await createHttpRequest(app)
        .post('/user')
        .send({ name: 'John', email: 'not-an-email', age: 30 });
      expect(res.status).toBe(500);          // caught by error filter
      expect(res.body.message).toContain('email');
    });
  });

  it('mocks an external service instead of calling it', async () => {
    mockClassProperty(UserService, 'findById', async (id: number) => ({ id, name: 'mock' }));
    const res = await createHttpRequest(app).get('/user/1');
    expect(res.body.data.name).toBe('mock');
  });
});
```

Run serially with `jest --runInBand` to avoid port conflicts, and set `process.env.MIDWAY_TS_MODE='true'` in a `jest.setup.js` for ts-mode.

Reference: [Midway Testing](https://midwayjs.org/docs/testing)
