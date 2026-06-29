---
title: Use Structured Logging via midwayLogger
impact: MEDIUM-HIGH
impactDescription: "Structured logging enables effective debugging and monitoring"
tags: devops, logging, logger, structured
---

## Use Structured Logging via midwayLogger

Midway ships `@midwayjs/logger` (winston-based) with default `coreLogger` (`midway-core.log`) and `appLogger` (`midway-app.log`), plus a context logger bound to each request; all errors also flow to `common-error.log`. Inject the context logger with `@Inject() logger` (request-bound) or `@Logger() logger` (app logger). Configure all loggers under `midwayLogger.clients.<name>` with level, format, rotation, and (v4) per-logger `contextFormat`. Never use `console.log` in production; never log secrets.

**Incorrect (console.log, logging secrets, unstructured text):**

```typescript
@Provide()
export class UserService {
  async login(email: string, password: string) {
    console.log('login', email, password);   // ❌ console.log + leaked secret
    const user = await this.repo.findOne({ where: { email } });
    console.log('result ' + JSON.stringify(user));  // ❌ unstructured, lost in prod
    return user;
  }
}
```

**Correct (typed ILogger, context logger, structured midwayLogger config):**

```typescript
import { Provide, Inject, ILogger, Logger } from '@midwayjs/core';

@Provide()
export class UserService {
  // @Inject() logger is the REQUEST-bound context logger (=== ctx.logger)
  @Inject() logger: ILogger;
  // @Logger() logger: ILogger;              // app logger (no request context)
  // @Logger('coreLogger') logger: ILogger;  // a specific named logger

  async login(email: string, password: string) {
    this.logger.info('login attempt %s', email);   // ✓ %s/%d/%j formatting, no secret
    try {
      const user = await this.verify(email, password);
      this.logger.info('login ok userId=%d', user.id);
      return user;
    } catch (err) {
      this.logger.error('login failed %s: %s', email, err.message);
      throw err;
    }
  }
}
```

```typescript
// config.default.ts — structured logging config
export default {
  midwayLogger: {
    default: {
      level: 'info',
      consoleLevel: process.env.NODE_ENV === 'local' ? 'info' : 'warn',
      dir: 'logs',
      maxSize: '200m',     // rotate at 200MB
      maxFiles: '31d',     // keep 31 days
    },
    clients: {
      coreLogger: { level: 'warn' },
      appLogger: {
        level: 'info',
        // v4: request-scoped log format moved here from koa.contextLoggerFormat
        contextFormat: (info: any) => {
          const ctx = info.ctx;
          return `${info.timestamp} ${info.LEVEL} ${info.pid} [${ctx?.userId}] ${info.message}`;
        },
      },
      // custom logger
      auditLogger: { fileLogName: 'audit.log', lazyLoad: true },
    },
  },
} as MidwayConfig;
```

Levels: `none(0) < error(1) < trace(2) < warn(3) < info(4) < verbose(5) < debug(6)`. Log root: dev `${appDir}/logs/<name>`, prod `$HOME/logs/<name>`.

Reference: [Midway Logger](https://midwayjs.org/docs/logger)
