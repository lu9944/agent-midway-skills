---
title: Implement Secure JWT Authentication
impact: HIGH
impactDescription: "Essential for stateless API security"
tags: security, jwt, authentication, token
---

## Implement Secure JWT Authentication

Use the `@midwayjs/jwt` component for token sign/verify/decode. Keep secrets in config (never hardcoded), use short-lived access tokens, store the secret via `@Config('jwt.secret')`, and verify tokens in a middleware or guard that skips public routes via `match`/`ignore`. Never put sensitive data (passwords) in the JWT payload.

**Incorrect (hardcoded secret, sensitive payload, blocking every route):**

```typescript
// ❌ secret hardcoded in source
@Configuration({ imports: [jwt] })
export class MainConfiguration {}

// config
export default { jwt: { secret: 'my-secret-key', sign: { expiresIn: '7d' } } };

// ❌ password in the JWT payload
async login(user) {
  return { token: this.jwtService.signSync({ id: user.id, password: user.password }) };
}
```

**Correct (config-driven secret, minimal payload, route-aware middleware):**

```typescript
// config.default.ts — secret from environment, short TTL
export default {
  jwt: {
    secret: process.env.JWT_SECRET,           // never inline
    sign: { expiresIn: '2d' },
  },
} as MidwayConfig;

// src/middleware/jwt.middleware.ts
import { Middleware, IMiddleware, Config } from '@midwayjs/core';
import { Context, NextFunction } from '@midwayjs/koa';
import { httpError } from '@midwayjs/core';
import { JwtService } from '@midwayjs/jwt';

@Middleware()
export class JwtMiddleware implements IMiddleware<Context, NextFunction> {
  @Config('jwt') jwtConfig;
  @Inject() jwtService: JwtService;

  // skip auth on login/register routes
  ignore(ctx: Context): boolean {
    return /\/login|\/register|\/captcha/.test(ctx.path);
  }

  resolve() {
    return async (ctx: Context, next: NextFunction) => {
      const auth = ctx.headers.authorization || '';
      const token = auth.startsWith('Bearer ') ? auth.slice(7) : '';
      if (!token) throw new httpError.UnauthorizedError('missing token');
      try {
        // minimal payload: only sub + roles
        ctx.user = await this.jwtService.verify(token);
      } catch {
        throw new httpError.UnauthorizedError('invalid token');
      }
      return await next();
    };
  }
  static getName(): string { return 'jwt'; }
}

// src/configuration.ts
async onReady(_, app) { app.useMiddleware(JwtMiddleware); }
```

Reference: [Midway JWT](https://midwayjs.org/docs/extensions/jwt)
