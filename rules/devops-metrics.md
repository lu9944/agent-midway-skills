---
title: Expose Metrics with @midwayjs/prometheus
impact: MEDIUM
impactDescription: "Auto-exposed /metrics for Prometheus scraping"
tags: devops, prometheus, metrics, monitoring, observability
---

## Expose Metrics with @midwayjs/prometheus

`@midwayjs/prometheus` auto-exposes a `/metrics` endpoint (requires an HTTP framework) with built-in metrics: per-path QPS, status-code distribution, response time, process CPU/memory, heap, and event loop. Configure `prometheus.labels` for grouping (e.g. `APP_NAME`). For Socket.io metrics, add `@midwayjs/prometheus-socket-io`. Scrape with Prometheus + visualize in Grafana.

**Incorrect (no metrics, manual instrumentation):**

```typescript
// ❌ no observability — blind to latency spikes and error rates in production
@Configuration({ imports: [koa] })
```

**Correct (auto-metrics + labels + Grafana scrape):**

```typescript
import * as prometheus from '@midwayjs/prometheus';
@Configuration({ imports: [koa, prometheus] })

// config.default.ts
export default {
  prometheus: {
    labels: { APP_NAME: 'order-service', env: process.env.NODE_ENV },
  },
} as MidwayConfig;
// /metrics endpoint now exposes: http_request_duration, http_requests_total,
// nodejs_heap_size, process_cpu_usage, event_loop_lag, etc.
```

```yaml
# prometheus.yml scrape config
scrape_configs:
  - job_name: 'midway'
    static_configs:
      - targets: ['app:7001']
```

Reference: [Midway Prometheus](https://midwayjs.org/docs/extensions/prometheus)
