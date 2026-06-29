---
title: Enable Distributed Tracing with OpenTelemetry
impact: MEDIUM-HIGH
impactDescription: "v4 merged tracing into core — trace across services"
tags: devops, tracing, opentelemetry, observability, distributed
---

## Enable Distributed Tracing with OpenTelemetry

In v4, tracing merged into `@midwayjs/core` — the `@midwayjs/otel` package is **removed**. Enable via `tracing.enable: true` (global) + per-component switches. `ctx.traceId` is available on all frameworks. Use `@Trace(name)` on methods and inject `MidwayTraceService`. Initialize the OpenTelemetry SDK **before** `Bootstrap.run()` in `bootstrap.js`. Configure context propagation (`extractor`/`injector`) for cross-service trace linking.

**Incorrect (no tracing, or using the removed @midwayjs/otel):**

```typescript
// ❌ v4 removed package
import { Trace } from '@midwayjs/otel';   // module not found

// ❌ no correlation IDs across microservice calls — impossible to debug latency
```

**Correct (core tracing + OTel SDK init + per-component config + @Trace):**

```typescript
// config.default.ts
export default {
  tracing: {
    enable: true,            // global switch
    onError: 'ignore',       // or 'throw'
  },
  koa: { tracing: { enable: true } },        // component-level on
  kafka: { tracing: { enable: false } },     // component-level off
  // context propagation + custom attributes
  koa: {
    tracing: {
      meta: { common: ({ ctx }) => ({ 'biz.userId': ctx?.user?.id ?? 'anon' }) },
      extractor: ({ request }) => request?.headers || {},   // read upstream trace
    },
  },
} as MidwayConfig;

// @Trace decorator on methods (from core, not otel)
import { Trace, MidwayTraceService, Inject } from '@midwayjs/core';
@Provide()
export class UserService {
  @Trace('user.get')
  async getUser(id: string) { /* ... */ }
}

// bootstrap.js — init OTel SDK BEFORE Bootstrap.run()
const { NodeTracerProvider } = require('@opentelemetry/sdk-trace-node');
const { SimpleSpanProcessor, ConsoleSpanExporter } = require('@opentelemetry/sdk-trace-base');
const { JaegerExporter } = require('@opentelemetry/exporter-jaeger');
const { Bootstrap } = require('@midwayjs/bootstrap');

const provider = new NodeTracerProvider();
provider.addSpanProcessor(new SimpleSpanProcessor(
  new JaegerExporter({ host: process.env.JAEGER_HOST || '127.0.0.1', port: 6832 }),
));
provider.register();              // ✓ must run before Bootstrap
Bootstrap.configure().run();
```

`ctx.traceId` returns a 32-hex string. Default propagator is W3C (`traceparent`); override with B3 via `propagation.setGlobalPropagator()` in bootstrap. In `dev` mode spans don't print (dev doesn't run bootstrap.js).

Reference: [Midway Tracing](https://midwayjs.org/docs/tracing)
