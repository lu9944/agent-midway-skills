---
title: Use @midwayjs/cron for Local Scheduled Tasks
impact: MEDIUM
impactDescription: "Simple per-process cron jobs without Redis"
tags: microservices, cron, schedule, task
---

## Use @midwayjs/cron for Local Scheduled Tasks

For simple scheduled jobs that should run on **every process**, use `@midwayjs/cron` with the `@Job` decorator and `IJob` interface. Use the `FORMAT.CRONTAB.*` constants from `@midwayjs/core` for common schedules. Control execution with `@InjectJob(Class)` (`start()`/`stop()`). Note: cron jobs are **local** — every machine/process runs them. For cluster-wide single execution, use BullMQ's `repeat` option instead.

**Incorrect (setInterval without error handling or cleanup):**

```typescript
@Provide()
export class CleanupService {
  @Init()
  init() {
    // ❌ no error handling, no cleanup, drifts over time
    setInterval(async () => {
      await this.cleanupOldRecords();
    }, 60 * 1000);
  }
}
```

**Correct (declarative @Job with framework-managed lifecycle):**

```typescript
// configuration.ts
import * as cron from '@midwayjs/cron';
@Configuration({ imports: [cron] })

// src/job/cleanup.job.ts
import { Job, IJob, InjectJob } from '@midwayjs/cron';
import { FORMAT, ILogger, Logger } from '@midwayjs/core';

@Job({
  cronTime: FORMAT.CRONTAB.EVERY_PER_30_MINUTE,   // predefined constant
  start: true,                                      // auto-start on server ready
})
export class CleanupJob implements IJob {
  @Logger() logger: ILogger;

  async onTick() {
    try {
      await this.cleanupOldRecords();
      this.logger.info('cleanup done');
    } catch (err) {
      this.logger.error('cleanup failed', err);
    }
  }

  async onComplete() {
    this.logger.info('cleanup tick completed');
  }
}

// control a job at runtime
@Provide()
export class JobController {
  @InjectJob(CleanupJob) cleanupJob: cron.CronJob;   // or @InjectJob('cleanupJob')

  async pause() { await this.cleanupJob.stop(); }
  async resume() { await this.cleanupJob.start(); }
}
```

`@Job` options: `cronTime` (cron string or `Date`), `start` (auto-start), `runOnInit`. Global defaults via `cron.defaultCronJobOptions`.

Reference: [Midway Cron](https://midwayjs.org/docs/extensions/cron)
