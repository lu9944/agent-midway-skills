---
title: Use BullMQ for Reliable Background Job Processing
impact: MEDIUM-HIGH
impactDescription: "Reliable, retryable, distributed background jobs"
tags: microservices, bullmq, queue, background-jobs, redis
---

## Use BullMQ for Reliable Background Job Processing

For long-running or retryable background work (emails, reports, file processing), use `@midwayjs/bullmq` (replaces Bull since v3.20). It requires Redis and has its own framework. Define a processor with `@Processor(queueName)` implementing `IProcessor.execute()`, and add jobs via `bullmqFramework.getQueue(name).addJobToQueue()`. Configure retry/backoff in job options. Use cron-style `repeat` for scheduled jobs, and Bull Board for monitoring.

**Incorrect (blocking HTTP handlers, fire-and-forget without retry):**

```typescript
@Post('/report')
async generate(@Body() dto: GenerateReportDto) {
  // ❌ blocks the request for minutes
  const data = await this.fetchLargeDataset(dto);
  const report = await this.processData(data);
  return report;   // client times out
}

@Provide()
export class EmailService {
  async sendWelcome(email: string) {
    await this.mailer.send({ to: email });  // ❌ no retry, lost on failure
  }
}
```

**Correct (queue + processor with retry/backoff + progress):**

```typescript
// configuration.ts
import * as bullmq from '@midwayjs/bullmq';
@Configuration({ imports: [bullmq] })

// config.default.ts
export default {
  bullmq: {
    defaultConnection: { host: '127.0.0.1', port: 6379 },
    defaultPrefix: '{midway-bullmq}',
    defaultQueueOptions: {
      defaultJobOptions: { removeOnComplete: 100, removeOnFail: 5000 },
    },
  },
} as MidwayConfig;

// src/queue/report.processor.ts
import { Processor, IProcessor } from '@midwayjs/bullmq';
import { Inject } from '@midwayjs/core';
import { Context } from '@midwayjs/bullmq';

@Processor('reports')
export class ReportProcessor implements IProcessor {
  @Inject() ctx: Context;   // { jobId, job, token?, from }

  async execute(data: any) {
    await this.ctx.job.updateProgress(50);
    const report = await this.processData(data);
    await this.ctx.job.updateProgress(100);
    return report;
  }
}

// producer — add jobs with retry/backoff
@Provide()
export class ReportService {
  @Inject() bullmqFramework: bullmq.Framework;

  async requestReport(dto: GenerateReportDto) {
    const queue = this.bullmqFramework.getQueue('reports');
    const job = await queue?.addJobToQueue(dto, {
      attempts: 3,
      backoff: { type: 'exponential', delay: 1000 },
      removeOnComplete: true,
    });
    return { jobId: job?.id };
  }

  // scheduled recurring job via cron repeat
  async scheduleDailyDigest() {
    const queue = this.bullmqFramework.getQueue('reports');
    await queue?.addJobToQueue({}, {
      repeat: { pattern: '0 0 * * *' },   // daily at midnight
      jobId: 'daily-digest',                // dedupe id
    });
  }
}
```

For distributed single-execution cron across cluster instances, prefer BullMQ's `repeat` over the per-process `@midwayjs/cron` jobs.

Reference: [Midway BullMQ](https://midwayjs.org/docs/extensions/bullmq)
