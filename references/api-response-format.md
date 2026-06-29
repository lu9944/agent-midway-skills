---
title: Standardize Responses with HttpServerResponse
impact: MEDIUM
impactDescription: "Consistent success/fail envelope and SSE support"
tags: api, response, http-server-response, sse
---

## Standardize Responses with HttpServerResponse

For a uniform API envelope and advanced response types (SSE, file, stream), use `HttpServerResponse` (since v3.17.0). It provides chainable `.success()`/`.fail()` with overridable JSON/TEXT/BLOB templates, plus `.sse()` for AI streaming with upstream forwarding. Subclass it for per-scenario formats without changing global defaults. Avoid hand-rolling `{ code, msg, data }` shapes in every handler.

**Incorrect (hand-rolled, inconsistent response shapes):**

```typescript
@Get('/:id')
async findOne(@Param('id') id: number) {
  const user = await this.userService.findById(id);
  if (!user) {
    // ❌ inconsistent shape, manual status plumbing
    this.ctx.status = 404;
    return { code: 404, msg: 'not found', data: null };
  }
  return { code: 0, msg: 'ok', data: user };   // ❌ different envelope than above
}
```

**Correct (HttpServerResponse chain + custom subclass + SSE):**

```typescript
import { HttpServerResponse } from '@midwayjs/core';

@Get('/:id')
async findOne(@Param('id') id: number) {
  const user = await this.userService.findById(id);
  if (!user) {
    // fail() → { success: 'false', message: {} }
    return new HttpServerResponse(this.ctx).fail().json({ id, reason: 'not found' });
  }
  // success() → { success: 'true', data: {} }; .json() must be called last
  return new HttpServerResponse(this.ctx).success().json(user);
}

// override the global templates via static properties (success/fail are on the base ServerResponse class)
HttpServerResponse.JSON_TPL = (data, isSuccess, ctx) =>
  JSON.stringify({ code: isSuccess ? 0 : 1, data, message: isSuccess ? 'ok' : data });

// per-scenario custom format: override the static *_TPL properties on a subclass
// (do NOT look for a getJsonTpl() method — it does not exist; use JSON_TPL/TEXT_TPL/BLOB_TPL)
class ApiGatewayResponse extends HttpServerResponse {
  static JSON_TPL = (data, isSuccess, ctx) =>
    JSON.stringify({ ok: isSuccess, payload: data });
}

// SSE with upstream AI forwarding
@Get('/chat')
async chat(@Query('q') q: string) {
  const sse = new HttpServerResponse(this.ctx).sse();
  await sse.forward(upstreamStream, { protocol: 'openai' });  // forwards OpenAI chunks
  sse.sendEnd();
}
```

Reference: [Midway Data Response](https://midwayjs.org/docs/data_response)
