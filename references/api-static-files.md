---
title: Serve Static Files with @midwayjs/static-file
impact: MEDIUM
impactDescription: "Config-driven static asset hosting"
tags: api, static-file, assets, koa-static-cache
---

## Serve Static Files with @midwayjs/static-file

`@midwayjs/static-file` (built on `koa-static-cache`) serves static assets via config — no decorators. Define multiple directories with `prefix`/`dir` pairs. For production, enable `buffer` and set `maxAge` for caching. Note: it does not support `index.html` natively — use `alias: { '/': '/index.html' }`. In egg (`@midwayjs/web`), disable the built-in static plugin to avoid conflicts.

**Incorrect (manual file reading in controllers):**

```typescript
@Get('/logo.png')
async logo() {
  const buf = await readFile(join(__dirname, 'public/logo.png'));  // ❌ manual, no caching, no streaming
  this.ctx.type = 'png';
  return buf;
}
```

**Correct (config-driven multi-directory static serving):**

```typescript
import * as staticFile from '@midwayjs/static-file';
@Configuration({ imports: [koa, staticFile] })

// config.default.ts
export default {
  staticFile: {
    dirs: {
      default: { prefix: '/public', dir: join(appInfo.appDir, 'public') },
      uploads: { prefix: '/uploads', dir: join(appInfo.appDir, 'uploads') },
    },
    dynamic: true,          // load files dynamically (dev)
    preload: false,
    // production-only tuning:
    // maxAge: 31536000,    // 1 year cache
    // buffer: true,        // buffer in memory
    // alias: { '/': '/index.html' },   // serve index.html at root
  },
} as MidwayConfig;
```

Supported frameworks: koa ✅, web(egg) ✅, faas 💬, express ❌. For FaaS, register a wildcard route so the gateway maps the path.

Reference: [Midway Static File](https://midwayjs.org/docs/extensions/static_file)
