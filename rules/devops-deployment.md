---
title: Implement Graceful Shutdown and Deployment Correctly
impact: MEDIUM-HIGH
impactDescription: "Zero-downtime deployments and clean resource release"
tags: devops, deployment, shutdown, docker, lifecycle
---

## Implement Graceful Shutdown and Deployment Correctly

`@midwayjs/bootstrap` enables graceful shutdown by handling `SIGTERM`/`SIGINT`: it stops accepting new requests, runs `onStop`/`@Destroy` hooks (close DB pools, queues, sockets), then exits. Implement `onStop` in your `@Configuration` and `@Destroy` on resource-holding services. For deployment, build with `mwtsc`, prune dev deps, and ship `dist/` + `bootstrap.js` + `package.json` + `node_modules`. Use multi-stage Docker builds. Enable shutdown hooks so container orchestrators can drain traffic.

**Incorrect (ignoring shutdown signals, abrupt exits, missing cleanup):**

```typescript
// bootstrap.js — no graceful handling
const { Bootstrap } = require('@midwayjs/bootstrap');
Bootstrap.run();   // ❌ SIGTERM kills instantly; in-flight requests fail; DB not closed

@Provide()
export class DatabaseService {
  @Init() async init() { this.pool = mysql.createPool(config); }
  // ❌ no @Destroy → pool leaks connections on shutdown
}
```

**Correct (lifecycle cleanup + multi-stage Docker + bootstrap):**

```typescript
// configuration.ts — implement onStop for graceful cleanup
@Configuration({})
export class MainConfiguration implements ILifeCycle {
  async onStop(container: IMidwayContainer, app: IMidwayApplication) {
    // framework waits for this before exiting (v4: core.stopTimeout)
    const queueFramework = await container.getAsync(bullmq.Framework);
    await queueFramework.getQueue('reports')?.close();
  }
}

// service — release resources on destroy
@Provide()
export class DatabaseService {
  @InjectDataSource() dataSource: DataSource;

  @Destroy()
  async destroy() {
    await this.dataSource.destroy();   // ✓ closes the pool
  }
}

// bootstrap.js — entry point (production)
const { Bootstrap } = require('@midwayjs/bootstrap');
Bootstrap.run();

// package.json scripts
// "build": "mwtsc --cleanOutDir",
// "start": "NODE_ENV=production node ./bootstrap.js"
```

```dockerfile
# Multi-stage Dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package.json pnpm-lock.yaml ./
RUN npm ci
COPY . .
RUN npm run build            # mwtsc → dist/

FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/src ./src          # source maps
COPY --from=builder /app/bootstrap.js ./
COPY --from=builder /app/package.json ./
RUN npm install --production && npm prune --production
EXPOSE 7001
CMD ["node", "./bootstrap.js"]
```

v4 dev mode uses `mwtsc --watch --run @midwayjs/mock/app.js` (single process, no `bootstrap.js` needed). Production baseDir is `dist` (not `src`).

Reference: [Midway Deployment](https://midwayjs.org/docs/deployment), [Lifecycle](https://midwayjs.org/docs/lifecycle)
