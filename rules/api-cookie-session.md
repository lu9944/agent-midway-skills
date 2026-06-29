---
title: Manage Cookies and Sessions Correctly
impact: MEDIUM
impactDescription: "Signed vs encrypted cookies, session config, rotation"
tags: api, cookie, session, security, koa
---

## Manage Cookies and Sessions Correctly

In `@midwayjs/koa`, cookies are signed by default (`signed: true`) but **not** encrypted — the browser sees plaintext. Use `encrypt: true` for hidden values, `httpOnly: false` for JS-readable cookies. Read with matching options (signed↔signed, encrypt↔encrypt). Sessions are stored encrypted in a cookie by default; for "remember me", set `ctx.session.maxAge`. Rotate cookie keys by prepending new keys to the `keys` array.

**Incorrect (plaintext sensitive cookies, wrong read options, session field naming):**

```typescript
// ❌ sensitive data in a plaintext cookie
ctx.cookies.set('role', 'admin', { signed: false, httpOnly: false });

// ❌ reading a signed cookie without signed option → garbled value
const role = ctx.cookies.get('role');   // wrong: set had signed:true

// ❌ session fields starting with _ → lost next request
ctx.session._internal = 'data';
```

**Correct (signed/encrypted cookies + session config + key rotation):**

```typescript
// config.default.ts
export default {
  keys: ['newKey', 'oldKey'],     // first signs/encrypts; all tried for verify; rotate by prepending
  session: {
    maxAge: 24 * 3600 * 1000,     // 1 day
    key: 'MW_SESS',
    httpOnly: true,
    renew: true,                   // extend when half-expired
  },
} as MidwayConfig;

@Controller('/')
export class HomeController {
  @Inject() ctx: Context;

  @Get('/login')
  async login() {
    // encrypted + httpOnly → fully hidden from client & JS
    this.ctx.cookies.set('token', jwtToken, { encrypt: true, httpOnly: true });

    // session — "remember me" extends expiry
    this.ctx.session.userId = user.id;
    if (rememberMe) {
      this.ctx.session.maxAge = FORMAT.MS.ONE_DAY * 30;   // import FORMAT from @midwayjs/core
    }
    // rotate CSRF secret after login
    this.ctx.rotateCsrfSecret();
    return { success: true };
  }

  @Get('/profile')
  async profile() {
    // read with matching options
    const token = this.ctx.cookies.get('token', { encrypt: true });
    return { userId: this.ctx.session.userId };
  }

  @Get('/logout')
  async logout() {
    this.ctx.session = null;   // destroy session
    return { success: true };
  }
}
```

For FaaS, manually add `@midwayjs/session` (v4). For Redis-backed sessions, use the framework's session store package.

Reference: [Midway Cookie & Session](https://midwayjs.org/docs/cookie_session)
