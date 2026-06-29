---
title: Proxy Requests with @midwayjs/http-proxy
impact: LOW-MEDIUM
impactDescription: "Reverse proxy for external/static resources"
tags: api, http-proxy, reverse-proxy, gateway
---

## Proxy Requests with @midwayjs/http-proxy

`@midwayjs/http-proxy` is a purely config-driven reverse proxy. Use `match` (regex) to select URLs, and either `host` (swap host, keep path) or `target` (rewrite path via `$1` capture groups). Define multiple strategies under `httpProxy.strategy`. Useful for proxying CDN assets, microservice gateways, or third-party APIs without writing controller code.

**Incorrect (manual proxying in a controller with the HTTP client):**

```typescript
@Get('/tfs/:file')
async proxy(@Param('file') file: string) {
  const res = await this.httpService.get(`https://gw.alicdn.com/tfs/${file}`);  // ❌ manual, no streaming
  return res.data;   // ❌ buffers entire body in memory
}
```

**Correct (config-driven proxy with path rewriting strategies):**

```typescript
import * as httpProxy from '@midwayjs/http-proxy';
@Configuration({ imports: [koa, httpProxy] })

// config.default.ts
export default {
  httpProxy: {
    default: { proxyTimeout: 10000 },     // shared, merged into each strategy
    strategy: {
      // host swap: keeps the matched path
      cdn: { match: /\/tfs\//, host: 'https://gw.alicdn.com' },
      // target rewrite: uses regex capture groups ($1)
      baidu: { match: /\/bdimg\/(.*)$/, target: 'https://sm.bdimg.com/$1' },
      // API gateway proxy
      api: { match: /\/external-api\/(.*)$/, target: 'https://api.example.com/$1', ignoreHeaders: { cookie: true } },
    },
  },
} as MidwayConfig;
// /tfs/logo.png → https://gw.alicdn.com/tfs/logo.png
// /bdimg/x.js   → https://sm.bdimg.com/x.js
```

Supported: koa ✅, web(egg) ✅, express ✅, faas 💬 (some platforms limit streaming). Use `host` OR `target`, not both on one strategy.

Reference: [Midway HTTP Proxy](https://midwayjs.org/docs/extensions/http-proxy)
