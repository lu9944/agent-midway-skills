---
title: Configure CORS and Security Headers
impact: HIGH
impactDescription: "Prevents cross-origin and XSS/CSRF attacks"
tags: security, cors, csrf, helmet, xss, cross-domain
---

## Configure CORS and Security Headers

Use `@midwayjs/cross-domain` for CORS/JSONP and `@midwayjs/security` for CSRF, CSP, HSTS, and XSS escaping. Both are config-driven — enable them in `@Configuration({ imports })` and tune via their config keys. When `credentials: true`, `origin` cannot be `*`. Rotate the CSRF secret on login via `ctx.rotateCsrfSecret()`.

**Incorrect (no CORS, no security headers, reflecting raw input):**

```typescript
// ❌ no cors/security component — browsers block cross-origin, no CSRF/XSS protection
@Configuration({ imports: [koa] })
export class MainConfiguration {}

@Controller('/')
export class HomeController {
  @Get('/')
  async echo(@Query() q) { return q; }   // ❌ reflected XSS if rendered as HTML
}
```

**Correct (config-driven CORS + CSRF + headers + escape):**

```typescript
import * as crossDomain from '@midwayjs/cross-domain';
import * as security from '@midwayjs/security';

@Configuration({ imports: [koa, crossDomain, security] })
export class MainConfiguration {}

// config.default.ts
export default {
  // CORS — origin callback for dynamic allow-list
  cors: {
    origin: (ctx) => (allowedOrigins.includes(ctx.headers.origin) ? ctx.headers.origin : ''),
    allowMethods: ['GET', 'POST', 'PUT', 'DELETE'],
    credentials: true,          // with cookies; origin must NOT be '*' when true
    maxAge: 600,
  },
  // Security headers + CSRF
  security: {
    csrf: { enable: true, type: 'ctoken', headerName: 'x-csrf-token' },
    xframe: { enable: true, value: 'SAMEORIGIN' },
    hsts: { enable: true, maxAge: 365 * 24 * 3600, includeSubdomains: true },
    nosniff: { enable: true },
    csp: { enable: true, policy: { 'default-src': ["'self'"] } },
  },
} as MidwayConfig;

// rotate CSRF secret on login; escape reflected output
@Controller('/auth')
export class AuthController {
  @Inject() ctx: Context;

  @Post('/login')
  async login() {
    this.ctx.session = { user };
    this.ctx.rotateCsrfSecret();   // ✓ prevent token reuse across sessions
    return { success: true };
  }

  @Get('/search')
  async search(@Query('q') q: string) {
    // ✓ HTML-escape user input before returning/rendering
    return { safe: this.ctx.security.escape(q) };
  }
}
```

Reference: [Midway Cross-Domain](https://midwayjs.org/docs/extensions/cross_domain), [Midway Security](https://midwayjs.org/docs/extensions/security)
