---
title: Version APIs for Backward Compatibility
impact: MEDIUM
impactDescription: "Evolve APIs without breaking existing clients"
tags: api, versioning, backward-compat, rest
---

## Version APIs for Backward Compatibility

Midway supports route versioning (URI/HEADER/MEDIA_TYPE/CUSTOM) so old clients keep working while new clients use updated endpoints. Enable it under the framework config key (`koa.versioning`), then set `version` on `@Controller` or individual routes. Use simple numeric versions (`'1'`, `'2'`), never semver strings. Access the request version via `ctx.apiVersion`. Not supported in Serverless.

**Incorrect (breaking changes to existing routes, manual v1/v2 path prefixes):**

```typescript
// ❌ breaking change to /users/:id response shape — old clients break
@Controller('/users')
export class UserController {
  @Get('/:id')
  async findOne() { return { firstName, lastName }; }   // was { name }
}

// ❌ manual path-based versioning — error-prone
@Controller('/v1/users') // ...
@Controller('/v2/users') // ...
```

**Correct (URI versioning + declarative @Controller version):**

```typescript
// config.default.ts
export default {
  koa: {
    versioning: {
      enabled: true,
      type: 'URI',            // /v1/users, /v2/users (recommended — cache-friendly)
      prefix: 'v',
      defaultVersion: '1',
      // alternatives:
      // type: 'HEADER', header: 'x-api-version',
      // type: 'MEDIA_TYPE', mediaTypeParam: 'version',  // Accept: application/json;version=1
      // type: 'CUSTOM', extractVersionFn: (ctx) => ctx.query.version,
    },
  },
} as MidwayConfig;

// v1 controller — old response shape
@Controller('/users', { version: '1' })
export class UserV1Controller {
  @Get('/:id')
  async findOne(@Param('id') id: string) {
    const u = await this.userService.findById(id);
    return { name: `${u.firstName} ${u.lastName}` };
  }
}

// v2 controller — new response shape (same path, different version)
@Controller('/users', { version: '2' })
export class UserV2Controller {
  @Get('/:id')
  async findOne(@Param('id') id: string) {
    const u = await this.userService.findById(id);
    return { firstName: u.firstName, lastName: u.lastName };
  }
}
```

For deprecation, set response headers (`X-API-Deprecated`, `X-API-Sunset-Date`).

Reference: [Midway Versioning](https://midwayjs.org/docs/versioning)
