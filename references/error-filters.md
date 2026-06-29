---
title: Use @Catch Error Filters for Centralized Handling
impact: HIGH
impactDescription: "Consistent, centralized error responses across the app"
tags: error-handling, filter, catch, centralized
---

## Use @Catch Error Filters for Centralized Handling

Never catch-and-format errors manually in every controller. Midway error filters (`@Catch` classes in `src/filter/*.filter.ts`) run **after** middleware and catch all business + framework errors, producing uniform responses. Register specific filters for specific error types, plus exactly **one** generic `@Catch()` filter (no argument) as the catch-all that always runs last. Throw errors from services so filters can map them to HTTP responses; never set status codes silently.

**Incorrect (manual try/catch + manual response shaping in controllers):**

```typescript
@Controller('/user')
export class UserController {
  @Inject() userService: UserService;

  @Get('/:id')
  async findOne(@Param('id') id: number, @Ctx() ctx: Context) {
    try {
      const user = await this.userService.findById(id);
      return ctx.body = user;
    } catch (e) {
      // ❌ manual error formatting repeated in every handler
      ctx.status = 500;
      ctx.body = { message: e.message };
    }
  }
}
```

**Correct (throw from service, handle centrally with @Catch filters):**

```typescript
// src/filter/default.filter.ts
import { Catch } from '@midwayjs/core';
import { Context } from '@midwayjs/koa';

@Catch()   // no arg = catch ALL unhandled errors (only one per app, always last)
export class DefaultErrorFilter {
  async catch(err: Error, ctx: Context) {
    // log manually — errors caught by a custom filter are NOT auto-logged
    ctx.logger.error(err);
    return { status: (err as any).status ?? 500, message: err.message };
  }
}

// src/filter/notfound.filter.ts
import { Catch, httpError, MidwayHttpError } from '@midwayjs/core';

@Catch(httpError.NotFoundError)   // specific error type
export class NotFoundFilter {
  async catch(err: MidwayHttpError, ctx: Context) {
    return { code: 404, message: err.message };
  }
}

// src/configuration.ts — register filters in onReady
async onReady(container, app) {
  app.useFilter([NotFoundFilter, DefaultErrorFilter]);   // order irrelevant: specific first, generic last
}
```

Key behaviors: specific filters match before the generic one; the generic `@Catch()` filter must be unique. To match a base class and all subclasses, use `{ matchPrototype: true }`: `@Catch([MidwayError], { matchPrototype: true })`.

Reference: [Midway Error Filters](https://midwayjs.org/docs/error_filter)
