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

Reference: [Midway PM2](https://midwayjs.org/docs/extensions/pm2), [cfork](https://midwayjs.org/docs/extensions/cfork)
