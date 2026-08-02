---
title: Manage Processes for Production (PM2/cfork)
impact: MEDIUM
impactDescription: "Cluster-mode deployment and zero-downtime restarts"
tags: devops, pm2, cfork, cluster, deployment, process-management
---

## Manage Processes for Production (PM2/cfork)

For cluster-mode production deployment, use `pm2` (CLI) or `cfork` (API-driven). Build with `mwtsc` first, then run `bootstrap.js`. **In Docker, use `pm2-runtime`** (foreground) — not `pm2` (which backgrounds and exits the container). cfork is useful where global installs aren't allowed; it forks `bootstrap.js` per CPU and auto-respawns workers. Always set `NODE_ENV=production`.

**Incorrect (single process, or pm2 in a container):**

```bash
# ❌ single process — no multi-core utilization
NODE_ENV=production node ./bootstrap.js

# ❌ pm2 in Docker — daemonizes, container exits immediately
pm2 start ./bootstrap.js --name app -i 4
```

**Correct (PM2 cluster + pm2-runtime for Docker + cfork alternative):**

```bash
# PM2 cluster mode (bare metal / VM)
npm run build                              # mwtsc → dist/
NODE_ENV=production pm2 start ./bootstrap.js --name midway_app -i 4   # 4 cluster workers
pm2 list                                   # verify
pm2 restart midway_app                     # zero-downtime reload
pm2 logs midway_app
```

```dockerfile
# Docker — pm2-runtime stays in foreground
CMD ["pm2-runtime", "start", "./bootstrap.js", "--name", "midway_app", "-i", "4"]
```

```javascript
// cfork alternative (API-driven, no global install) — server.js
const cfork = require('cfork');
const os = require('os');
cfork({ exec: path.join(__dirname, './bootstrap.js'), count: os.cpus().length })
  .on('fork', w => console.log(`[worker:${w.process.pid}] start`))
  .on('disconnect', w => console.log(`worker ${w.process.pid} disconnect`))
  .on('exit', (w, code, signal) => { throw new Error(`worker ${w.process.pid} died`); });
// run: NODE_ENV=production node server.js
```

Reference: [Midway PM2](https://midwayjs.org/docs/extensions/pm2), [cfork](https://midwayjs.org/docs/extensions/cfork), [process-agent](https://midwayjs.org/docs/extensions/process_agent)

**Multi-process consistency (`@midwayjs/process-agent`):** in cluster/multi-process deployments (PM2, cfork, egg-script), in-memory state is per-process — a request can land on a different worker than the one that wrote the state (cache misses, inconsistent `/metrics`, health checks). Add the component and mark methods that must run in the master process with `@RunInPrimary()`:

> **⚠️ Note:** `@midwayjs/process-agent` v4 is still in alpha on npm (latest published tag: `3.20.24`; the v4-next repo tracks `4.0.0-alpha.1`+). Verify your installed version supports v4 before relying on it; on v3, the package works with v3 apps.

```typescript
// configuration.ts
import * as processAgent from '@midwayjs/process-agent';
@Configuration({ imports: [processAgent] })

// src/service/state.service.ts — only the master executes setData/getData
import { Provide, Scope, ScopeEnum } from '@midwayjs/core';
import { RunInPrimary } from '@midwayjs/process-agent';

@Provide()
@Scope(ScopeEnum.Singleton)
export class StateService {
  data: any = 0;

  @RunInPrimary()
  async setData(b) { this.data = b; return this.data; }

  @RunInPrimary()
  async getData() { return this.data; }
}
```

Note: return values must be JSON-serializable (no methods/non-serializable objects) since they cross process boundaries. Same idea applies to per-process caches (see `perf-caching.md`) — choose Redis when the state must be shared, `@RunInPrimary` when single-owner state in the master suffices.
