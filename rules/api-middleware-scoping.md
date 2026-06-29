---
title: Apply Middleware at the Right Level with Match/Ignore
impact: MEDIUM-HIGH
impactDescription: "Correct scoping prevents auth/logging bypass and perf waste"
tags: api, middleware, onion-model, match, ignore
---

## Apply Middleware at the Right Level with Match/Ignore

Midway middleware uses the **onion model** (Koa/Egg) — code runs both before AND after the controller, and `await next()` returns the downstream result. Apply middleware at three levels: route (`@Get('/', { middleware: [...] })`), controller (`@Controller('/', { middleware: [...] })`), and global (`app.useMiddleware(...)`). Use `match(ctx)` or `ignore(ctx)` (only one takes effect; strings/regex/arrays/functions) to scope execution — essential for auth/login routes. Order global middleware with `app.getMiddleware().insertBefore/After`.

**Incorrect (auth middleware on every route including login, manual next handling):**

```typescript
async onReady(_, app) {
  app.useMiddleware(JwtMiddleware);   // ❌ blocks /login, /register, /captcha
}

@Middleware()
export class JwtMiddleware implements IMiddleware<Context, NextFunction> {
  resolve() {
    return async (ctx, next) => {
      const token = verifyToken(ctx);   // ❌ throws on public routes
      ctx.user = token;
      next();                            // ❌ result of next() discarded, breaks onion
    };
  }
}
```

**Correct (ignore public routes, propagate result, ordered global registration):**

```typescript
@Middleware()
export class JwtMiddleware implements IMiddleware<Context, NextFunction> {
  // skip auth on public routes (only one of match/ignore is used)
  ignore(ctx: Context) {
    return /\/login|\/register|\/captcha|\/health/.test(ctx.path);
  }

  resolve() {
    return async (ctx: Context, next: NextFunction) => {
      const token = extractToken(ctx);
      if (token) ctx.user = verifyToken(token);
      const result = await next();     // ✓ await + return preserves onion model
      return result;
    };
  }
  static getName(): string { return 'jwt'; }
}

// configuration.ts — order matters: cors before auth
async onReady(_, app) {
  app.getMiddleware().insertFirst(CorsMiddleware);
  app.useMiddleware(JwtMiddleware);
  // or: app.getMiddleware().insertBefore(JwtMiddleware, 'cors');
}

// reuse a middleware with different options
app.useMiddleware(createMiddleware(ReportMiddleware, { text: 'x' }, 'report2'));
```

Note: returning `null` from a Koa handler sets status **204**. To return 200, set `ctx.status = 200` explicitly.

Reference: [Midway Middleware](https://midwayjs.org/docs/middleware)
