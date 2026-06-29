---
title: Use Passport for OAuth and Strategy-Based Authentication
impact: HIGH
impactDescription: "Standardized third-party login (GitHub, Google, JWT, local)"
tags: security, passport, oauth, authentication, jwt
---

## Use Passport for OAuth and Strategy-Based Authentication

`@midwayjs/passport` (self-maintained since v3.4.0 — no need for the community `passport` package) standardizes authentication via strategies. Each strategy is a `@CustomStrategy()` class extending `PassportStrategy(Strategy, name?)`, implementing a Promise-based `validate()` (no `done` callback) and `getStrategyOptions()`. Apply strategies via a `PassportMiddleware(StrategyClass)` on routes. The authenticated user lands on `ctx.state.user`.

**Incorrect (hand-rolled OAuth, community passport package, done callback):**

```typescript
// ❌ community passport package (not needed in Midway)
import passport from 'passport';
import { Strategy as GitHubStrategy } from 'passport-github';

passport.use(new GitHubStrategy({ ... }, (accessToken, profile, done) => {
  done(null, profile);   // ❌ Midway uses async validate(), not done()
}));
```

**Correct (Midway passport: JWT strategy + middleware + route binding):**

```typescript
import * as passport from '@midwayjs/passport';
import * as jwt from '@midwayjs/jwt';
@Configuration({ imports: [jwt, passport] })

// config.default.ts
export default { jwt: { secret: process.env.JWT_SECRET }, passport: { session: false } } as MidwayConfig;

// src/strategy/jwt.strategy.ts
import { CustomStrategy, PassportStrategy } from '@midwayjs/passport';
import { Strategy, ExtractJwt } from 'passport-jwt';
import { Config } from '@midwayjs/core';

@CustomStrategy()
export class JwtStrategy extends PassportStrategy(Strategy, 'jwt') {
  @Config('jwt') jwtConfig;

  async validate(payload: any) {        // Promise-based, no done()
    return payload;                      // → becomes ctx.state.user
  }

  getStrategyOptions() {
    return { secretOrKey: this.jwtConfig.secret, jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken() };
  }
}

// src/middleware/jwt-passport.middleware.ts
import { PassportMiddleware } from '@midwayjs/passport';
@Middleware()
export class JwtPassportMiddleware extends PassportMiddleware(JwtStrategy) {
  getAuthenticateOptions() { return {}; }
}

// route
@Controller('/user')
export class UserController {
  @Inject() ctx: Context;

  @Post('/me', { middleware: [JwtPassportMiddleware] })
  async me() { return this.ctx.state.user; }   // authenticated user
}
```

For OAuth providers (GitHub/Google), swap the `Strategy` import and implement `validate(accessToken, refreshToken, profile)`. Disable session serialization for JWT (`passport.session = false`) to avoid "Failed to serialize user into session".

Reference: [Midway Passport](https://midwayjs.org/docs/extensions/passport)
