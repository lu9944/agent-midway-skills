# Midway Best Practices

**Version 1.0.0**
Midway Best Practices
June 2026

> **Note:**
> This document is mainly for agents and LLMs to follow when maintaining,
> generating, or refactoring Midway.js v4 codebases. Humans may also find it
> useful, but guidance here is optimized for automation and consistency
> by AI-assisted workflows.

---

## Abstract

Comprehensive best practices and architecture guide for Midway.js v4 applications, designed for AI agents and LLMs. Contains 78 rules across 10 categories, prioritized by impact from critical (architecture, dependency injection) to incremental (DevOps patterns). Each rule includes detailed explanations, real-world examples comparing incorrect vs. correct implementations, and v4-specific guidance to reduce magic variables and write compliant Midway code.

---

## Table of Contents

1. [Architecture](#1-architecture) — **CRITICAL**
   - 1.1 [Use @Autoload for Self-Initializing Classes](#11-use-autoload-for-self-initializing-classes)
   - 1.2 [Develop Reusable Components and Custom Frameworks](#12-develop-reusable-components-and-custom-frameworks)
   - 1.3 [Use @Configuration with Explicit Component Composition](#13-use-configuration-with-explicit-component-composition)
   - 1.4 [Extend the Context with Typed Custom Properties](#14-extend-the-context-with-typed-custom-properties)
   - 1.5 [Build Custom Decorators with DecoratorManager and MetadataManager](#15-build-custom-decorators-with-decoratormanager-and-metadatamanager)
   - 1.6 [Enable ESM Support Correctly](#16-enable-esm-support-correctly)
   - 1.7 [Write FaaS Functions with @ServerlessTrigger](#17-write-faas-functions-with-serverlesstrigger)
   - 1.8 [Declare the v4 File Detector Explicitly](#18-declare-the-v4-file-detector-explicitly)
   - 1.9 [Use the Functional API for Declarative Routes](#19-use-the-functional-api-for-declarative-routes)
   - 1.10 [Build Fullstack Apps with Midway Hooks](#110-build-fullstack-apps-with-midway-hooks)
   - 1.11 [Organize by Feature Modules with Directory Conventions](#111-organize-by-feature-modules-with-directory-conventions)
   - 1.12 [Extend ServiceFactory for Multi-Instance Services](#112-extend-servicefactory-for-multi-instance-services)
2. [Dependency Injection](#2-dependency-injection) — **CRITICAL**
   - 2.1 [Resolve Circular Dependencies with @LazyInject](#21-resolve-circular-dependencies-with-lazyinject)
   - 2.2 [Use Defined Injection Identifiers, Not Magic Strings](#22-use-defined-injection-identifiers-not-magic-strings)
   - 2.3 [Never Access Injected Dependencies in the Constructor](#23-never-access-injected-dependencies-in-the-constructor)
   - 2.4 [Use Property Injection with @Provide/@Inject Pairing](#24-use-property-injection-with-provide-inject-pairing)
   - 2.5 [Resolve Request-Scoped Services in Singleton Middleware](#25-resolve-request-scoped-services-in-singleton-middleware)
   - 2.6 [Understand Singleton, Request, and Prototype Scopes](#26-understand-singleton-request-and-prototype-scopes)
3. [Error Handling](#3-error-handling) — **HIGH**
   - 3.1 [Use @Catch Error Filters for Centralized Handling](#31-use-catch-error-filters-for-centralized-handling)
   - 3.2 [Extend MidwayError and MidwayHttpError](#32-extend-midwayerror-and-midwayhttperror)
4. [Security](#4-security) — **HIGH**
   - 4.1 [Implement Captcha for Anti-Bot Protection](#41-implement-captcha-for-anti-bot-protection)
   - 4.2 [Use Casbin for Fine-Grained RBAC/ABAC Authorization](#42-use-casbin-for-fine-grained-rbac-abac-authorization)
   - 4.3 [Configure CORS and Security Headers](#43-configure-cors-and-security-headers)
   - 4.4 [Use @Guard for Route-Level Authorization](#44-use-guard-for-route-level-authorization)
   - 4.5 [Implement Secure JWT Authentication](#45-implement-secure-jwt-authentication)
   - 4.6 [Use Passport for OAuth and Strategy-Based Authentication](#46-use-passport-for-oauth-and-strategy-based-authentication)
   - 4.7 [Validate All Input with @midwayjs/validation (v4)](#47-validate-all-input-with-midwayjs-validation-v4-)
5. [Performance](#5-performance) — **HIGH**
   - 5.1 [Enable the Async Context Manager for Request Context](#51-enable-the-async-context-manager-for-request-context)
   - 5.2 [Use @midwayjs/cache-manager for Strategic Caching](#52-use-midwayjs-cache-manager-for-strategic-caching)
   - 5.3 [Subscribe to Changing Data with DataListener](#53-subscribe-to-changing-data-with-datalistener)
   - 5.4 [Use Async Lifecycle Hooks Correctly](#54-use-async-lifecycle-hooks-correctly)
   - 5.5 [Retry Fallible Operations with retryWithAsync](#55-retry-fallible-operations-with-retrywithasync)
   - 5.6 [Offload CPU Work to Thread Pools (@midwayjs/piscina)](#56-offload-cpu-work-to-thread-pools-midwayjs-piscina-)
6. [Testing](#6-testing) — **MEDIUM-HIGH**
   - 6.1 [Test FaaS Functions with createFunctionApp](#61-test-faas-functions-with-createfunctionapp)
   - 6.2 [Use Supertest for HTTP E2E Testing](#62-use-supertest-for-http-e2e-testing)
   - 6.3 [Use @midwayjs/mock for App Bootstrap Testing](#63-use-midwayjs-mock-for-app-bootstrap-testing)
7. [Database & ORM](#7-database-orm) — **MEDIUM-HIGH**
   - 7.1 [Use MikroORM with the Correct v6/v7 Patterns](#71-use-mikroorm-with-the-correct-v6-v7-patterns)
   - 7.2 [Use MongoDB with Typegoose/Mongoose Datasource Form](#72-use-mongodb-with-typegoose-mongoose-datasource-form)
   - 7.3 [Manage Multiple Datasources and Transactions](#73-manage-multiple-datasources-and-transactions)
   - 7.4 [Access Object Storage via Service Factories (COS/OSS/TableStore)](#74-access-object-storage-via-service-factories-cos-oss-tablestore-)
   - 7.5 [Use Prisma ORM in Fullstack Projects](#75-use-prisma-orm-in-fullstack-projects)
   - 7.6 [Use the Redis Service Factory Correctly](#76-use-the-redis-service-factory-correctly)
   - 7.7 [Follow Sequelize v4 Patterns (@Table replaces @BaseTable)](#77-follow-sequelize-v4-patterns-table-replaces-basetable-)
   - 7.8 [Follow TypeORM v4 Datasource Patterns](#78-follow-typeorm-v4-datasource-patterns)
8. [API Design](#8-api-design) — **MEDIUM**
   - 8.1 [Use @Aspect for Cross-Cutting AOP Interception](#81-use-aspect-for-cross-cutting-aop-interception)
   - 8.2 [Write Controllers with Declarative Routing and Param Decorators](#82-write-controllers-with-declarative-routing-and-param-decorators)
   - 8.3 [Manage Cookies and Sessions Correctly](#83-manage-cookies-and-sessions-correctly)
   - 8.4 [Generate CRUD Routes with @midwayjs/crud](#84-generate-crud-routes-with-midwayjs-crud)
   - 8.5 [Handle File Uploads with @midwayjs/busboy](#85-handle-file-uploads-with-midwayjs-busboy)
   - 8.6 [Build GraphQL APIs with @midwayjs/apollo](#86-build-graphql-apis-with-midwayjs-apollo)
   - 8.7 [Use the HTTP Client for Outbound Requests](#87-use-the-http-client-for-outbound-requests)
   - 8.8 [Proxy Requests with @midwayjs/http-proxy](#88-proxy-requests-with-midwayjs-http-proxy)
   - 8.9 [Internationalize with @midwayjs/i18n](#89-internationalize-with-midwayjs-i18n)
   - 8.10 [Apply Middleware at the Right Level with Match/Ignore](#810-apply-middleware-at-the-right-level-with-match-ignore)
   - 8.11 [Use Pipes for Input Transformation](#811-use-pipes-for-input-transformation)
   - 8.12 [Standardize Responses with HttpServerResponse](#812-standardize-responses-with-httpserverresponse)
   - 8.13 [Serve Static Files with @midwayjs/static-file](#813-serve-static-files-with-midwayjs-static-file)
   - 8.14 [Generate API Docs with @midwayjs/swagger](#814-generate-api-docs-with-midwayjs-swagger)
   - 8.15 [Implement Multi-Tenancy with @midwayjs/tenant](#815-implement-multi-tenancy-with-midwayjs-tenant)
   - 8.16 [Version APIs for Backward Compatibility](#816-version-apis-for-backward-compatibility)
   - 8.17 [Render Views with Template Engines](#817-render-views-with-template-engines)
9. [Microservices & Messaging](#9-microservices-messaging) — **MEDIUM**
   - 9.1 [Use @midwayjs/cron for Local Scheduled Tasks](#91-use-midwayjs-cron-for-local-scheduled-tasks)
   - 9.2 [Use @OnEvent for Decoupled Event-Driven Processing](#92-use-onevent-for-decoupled-event-driven-processing)
   - 9.3 [Build MCP Servers with @midwayjs/mcp](#93-build-mcp-servers-with-midwayjs-mcp)
   - 9.4 [Use MQTT for IoT Messaging](#94-use-mqtt-for-iot-messaging)
   - 9.5 [Use the Correct Microservice Provider/Consumer Pattern](#95-use-the-correct-microservice-provider-consumer-pattern)
   - 9.6 [Use BullMQ for Reliable Background Job Processing](#96-use-bullmq-for-reliable-background-job-processing)
10. [DevOps & Deployment](#10-devops-deployment) — **LOW-MEDIUM**
   - 10.1 [Use the Midway Toolchain (mwtsc, mwts, create-midway)](#101-use-the-midway-toolchain-mwtsc-mwts-create-midway-)
   - 10.2 [Build CLI Tools with @midwayjs/commander](#102-build-cli-tools-with-midwayjs-commander)
   - 10.3 [Use Config Centers and Service Discovery (Consul/etcd)](#103-use-config-centers-and-service-discovery-consul-etcd-)
   - 10.4 [Debug with Request Tracing (@midwayjs/code-dye)](#104-debug-with-request-tracing-midwayjs-code-dye-)
   - 10.5 [Implement Graceful Shutdown and Deployment Correctly](#105-implement-graceful-shutdown-and-deployment-correctly)
   - 10.6 [Deploy FaaS to Aliyun FC and AWS Lambda](#106-deploy-faas-to-aliyun-fc-and-aws-lambda)
   - 10.7 [Use Structured Logging via midwayLogger](#107-use-structured-logging-via-midwaylogger)
   - 10.8 [Expose Metrics with @midwayjs/prometheus](#108-expose-metrics-with-midwayjs-prometheus)
   - 10.9 [Use Multi-Environment Configuration with Validation](#109-use-multi-environment-configuration-with-validation)
   - 10.10 [Manage Processes for Production (PM2/cfork)](#1010-manage-processes-for-production-pm2-cfork-)
   - 10.11 [Enable Distributed Tracing with OpenTelemetry](#1011-enable-distributed-tracing-with-opentelemetry)

---

## 1. Architecture

**Section Impact: CRITICAL**

### 1.1 Use @Autoload for Self-Initializing Classes

**Impact: MEDIUM** — "Keeps onReady lean by auto-initializing independent background logic"

When a class contains independent background logic (event listeners, data sync, health monitors) that isn't part of the main request flow, use `@Autoload()` instead of manually resolving it in `onReady`. The framework auto-creates the instance and runs its `@Init()` method at startup — no `container.getAsync(Class)` needed in `@Configuration`. Always pair `@Autoload` with `@Scope(ScopeEnum.Singleton)` and clean up resources in `@Destroy()`. This keeps `onReady` focused on main-flow wiring.

**Incorrect (cluttering onReady with unrelated initialization):**

```typescript
@Configuration({})
export class MainConfiguration {
  async onReady(container) {
    await container.getAsync(RedisErrorListener);   // ❌ unrelated listeners bloating onReady
    await container.getAsync(DataSyncListener);
    await container.getAsync(MetricsCollector);
    await container.getAsync(HealthChecker);
  }
}
```

**Correct (@Autoload self-initializes + @Init + @Destroy cleanup):**

```typescript
import { Autoload, Scope, ScopeEnum, Init, Destroy } from '@midwayjs/core';

@Autoload()
@Scope(ScopeEnum.Singleton)
export class RedisErrorListener {
  private redis: Redis;

  @Init()
  async init() {
    this.redis = new Redis();
    this.redis.on('error', (err) => this.handleError(err));
  }

  @Destroy()
  async destroy() {
    this.redis.disconnect();   // ✓ clean up on shutdown
  }

  private handleError(err: Error) { /* ... */ }
}

// no need to touch onReady — the class starts itself
@Configuration({})
export class MainConfiguration {
  async onReady(container) {
    // only main-flow wiring here (middleware, filters, DB connect)
  }
}
```

`@Autoload` is how many framework components (like RabbitMQ producers) self-register without manual instantiation. The class is scanned, instantiated as Singleton, and `@Init()` runs before `onReady`.

Reference: [Midway Auto Run (@Autoload)](https://midwayjs.org/docs/auto_run)

---

### 1.2 Develop Reusable Components and Custom Frameworks

**Impact: HIGH** — "Components are the unit of reuse; frameworks are components with a server lifecycle"

A Midway component is a self-contained mini-application: a unique `namespace`, `@Configuration` entry, explicit exports, and typed config augmentation via `declare module`. Test it with `createLightApp({ imports: [custom] })`. Since v3, a **Framework** is part of a component — extend an existing one (inject `koa.Framework`, add middleware/filters, extend context) or build a custom protocol framework by extending `BaseFramework` and implementing `configure()`/`applicationInitialize()`/`run()`.

### Component basics: namespace, entry export, typed config

**Incorrect (no namespace, missing entry export, untyped config):**

```typescript
// ❌ no namespace — collides with host config keys
@Configuration({ importConfigs: [{ default: DefaultConfig }] })
export class BookConfiguration {}

// ❌ index.ts forgets to export Configuration — component never registers
// ❌ index.d.ts missing → @Config('book.*') has no type info
```

**Correct:**

```typescript
// src/configuration.ts
@Configuration({
  namespace: 'book',                                    // REQUIRED unique namespace
  importConfigs: [{ default: DefaultConfig }],
})
export class BookConfiguration implements ILifeCycle {
  async onReady(container: IMidwayContainer) {}
}

// src/index.ts — MANDATORY: only explicitly exported decorated classes are scanned
export { BookConfiguration as Configuration } from './configuration';
export * from './service/book.service';

// index.d.ts — typed config (declaration merging, eliminates magic strings)
import '@midwayjs/core';
// if depending on another component, import it so its types are visible:
// import '@midwayjs/axios';
export * from './dist/index';
declare module '@midwayjs/core' {
  interface MidwayConfig {
    book?: { pageSize: number; apiBase: string };
  }
}
```

```json
// package.json
{ "main": "dist/index.js", "typings": "index.d.ts",
  "files": ["dist/**/*.js", "dist/**/*.d.ts", "index.d.ts"] }
```

### Test components with createLightApp

```typescript
import { createLightApp, close } from '@midwayjs/mock';
import * as custom from '../src';

describe('book component', () => {
  it('service works', async () => {
    const app = await createLightApp('', { imports: [custom] });
    const svc = await app.getApplicationContext().getAsync(custom.BookService);
    expect(await svc.getBookById()).toEqual({ data: 'hello world' });
    await close(app);
  });
});

// If the component needs HTTP context, test with a full app + createHttpRequest:
// const app = await createApp(join(__dirname, 'fixtures/base-app'), { imports: [custom] });
// const res = await createHttpRequest(app).get('/');
```

### Extend an existing Framework

Inject the framework (e.g. `koa.Framework`) in `onReady` to add middleware/filters and extend context:

```typescript
@Configuration({ namespace: 'myKoa', imports: [koa] })
export class MyKoaConfiguration {
  @Inject() framework: koa.Framework;

  async onReady() {
    this.framework.useMiddleware(CustomMiddleware);    // app.useMiddleware proxies this
    this.framework.useFilter(CustomFilter);

    // extend koa context (pair with declare module in index.d.ts — see context_definition)
    const app = this.framework.getApplication();
    Object.defineProperty(app.context, 'user', {
      get() { return 'xxx'; },   // ctx.user now available everywhere
      enumerable: true,
    });
  }

  async onServerReady() {
    const server = this.framework.getServer();   // raw HTTP server
  }
}
```

### Write a custom Framework (custom protocol)

Extend `BaseFramework` and implement the three required methods. Export `Application`, `Context`, `Framework`, and the options interface:

```typescript
import { Framework, BaseFramework, IConfigurationOptions, IMidwayApplication, IMidwayContext } from '@midwayjs/core';
import * as http from 'http';

export interface Context extends IMidwayContext {}
export interface Application extends IMidwayApplication<Context> {}
export interface IMidwayCustomConfigurationOptions extends IConfigurationOptions {
  port: number;
}

@Framework()
export class MidwayCustomHTTPFramework extends BaseFramework<Application, Context, IMidwayCustomConfigurationOptions> {
  configure(): IMidwayCustomConfigurationOptions {
    return this.configService.getConfiguration('customKey');
  }

  async applicationInitialize() {
    this.app = http.createServer((req, res) => {
      const ctx = this.app.createAnonymousContext();    // request context with logger, DI scope
      ctx.requestContext.getAsync('xxx').then(ins => ins.doWork()).then(() => res.end());
    });
    this.defineApplicationProperties();   // bind getConfig, getLogger, etc. to app
  }

  async run() {
    if (this.configurationOptions.port) {
      await new Promise<void>(resolve => this.app.listen(this.configurationOptions.port, resolve));
    }
  }
}

// src/index.ts — export the framework
export { Application, Context, MidwayCustomHTTPFramework as Framework, IMidwayCustomConfigurationOptions } from './framework';
export { MyConfiguration as Configuration } from './configuration';
```

### Strong vs weak dependencies, publishing

- **Strong dependency** (always needed): declare in `imports` — e.g. `imports: [axios]`.
- **Weak dependency** (optional): check at runtime — `if (container.hasNamespace('axios')) { ... }`.
- Publish: `npm run build && npm publish` (standard Node.js package).
- Monorepo dev: use lerna with hoist, or place components under `src/components/` with `main` pointing to `src/index` (change back to `dist/index` before publish).

Reference: [Midway Component Development](https://midwayjs.org/docs/component_development), [Context Definition](https://midwayjs.org/docs/context_definition)

---

### 1.3 Use @Configuration with Explicit Component Composition

**Impact: CRITICAL** — "The entry point that wires the entire application together"

In Midway v4, the `@Configuration` class is the single entry point that wires components, loads config, and runs lifecycle hooks. Components are always imported as namespace objects (`import * as koa from '@midwayjs/koa'`) and listed in `imports`. Each component is a self-contained mini-application that registers its own framework, services, and config namespace.

**Incorrect (mismatched imports, manual wiring, missing detector):**

```typescript
// configuration.ts — v3 implicit auto-scan (BROKEN in v4)
import { Configuration } from '@midwayjs/core';
import * as koa from '@midwayjs/koa';

@Configuration({
  imports: [koa],
  // ❌ v4 REMOVED implicit scanning — nothing loads without a detector
  conflictCheck: true,           // ❌ moved to detector in v4
  detectorOptions: {},            // ❌ moved to detector in v4
})
export class MainConfiguration {}
```

**Correct (v4 explicit detector + namespace component imports + typed lifecycle):**

```typescript
// src/configuration.ts
import { Configuration, App, CommonJSFileDetector } from '@midwayjs/core';
import { ILifeCycle, IMidwayContainer, IMidwayApplication } from '@midwayjs/core';
import * as koa from '@midwayjs/koa';
import * as orm from '@midwayjs/typeorm';
import * as validation from '@midwayjs/validation';
import { join } from 'path';
import DefaultConfig from './config/config.default';
import LocalConfig from './config/config.local';

@Configuration({
  imports: [
    koa,          // web server framework
    orm,          // typeorm (each is a namespace object)
    validation,   // v4 validation component
    // environment-gated component (only loaded in dev/prod)
    // { component: require('@midwayjs/info'), enabledEnvironment: ['local'] },
  ],
  // v4: explicit file detector replaces implicit auto-scan
  detector: new CommonJSFileDetector({
    ignore: ['**/logs/**'],
    conflictCheck: true,   // detect duplicate class names
  }),
  importConfigs: [
    {
      default: DefaultConfig,
      local: LocalConfig,
    },
  ],
})
export class MainConfiguration implements ILifeCycle {
  @App() app: IMidwayApplication;

  async onReady(container: IMidwayContainer, app: IMidwayApplication) {
    // container is ready; register global middleware/guards/filters here
  }

  async onStop?(container: IMidwayContainer, app: IMidwayApplication) {
    // cleanup resources
  }
}
```

Reference: [Midway Component Development](https://midwayjs.org/docs/component_development), [v4 Upgrade Guide](https://midwayjs.org/docs/upgrade_v4)

---

### 1.4 Extend the Context with Typed Custom Properties

**Impact: MEDIUM-HIGH** — "Type-safe custom ctx fields via declaration merging"

Midway discourages dynamically attaching properties to `ctx` because TypeScript can't type them. The correct way to add custom context fields (e.g. `ctx.userId`, `ctx.tenantId`) is via **TypeScript declaration merging** (`declare module`) so both the runtime assignment and type checking work. In a **project**, augment the Context interface in `src/interface.ts`. In a **component**, augment it in the component's `index.d.ts` — either globally (`@midwayjs/core`) or framework-specifically (`@midwayjs/koa/dist/interface`). Always `import` the module before augmenting it.

**Incorrect (dynamic untyped ctx properties — TS errors, no autocomplete):**

```typescript
@Middleware()
export class AuthMiddleware implements IMiddleware<Context, NextFunction> {
  resolve() {
    return async (ctx: Context, next: NextFunction) => {
      ctx.userId = user.id;   // ❌ TS2339: Property 'userId' does not exist on type 'Context'
      await next();
    };
  }
}

@Controller('/')
export class HomeController {
  @Inject() ctx: Context;
  @Get('/')
  async index() {
    return this.ctx.userId;   // ❌ TS2339 — no type info, no autocomplete
  }
}
```

**Correct (project-level augmentation + middleware assignment + typed access):**

```typescript
// src/interface.ts — augment the global Context interface
import '@midwayjs/core';     // ✓ must import before declare module

declare module '@midwayjs/core' {
  interface Context {
    userId: number;          // now typed everywhere ctx is used
    tenantId?: number;
    requestId: string;
  }
}

// src/middleware/auth.middleware.ts — set values in middleware
import { Middleware, IMiddleware } from '@midwayjs/core';
import { Context, NextFunction } from '@midwayjs/koa';

@Middleware()
export class AuthMiddleware implements IMiddleware<Context, NextFunction> {
  resolve() {
    return async (ctx: Context, next: NextFunction) => {
      const token = extractToken(ctx);
      if (token) {
        ctx.userId = verifyToken(token).sub;   // ✓ typed assignment
        ctx.requestId = ctx.headers['x-request-id'] || randomUUID();
      }
      await next();
    };
  }
  static getName(): string { return 'auth'; }
}

// src/controller/user.controller.ts — typed access in any handler
@Controller('/user')
export class UserController {
  @Inject() ctx: Context;

  @Get('/me')
  async me() {
    return { id: this.ctx.userId, requestId: this.ctx.requestId };  // ✓ typed read
  }
}
```

**Component-level augmentation (in the component's `index.d.ts`):**

```typescript
// index.d.ts — extend ALL frameworks' Context
declare module '@midwayjs/core' {
  interface Context {
    tenantId?: number;
  }
}

// OR extend only a specific framework's Context:
declare module '@midwayjs/koa/dist/interface' {
  interface Context { tenantId?: number; }
}
// @midwayjs/web → '@midwayjs/web/dist/interface'
// @midwayjs/faas → '@midwayjs/faas/dist/interface'
// @midwayjs/express → '@midwayjs/express/dist/interface'
```

> **Caveat:** `declare module` merges (not replaces) interface definitions, but you must `import` the target module first. In components, use the `index.d.ts` form (not `src/interface.ts`) to avoid cross-component augmentation conflicts.

Reference: [Midway Context Definition](https://midwayjs.org/docs/context_definition)

---

### 1.5 Build Custom Decorators with DecoratorManager and MetadataManager

**Impact: MEDIUM-HIGH** — "v4 metadata system for class/property/method/parameter decorators"

v4 replaced `reflect-metadata` with two static utility classes: `DecoratorManager` (registers/saves modules, creates custom decorators) and `MetadataManager` (read/write metadata with prototype-chain awareness). To make a decorator **functional** (not just metadata-carrying), register an implementation handler on `MidwayDecoratorService` (`registerPropertyHandler`/`registerMethodHandler`/`registerParameterHandler`) in `@Init` or `onReady`. Method decorators require the decorated method to be **async**.

### Custom property decorator (e.g. a `@MemoryCache` injector)

```typescript
import { Provide, Scope, ScopeEnum, Init, Inject } from '@midwayjs/core';
import { DecoratorManager, MidwayDecoratorService } from '@midwayjs/core';

const MEMORY_CACHE_KEY = 'decorator:memoryCache';

// 1. define the decorator
export function MemoryCache(key?: string): PropertyDecorator {
  return DecoratorManager.createCustomPropertyDecorator(MEMORY_CACHE_KEY, { key });
}

// 2. implement the handler in a Singleton service
@Provide()
@Scope(ScopeEnum.Singleton)
export class DecoratorInitService {
  @Inject() decoratorService: MidwayDecoratorService;
  private store = new Map<string, any>();

  @Init()
  async init() {
    this.decoratorService.registerPropertyHandler(MEMORY_CACHE_KEY, (propertyName, meta) => {
      return this.store.get(meta.key);   // returns the injected value
    });
  }
}

// 3. usage — the property gets the handler's return value
@Provide()
export class ConfigService {
  @MemoryCache('appConfig')
  config: Record<string, any>;   // injected from store.get('appConfig')
}
```

### Custom method decorator (e.g. `@LoggingTime` — must be async)

```typescript
const LOGGING_KEY = 'decorator:loggingTime';

export function LoggingTime(unit: 'ms' | 's' = 'ms'): MethodDecorator {
  return DecoratorManager.createCustomMethodDecorator(LOGGING_KEY, { unit });
}

// implement with registerMethodHandler — returns an { around } join point
@Init()
async init() {
  this.decoratorService.registerMethodHandler(LOGGING_KEY, (options) => {
    return {
      around: async (joinPoint: JoinPoint) => {
        const start = Date.now();
        const result = await joinPoint.proceed(...joinPoint.args);  // call original
        this.logger.info(`${joinPoint.methodName} took ${Date.now() - start}${options.metadata.unit}`);
        return result;
      },
    };
  });
}

// usage — the decorated method MUST be async
@Provide()
export class ReportService {
  @LoggingTime('ms')
  async generate() { /* ... */ }   // ✓ async required
}
```

For a **no-impl** method decorator (metadata only, no interception), pass `false`: `createCustomMethodDecorator(KEY, {}, false)`.

### Custom parameter decorator (e.g. `@CurrentUser()`)

```typescript
import { REQUEST_OBJ_CTX_KEY } from '@midwayjs/core';

const USER_KEY = 'decorator:currentUser';

export function CurrentUser(): ParameterDecorator {
  return DecoratorManager.createCustomParamDecorator(USER_KEY, {});
}

// implement — options has { target, propertyName, metadata, originArgs, parameterIndex }
this.decoratorService.registerParameterHandler(USER_KEY, (options) => {
  // if it throws, the framework falls back to the original arg (no exception propagated)
  const ctx = options.target[REQUEST_OBJ_CTX_KEY];
  return ctx?.user ?? {};
});

// usage
@Controller('/user')
export class UserController {
  @Get('/me')
  async me(@CurrentUser() user: User) {   // user injected from ctx
    return user;
  }
}
```

### Custom class decorator + metadata (e.g. register a category of classes)

```typescript
import { DecoratorManager, MetadataManager, Scope, ScopeEnum, Provide } from '@midwayjs/core';

export const MODEL_KEY = 'decorator:model';

export function Model(): ClassDecorator {
  return (target: any) => {
    DecoratorManager.saveModule(MODEL_KEY, target);     // register for later listModule()
    MetadataManager.defineMetadata(MODEL_KEY, { test: 'abc' }, target);
    Scope(ScopeEnum.Request)(target);
    Provide()(target);   // make it injectable
  };
}

// in onReady — discover all @Model classes
const models = DecoratorManager.listModule(MODEL_KEY, (m) => m.name.includes('User'));
```

### MetadataManager API reference

```typescript
MetadataManager.defineMetadata(key, value, target, propertyName?);   // set
MetadataManager.attachMetadata(key, value, target, propertyName?);   // append to array
MetadataManager.getMetadata(key, target, propertyName?);             // prototype-chain lookup
MetadataManager.getOwnMetadata(key, target, propertyName?);          // own only
MetadataManager.copyMetadata(source, target);                        // includes prototype chain
MetadataManager.getMethodParamTypes(target, methodName);             // param types
```

Reference: [Midway Custom Decorator](https://midwayjs.org/docs/custom_decorator)

---

### 1.6 Enable ESM Support Correctly

**Impact: MEDIUM** — "v4 first-class ESM requires specific config changes"

v4 provides a first-class ESM scaffold (`koa-v4-esm`). ESM requires: `"type": "module"` in `package.json`, `moduleResolution: Node16/NodeNext` in `tsconfig.json`, imports **must include `.js` extension** (even for `.ts` source), no `require`/`__dirname`/`module.exports`, and the `ESModuleFileDetector` in `@Configuration`. Config loading must use the object form (`importConfigs: [{ default, local }]`). Not supported: alias paths (use Node subpath exports), build-time non-JS copying.

**Incorrect (CJS patterns in an ESM project):**

```typescript
// ❌ missing .js extension (breaks in ESM)
import { UserService } from './service/user';

// ❌ CJS-only APIs
const dir = __dirname;                    // undefined in ESM
module.exports = { foo };                 // syntax error

// ❌ directory-based config loading (unsupported in ESM)
@Configuration({ importConfigs: [join(__dirname, './config')] })
```

**Correct (ESM conventions + ESModuleFileDetector + object config form):**

```json
// package.json
{ "type": "module" }
// tsconfig.json
{ "compilerOptions": { "module": "ESNext", "moduleResolution": "Node16", "target": "ESNext" } }
```

```typescript
// src/configuration.ts
import { Configuration, ESModuleFileDetector } from '@midwayjs/core';
import DefaultConfig from './config/config.default.js';   // ✓ .js extension
import LocalConfig from './config/config.local.js';

@Configuration({
  imports: [/* ... */],
  detector: new ESModuleFileDetector(),                   // ✓ ESM detector
  importConfigs: [{ default: DefaultConfig, local: LocalConfig }],  // ✓ object form
})
export class MainConfiguration {}

// __dirname replacement in ESM
import { dirname } from 'node:path';
import { fileURLToPath } from 'node:url';
const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);
```

Scaffold: `npm init midway@latest -- --type=koa-v4-esm`. Dev/build use `mwtsc`; test with `mocha + ts-node`.

Reference: [Midway ESM](https://midwayjs.org/docs/esm)

---

### 1.7 Write FaaS Functions with @ServerlessTrigger

**Impact: CRITICAL** — "The core entry pattern for @midwayjs/faas projects"

Midway FaaS (`@midwayjs/faas`) is the Serverless project mode. Functions are `@Provide()` classes with methods decorated by `@ServerlessTrigger(type, options)` (from `@midwayjs/core`). Each trigger binds an invocation source: `HTTP`, `API_GATEWAY`, `TIMER`, `OS` (OSS), `MQ` (MNS), `EVENT`, `KAFKA`, `HSF`, `MTOP`, `SSR`, `CDN`, or `LOG`. In v4 the old `@Func`/`@FuncHook` decorators are **gone** — use `@ServerlessTrigger` + `@ServerlessFunction`. Inject the FaaS `Context` from `@midwayjs/faas`. For Aliyun, register starter types via `import type {} from '@midwayjs/fc-starter'` in `interface.ts`.

**Incorrect (v1 @Func decorator, missing @Provide, no trigger):**

```typescript
// ❌ @Func/@FuncHook removed in v4
import { Func } from '@midwayjs/core';
@Func()
export class HelloService {
  async handler() { return 'hello'; }   // ❌ no trigger bound, never invoked
}
```

**Correct (v4 @ServerlessTrigger with typed events + Context injection):**

```typescript
// src/function/hello.ts
import {
  Provide, Inject, Query,
  ServerlessTrigger, ServerlessTriggerType,
} from '@midwayjs/core';
import { Context } from '@midwayjs/faas';
import type { TimerEvent, OSSEvent } from '@midwayjs/fc-starter';

@Provide()
export class HelloService {
  @Inject() ctx: Context;

  // HTTP trigger — auto-normalizes to a Koa-like context (ctx.query, ctx.request.body)
  @ServerlessTrigger(ServerlessTriggerType.HTTP, { path: '/', method: 'get' })
  async handleHTTP(@Query() name = 'midway') {
    return `hello ${name}`;
  }

  // Timer trigger — typed event from the starter package
  @ServerlessTrigger(ServerlessTriggerType.TIMER, { name: 'cleanupTimer' })
  async handleTimer(event: TimerEvent) {
    this.ctx.logger.info('timer fired at %s', event.triggerTime);
  }

  // OSS event trigger (file create/update)
  @ServerlessTrigger(ServerlessTriggerType.OS)
  async handleOSS(event: OSSEvent) { /* process file event */ }
}

// multiple triggers on one function → use @ServerlessFunction for the name
@ServerlessFunction({ functionName: 'abcde' })
@ServerlessTrigger(ServerlessTriggerType.EVENT)
async handleEvent(event: any) { return event; }
```

Project structure: `f.yml` declares the platform + starter (`provider.name: aliyun`, `provider.starter: '@midwayjs/fc-starter'`); function code lives under `src/function/`. **Pitfall:** Aliyun forbids mixing HTTP + non-HTTP triggers in the same function. FaaS is unsuitable for long connections (>5s), large uploads (>2MB gateway limit), or stateful workloads.

Reference: [Midway Serverless Intro](https://midwayjs.org/docs/serverless/serverless_intro), [Aliyun FC](https://midwayjs.org/docs/serverless/aliyun_faas)

---

### 1.8 Declare the v4 File Detector Explicitly

**Impact: CRITICAL** — "v4 removed implicit auto-scan — code silently fails to load without it"

The single most common v4 migration breakage: the framework **removed implicit automatic scanning**. Without an explicit `detector`, decorated classes are never registered with the IoC container and injection silently fails. Always declare a `CommonJSFileDetector` (CommonJS) or `ESModuleFileDetector` (ESM) in `@Configuration`. The `conflictCheck` and `detectorOptions` that previously lived on `@Configuration` are now properties of the detector.

**Incorrect (relying on removed implicit scanning):**

```typescript
import { Configuration } from '@midwayjs/core';

@Configuration({
  imports: [koa],
  // ❌ No detector — NOTHING is scanned in v4. @Provide classes never register.
  conflictCheck: true,   // ❌ ignored here in v4
})
export class MainConfiguration {}
```

**Correct (explicit detector with conflict checking):**

```typescript
import { Configuration, CommonJSFileDetector } from '@midwayjs/core';

@Configuration({
  imports: [koa],
  detector: new CommonJSFileDetector({
    // optional: pattern (default '**/*.{ts,tsx,js,mts,mjs,cts,cjs}')
    // optional: ignore (default logs, run, public, node_modules, *.test.*, __test__, *.d.ts)
    ignore: [
      '**/logs/**',
      '**/run/**',
    ],
    conflictCheck: true,   // throws on duplicate class names across the codebase
  }),
})
export class MainConfiguration {}

// ESM projects use the ESM detector instead:
// import { ESModuleFileDetector } from '@midwayjs/core';
// detector: new ESModuleFileDetector()
```

Reference: [v4 Upgrade — Detector](https://midwayjs.org/docs/upgrade_v4)

---

### 1.9 Use the Functional API for Declarative Routes

**Impact: MEDIUM-HIGH** — "Function-style routes with end-to-end type safety (zod)"

Midway's Functional API (`@midwayjs/core/functional`) is an alternative to class-based decorators for defining routes. Use `defineApi()` with a builder chain (`api.get(path).input(schema).output(schema).handle(fn)`), and `useInject`/`useConfig`/`useContext` hooks inside handlers. It **coexists** with `@Controller` — use it when co-developing a frontend in the same repo (the typed client enables "zero API" RPC calls). Entry point is `defineConfiguration()` instead of `@Configuration`.

### Core pattern: defineApi + zod + hooks + typed client

**Incorrect (hand-written fetch, no type safety, no validation):**

```typescript
// ❌ hand-written fetch with no type safety
async function getUser(id: string) {
  const res = await fetch(`/api/users/${id}`);
  return res.json();   // ❌ untyped, no validation
}
```

**Correct:**

```typescript
// src/server/index.ts — functional entry
import { defineConfiguration } from '@midwayjs/core/functional';
import * as koa from '@midwayjs/koa';
export default defineConfiguration({ imports: [koa] });

// src/server/api/user.api.ts — declarative route with validation
import { defineApi, useInject, useContext, useConfig, useLogger } from '@midwayjs/core/functional';
import { z } from 'zod';
import { UserService } from '../service/user.service';

export const userApi = defineApi('/users', (api) => ({
  getUser: api
    .get('/:id')
    .input({ params: z.object({ id: z.string().min(1) }) })
    .output(z.object({ id: z.string(), name: z.string() }))
    .handle(async ({ input }) => {
      const ctx = useContext();                         // request context
      const logger = useLogger();                        // logger
      const cfg = useConfig('app');                      // config
      const userService = await useInject(UserService);  // async IoC
      return userService.findById(input.params.id);
    }),

  createUser: api
    .post('/')
    .input({ body: z.object({ name: z.string() }) })
    .handle(async ({ input }) => {
      const svc = await useInject(UserService);
      return svc.create(input.body);
    }),
}));

// src/web/api/client.ts — typed frontend client (zero-API RPC)
import { createClient } from '@midwayjs/react';   // or '@midwayjs/vue'
import { userApi } from '../../server/api/user.api';
export const api = createClient({ user: userApi }, { basePath: '/api' });
// usage: const u = await api.user.getUser({ params: { id: 'u-1' } });  // fully typed
```

Hooks (`useInject`, `useInjectSync`, `useConfig`, `useLogger`, `useContext`, `useApp`, `useMainApp`, `useInjectClient`, `useInjectDataSource`, `usePlugin`) must be called **inside** `.handle()`. `input(...)` validates before business logic; `output(...)` validates before the response is sent.

### Frontend integration (React / Vue + Vite)

Configure the Vite bridge so the frontend imports server API contracts and the dev server runs the backend:

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';        // or '@vitejs/plugin-vue'
import { devPlugin } from '@midwayjs/mock/vite';
import { apiPlugin } from '@midwayjs/web-bridge/vite';

export default defineConfig({
  plugins: [
    devPlugin({ appDir: process.cwd(), baseDir: 'src/server', basePath: '/api' }), // runs backend in dev
    react(),                        // or vue()
    apiPlugin({ root: process.cwd(), apiDir: 'src/server/api', target: 'both' }), // type bridge
  ],
});
// Rspack alternative: createApiRspackRule({ root: process.cwd(), apiDir: 'src/server/api' })
```

React usage: `api.user.getUser({ params: { id: 'u-1' } }).then(u => setName(u.name))`. Vue: `const user = await api.user.getUser({ params: { id: 'u-1' } })` in `setup()`.

### Workspace boundaries

```
src
├── server                    # serverDir (default: src/server)
│   ├── index.ts              # defineConfiguration entry
│   └── api                   # apiDir (default: src/server/api)
│       └── user.api.ts       # API definitions = route contracts + type source
└── web                       # webDir (default: src/web)
    ├── main.tsx              # frontend entry
    └── api
        └── client.ts         # createClient
```

**Frontend CAN import:** `src/server/api/*.api.ts` exports (API definitions, types, zod schemas).
**Frontend CANNOT import:** Node-only modules (`fs`/`path`/`net`), server runtime code, handler internals.
Directories are customizable — just keep `client.ts` import paths and the plugin's `apiDir` in sync.

### Gradual migration from @Controller

`@Controller` and `defineApi` coexist in the same project — no forced split. Midway detects conflicts by `method + fullPath`. Recommended order:

1. Keep existing Service layer untouched
2. New APIs use `defineApi`
3. Migrate old controllers module-by-module
4. Add `input`/`output` schemas to critical APIs first
5. Delete old controllers once stable

### Build & deploy (unified dev, separate deploy)

```json
{
  "scripts": {
    "dev": "vite",                              // runs both via devPlugin
    "build:server": "tsc -p tsconfig.server.json",
    "build:web": "vite build",
    "build": "npm run build:server && npm run build:web"
  }
}
```

Artifacts: `dist/server` (Node runtime) + `dist/web` (static → Nginx/CDN). Deploy: `node dist/server/bootstrap.js`, serve `dist/web` statically, proxy `/api/*` to server.

### Testing (3 layers)

```typescript
// Layer 1 — contract test (fast): call API directly, cover valid/invalid/boundary input
// Layer 2 — server integration test (primary, highest ROI):
import { close, createApp, createHttpRequest } from '@midwayjs/mock';

describe('functional api', () => {
  let app;
  beforeAll(async () => { app = await createApp(); });
  afterAll(async () => { await close(app); });

  it('GET /api/users/:id', async () => {
    const res = await createHttpRequest(app).get('/api/users/u-1');
    expect(res.status).toBe(200);
    expect(res.body.id).toBe('u-1');
  });

  it('rejects invalid input (validation failure)', async () => {
    const res = await createHttpRequest(app).post('/api/users').send({ /* invalid */ });
    expect(res.status).toBe(500);   // input() schema rejects
  });
});
// Layer 3 — frontend client test (as needed): mock client returns in unit tests,
//           verify one real end-to-end chain in E2E.
```

Priority: 1) server integration tests (highest ROI), 2) key contract tests, 3) frontend interaction tests.

Reference: [Functional Intro](https://midwayjs.org/docs/functional/intro), [API Reference](https://midwayjs.org/docs/functional/api-reference), [Testing](https://midwayjs.org/docs/functional/testing), [Migration](https://midwayjs.org/docs/functional/migration), [Workspace](https://midwayjs.org/docs/functional/workspace), [React](https://midwayjs.org/docs/functional/react), [Vue](https://midwayjs.org/docs/functional/vue)

---

### 1.10 Build Fullstack Apps with Midway Hooks

**Impact: MEDIUM** — "Zero-API RPC with React/Vue + type-safe backend"

Midway Hooks (`@midwayjs/hooks` + `@midwayjs/hooks-kit`) is the function-style fullstack framework: a single codebase contains frontend (React/Vue) and Node.js backend. Backend API functions are imported and called directly on the frontend — the toolkit compiles this into an HTTP RPC call (`{ args: [...] }` wire format) with shared TypeScript types. Define APIs via `Api(Get(), async () => ...)` operators; use `useContext()` for request context. Validate with zod via `Validate(...)`. Use `hooks dev`/`hooks build`/`hooks start`.

> **Note:** The docs carry a deprecation caution — existing fullstack projects may continue, but evaluate carefully for new projects.

**Incorrect (writing separate API client + fetch calls, no shared types):**

```typescript
// ❌ frontend: untyped fetch, types drift from backend
const user = await fetch('/api/user').then(r => r.json()) as any;
```

**Correct (Api operators + useContext + Validate + typed import):**

```typescript
// backend: src/api/hello.ts
import { Api, Get, Post, Validate } from '@midwayjs/hooks';
import { z } from 'zod';

// auto-route: /api/hello (functionName + fileName, prefixed /api)
export default Api(Get(), async () => 'Hello World!');

// explicit path + zod validation (positional to handler args)
export const say = Api(
  Post('/say'),
  Validate(z.string()),
  async (name: string) => `Hello ${name}!`,
);

// useContext for request data
import { useContext } from '@midwayjs/hooks';
import { Context } from '@midwayjs/koa';
export const ip = Api(Get(), async () => {
  const ctx = useContext<Context>();
  return { ip: ctx.ip };
});

// frontend: import & call directly — full type inference
import say, { ip } from './api/hello';
const greeting = await say('Midway');   // → POST /api/say {args:['Midway']}
const { ip: clientIp } = await ip();    // → GET /api/ip
```

Middleware: function-style `(next) => { const ctx = useContext(); ... await next(); }`, scoped global/file/function. Test via `getApiTrigger(api)` + `createHttpRequest(app)`.

### Filesystem routing (simple mode)

Enable file-system routing in `midway.config.ts` — any async function exported from `.ts` files under the route `baseDir` auto-becomes an API. Functions returning without `Api()` default to GET (no args) or POST (with args). `index.ts` → root; nested files → nested paths; `[name]` → path param; `[...file]` → wildcard:

```typescript
// midway.config.ts — enable file routing
import { defineConfig } from '@midwayjs/hooks';
export default defineConfig({
  source: './src/apis',
  routes: [{ baseDir: 'lambda', basePath: '/api' }],
});
// lambda/index.ts → /api/         (export default = root)
// lambda/about.ts → /api/about    (export default)
// lambda/about/contact.ts → /api/about/contact (named export)
// lambda/[name]/project.ts → /api/:name/project (path param, needs Params<T>())
// lambda/[...index].ts → /api/*   (wildcard)
```

```typescript
// lambda/[name]/project.ts — path param with filesystem routing
import { Api, Get, Params, useContext } from '@midwayjs/hooks';
export default Api(Get(), Params<{ name: string }>(), async () => {
  const ctx = useContext();
  return { name: ctx.params.name };
});
```

### CORS, upload, and other sub-features

```typescript
// CORS — pass @koa/cors as middleware (global, file, or function level)
import cors from '@koa/cors';
export const config: ApiConfig = { middleware: [cors()] };

// File upload — handler args accept FormData on POST
export default Api(Post(), async (formData: FormData) => {
  const file = formData.get('file');
  /* process file */
});

// Input validation/security — Validate(zodSchema...) positional to handler args
import { Validate } from '@midwayjs/hooks';
import { z } from 'zod';
export const safe = Api(Post(), Validate(z.string().email()), async (email: string) => {
  return { ok: true };
});
```

Reference: [Midway Hooks Intro](https://midwayjs.org/docs/hooks/intro), [Hooks API](https://midwayjs.org/docs/hooks/api), [File Routing](https://midwayjs.org/docs/hooks/file-route), [Middleware](https://midwayjs.org/docs/hooks/middleware), [CORS](https://midwayjs.org/docs/hooks/cors), [Upload](https://midwayjs.org/docs/hooks/upload), [Safe](https://midwayjs.org/docs/hooks/safe)

---

### 1.11 Organize by Feature Modules with Directory Conventions

**Impact: CRITICAL** — "3-5x faster onboarding and feature development"

Midway scans a standard directory layout by convention. Organize the application into self-contained feature directories (one folder per domain) instead of grouping by technical layer. Each module bundles its own controllers, services, entities, events, and queues. This mirrors the `cool-admin-midway` pattern where all business code lives under `src/modules/{name}/` and is auto-discovered.

**Incorrect (technical-layer organization, scattered files):**

```typescript
// ❌ All controllers/services/entities dumped in flat top-level folders
src/
├── controllers/        // users, orders, products all mixed
├── services/
├── entities/
└── configuration.ts
```

**Correct (feature-module + conventional Midway directories):**

```typescript
// Convention: src/<layer>/<name>.<layer>.ts
src/
├── configuration.ts            // the single @Configuration entry
├── interface.ts                // shared TS type definitions
├── config/
│   ├── config.default.ts       // loaded in ALL environments
│   ├── config.local.ts         // dev only
│   └── config.prod.ts          // prod only
├── controller/                 // web controllers (or modules/*/controller)
│   └── user.controller.ts
├── service/
│   └── user.service.ts
├── entity/
│   └── user.entity.ts
├── middleware/                 // *.middleware.ts
├── guard/                      // *.guard.ts
├── filter/                     // *.filter.ts
├── aspect/                     // *.ts (AOP)
├── pipe/                       // *.pipe.ts
├── dto/                        // validation DTOs
├── error/                      // custom *.error.ts
└── modules/                    // (recommended) self-contained feature modules
    └── order/
        ├── controller/
        ├── service/
        └── entity/

// A self-contained module file
// src/modules/order/service/order.service.ts
import { Provide } from '@midwayjs/core';

@Provide()
export class OrderService {
  async create(userId: number) {
    return { id: 1, userId };
  }
}
```

The `cool-admin-midway` best practice goes further: every module exposes a typed `config.ts` factory that declares its name, description, load `order`, module-scoped middleware, and arbitrary config keys readable via `@Config('module.<name>.<key>')`. Adding a folder makes the module live without manual registration.

Reference: [Midway Directory Conventions](https://midwayjs.org/docs/quickstart), [cool-admin-midway](https://github.com/cool-team-official/cool-admin-midway)

---

### 1.12 Extend ServiceFactory for Multi-Instance Services

**Impact: MEDIUM-HIGH** — "The core pattern behind redis/oss/cos/kafka multi-instance components"

`ServiceFactory<T>` (from `@midwayjs/core`) is the base class for components that manage multiple named instances of an SDK client (e.g. Redis, OSS, COS, Kafka producer). Extend it as a `@Singleton`, implement `createClient(config)` and `getName()`, call `initClients(config)` in `@Init`. Consumers get instances via `factory.get('name')` or `@InjectClient(FactoryClass, 'name')`. This is distinct from `DataSourceManager` (which also binds entities).

**Incorrect (manual client management, no lifecycle, no multi-instance support):**

```typescript
@Provide()
export class MyClientService {
  private client: MyClient;
  @Init() init() { this.client = new MyClient(config); }  // ❌ single instance, no cleanup, no config-driven
}
```

**Correct (extend ServiceFactory + config-driven multi-instance + InjectClient):**

```typescript
import { ServiceFactory, Provide, Scope, ScopeEnum, Init, Config, InjectClient } from '@midwayjs/core';

@Provide()
@Scope(ScopeEnum.Singleton)
export class MyClientServiceFactory extends ServiceFactory<MyClient> {
  @Config('myClient') myClientConfig;

  @Init()
  async init() {
    await this.initClients(this.myClientConfig, { concurrent: true });  // v4 parallel option
  }

  protected createClient(config: any, clientName?: string): MyClient {
    return new MyClient(config);     // create one instance from config (clientName optional)
  }

  getName(): string { return 'myClient'; }

  async destroyClient(client: MyClient, clientName: string) {
    await client.close();            // optional cleanup
  }
}

// config.default.ts — 3 config forms
export default {
  myClient: {
    default: { timeout: 5000 },                  // shared defaults
    client: { baseUrl: 'https://a.com' },        // single instance → becomes clients.default
    // OR multiple:
    // clients: { aaa: { baseUrl: '...' }, bbb: { baseUrl: '...' } },
    // defaultClientName: 'aaa',
  },
} as MidwayConfig;

// consumer — default instance
@Provide()
export class MyService {
  @Inject() factory: MyClientServiceFactory;
  async call() { return this.factory.get().request(); }        // default
  async callNamed() { return this.factory.get('bbb').request(); }
}

// consumer — named instance via decorator
@Provide()
export class AnotherService {
  @InjectClient(MyClientServiceFactory, 'aaa') aaaClient: MyClient;
}

// dynamic creation at runtime
const dyn = await this.factory.createInstance({ baseUrl: '...' }, 'dynamicName');
```

Config type augmentation + default proxy pattern:
```typescript
import { ServiceFactoryConfigOption } from '@midwayjs/core';
declare module '@midwayjs/core' {
  interface MidwayConfig {
    myClient?: ServiceFactoryConfigOption<MyClientConfig>;
  }
}
// optional: delegate all prototype methods to the default instance
// delegateTargetAllPrototypeMethod(MyClientWrapperService, MyClient);
```

Reference: [Midway Service Factory](https://midwayjs.org/docs/service_factory)

---

## 2. Dependency Injection

**Section Impact: CRITICAL**

### 2.1 Resolve Circular Dependencies with @LazyInject

**Impact: HIGH** — "v4 auto-detects cycles; @LazyInject is the safe escape hatch"

Circular dependencies (A → B → C → A) indicate an architectural smell and should ideally be resolved by extracting shared logic or using events. However, when unavoidable, v4 **auto-detects** cycles and throws a clear error (`Circular dependency detected: A -> B -> C -> A`). The correct escape hatch is `@LazyInject(() => Clazz)`, which defers resolution until first access. Use it **only** for genuine circular dependencies, not as a general pattern.

**Incorrect (mutual @Inject creates a cycle that crashes startup):**

```typescript
@Provide()
export class OrderService {
  @Inject() userService: UserService;   // Order → User
}

@Provide()
export class UserService {
  @Inject() orderService: OrderService;  // User → Order → cycle! startup crash
}
```

**Correct (break the cycle with @LazyInject on one side):**

```typescript
import { Provide, Inject, LazyInject } from '@midwayjs/core';

@Provide()
export class OrderService {
  @Inject() userService: UserService;   // eager — fine
}

@Provide()
export class UserService {
  // @LazyInject defers resolution to first property access, breaking the cycle
  @LazyInject(() => OrderService)
  orderService: OrderService;
}
```

**Prefer these structural fixes over `@LazyInject`:**

```typescript
// Fix 1: extract shared logic into a third, dependency-free module
@Provide()
export class SharedNotificationService { /* ... */ }
// Both OrderService and UserService depend on SharedNotificationService

// Fix 2: use the event emitter for fire-and-forget decoupling
import { OnEvent, EventEmitterService } from '@midwayjs/event-emitter';
@Singleton()
export class UserService {
  @Inject() eventEmitter: EventEmitterService;
  async createUser() {
    this.eventEmitter.emitAsync('user.created', user);  // no direct dependency
  }
}
```

Reference: [Midway Container — Circular Dependency](https://midwayjs.org/docs/container)

---

### 2.2 Use Defined Injection Identifiers, Not Magic Strings

**Impact: HIGH** — "Reduces magic variables and makes DI intent explicit"

TypeScript interfaces are erased at runtime, so they cannot serve as injection keys. When you need to inject an abstraction (e.g. swap implementations per environment), define the identifier as a **constant symbol/string** in one place and reuse it — never scatter raw string literals. For multi-instance service factories, use `@InjectClient(Factory, 'name')` with a named constant. This keeps DI intent explicit and grep-able, eliminating magic variables.

**Incorrect (raw string identifiers scattered as magic variables):**

```typescript
// ❌ magic string 'payment' duplicated and typo-prone across files
@Configuration({ imports: [PaymentModule] })
export class AppModule {}

@Provide('payment')
export class StripeService implements PaymentGateway {}

@Controller('/pay')
export class PayController {
  @Inject('payment') payment: PaymentGateway;   // ❌ 'payment' literal — no single source of truth
}

// ❌ magic instance name for a service factory
@InjectClient(RedisServiceFactory, 'instance1') redis1: RedisService;
```

**Correct (centralized constants + registerObject + typed config keys):**

```typescript
// src/constants.ts — single source of truth for identifiers
export const PAYMENT_GATEWAY = Symbol('PAYMENT_GATEWAY');

// provider
@Provide()
export class StripeService implements PaymentGateway { /* ... */ }

// register an implementation by token (e.g. in onReady or a factory)
container.registerObject(PAYMENT_GATEWAY, new StripeService());

// consumer — uses the constant, not a literal
@Provide()
export class OrderService {
  @Inject(PAYMENT_GATEWAY) payment: PaymentGateway;
}

// service-factory instances: name them via typed config, not inline literals
// config.default.ts → redis.clients.cache / redis.clients.session
@InjectClient(RedisServiceFactory, 'cache') cacheRedis: RedisService;

// register a global object once and inject by a named constant
export const LODASH_KEY = 'lodash';
async onReady(container: IMidwayContainer) {
  container.registerObject(LODASH_KEY, _);
}
```

Reference: [Midway Container — Identifiers](https://midwayjs.org/docs/container), [Service Factory](https://midwayjs.org/docs/service_factory)

---

### 2.3 Never Access Injected Dependencies in the Constructor

**Impact: CRITICAL** — "The #1 cause of undefined property bugs in Midway"

Injected properties (`@Inject`, `@Config`, `@Logger`) are populated by the container **after** construction. Reading them inside the `constructor` returns `undefined`. Perform all initialization that depends on injected values in an `@Init()` method instead — it is always invoked asynchronously after injection is complete. This is the most common mistake for developers coming from constructor-injection frameworks.

**Incorrect (reading injected deps in constructor):**

```typescript
@Provide()
export class DatabaseService {
  @Config('mysql.host')
  host: string;

  constructor() {
    // ❌ this.host is undefined here — injected AFTER construction
    this.pool = mysql.createPool({ host: this.host });
  }
}
```

**Correct (use @Init for async post-injection setup):**

```typescript
import { Provide, Init, Config, Destroy } from '@midwayjs/core';

@Provide()
export class DatabaseService {
  @Config('mysql.host')
  host: string;

  private pool: any;

  @Init()
  async init() {
    // ✓ this.host is now populated; @Init runs after injection
    this.pool = mysql.createPool({ host: this.host });
  }

  @Destroy()
  async destroy() {
    // cleanup when the instance is disposed
    await this.pool.end();
  }

  async query(sql: string) {
    return this.pool.query(sql);
  }
}
```

The same rule applies to `@Config(...)`, `@Logger(...)`, and `@App()` — none are available in the constructor. Use `@Init()` (only one per class, always async) and pair cleanup with `@Destroy()`.

Reference: [Midway Lifecycle](https://midwayjs.org/docs/lifecycle)

---

### 2.4 Use Property Injection with @Provide/@Inject Pairing

**Impact: CRITICAL** — "The fundamental DI contract of Midway"

Midway's primary DI style is **property injection**: a class decorated with `@Provide()` (or implicitly via `@Controller`/`@Singleton`) registers itself with the IoC container, and consumers inject it with `@Inject()`. The two are paired — `@Inject` resolves by the TypeScript class type, so the dependency must be `@Provide`-d. Prefer this over reaching into the container manually. v4 also restores constructor injection, but property injection remains the idiomatic, zero-arg-constructable default used throughout `cool-admin-midway`.

**Incorrect (manual container access, missing @Provide, unpaired injection):**

```typescript
// ❌ Service missing @Provide — cannot be injected
export class UserService {
  async getUser(id: number) { return { id }; }
}

@Controller('/api')
export class UserController {
  // ❌ UserService was never registered, so this is undefined
  @Inject() userService: UserService;

  // ❌ Service locator anti-pattern — hides dependencies
  async list(@Query() q, @ApplicationContext() ctx) {
    const svc = await ctx.requestContext.getAsync(UserService);
  }
}
```

**Correct (@Provide/@Inject pairing, idiomatic property injection):**

```typescript
import { Provide, Inject, Controller, Get, Query } from '@midwayjs/core';

// @Provide() registers the class with the container (makes it injectable)
@Provide()
export class UserService {
  async getUser(id: number) {
    return { id, name: 'Harry' };
  }
}

@Controller('/api/user')   // @Controller implicitly includes @Provide
export class UserController {
  // @Inject resolves by the UserService type — paired with @Provide above
  @Inject()
  userService: UserService;

  @Get('/')
  async getUser(@Query('id') uid: number) {
    const user = await this.userService.getUser(uid);
    return { success: true, message: 'OK', data: user };
  }
}

// v4 also supports constructor injection (only @Inject works in constructors):
@Provide()
export class OrderService {
  constructor(@Inject() private userService: UserService) {}
}
```

When you need an explicit identifier (multiple implementations, or a non-class dependency), pair string identifiers explicitly: `@Provide('cache')` + `@Inject('cache')`.

Reference: [Midway Container & DI](https://midwayjs.org/docs/container)

---

### 2.5 Resolve Request-Scoped Services in Singleton Middleware

**Impact: HIGH** — "Middleware/Aspect are always Singleton — direct @Inject of Request scope fails"

Middleware classes (`@Middleware()`) and AOP aspects (`@Aspect()`) are **always Singleton scope**. You cannot `@Inject()` a Request-scoped service into them — the injected instance has no request context (`ctx` is undefined). To call Request-scoped logic from a Singleton middleware, resolve the service per-request via `ctx.requestContext.getAsync(Service)` inside the `resolve()` handler.

**Incorrect (injecting Request-scoped service into Singleton middleware):**

```typescript
@Middleware()
export class AuthMiddleware implements IMiddleware<Context, NextFunction> {
  @Inject() userService: UserService;   // ❌ not bound to the current request/ctx
  resolve() {
    return async (ctx: Context, next: NextFunction) => {
      // ❌ this.userService has no ctx — behaves like a stale/frozen instance
      const user = await this.userService.getCurrentUser();
      ctx.user = user;
      await next();
    };
  }
}
```

**Correct (resolve per-request via requestContext.getAsync):**

```typescript
import { Middleware, IMiddleware } from '@midwayjs/core';
import { Context, NextFunction } from '@midwayjs/koa';

@Middleware()
export class AuthMiddleware implements IMiddleware<Context, NextFunction> {
  resolve() {
    return async (ctx: Context, next: NextFunction) => {
      // ✓ resolve a fresh Request-scoped instance bound to THIS ctx
      const userService = await ctx.requestContext.getAsync<UserService>(UserService);
      ctx.user = await userService.getCurrentUser();
      await next();
    };
  }
  static getName(): string { return 'auth'; }
}
```

The same applies to AOP aspects: inside `before`/`around`, access the request context via `joinPoint.target[REQUEST_OBJ_CTX_KEY]` (imported from `@midwayjs/core`) rather than injected `ctx`.

Reference: [Midway Middleware](https://midwayjs.org/docs/middleware)

---

### 2.6 Understand Singleton, Request, and Prototype Scopes

**Impact: CRITICAL** — "Wrong scope causes data races, memory leaks, and undefined injections"

Midway has three scopes via `ScopeEnum` (imported from `@midwayjs/core`): `Singleton` (one instance per process), `Request` (the **default**; one instance per request chain, destroyed after), and `Prototype` (new instance every time). Controllers are always Request-scoped and cannot be changed. The critical rule: if a Singleton injects a Request-scoped dependency, that dependency gets **frozen** to a single instance — usually a bug. Use the `@Singleton()` shorthand for stateless, cross-request services (matches `cool-admin-midway`'s `Utils`, `PluginCenterService`, swagger builders).

**Incorrect (wrong scope, mutable shared state in singleton, injecting ctx into singleton):**

```typescript
// ❌ Mutable per-request state on a Singleton — shared across all concurrent requests
@Provide()
@Scope(ScopeEnum.Singleton)
export class RequestContextService {
  private userId: string;          // overwritten by every concurrent request!

  setUserId(id: string) { this.userId = id; }
  getUserId() { return this.userId; }   // returns WRONG user
}

// ❌ Injecting ctx (Request scope) into a Singleton — ctx is undefined
@Provide()
@Scope(ScopeEnum.Singleton)
export class AuditService {
  @Inject() ctx: any;              // undefined — no request in a Singleton
}
```

**Correct (appropriate scope per use case, @Singleton shorthand):**

```typescript
import { Provide, Scope, ScopeEnum, Singleton } from '@midwayjs/core';

// Stateless, cross-request service → use the @Singleton() shorthand
@Singleton()                        // equivalent to @Provide() @Scope(ScopeEnum.Singleton)
export class Utils {
  formatDate(d: Date) { return d.toISOString(); }
}

// Request-scoped service holding per-request state (the DEFAULT)
@Provide()
export class UserService {
  @Inject() ctx: any;              // ✓ valid: this is Request scope
  async getCurrentUser() { return this.ctx.user; }
}

// A Singleton that MUST depend on a Request-scoped service:
// explicitly allow downgrade so v4 doesn't throw MidwaySingletonInjectRequestError
@Provide()
@Scope(ScopeEnum.Request, { allowDowngrade: true })
export class ReportService {
  @Inject() userService: UserService;   // degrades to singleton when no request context
}
```

By default v4 **throws** `MidwaySingletonInjectRequestError` (code `MIDWAY_10010`) when a Singleton tries to inject a Request-scoped object. Use `{ allowDowngrade: true }` only when you understand the freezing behavior.

Reference: [Midway Injection Scopes](https://midwayjs.org/docs/container)

---

## 3. Error Handling

**Section Impact: HIGH**

### 3.1 Use @Catch Error Filters for Centralized Handling

**Impact: HIGH** — "Consistent, centralized error responses across the app"

Never catch-and-format errors manually in every controller. Midway error filters (`@Catch` classes in `src/filter/*.filter.ts`) run **after** middleware and catch all business + framework errors, producing uniform responses. Register specific filters for specific error types, plus exactly **one** generic `@Catch()` filter (no argument) as the catch-all that always runs last. Throw errors from services so filters can map them to HTTP responses; never set status codes silently.

**Incorrect (manual try/catch + manual response shaping in controllers):**

```typescript
@Controller('/user')
export class UserController {
  @Inject() userService: UserService;

  @Get('/:id')
  async findOne(@Param('id') id: number, @Ctx() ctx: Context) {
    try {
      const user = await this.userService.findById(id);
      return ctx.body = user;
    } catch (e) {
      // ❌ manual error formatting repeated in every handler
      ctx.status = 500;
      ctx.body = { message: e.message };
    }
  }
}
```

**Correct (throw from service, handle centrally with @Catch filters):**

```typescript
// src/filter/default.filter.ts
import { Catch } from '@midwayjs/core';
import { Context } from '@midwayjs/koa';

@Catch()   // no arg = catch ALL unhandled errors (only one per app, always last)
export class DefaultErrorFilter {
  async catch(err: Error, ctx: Context) {
    // log manually — errors caught by a custom filter are NOT auto-logged
    ctx.logger.error(err);
    return { status: (err as any).status ?? 500, message: err.message };
  }
}

// src/filter/notfound.filter.ts
import { Catch, httpError, MidwayHttpError } from '@midwayjs/core';

@Catch(httpError.NotFoundError)   // specific error type
export class NotFoundFilter {
  async catch(err: MidwayHttpError, ctx: Context) {
    return { code: 404, message: err.message };
  }
}

// src/configuration.ts — register filters in onReady
async onReady(container, app) {
  app.useFilter([NotFoundFilter, DefaultErrorFilter]);   // order irrelevant: specific first, generic last
}
```

Key behaviors: specific filters match before the generic one; the generic `@Catch()` filter must be unique. To match a base class and all subclasses, use `{ matchPrototype: true }`: `@Catch([MidwayError], { matchPrototype: true })`.

Reference: [Midway Error Filters](https://midwayjs.org/docs/error_filter)

---

### 3.2 Extend MidwayError and MidwayHttpError

**Impact: HIGH** — "Structured errors carry status codes and codes for reliable handling"

Midway provides a structured error hierarchy: `MidwayError` (base, carries `code` and `cause`) and `MidwayHttpError` (adds an HTTP status code). Throw these instead of generic `Error` so filters and logging can act on them. For predefined HTTP statuses, use the `httpError.*` helpers (`BadRequestError`, `UnauthorizedError`, `NotFoundError`, etc.). Custom business errors extend `MidwayError` with a stable error code; HTTP-facing errors extend `MidwayHttpError`.

**Incorrect (generic Error, manual status code plumbing):**

```typescript
@Provide()
export class UserService {
  async findById(id: number) {
    const user = await this.repo.findOne({ where: { id } });
    if (!user) {
      throw new Error('User not found');   // ❌ no status, no code, unstructured
    }
    return user;
  }
}
```

**Correct (structured Midway errors with codes and HTTP status):**

```typescript
import { MidwayError, MidwayHttpError, httpError, HttpStatus } from '@midwayjs/core';

// Throw predefined HTTP errors for common cases
@Provide()
export class UserService {
  async findById(id: number) {
    const user = await this.repo.findOne({ where: { id } });
    if (!user) {
      throw new httpError.NotFoundError(`User #${id} not found`);  // 404, structured
    }
    return user;
  }
}

// Custom business error — extends MidwayError, carries a stable code
export class InsufficientBalanceError extends MidwayError {
  constructor(accountId: number, needed: number) {
    super(
      `Account ${accountId} needs ${needed}`,
      'BIZ_INSUFFICIENT_BALANCE',     // stable, non-colliding code
    );
  }
}

// Custom HTTP error — extends MidwayHttpError, carries status
export class UserNotFoundException extends MidwayHttpError {
  constructor(userId: number) {
    super(`User ${userId} not found`, HttpStatus.NOT_FOUND);
  }
}
```

### Register Error Codes (non-colliding)

Use `registerErrorCode` to generate stable, prefixed error codes for a module/component. This prevents collisions across packages:

```typescript
import { registerErrorCode } from '@midwayjs/core';

// generates codes like 'ORDER_10000', 'ORDER_10001'
export const OrderErrorEnum = registerErrorCode('order', {
  INSUFFICIENT_STOCK: 10000,
  PAYMENT_FAILED: 10001,
  ADDRESS_INVALID: 10002,
} as const);

// use in custom errors
export class InsufficientStockError extends MidwayError {
  constructor(productId: string) {
    super(`Product ${productId} out of stock`, OrderErrorEnum.INSUFFICIENT_STOCK);
  }
}
```

The framework's own codes are `MIDWAY_10000`–`MIDWAY_10022` (e.g. `MIDWAY_10003` = DI definition not found, `MIDWAY_10010` = Singleton injecting Request scope).

Reference: [Midway Custom Errors](https://midwayjs.org/docs/custom_error), [Error Codes](https://midwayjs.org/docs/error_code)

---

## 4. Security

**Section Impact: HIGH**

### 4.1 Implement Captcha for Anti-Bot Protection

**Impact: MEDIUM** — "Prevents brute-force and automated abuse"

Use `@midwayjs/captcha` for image, math-formula, and text captchas. It stores verification state via `@midwayjs/cache-manager` (default memory store; swap to Redis for multi-instance). The component does **not** send SMS/email — `text()` returns a code you deliver yourself. Always verify via `check(id, answer)` before processing sensitive actions.

**Incorrect (no captcha on login/forgot-password, raw verification):**

```typescript
@Post('/login')
async login(@Body() dto: LoginDTO) {
  // ❌ no captcha → brute-force attack surface
  return this.authService.login(dto);
}
```

**Correct (image captcha + verification + Redis store):**

```typescript
import * as captcha from '@midwayjs/captcha';
import * as cacheManager from '@midwayjs/cache-manager';
@Configuration({ imports: [captcha, cacheManager] })

// config.default.ts — use Redis for multi-instance consistency
export default {
  captcha: {
    default: { size: 4, noise: 1, width: 120, height: 40 },
    image: { type: 'mixed' },     // 'mixed' | 'letter' | 'number'
    expirationTime: 300,          // 5 min (seconds)
  },
  cacheManager: { clients: { captcha: { store: createRedisStore('default') } } },
} as MidwayConfig;

// controller
import { CaptchaService } from '@midwayjs/captcha';
@Controller('/auth')
export class AuthController {
  @Inject() captchaService: CaptchaService;

  @Get('/captcha')
  async getCaptcha() {
    const { id, imageBase64 } = await this.captchaService.image();
    return { id, imageBase64 };
  }

  @Post('/login')
  async login(@Body('captchaId') captchaId: string, @Body('captchaCode') code: string) {
    const passed = await this.captchaService.check(captchaId, code);
    if (!passed) throw new httpError.BadRequestError('invalid captcha');
    return this.authService.login(/* ... */);
  }

  // SMS/email code: component returns the code, you send it
  @Get('/sms-code')
  async sendSms(@Query('phone') phone: string) {
    const { id, text } = await this.captchaService.text({ type: 'number' });
    await this.smsService.send(phone, text);   // you deliver the code
    return { id };
  }
}
```

Reference: [Midway Captcha](https://midwayjs.org/docs/extensions/captcha)

---

### 4.2 Use Casbin for Fine-Grained RBAC/ABAC Authorization

**Impact: MEDIUM** — "Policy-driven access control with externalized rules"

`@midwayjs/casbin` provides policy-driven authorization (RBAC, ABAC). Casbin does **authorization only** — pair it with `@midwayjs/passport` for authentication. Define a model file + policy, configure `usernameFromContext` to read the authenticated user, and apply the built-in `AuthGuard` + `@UsePermission` decorator. For multi-instance, use the Redis or TypeORM adapter with a watcher for policy sync.

**Incorrect (hardcoded role checks, no policy model):**

```typescript
@Get('/users')
async findAll(@Ctx() ctx) {
  if (ctx.user.role !== 'admin') throw new httpError.ForbiddenError();  // ❌ hardcoded, not scalable
  return this.userService.findAll();
}
```

**Correct (declarative @UsePermission + AuthGuard + policy files):**

```typescript
import * as casbin from '@midwayjs/casbin';
@Configuration({ imports: [passport, casbin] })

// config.default.ts
export default {
  casbin: {
    modelPath: join(appInfo.appDir, 'basic_model.conf'),
    policyAdapter: join(appInfo.appDir, 'basic_policy.csv'),
    usernameFromContext: (ctx) => ctx.user,   // read from passport auth
  },
};

// controller — declarative permission
import { AuthGuard, UsePermission, AuthActionVerb, AuthPossession } from '@midwayjs/casbin';

@Controller('/users')
@UseGuard(AuthGuard)
export class UserController {
  @UsePermission({ action: AuthActionVerb.READ, resource: 'users', possession: AuthPossession.ANY })
  @Get('/')
  async findAll() { return this.userService.findAll(); }

  @UsePermission({ action: AuthActionVerb.UPDATE, resource: 'users', possession: AuthPossession.OWN })
  @Patch('/:id')
  async update() { /* ... */ }
}

// imperative check (custom guard)
@Guard()
export class CustomGuard implements IGuard {
  @Inject() casbinEnforcerService: CasbinEnforcerService;
  async canActivate(ctx, clz, method) {
    return this.casbinEnforcerService.enforce(ctx.user, 'users', 'read');
  }
}
```

For distributed deployments use adapters: `@midwayjs/casbin-redis-adapter` (`createAdapter({ clientName })`) or `@midwayjs/casbin-typeorm-adapter` (`createAdapter({ dataSourceName })`, register `CasbinRule` entity). Add a watcher (`createWatcher`) for pub/sub policy sync — note the subscriber needs a **dedicated** Redis client.

Reference: [Midway Casbin](https://midwayjs.org/docs/extensions/casbin)

---

### 4.3 Configure CORS and Security Headers

**Impact: HIGH** — "Prevents cross-origin and XSS/CSRF attacks"

Use `@midwayjs/cross-domain` for CORS/JSONP and `@midwayjs/security` for CSRF, CSP, HSTS, and XSS escaping. Both are config-driven — enable them in `@Configuration({ imports })` and tune via their config keys. When `credentials: true`, `origin` cannot be `*`. Rotate the CSRF secret on login via `ctx.rotateCsrfSecret()`.

**Incorrect (no CORS, no security headers, reflecting raw input):**

```typescript
// ❌ no cors/security component — browsers block cross-origin, no CSRF/XSS protection
@Configuration({ imports: [koa] })
export class MainConfiguration {}

@Controller('/')
export class HomeController {
  @Get('/')
  async echo(@Query() q) { return q; }   // ❌ reflected XSS if rendered as HTML
}
```

**Correct (config-driven CORS + CSRF + headers + escape):**

```typescript
import * as crossDomain from '@midwayjs/cross-domain';
import * as security from '@midwayjs/security';

@Configuration({ imports: [koa, crossDomain, security] })
export class MainConfiguration {}

// config.default.ts
export default {
  // CORS — origin callback for dynamic allow-list
  cors: {
    origin: (ctx) => (allowedOrigins.includes(ctx.headers.origin) ? ctx.headers.origin : ''),
    allowMethods: ['GET', 'POST', 'PUT', 'DELETE'],
    credentials: true,          // with cookies; origin must NOT be '*' when true
    maxAge: 600,
  },
  // Security headers + CSRF
  security: {
    csrf: { enable: true, type: 'ctoken', headerName: 'x-csrf-token' },
    xframe: { enable: true, value: 'SAMEORIGIN' },
    hsts: { enable: true, maxAge: 365 * 24 * 3600, includeSubdomains: true },
    nosniff: { enable: true },
    csp: { enable: true, policy: { 'default-src': ["'self'"] } },
  },
} as MidwayConfig;

// rotate CSRF secret on login; escape reflected output
@Controller('/auth')
export class AuthController {
  @Inject() ctx: Context;

  @Post('/login')
  async login() {
    this.ctx.session = { user };
    this.ctx.rotateCsrfSecret();   // ✓ prevent token reuse across sessions
    return { success: true };
  }

  @Get('/search')
  async search(@Query('q') q: string) {
    // ✓ HTML-escape user input before returning/rendering
    return { safe: this.ctx.security.escape(q) };
  }
}
```

Reference: [Midway Cross-Domain](https://midwayjs.org/docs/extensions/cross_domain), [Midway Security](https://midwayjs.org/docs/extensions/security)

---

### 4.4 Use @Guard for Route-Level Authorization

**Impact: HIGH** — "Guards sit inside the route, know the matched handler, ideal for authz"

Guards (`@Guard`, since v3.6.0) run **after global middleware** but **before** the route method. Unlike middleware (which has no knowledge of the matched controller/method), a guard receives the exact `supplierClz` and `methodName`, making it the correct tool for authorization. Combine a custom metadata decorator with `getPropertyMetadata` for declarative role-based access control. Return `true` to proceed, `false` to throw a 403, or throw a custom error.

**Incorrect (manual auth checks repeated in every handler):**

```typescript
@Controller('/admin')
export class AdminController {
  @Get('/users')
  async getUsers(@Ctx() ctx: Context) {
    if (!ctx.user) throw new httpError.UnauthorizedError();   // ❌ repeated everywhere
    if (!ctx.user.roles.includes('admin')) throw new httpError.ForbiddenError();
    return this.adminService.getUsers();
  }
}
```

**Correct (declarative @UseGuard + custom @Role metadata):**

```typescript
import { Guard, IGuard, savePropertyMetadata, getPropertyMetadata, UseGuard } from '@midwayjs/core';
import { httpError } from '@midwayjs/core';
import { Context } from '@midwayjs/koa';

// 1. Custom metadata decorator carrying required roles
const ROLE_META_KEY = 'role:name';
export function Role(roleName: string | string[]): MethodDecorator {
  return (target, key, desc) => {
    savePropertyMetadata(ROLE_META_KEY, [].concat(roleName), target, key);
  };
}

// 2. The guard reads metadata and checks the user
@Guard()
export class AuthGuard implements IGuard<Context> {
  async canActivate(
    ctx: Context,
    supplierClz: any,
    methodName: string,
  ): Promise<boolean> {
    const roles = getPropertyMetadata<string[]>(ROLE_META_KEY, supplierClz, methodName);
    // no @Role metadata → public route
    if (!roles?.length) return true;
    if (!ctx.user?.role) throw new httpError.UnauthorizedError();
    if (!roles.includes(ctx.user.role)) throw new httpError.ForbiddenError();
    return true;
  }
}

// 3. Apply globally, then decorate routes
async onReady(_, app) { app.useGuard(AuthGuard); }

@Controller('/user')
export class UserController {
  @UseGuard(AuthGuard)
  @Role(['admin'])
  @Get('/roles')
  async getRoles() { return this.userService.getRoles(); }
}
```

Reference: [Midway Guards](https://midwayjs.org/docs/guard)

---

### 4.5 Implement Secure JWT Authentication

**Impact: HIGH** — "Essential for stateless API security"

Use the `@midwayjs/jwt` component for token sign/verify/decode. Keep secrets in config (never hardcoded), use short-lived access tokens, store the secret via `@Config('jwt.secret')`, and verify tokens in a middleware or guard that skips public routes via `match`/`ignore`. Never put sensitive data (passwords) in the JWT payload.

**Incorrect (hardcoded secret, sensitive payload, blocking every route):**

```typescript
// ❌ secret hardcoded in source
@Configuration({ imports: [jwt] })
export class MainConfiguration {}

// config
export default { jwt: { secret: 'my-secret-key', sign: { expiresIn: '7d' } } };

// ❌ password in the JWT payload
async login(user) {
  return { token: this.jwtService.signSync({ id: user.id, password: user.password }) };
}
```

**Correct (config-driven secret, minimal payload, route-aware middleware):**

```typescript
// config.default.ts — secret from environment, short TTL
export default {
  jwt: {
    secret: process.env.JWT_SECRET,           // never inline
    sign: { expiresIn: '2d' },
  },
} as MidwayConfig;

// src/middleware/jwt.middleware.ts
import { Middleware, IMiddleware, Config } from '@midwayjs/core';
import { Context, NextFunction } from '@midwayjs/koa';
import { httpError } from '@midwayjs/core';
import { JwtService } from '@midwayjs/jwt';

@Middleware()
export class JwtMiddleware implements IMiddleware<Context, NextFunction> {
  @Config('jwt') jwtConfig;
  @Inject() jwtService: JwtService;

  // skip auth on login/register routes
  ignore(ctx: Context): boolean {
    return /\/login|\/register|\/captcha/.test(ctx.path);
  }

  resolve() {
    return async (ctx: Context, next: NextFunction) => {
      const auth = ctx.headers.authorization || '';
      const token = auth.startsWith('Bearer ') ? auth.slice(7) : '';
      if (!token) throw new httpError.UnauthorizedError('missing token');
      try {
        // minimal payload: only sub + roles
        ctx.user = await this.jwtService.verify(token);
      } catch {
        throw new httpError.UnauthorizedError('invalid token');
      }
      return await next();
    };
  }
  static getName(): string { return 'jwt'; }
}

// src/configuration.ts
async onReady(_, app) { app.useMiddleware(JwtMiddleware); }
```

Reference: [Midway JWT](https://midwayjs.org/docs/extensions/jwt)

---

### 4.6 Use Passport for OAuth and Strategy-Based Authentication

**Impact: HIGH** — "Standardized third-party login (GitHub, Google, JWT, local)"

`@midwayjs/passport` (self-maintained since v3.4.0 — no need for the community `passport` package) standardizes authentication via strategies. Each strategy is a `@CustomStrategy()` class extending `PassportStrategy(Strategy, name?)`, implementing a Promise-based `validate()` (no `done` callback) and `getStrategyOptions()`. Apply strategies via a `PassportMiddleware(StrategyClass)` on routes. The authenticated user lands on `ctx.state.user`.

**Incorrect (hand-rolled OAuth, community passport package, done callback):**

```typescript
// ❌ community passport package (not needed in Midway)
import passport from 'passport';
import { Strategy as GitHubStrategy } from 'passport-github';

passport.use(new GitHubStrategy({ ... }, (accessToken, profile, done) => {
  done(null, profile);   // ❌ Midway uses async validate(), not done()
}));
```

**Correct (Midway passport: JWT strategy + middleware + route binding):**

```typescript
import * as passport from '@midwayjs/passport';
import * as jwt from '@midwayjs/jwt';
@Configuration({ imports: [jwt, passport] })

// config.default.ts
export default { jwt: { secret: process.env.JWT_SECRET }, passport: { session: false } } as MidwayConfig;

// src/strategy/jwt.strategy.ts
import { CustomStrategy, PassportStrategy } from '@midwayjs/passport';
import { Strategy, ExtractJwt } from 'passport-jwt';
import { Config } from '@midwayjs/core';

@CustomStrategy()
export class JwtStrategy extends PassportStrategy(Strategy, 'jwt') {
  @Config('jwt') jwtConfig;

  async validate(payload: any) {        // Promise-based, no done()
    return payload;                      // → becomes ctx.state.user
  }

  getStrategyOptions() {
    return { secretOrKey: this.jwtConfig.secret, jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken() };
  }
}

// src/middleware/jwt-passport.middleware.ts
import { PassportMiddleware } from '@midwayjs/passport';
@Middleware()
export class JwtPassportMiddleware extends PassportMiddleware(JwtStrategy) {
  getAuthenticateOptions() { return {}; }
}

// route
@Controller('/user')
export class UserController {
  @Inject() ctx: Context;

  @Post('/me', { middleware: [JwtPassportMiddleware] })
  async me() { return this.ctx.state.user; }   // authenticated user
}
```

For OAuth providers (GitHub/Google), swap the `Strategy` import and implement `validate(accessToken, refreshToken, profile)`. Disable session serialization for JWT (`passport.session = false`) to avoid "Failed to serialize user into session".

Reference: [Midway Passport](https://midwayjs.org/docs/extensions/passport)

---

### 4.7 Validate All Input with @midwayjs/validation (v4)

**Impact: HIGH** — "First line of defense — and v4 requires the component for auto DTO conversion"

In v4, automatic request-to-DTO conversion only happens when a validation component is enabled. Use `@midwayjs/validation` (the v4 successor to `@midwayjs/validate`) — it supports pluggable joi/zod/class-validator. Define DTOs with `@Rule(schema)` properties, and apply `@Validate` on handlers. The v4 component removed `RuleType`; use the validator's native API directly (`Joi.*`, `z.*`). Never trust raw `@Body`/`@Query` without a validated DTO.

**Incorrect (v3-style unvalidated input, deprecated RuleType):**

```typescript
// ❌ raw body with no validation — SQL injection / type confusion risk
@Post('/')
async create(@Body() body: any) {
  return this.userService.create(body);
}

// ❌ v3 deprecated package + RuleType (removed in v4 validation component)
import { Rule, RuleType } from '@midwayjs/validate';
export class UserDTO {
  @Rule(RuleType.string().required()) name: string;
}

// ❌ expecting auto-DTO conversion without a validation component (v4 disabled this)
async create(@Body() user: CreateUserDTO) { /* user is a plain object, not instance */ }
```

**Correct (v4 @midwayjs/validation with joi + validated DTO):**

```typescript
// configuration.ts
import * as validation from '@midwayjs/validation';
@Configuration({ imports: [koa, validation] })

// config.default.ts — register the joi validator
import joi from '@midwayjs/validation-joi';   // registers the validator adapter (NOT the Joi API itself)
export default {
  validation: { validators: { joi }, defaultValidator: 'joi' },
} as MidwayConfig;

// src/dto/user.dto.ts — @Rule uses Joi's native schema directly (import Joi from 'joi')
import { Rule } from '@midwayjs/validation';
import { getSchema } from '@midwayjs/validation';
import Joi from 'joi';   // import Joi from the 'joi' package, NOT from '@midwayjs/validation-joi'

export class CreateUserDTO {
  @Rule(Joi.string().min(2).max(100).required())
  name: string;

  @Rule(Joi.string().email().required())
  email: string;

  @Rule(Joi.number().integer().min(0).max(150))
  age: number;
}

// src/controller/user.controller.ts
import { Validate } from '@midwayjs/validation';

@Controller('/user')
export class UserController {
  @Post('/')
  @Validate()                          // triggers validation + DTO conversion
  async create(@Body() dto: CreateUserDTO) {
    // dto is a validated CreateUserDTO instance
    return this.userService.create(dto);
  }
}
```

For cascading/nested DTOs, use the arrow-function form because the validator isn't registered at class-eval time: `@Rule(() => getSchema(AddressDTO).required())`. Validation failures throw `MidwayValidationError` — catch it with a `@Catch(MidwayValidationError)` filter.

Reference: [Midway Validation (v4)](https://midwayjs.org/docs/extensions/validation)

---

## 5. Performance

**Section Impact: HIGH**

### 5.1 Enable the Async Context Manager for Request Context

**Impact: MEDIUM-HIGH** — "Lets Singletons access request-scoped data without scope downgrade"

A common need: a Singleton service must know the current request/user without being Request-scoped (which causes scope-downgrade freezes). v4 enables `async_local_storage` by default (merged into core). Enable the async context manager in config so any code can read the current context via `AsyncContextManager` / `ASYNC_CONTEXT_KEY`, decoupling request data from DI scope. `cool-admin-midway` requires `asyncContextManager.enable: true` for its multi-tenant system.

**Incorrect (storing request state on a Singleton, or scope-downgrade everywhere):**

```typescript
// ❌ shared mutable state across concurrent requests
@Singleton()
export class TenantService {
  private currentTenantId: number;   // clobbered by every request!
  setTenant(id: number) { this.currentTenantId = id; }
}

// ❌ forcing every consumer into Request scope just to read ctx
@Scope(ScopeEnum.Request)
export class AuditService {
  @Inject() ctx: Context;
  log(msg: string) { console.log(this.ctx.tenantId, msg); }
}
```

**Correct (async context manager — read request data from anywhere):**

```typescript
// config.default.ts
export default {
  asyncContextManager: { enable: true },   // REQUIRED for ALS-based ctx
} as MidwayConfig;

// a Singleton can read the current request's tenant via the async context
import { Provide, Singleton } from '@midwayjs/core';
import { getCurrentAsyncContextManager } from '@midwayjs/core';

// define a symbol key for your context value
const TENANT_ID_KEY = Symbol('tenantId');

@Singleton()
export class TenantService {
  getTenantId(): number | undefined {
    // AsyncContextManager is an interface; use getCurrentAsyncContextManager() to get the instance
    const manager = getCurrentAsyncContextManager();
    if (!manager?.active()) return undefined;          // no active context (e.g. outside request)
    return manager.active().getValue(TENANT_ID_KEY);   // read by symbol key
  }
}

// set values early in a middleware (runs within the request chain)
@Middleware()
export class TenantMiddleware implements IMiddleware<Context, NextFunction> {
  resolve() {
    return async (ctx: Context, next: NextFunction) => {
      const manager = getCurrentAsyncContextManager();
      manager?.active()?.setValue(TENANT_ID_KEY, extractTenant(ctx));
      await next();
    };
  }
}
```

Reference: [Midway Context Definition](https://midwayjs.org/docs/context_definition)

---

### 5.2 Use @midwayjs/cache-manager for Strategic Caching

**Impact: HIGH** — "Dramatically reduces DB load and response times"

In v4, prefer `@midwayjs/cache-manager` (based on cache-manager v5) over the legacy `@midwayjs/cache`. It supports multi-instance stores (`clients`), Redis-backed stores (reusing the redis component), multi-level caching, and method-level caching via the `@Caching(instanceName, ttl?)` decorator. Note the TTL unit difference: cache-manager v5 uses **milliseconds**. For cross-worker consistency, use a Redis store — in-memory cache is per-process.

**Incorrect (no caching for expensive queries, or per-process memory cache shared across workers):**

```typescript
@Provide()
export class ProductService {
  async getPopular() {
    // ❌ runs a heavy aggregation on EVERY request
    return this.repo.createQueryBuilder('p')
      .leftJoin('p.orders', 'o')
      .groupBy('p.id').orderBy('COUNT(o.id)', 'DESC').limit(20).getMany();
  }
}
```

**Correct (Redis-backed store + @Caching decorator + invalidation):**

```typescript
// config.default.ts — Redis store reuses the redis component instance
import { createRedisStore } from '@midwayjs/cache-manager';
export default {
  cacheManager: {
    clients: {
      default: { store: createRedisStore('default'), options: { ttl: 60000 } }, // ms
    },
  },
  redis: { clients: { default: { port: 6379, host: '127.0.0.1' } } },
} as MidwayConfig;

// service — imperative access for granular control + invalidation
import { CachingFactory, MidwayCache, Caching } from '@midwayjs/cache-manager';
import { InjectClient, Provide } from '@midwayjs/core';

@Provide()
export class ProductService {
  @InjectClient(CachingFactory, 'default')
  cache: MidwayCache;

  async getPopular() {
    const cached = await this.cache.get('products:popular');
    if (cached) return cached;
    const products = await this.fetchPopular();
    await this.cache.set('products:popular', products, 5 * 60 * 1000); // 5 min
    return products;
  }

  async update(id: number, dto: any) {
    const product = await this.repo.save({ id, ...dto });
    await this.cache.del('products:popular');   // invalidate on change
    return product;
  }
}

// method-level caching via decorator — simplest form
@Provide()
export class ConfigService {
  @Caching('default', 10000)          // cache result for 10s
  async getAppConfig() { return this.repo.find(); }

  // dynamic key via callback: return null to skip caching
  @Caching('default', (ctx) => ctx.methodArgs[0]?.id ?? null, 30000)
  async getUser(id: number) { return this.repo.findOne({ where: { id } }); }
}
```

Reference: [Midway Cache Manager](https://midwayjs.org/docs/extensions/caching)

---

### 5.3 Subscribe to Changing Data with DataListener

**Impact: MEDIUM** — "Caches remote/volatile data behind a stable read API"

`DataListener<T>` (from `@midwayjs/core`) is a base class for the "data subscription" pattern: a Singleton holds a piece of data that updates periodically (from a remote API, DB, or event stream), while consumers read the latest value via a stable `getData()` API without worrying about freshness. Implement `initData()` (initial/async fetch) and `onData(setData)` (periodic update); clean up timers/connections in `destroyListener()`.

**Incorrect (every consumer fetches remote data independently, or stale cached value):**

```typescript
@Provide()
export class UserService {
  async getConfig() {
    return await this.httpService.get('/remote-config');  // ❌ network call every request
  }
}

// OR a stale singleton with no update mechanism
@Singleton()
export class ConfigCache {
  private data = {};   // ❌ never refreshes after first load
}
```

**Correct (DataListener Singleton + periodic update + cleanup):**

```typescript
import { Provide, Scope, ScopeEnum, DataListener, Inject } from '@midwayjs/core';

@Provide()
@Scope(ScopeEnum.Singleton)
export class RemoteConfigListener extends DataListener<Record<string, any>> {
  @Inject() httpService: HttpService;
  private timer: NodeJS.Timeout;

  // initial data (can be async)
  async initData() {
    const { data } = await this.httpService.get('/remote-config');
    return data;
  }

  // periodic update — call setData() to refresh the cached value
  onData(setData: (data: Record<string, any>) => void) {
    this.timer = setInterval(async () => {
      const { data } = await this.httpService.get('/remote-config');
      setData(data);   // ✓ updates the internal cache; consumers see it instantly
    }, 60_000);         // refresh every minute
  }

  // clean up resources on shutdown
  async destroyListener() {
    clearInterval(this.timer);
  }
}

// consumer — always reads the latest value, no network calls
@Provide()
export class UserService {
  @Inject() remoteConfig: RemoteConfigListener;

  async getFeatureFlags() {
    return this.remoteConfig.getData();   // ✓ instant read, always fresh
  }
}
```

The pattern hides volatility behind a stable API: the listener manages freshness (timers, subscriptions), while business code simply calls `getData()`. Pair with `@Autoload` if the listener should self-start without `onReady` wiring.

Reference: [Midway Data Listener](https://midwayjs.org/docs/data_listener)

---

### 5.4 Use Async Lifecycle Hooks Correctly

**Impact: HIGH** — "Misuse blocks startup or causes race conditions"

Midway lifecycle hooks (`onConfigLoad`, `onReady`, `onServerReady`, `onStop`, `onHealthCheck`) are async and awaited by the framework. Use the right hook for the right job: `onConfigLoad` to merge remote config, `onReady` for container setup (DB connect, register middleware), `onServerReady` to access the running server/framework, `onStop` for cleanup. v4 adds built-in timeouts (`core.readyTimeout` etc.) and an `AbortController` to prevent startup hangs. Never do heavy synchronous work in a constructor — move it to `@Init` or `onReady`.

**Incorrect (fire-and-forget async, blocking constructor, wrong hook):**

```typescript
@Provide()
export class DatabaseService {
  constructor() {
    // ❌ BLOCKS module instantiation synchronously
    this.config = fs.readFileSync('config.json');
  }
}

@Configuration({})
export class MainConfiguration {
  async onReady() {
    connectDatabase();   // ❌ not awaited — app starts before DB is ready
  }
}
```

**Correct (awaited hooks, async @Init, v4 AbortController awareness):**

```typescript
@Provide()
export class DatabaseService {
  @Config('mysql') mysqlConfig;
  private pool: any;

  @Init()
  async init() {                       // ✓ async, awaited by container
    this.pool = await mysql.createPool(this.mysqlConfig);
  }

  @Destroy()
  async destroy() { await this.pool.end(); }
}

@Configuration({})
export class MainConfiguration implements ILifeCycle {
  // onConfigLoad's RETURN VALUE is merged into global config
  async onConfigLoad(container: IMidwayContainer) {
    const remote = await fetchRemoteConfig();
    return { mysql: remote.mysql };
  }

  // onReady: container ready, register middleware/filters
  async onReady(container: IMidwayContainer, app: IMidwayApplication) {
    app.useMiddleware(JwtMiddleware);
    app.useFilter([DefaultErrorFilter]);
  }

  // onServerReady: server is listening — safe to get framework/server
  async onServerReady(container: IMidwayContainer) {
    const framework = await container.getAsync(koa.Framework);
    const server = framework.getServer();
  }

  // onStop: cleanup (v4 timeout via core.stopTimeout, unlimited by default)
  async onStop(container: IMidwayContainer) {
    await closeAllConnections();
  }

  // onHealthCheck: v4 timeout 1s (core.healthCheckTimeout)
  async onHealthCheck(container: IMidwayContainer): Promise<HealthResult> {
    return { status: true, reason: 'ok' };
  }
}
```

Reference: [Midway Lifecycle](https://midwayjs.org/docs/lifecycle)

---

### 5.5 Retry Fallible Operations with retryWithAsync

**Impact: MEDIUM** — "Built-in retry for transient failures without external libs"

Midway provides `retryWithAsync(fn, retryTimes, options)` / `retryWith(fn, retryTimes)` from `@midwayjs/core` for retrying transient failures. `retryTimes` is the **number of extra retries** (total executions = 1 + retryTimes). Use the `receiver` option to bind `this` for class methods (avoids manual `.bind()`), `retryInterval` for backoff delay, and `throwOriginError` to surface the original error instead of `MidwayRetryExceededMaxTimesError`.

**Incorrect (manual retry loops, no backoff, wrong error handling):**

```typescript
async function callWithRetry() {
  for (let i = 0; i < 3; i++) {          // ❌ manual loop, no delay, no error chaining
    try { return await flakyApi(); }
    catch (e) { if (i === 2) throw e; }
  }
}
```

**Correct (retryWithAsync + receiver binding + interval + typed errors):**

```typescript
import { retryWithAsync, MidwayRetryExceededMaxTimesError } from '@midwayjs/core';

@Provide()
export class PaymentService {
  async charge(orderId: string) {
    // wrap a class method: receiver binds `this`, 2 retries with 500ms backoff
    const charge = retryWithAsync(this.callPaymentGateway, 2, {
      receiver: this,
      retryInterval: 500,
    });
    try {
      return await charge(orderId);
    } catch (err) {
      if (err instanceof MidwayRetryExceededMaxTimesError) {
        this.logger.error('payment failed after retries; cause:', err.cause);
        throw new httpError.BadGatewayError('payment gateway unavailable');
      }
      throw err;
    }
  }

  private async callPaymentGateway(orderId: string) {
    return this.httpService.post('/charge', { orderId });
  }
}

// sync variant (no retryInterval)
import { retryWith } from '@midwayjs/core';
const parseSafe = retryWith(JSON.parse, 1);
```

Use `throwOriginError: true` when you want the underlying error (not the wrapper) to propagate.

Reference: [Midway Retry](https://midwayjs.org/docs/retry)

---

### 5.6 Offload CPU Work to Thread Pools (@midwayjs/piscina)

**Impact: MEDIUM** — "Keep the event loop free for heavy CPU tasks"

`@midwayjs/piscina` offloads CPU-intensive work to worker threads via the `piscina` pool. Two modes: **plain worker** (`run({ handler, payload })`) for low-overhead function execution, and **container mode** (`runInContainer('taskName', payload)`) where the worker runs a separate Midway app with DI. Configure pool size via `piscina.client`; the main app **must ignore the worker directory** in its detector to avoid double-loading. Data is serialized (structured clone) — use `transferList` for large buffers.

**Incorrect (CPU-heavy work on the event loop):**

```typescript
@Get('/hash')
async hash(@Query('input') input: string) {
  // ❌ blocks the event loop for all requests while hashing
  return heavyHash(input);
}
```

**Correct (offload to a thread pool with cancellation):**

```typescript
import * as piscina from '@midwayjs/piscina';
@Configuration({
  imports: [piscina],
  detector: new CommonJSFileDetector({ ignore: ['**/worker/**'] }),  // ✓ ignore worker dir
})

// config.default.ts
export default {
  piscina: {
    client: {
      workerFile: join(__dirname, '../worker/index'),   // no extension; searches .js→.ts
      minThreads: 1, maxThreads: 4, idleTimeout: 60000,
    },
    // OR multi-pool: clients: { compute: {...}, image: {...} }
  },
} as MidwayConfig;

// worker/task (container mode) — src/worker/index.ts uses functional API
import { defineConfiguration } from '@midwayjs/core/functional';
export default defineConfiguration({ namespace: 'worker', detector: new CommonJSFileDetector(), imports: [piscina] });

// src/worker/calculate.task.ts
import { PiscinaTask, IPiscinaTask } from '@midwayjs/piscina';
@PiscinaTask('calculate')
export class CalculateTask implements IPiscinaTask {
  async execute(payload: { a: number; b: number }) { return payload.a + payload.b; }
}

// main app — run with cancellation
import { PiscinaService } from '@midwayjs/piscina';
@Provide()
export class ComputeService {
  @Inject() piscinaService: PiscinaService;

  async compute() {
    return this.piscinaService.runInContainer('calculate', { a: 5, b: 6 });
  }
  async cancelable() {
    const ac = new AbortController();
    setTimeout(() => ac.abort(), 3000);
    return this.piscinaService.run({ handler: 'longTask', payload: { ms: 10000 } }, { signal: ac.signal });
  }
}
```

No functions/class instances across the thread boundary (structured clone only). Use `transferList` for large `ArrayBuffer`/`Buffer`.

Reference: [Midway Piscina](https://midwayjs.org/docs/extensions/piscina)

---

## 6. Testing

**Section Impact: MEDIUM-HIGH**

### 6.1 Test FaaS Functions with createFunctionApp

**Impact: MEDIUM-HIGH** — "Correct HTTP + non-HTTP trigger testing patterns"

Use `createFunctionApp<Framework>()` from `@midwayjs/mock` (the FaaS-specific variant of `createApp`) to boot a function app for tests. HTTP/API-Gateway triggers are tested with `createHttpRequest(app)` (supertest-style). Non-HTTP triggers (Timer/OS/MQ/Event) **cannot** be triggered via `npm run dev` — test them by getting the instance via `app.getServerlessInstance(Class)` and calling the method directly with mocked events from `@midwayjs/fc-starter`. Always pair with `close(app)` for teardown.

**Incorrect (mocking process.argv, only testing happy-path HTTP):**

```typescript
// ❌ mocking argv instead of booting the real function app
process.argv = ['node', 'bootstrap', '--trigger', 'timer'];
// ❌ non-HTTP triggers never tested at all
```

**Correct (createFunctionApp + HTTP supertest + non-HTTP instance calls + mock events):**

```typescript
import { createFunctionApp, close, createHttpRequest } from '@midwayjs/mock';
import { Framework, Application } from '@midwayjs/faas';
import { mockContext, mockTimerEvent, mockOSSEvent } from '@midwayjs/fc-starter';
import { HelloService } from '../src/function/hello';

describe('FaaS function', () => {
  let app: Application;

  beforeAll(async () => {
    app = await createFunctionApp<Framework>(join(__dirname, '../'), {
      initContext: mockContext(),   // simulate the Aliyun FC Context struct
    });
  });
  afterAll(async () => { await close(app); });

  // HTTP trigger — real HTTP request
  it('GET / returns greeting', async () => {
    const res = await createHttpRequest(app).get('/').query({ name: 'zhang' });
    expect(res.text).toEqual('hello zhang');
  });

  // Timer trigger — get instance + call with mocked event
  it('handles timer event', async () => {
    const instance = await app.getServerlessInstance<HelloService>(HelloService);
    await instance.handleTimer(mockTimerEvent());
    // assert side effects
  });

  // OSS trigger — mocked event
  it('handles OSS event', async () => {
    const instance = await app.getServerlessInstance<HelloService>(HelloService);
    await instance.handleOSS(mockOSSEvent());
  });

  // override mock context fields
  it('with custom function metadata', async () => {
    const customApp = await createFunctionApp<Framework>(join(__dirname, '../'), {
      initContext: Object.assign(mockContext(), { function: { name: 'myFn', handler: 'x.y' } }),
    });
    await close(customApp);
  });
});
```

Reference: [Midway Serverless Testing](https://midwayjs.org/docs/serverless/serverless_testing)

---

### 6.2 Use Supertest for HTTP E2E Testing

**Impact: HIGH** — "Validates the full request/response cycle"

End-to-end tests use `createHttpRequest(app)` (wrapping supertest) to make real HTTP requests against the booted app — exercising routes, middleware, guards, pipes, and filters together. Always set up the app in `beforeAll`, tear it down in `afterAll`, and assert both status codes and response bodies. Use `mockClassProperty`/`mockContext` to stub external dependencies without hitting real services.

**Incorrect (only unit-testing controllers, no route/filter coverage):**

```typescript
describe('UserController', () => {
  it('returns users', async () => {
    const ctrl = new UserController({ findAll: async () => [] } as any);
    const result = await ctrl.findAll();
    expect(result).toEqual([]);
    // ❌ never tests routing, validation pipe, or error filter
  });
});
```

**Correct (real HTTP requests with supertest + dependency mocking):**

```typescript
import { createApp, close, createHttpRequest, mockClassProperty } from '@midwayjs/mock';
import { Framework, Application } from '@midwayjs/koa';
import { UserService } from '../src/service/user.service';

describe('UserController (e2e)', () => {
  let app: Application;

  beforeAll(async () => { app = await createApp<Framework>(); });
  afterAll(async () => { await close(app); });

  describe('POST /user', () => {
    it('creates a user (201)', async () => {
      const res = await createHttpRequest(app)
        .post('/user')
        .send({ name: 'John', email: 'john@test.com', age: 30 });
      expect(res.status).toBe(200);
      expect(res.body.data).toHaveProperty('id');
    });

    it('rejects invalid email (validation error)', async () => {
      const res = await createHttpRequest(app)
        .post('/user')
        .send({ name: 'John', email: 'not-an-email', age: 30 });
      expect(res.status).toBe(500);          // caught by error filter
      expect(res.body.message).toContain('email');
    });
  });

  it('mocks an external service instead of calling it', async () => {
    mockClassProperty(UserService, 'findById', async (id: number) => ({ id, name: 'mock' }));
    const res = await createHttpRequest(app).get('/user/1');
    expect(res.body.data.name).toBe('mock');
  });
});
```

Run serially with `jest --runInBand` to avoid port conflicts, and set `process.env.MIDWAY_TS_MODE='true'` in a `jest.setup.js` for ts-mode.

Reference: [Midway Testing](https://midwayjs.org/docs/testing)

---

### 6.3 Use @midwayjs/mock for App Bootstrap Testing

**Impact: HIGH** — "Proper DI-isolated test environments"

Use `@midwayjs/mock` to create isolated test apps with real DI wiring. `createApp<Framework>()` boots the full app (matching production), `createLightApp` boots a minimal container, and `close(app)` tears it down (auto-restoring all mocks). Get services from the container with `getApplicationContext().getAsync(Class)`. For request-scoped services that read `ctx`, resolve them via `app.createAnonymousContext().requestContext.getAsync(Class)`. v4 dropped the trailing array param — pass extra components via `options.imports`.

**Incorrect (manual instantiation bypassing DI, no teardown):**

```typescript
describe('UserService', () => {
  it('creates user', async () => {
    const service = new UserService(new UserRepository()); // ❌ bypasses DI, hits real DB
    const user = await service.create({ name: 'x' });
    // ❌ no app teardown, no isolation, mocks never restored
  });
});
```

**Correct (createApp + container resolution + mock groups + teardown):**

```typescript
import { createApp, close } from '@midwayjs/mock';
import { Framework, Application } from '@midwayjs/koa';
import { UserService } from '../src/service/user.service';

describe('UserService', () => {
  let app: Application;

  beforeAll(async () => {
    // boots the real app (reads configuration.ts); v4 extra components via options.imports
    app = await createApp<Framework>(process.cwd(), { imports: [] });
  });

  afterAll(async () => {
    await close(app, { sleep: 50 });   // closes app + restores all mocks
  });

  it('finds user by id', async () => {
    // resolve from container — full DI graph
    const userService = await app
      .getApplicationContext()
      .getAsync<UserService>(UserService);

    // for a Request-scoped service that needs ctx:
    // const svc = await app.createAnonymousContext().requestContext.getAsync(UserService);

    const user = await userService.findById(1);
    expect(user).toBeDefined();
  });
});

// Light app for fast component-only tests
import { createLightApp } from '@midwayjs/mock';
it('tests a component in isolation', async () => {
  const app = await createLightApp('', { imports: [myComponent] });
  const svc = await app.getApplicationContext().getAsync(MyService);
  await close(app);
});
```

Reference: [Midway Testing](https://midwayjs.org/docs/testing), [Mock](https://midwayjs.org/docs/mock)

---

## 7. Database & ORM

**Section Impact: MEDIUM-HIGH**

### 7.1 Use MikroORM with the Correct v6/v7 Patterns

**Impact: MEDIUM** — "v6 removed Repository.persist; v7 is a separate package"

`@midwayjs/mikro` pins MikroORM **v6**; for v7 (native ESM) use the separate `@midwayjs/mikro7` package — never mix them. Use the datasource config form (`mikro.dataSource.<name>`), pass the **driver class** (not a string) in v6, and inject via `@InjectRepository(Entity)` / `@InjectEntityManager()`. **Critical v6 change:** `EntityRepository.persist/flush` were removed — use `EntityManager.persist()` + `em.flush()`.

**Incorrect (v5 string driver, removed Repository.persist):**

```typescript
export default { mikro: { dataSource: { default: { type: 'sqlite' /* ❌ v5 string */ } } } };

@Provide()
export class BookService {
  @InjectRepository(Book) repo: EntityRepository<Book>;
  async create() {
    this.repo.persist(book);   // ❌ removed in MikroORM v6
    await this.repo.flush();   // ❌ removed in MikroORM v6
  }
}
```

**Correct (v6 driver class + EntityManager + datasource config):**

```typescript
import * as mikro from '@midwayjs/mikro';
@Configuration({ imports: [mikro] })

// config.default.ts — v6 passes the driver CLASS
import { SqliteDriver } from '@mikro-orm/sqlite';
export default {
  mikro: {
    dataSource: {
      default: {
        dbName: join(__dirname, '../../test.sqlite'),
        driver: SqliteDriver,              // ✓ class, not string
        allowGlobalContext: true,          // needed outside request scope
        entities: [Author, Book, '**/*.entity.{j,t}s'],
        logger: 'mikroLogger',
      },
    },
  },
} as MidwayConfig;

// service — EntityManager.persist + flush (v6 pattern)
import { InjectEntityManager, InjectRepository } from '@midwayjs/mikro';
import { EntityRepository, EntityManager } from '@mikro-orm/core';
@Provide()
export class BookService {
  @InjectRepository(Book) bookRepo: EntityRepository<Book>;
  @InjectEntityManager() em: EntityManager;

  async create() {
    const book = new Book({ title: 'b1' });
    this.em.persist(book);          // ✓ EntityManager, not Repository
    await this.em.flush();          // ✓ single flush commits all
    return book;
  }
  async findAll() {
    return this.bookRepo.findAll({ populate: ['author'], limit: 20 });
  }
}
```

For MikroORM **v7** (ESM): switch to `@midwayjs/mikro7`, import `@Entity` from `@mikro-orm/decorators/legacy` (with legacy TS decorators), and set `metadataProvider: ReflectMetadataProvider`.

Reference: [Midway MikroORM](https://midwayjs.org/docs/extensions/mikro)

---

### 7.2 Use MongoDB with Typegoose/Mongoose Datasource Form

**Impact: MEDIUM-HIGH** — "v4 removed EntityModel; use dataSource config form"

In v4, MongoDB uses the datasource config form (`mongoose.dataSource.<name>`); the old `@EntityModel` decorator and `mongoose.clients` form are **removed**. Use `@midwayjs/typegoose` for class-based entities (`@prop()` from `@typegoose/typegoose`) injected via `@InjectEntityModel(Entity)`. For raw mongoose, get the connection via `MongooseDataSourceManager.getDataSource()` and define schemas manually.

**Incorrect (v3 EntityModel + clients config):**

```typescript
import { EntityModel } from '@midwayjs/typegoose';   // ❌ removed in v4
@EntityModel()
export class User { /* ... */ }

export default { mongoose: { clients: { default: { uri: '...' } } } };  // ❌ v3 clients form
```

**Correct (v4 dataSource form + typegoose entity + injection):**

```typescript
import * as typegoose from '@midwayjs/typegoose';
@Configuration({ imports: [typegoose] })

// src/entity/user.entity.ts — @prop from @typegoose/typegoose
import { prop } from '@typegoose/typegoose';
export class User {
  @prop() name?: string;
  @prop({ type: () => [String] }) jobs?: string[];
}

// config.default.ts — dataSource form
export default {
  mongoose: {
    dataSource: {
      default: {
        uri: 'mongodb://localhost:27017/test',
        options: { useNewUrlParser: true, useUnifiedTopology: true, user: '...', pass: '...' },
        entities: [User],    // declare entities per datasource
      },
    },
  },
} as MidwayConfig;

// service — inject typegoose model
import { InjectEntityModel } from '@midwayjs/typegoose';
import { ReturnModelType } from '@typegoose/typegoose';
@Provide()
export class UserService {
  @InjectEntityModel(User) userModel: ReturnModelType<typeof User>;

  async create() { return this.userModel.create({ name: 'John', jobs: ['dev'] } as User); }
  async find(id: string) { return this.userModel.findById(id).exec(); }
}

// raw mongoose: get the connection, define schemas yourself
import { MongooseDataSourceManager } from '@midwayjs/mongoose';
@Provide()
export class RawService {
  @Inject() dataSourceManager: MongooseDataSourceManager;
  @Init() async init() { this.conn = this.dataSourceManager.getDataSource('default'); }
  async invoke() {
    const schema = new Schema({ name: { type: String, required: true } });
    const Model = this.conn.model('User', schema);
    return new Model({ name: 'Bill' }).save();
  }
}
```

Set global schema options in `onConfigLoad` via `Typegoose.setGlobalOptions(...)`. Version matrix: MongoDB 6.x → mongoose ^7 + typegoose ^10.

Reference: [Midway MongoDB (Typegoose)](https://midwayjs.org/docs/extensions/mongodb)

---

### 7.3 Manage Multiple Datasources and Transactions

**Impact: HIGH** — "Correct multi-DB injection and atomic multi-step operations"

For multi-database setups, name each datasource in config and target it explicitly: `@InjectEntityModel(Entity, 'dsName')` or `@InjectDataSource('dsName')`. For atomic multi-step operations, obtain the `DataSource` via `@InjectDataSource()` and call `dataSource.transaction(async (manager) => {...})` — if the callback throws, everything rolls back. Avoid running separate `save` calls outside a transaction when they must succeed together.

**Incorrect (separate saves without a transaction — partial updates on failure):**

```typescript
@Provide()
export class OrderService {
  @InjectEntityModel(Order) orderRepo: Repository<Order>;
  @InjectEntityModel(OrderItem) itemRepo: Repository<OrderItem>;

  async createOrder(userId: number, items: any[]) {
    const order = await this.orderRepo.save({ userId, status: 'pending' });
    for (const item of items) {
      await this.itemRepo.save({ orderId: order.id, ...item });
    }
    // ❌ if a later save fails, the order exists with missing items
    await this.paymentService.charge(order.id);   // ❌ payment + DB inconsistent on error
    return order;
  }
}
```

**Correct (multi-datasource injection + DataSource.transaction for atomicity):**

```typescript
import { Provide } from '@midwayjs/core';
import { InjectDataSource, InjectEntityModel } from '@midwayjs/typeorm';
import { DataSource, EntityManager } from 'typeorm';

@Provide()
export class OrderService {
  // default datasource repository
  @InjectEntityModel(Order) orderRepo: Repository<Order>;
  // named datasource (multi-DB)
  @InjectEntityModel(User, 'analytics') analyticsUserRepo: Repository<User>;
  // raw datasource for transactions
  @InjectDataSource() defaultDataSource: DataSource;
  @InjectDataSource('analytics') analyticsDataSource: DataSource;

  async createOrder(userId: number, items: any[]) {
    // atomic: all saves + payment must succeed together, else full rollback
    return this.defaultDataSource.transaction(async (manager: EntityManager) => {
      const order = await manager.save(Order, { userId, status: 'pending' });
      for (const item of items) {
        await manager.save(OrderItem, { orderId: order.id, ...item });
      }
      await this.paymentService.chargeWithManager(manager, order.id);
      return order;
    });
  }
}
```

Config for multiple datasources:
```typescript
export default {
  typeorm: {
    dataSource: {
      default: { type: 'mysql', host: '...', entities: [Order, OrderItem] },
      analytics: { type: 'postgres', host: '...', entities: [User] },
    },
  },
} as MidwayConfig;
```

Reference: [Midway TypeORM](https://midwayjs.org/docs/extensions/orm), [Data Source Management](https://midwayjs.org/docs/data_source)

---

### 7.4 Access Object Storage via Service Factories (COS/OSS/TableStore)

**Impact: MEDIUM** — "Correct DI-managed multi-instance cloud storage"

Cloud storage components (`@midwayjs/cos`, `@midwayjs/oss`, `@midwayjs/tablestore`) all follow the Service Factory pattern: configure via `<name>.client` (single) or `<name>.clients` (multi), inject the default `XxxService`, and use `XxxServiceFactory.get('name')` for named instances. Never hardcode SDK credentials — put them in config (env-driven). OSS supports cluster and STS modes; TableStore re-exports SDK types directly.

**Incorrect (raw SDK, manual client, hardcoded credentials):**

```typescript
import OSS from 'ali-oss';   // ❌ bypasses DI, no lifecycle, hardcoded
const client = new OSS({ accessKeyId: 'xxx', accessKeySecret: 'yyy' });
await client.put('file', buffer);
```

**Correct (DI-managed service factory + multi-instance config):**

```typescript
// === Aliyun OSS ===
import * as oss from '@midwayjs/oss';
@Configuration({ imports: [oss] })
// config.default.ts
export default {
  oss: {
    default: { timeout: 5000 },
    clients: {
      default: { bucket: 'app-bucket', region: 'oss-cn-hangzhou', accessKeyId: process.env.OSS_KEY, accessKeySecret: process.env.OSS_SECRET },
      backup: { bucket: 'backup-bucket', region: 'oss-cn-hangzhou', accessKeyId: process.env.OSS_KEY, accessKeySecret: process.env.OSS_SECRET },
    },
  },
} as MidwayConfig;

import { OSSService, OSSServiceFactory } from '@midwayjs/oss';
@Provide()
export class FileService {
  @Inject() ossService: OSSService;                          // default
  @Inject() ossFactory: OSSServiceFactory;
  async upload(buf: Buffer, key: string) { return this.ossService.put(key, buf); }
  async backup(buf: Buffer, key: string) { return this.ossFactory.get('backup').put(key, buf); }
  // STS mode: inject OSSSTSService, call assumeRole(roleArn)
}

// === Tencent COS ===
import * as cos from '@midwayjs/cos';
// config: cos.clients: { default: { SecretId, SecretKey } }
import { COSService } from '@midwayjs/cos';
@Provide()
export class CosService {
  @Inject() cosService: COSService;
  async upload() { return this.cosService.sliceUploadFile({ Bucket, Region, Key, FilePath }); }
}

// === Aliyun TableStore ===
import * as tablestore from '@midwayjs/tablestore';
import { TableStoreService, Long } from '@midwayjs/tablestore';
@Provide()
export class TsService {
  @Inject() tsService: TableStoreService;
  async get(id: number) {
    return this.tsService.getRow({ tableName: 't', primaryKey: [{ gid: Long.fromNumber(id) }] });
  }
}
```

Dynamic instance creation: `factory.createInstance(config, 'bucket3')`.

Reference: [Midway OSS](https://midwayjs.org/docs/extensions/oss), [COS](https://midwayjs.org/docs/extensions/cos), [TableStore](https://midwayjs.org/docs/extensions/tablestore)

---

### 7.5 Use Prisma ORM in Fullstack Projects

**Impact: MEDIUM** — "Schema-first ORM with generated typed client"

Prisma is the recommended ORM for Midway Hooks fullstack projects — it provides a schema-first workflow (`schema.prisma`), a generated fully-typed client (`@prisma/client`), and end-to-end type safety when combined with the Hooks zero-API RPC. Initialize the client as a module-level singleton in `src/api/prisma.ts`, then use it directly inside `Api()` handlers — no DI decorator needed. Set the engine mirror for poor-network regions.

**Incorrect (untyped queries, manual SQL, no schema):**

```typescript
// ❌ no schema, no type safety, manual SQL
@Provide()
export class UserService {
  async getUsers() {
    return db.query('SELECT * FROM users');  // ❌ untyped results
  }
}
```

**Correct (Prisma schema + generated client + Hooks API):**

```prisma
// prisma/schema.prisma
model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  name  String?
  posts Post[]
}
```

```typescript
// src/api/prisma.ts — singleton client (module-level, no DI needed)
import { PrismaClient } from '@prisma/client';
export const prisma = new PrismaClient();

// src/api/user.ts — use in Hooks API handlers with Validate
import { Api, Get, Post, Validate } from '@midwayjs/hooks';
import { z } from 'zod';
import { prisma } from './prisma';

export default Api(Get(), async () => {
  return prisma.post.findMany({
    where: { published: true },
    include: { author: true },   // ✓ fully typed relation
  });
});

export const signUp = Api(
  Post(),
  Validate(z.string(), z.string().email()),   // positional zod validation
  async (name: string, email: string) => {
    return prisma.user.create({ data: { name, email } });
  },
);

// frontend: typed zero-API call
// import { signUp } from '../api/user';
// const user = await signUp('John', 'test@test.com');  // fully typed
```

For poor network, set the engine mirror before install:
```bash
PRISMA_ENGINES_MIRROR=https://registry.npmmirror.com/-/binary/prisma/
```

Prisma works best in the Hooks/functional fullstack context. For class-based standard projects, prefer TypeORM/Sequelize/MikroORM (which have dedicated Midway component wrappers).

Reference: [Midway Hooks + Prisma](https://midwayjs.org/docs/hooks/prisma), [Prisma Docs](https://www.prisma.io/)

---

### 7.6 Use the Redis Service Factory Correctly

**Impact: MEDIUM-HIGH** — "Correct single/cluster/multi-instance patterns"

The `@midwayjs/redis` component exposes a default `RedisService` and a `RedisServiceFactory` for multiple instances. Configure via `redis.client` (single), `redis.clients` (multi), or `redis.client.cluster` (cluster mode). Inject the default service with `@Inject()`, and named instances with `@InjectClient(RedisServiceFactory, 'name')`. Always set TTL explicitly and prefer Redis stores over per-process memory for cross-worker consistency.

**Incorrect (accessing redis directly, ignoring cluster mode, magic instance names):**

```typescript
// ❌ raw ioredis, bypassing the component's lifecycle/health checks
import Redis from 'ioredis';
const redis = new Redis('redis://localhost');   // ❌ not managed, not closed on shutdown

// ❌ magic string instance name
@InjectClient(RedisServiceFactory, 'instance1') redis1: RedisService;
```

**Correct (component-managed service + typed instance names + cluster config):**

```typescript
// config.default.ts
export default {
  redis: {
    // single client
    client: { port: 6379, host: '127.0.0.1', password: process.env.REDIS_PASSWORD, db: 0 },
    // OR cluster
    // client: { cluster: true, nodes: [{host,port}], redisOptions: { password: '...' } },
    // OR multiple named instances
    // clients: { cache: {...}, session: {...} },
  },
} as MidwayConfig;

// default service
import { Provide, Inject, InjectClient } from '@midwayjs/core';
import { RedisService, RedisServiceFactory } from '@midwayjs/redis';

@Provide()
export class CacheService {
  @Inject() redisService: RedisService;

  async getCached(key: string) {
    const cached = await this.redisService.get(key);
    if (cached) return JSON.parse(cached);
    const data = await this.fetch(key);
    await this.redisService.set(key, JSON.stringify(data), 'EX', 60);  // 60s TTL
    return data;
  }
}

// named instance via factory
@Provide()
export class SessionService {
  @InjectClient(RedisServiceFactory, 'session') sessionRedis: RedisService;
}
```

Reference: [Midway Redis](https://midwayjs.org/docs/extensions/redis)

---

### 7.7 Follow Sequelize v4 Patterns (@Table replaces @BaseTable)

**Impact: HIGH** — "v4 removed @BaseTable and changed config form"

In v4, `@midwayjs/sequelize` changed: `@BaseTable` is **removed** (use `@Table` from `sequelize-typescript`), config uses the datasource form (`sequelize.dataSource.<name>`), and `dialect` is now **required**. Enable `repositoryMode` for multi-datasource safety (static methods stop working once enabled). Inject repositories with `@InjectRepository(Model)` from `@midwayjs/sequelize`.

**Incorrect (v3 @BaseTable, flat config, missing dialect):**

```typescript
import { BaseTable } from '@midwayjs/sequelize';   // ❌ removed in v4
@BaseTable()
export class User extends Model {}

export default { sequelize: { database: 'test', username: 'root' } };  // ❌ v3 flat form, no dialect
```

**Correct (v4 @Table + datasource config + repository injection):**

```typescript
import * as sequelize from '@midwayjs/sequelize';
@Configuration({ imports: [sequelize] })

// src/entity/user.entity.ts — @Table from sequelize-typescript
import { Table, Model, Column, CreatedAt, DeletedAt, DataType } from 'sequelize-typescript';
@Table
export class User extends Model {
  @Column({ primaryKey: true, autoIncrement: true, type: DataType.BIGINT })
  id?: number;
  @Column name: string;
  @CreatedAt creationDate: Date;
  @DeletedAt deletedDate: Date;   // enables paranoid (soft-delete) mode
}

// config.default.ts — datasource form, dialect REQUIRED
export default {
  sequelize: {
    dataSource: {
      default: {
        database: 'test', username: 'root', password: 'xxx',
        host: '127.0.0.1', port: 3306, dialect: 'mysql',   // REQUIRED in v4
        timezone: '+08:00', sync: false,
        repositoryMode: true,            // enables @InjectRepository (disables static methods)
        entities: [User, '**/*.entity.{j,t}s'],
      },
    },
  },
} as MidwayConfig;

// service — inject repository
import { InjectRepository } from '@midwayjs/sequelize';
import { Repository } from 'sequelize-typescript';
@Provide()
export class UserService {
  @InjectRepository(User) userRepo: Repository<User>;
  @InjectRepository(User, 'default') namedRepo: Repository<User>;   // specify datasource

  async findAll() { return this.userRepo.findAll(); }
  async create(name: string) { return this.userRepo.create({ name }); }
}
```

Relationships: `@ForeignKey(() => Team)`, `@BelongsTo`, `@HasMany`, `@BelongsToMany`. For circular refs wrap the type: `team: ReturnType<() => Team>`.

Reference: [Midway Sequelize](https://midwayjs.org/docs/extensions/sequelize)

---

### 7.8 Follow TypeORM v4 Datasource Patterns

**Impact: HIGH** — "v4 changed the package, config key, and removed EntityModel"

In v4, TypeORM moved to the `@midwayjs/typeorm` package (config key `typeorm`), uses the **datasource** config form (`typeorm.dataSource.<name>`), and **removed the `@EntityModel` decorator** — use TypeORM's native decorators (`@Entity`, `@Column`, ...) from the `typeorm` package directly. Inject repositories with `@InjectEntityModel(Entity)` from `@midwayjs/typeorm`. Declare entities in each datasource's `entities` array (class refs or globs). Never enable `synchronize: true` in production.

**Incorrect (v3 @midwayjs/orm, EntityModel, flat config):**

```typescript
// ❌ v3 package + removed decorator + wrong config key
import { EntityModel } from '@midwayjs/orm';

@EntityModel('photo')        // ❌ removed in v4
export class Photo {}

// ❌ v3 flat config
export default { orm: { type: 'mysql', host: '...', entities: [Photo] } };

// ❌ production synchronize
export default { typeorm: { dataSource: { default: { synchronize: true } } } };
```

**Correct (v4 package, native decorators, datasource config, repository injection):**

```typescript
// configuration.ts
import * as orm from '@midwayjs/typeorm';
@Configuration({ imports: [orm] })

// src/entity/photo.entity.ts — NATIVE typeorm decorators
import { Entity, Column, PrimaryGeneratedColumn, CreateDateColumn } from 'typeorm';

@Entity('photo')
export class Photo {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ length: 100 })
  name: string;

  @CreateDateColumn()
  createdAt: Date;
}

// config.default.ts — v4 datasource form
import { Photo } from '../entity/photo.entity';
export default {
  typeorm: {
    dataSource: {
      default: {
        type: 'mysql',
        host: process.env.DB_HOST,
        port: 3306,
        entities: [Photo, '**/*.entity.{j,t}s'],   // class refs OR globs
        synchronize: false,                          // NEVER true in prod
        logging: false,
      },
    },
    defaultDataSourceName: 'default',
  },
} as MidwayConfig;

// src/service/photo.service.ts — inject repository
import { Provide, Inject } from '@midwayjs/core';
import { InjectEntityModel } from '@midwayjs/typeorm';
import { Repository } from 'typeorm';

@Provide()
export class PhotoService {
  @InjectEntityModel(Photo)
  photoModel: Repository<Photo>;

  async findAll() { return this.photoModel.find({}); }
  async save(name: string) {
    const photo = new Photo(); photo.name = name;
    return this.photoModel.save(photo);
  }
}
```

Reference: [Midway TypeORM (v4)](https://midwayjs.org/docs/extensions/orm)

---

## 8. API Design

**Section Impact: MEDIUM**

### 8.1 Use @Aspect for Cross-Cutting AOP Interception

**Impact: MEDIUM-HIGH** — "Clean separation for logging, timing, and cross-cutting logic"

AOP aspects (`@Aspect` + `IMethodAspect`) intercept method calls for cross-cutting concerns (logging, timing, validation, caching) without polluting business logic. Aspects are always **Singleton scope** and live in `src/aspect/`. They target a class (or array of classes) and optionally a picomatch method pattern. Use the `IMethodAspect` lifecycle (`before`, `around`, `afterReturn`, `afterThrow`, `after`) — `around` gives full control. Remember: aspects do **not** apply to inherited (parent class) methods.

**Incorrect (duplicating logging/timing logic in every method):**

```typescript
@Provide()
export class OrderService {
  async createOrder(dto: any) {
    const start = Date.now();
    this.logger.info('createOrder called');        // ❌ repeated boilerplate
    const result = await this.repo.save(dto);
    this.logger.info(`createOrder done in ${Date.now() - start}ms`);
    return result;
  }
  async cancelOrder(id: number) {
    const start = Date.now();                       // ❌ same boilerplate again
    this.logger.info('cancelOrder called');
    /* ... */
  }
}
```

**Correct (a single @Aspect handles timing for the whole class):**

```typescript
import { Aspect, IMethodAspect, JoinPoint } from '@midwayjs/core';
import { ILogger, Logger } from '@midwayjs/core';
import { OrderService } from '../service/order.service';

// src/aspect/timing.aspect.ts
@Aspect(OrderService)                       // intercept ALL methods of OrderService
export class TimingAspect implements IMethodAspect {
  @Logger() logger: ILogger;

  async around(point: JoinPoint) {          // around = full control
    const start = Date.now();
    const name = point.methodName;
    try {
      const result = await point.proceed(...point.args);  // call original
      this.logger.info(`${name} ok in ${Date.now() - start}ms`);
      return result;                         // can modify the return value
    } catch (err) {
      this.logger.error(`${name} failed in ${Date.now() - start}ms`, err);
      throw err;
    }
  }
}

// target multiple classes + filter by method name (picomatch glob)
@Aspect([OrderService, PaymentService], '*By*')   // only methods containing "By"
export class AuditAspect implements IMethodAspect {
  async before(point: JoinPoint) { /* runs before each matched method */ }
}
```

`@Aspect(target, matchPattern?, priority?)` — higher priority registers first and is called later (onion model).

Reference: [Midway Aspect (AOP)](https://midwayjs.org/docs/aspect)

---

### 8.2 Write Controllers with Declarative Routing and Param Decorators

**Impact: MEDIUM** — "Clean, typed, swagger-friendly route definitions"

Use `@Controller(prefix?)` to group routes, HTTP-method decorators (`@Get`, `@Post`, `@Del`, ...) to bind paths, and parameter decorators (`@Body`, `@Query`, `@Param`, `@Headers`, `@Session`) to extract typed request data. Compose multiple param decorators on one handler. Use `@SetHeader`, `@HttpCode`, `@ContentType` for response shaping, and `{ summary: '...' }` in route options for Swagger. Delegate business logic to injected services — controllers should stay thin.

**Incorrect (manual ctx parsing, manual response writing, fat controller):**

```typescript
@Controller('/user')
export class UserController {
  @Inject() ctx: Context;
  @Inject() userService: UserService;

  @Get('/:id')
  async findOne() {
    // ❌ manually parsing everything from ctx
    const id = this.ctx.params.id;
    const fields = this.ctx.query.fields;
    const user = await this.userService.findById(Number(id));
    this.ctx.body = user;           // ❌ manual response writing
  }
}
```

**Correct (declarative decorators, thin controller, typed params):**

```typescript
import { Controller, Get, Post, Body, Query, Param, Provide, Inject, HttpCode } from '@midwayjs/core';
import { DefaultValuePipe, ParseIntPipe } from '@midwayjs/validation';
import { UserService } from '../service/user.service';
import { CreateUserDTO } from '../dto/user.dto';

@Controller('/user')
export class UserController {
  @Inject() userService: UserService;

  // compose multiple param decorators; { summary } for swagger docs
  @Get('/')
  async findAll(
    @Query('page', new DefaultValuePipe(1), ParseIntPipe) page: number,
    @Query('limit', new DefaultValuePipe(20), ParseIntPipe) limit: number,
  ) {
    return this.userService.findAll(page, limit);
  }

  @Get('/:id')
  async findOne(@Param('id', ParseIntPipe) id: number) {   // ParseIntPipe converts string→number
    return this.userService.findById(id);
  }

  @Post('/')
  @HttpCode(201)
  async create(@Body() dto: CreateUserDTO) {  // validated DTO
    return this.userService.create(dto);
  }
}

// global route prefix via config:
// config.default.ts → koa: { globalPrefix: '/v1' }
// opt out per-controller: @Controller('/health', { ignoreGlobalPrefix: true })
```

### Dynamic route registration and introspection (MidwayWebRouterService)

For runtime-driven routes (plugins, dynamic APIs) or route introspection in middleware/guards, inject `MidwayWebRouterService` (available at `onReady` and after):

```typescript
import { MidwayWebRouterService } from '@midwayjs/core';

@Configuration({})
export class MainConfiguration {
  @Inject() webRouterService: MidwayWebRouterService;

  async onReady() {
    // introspect: get all routes (sorted by priority)
    const table = this.webRouterService.getFlattenRouterTable();
    const priorityList = this.webRouterService.getRoutePriorityList();

    // find the matched route for a URL (useful in middleware/guards for authz)
    const matched = this.webRouterService.getMatchedRouterInfo('/user/1', 'get');

    // dynamically register a controller WITHOUT @Controller decorator
    this.webRouterService.addController(DynamicController, {
      prefix: '/plugin',
      routerOptions: { middleware: [AuthMiddleware] },
    });

    // dynamically add a raw route handler
    this.webRouterService.addRouter(
      async (ctx) => { ctx.body = 'dynamic'; },
      { url: '/dynamic', requestMethod: 'GET' },
    );
  }
}
```

Reference: [Midway Controller](https://midwayjs.org/docs/controller), [Router Table](https://midwayjs.org/docs/router_table)

---

### 8.3 Manage Cookies and Sessions Correctly

**Impact: MEDIUM** — "Signed vs encrypted cookies, session config, rotation"

In `@midwayjs/koa`, cookies are signed by default (`signed: true`) but **not** encrypted — the browser sees plaintext. Use `encrypt: true` for hidden values, `httpOnly: false` for JS-readable cookies. Read with matching options (signed↔signed, encrypt↔encrypt). Sessions are stored encrypted in a cookie by default; for "remember me", set `ctx.session.maxAge`. Rotate cookie keys by prepending new keys to the `keys` array.

**Incorrect (plaintext sensitive cookies, wrong read options, session field naming):**

```typescript
// ❌ sensitive data in a plaintext cookie
ctx.cookies.set('role', 'admin', { signed: false, httpOnly: false });

// ❌ reading a signed cookie without signed option → garbled value
const role = ctx.cookies.get('role');   // wrong: set had signed:true

// ❌ session fields starting with _ → lost next request
ctx.session._internal = 'data';
```

**Correct (signed/encrypted cookies + session config + key rotation):**

```typescript
// config.default.ts
export default {
  keys: ['newKey', 'oldKey'],     // first signs/encrypts; all tried for verify; rotate by prepending
  session: {
    maxAge: 24 * 3600 * 1000,     // 1 day
    key: 'MW_SESS',
    httpOnly: true,
    renew: true,                   // extend when half-expired
  },
} as MidwayConfig;

@Controller('/')
export class HomeController {
  @Inject() ctx: Context;

  @Get('/login')
  async login() {
    // encrypted + httpOnly → fully hidden from client & JS
    this.ctx.cookies.set('token', jwtToken, { encrypt: true, httpOnly: true });

    // session — "remember me" extends expiry
    this.ctx.session.userId = user.id;
    if (rememberMe) {
      this.ctx.session.maxAge = FORMAT.MS.ONE_DAY * 30;   // import FORMAT from @midwayjs/core
    }
    // rotate CSRF secret after login
    this.ctx.rotateCsrfSecret();
    return { success: true };
  }

  @Get('/profile')
  async profile() {
    // read with matching options
    const token = this.ctx.cookies.get('token', { encrypt: true });
    return { userId: this.ctx.session.userId };
  }

  @Get('/logout')
  async logout() {
    this.ctx.session = null;   // destroy session
    return { success: true };
  }
}
```

For FaaS, manually add `@midwayjs/session` (v4). For Redis-backed sessions, use the framework's session store package.

Reference: [Midway Cookie & Session](https://midwayjs.org/docs/cookie_session)

---

### 8.4 Generate CRUD Routes with @midwayjs/crud

**Impact: MEDIUM-HIGH** — "Declarative REST CRUD with query/filter/sort out of the box"

`@midwayjs/crud` generates standard REST endpoints (list/detail/create/update/delete) from a `@Crud()` declaration, with built-in pagination, filtering, sorting, search, and soft-delete. Pick a DB adapter via subpath imports (`@midwayjs/crud/typeorm`, `/mikro`, `/sequelize`, `/mongoose`). The `@Crud()` decorator only **declares** routes; the injected `CrudService` does the work. Put complex business logic in a separate service, not in CRUD overrides.

**Incorrect (hand-writing identical CRUD controllers for every entity):**

```typescript
@Controller('/users')
export class UserController {
  @Inject() userRepo: Repository<User>;
  @Get('/') async list(@Query() q) { /* pagination logic, repeated... */ }
  @Get('/:id') async detail(@Param('id') id) { /* ... */ }
  @Post('/') async create(@Body() b) { /* ... */ }
  @Patch('/:id') async update() { /* ... */ }
  @Del('/:id') async delete() { /* ... */ }
}
// ❌ same boilerplate for orders, products, etc.
```

**Correct (declarative @Crud with TypeORM adapter + query protocol):**

```typescript
import * as crud from '@midwayjs/crud';
import { TypeOrmCrudService } from '@midwayjs/crud/typeorm';
@Configuration({ imports: [koa, typeorm, crud, validation] })

// service extends the adapter base
@Provide()
export class UserCrudService extends TypeOrmCrudService<UserEntity> {
  @InjectEntityModel(UserEntity) repo: Repository<UserEntity>;
}

// controller — declarative routes + query options
import { Crud } from '@midwayjs/crud';
@Crud({
  model: UserEntity,
  service: UserCrudService,
  dto: { create: CreateUserDTO, update: UpdateUserDTO },
  query: {
    defaultLimit: 20, maxLimit: 100,
    sortable: ['id', 'createdAt'],        // whitelist sort fields
    filterable: ['status'],               // whitelist filter fields
    searchable: ['name', 'email'],        // full-text search fields
    join: ['profile'],                     // allowed relations
  },
  routes: { only: ['list', 'detail', 'create'] },  // or exclude
  delete: { mode: 'soft' },               // opt-in soft delete
})
@Controller('/users')
export class UserController {
  @Inject() crudService: UserCrudService;   // still required — does the work
  // override a default route by redefining the same path, or add custom routes
}

// list endpoint query protocol: ?page=1&limit=20&sort=id:DESC&filter=status||eq||active&search=john
// returns: { data, meta: { page, limit, total, pageCount, hasNext, hasPrev } }
```

DTO validation requires `@midwayjs/validation` installed. CRUD routes auto-emit Swagger metadata. Not suitable for transaction-heavy/workflow modules — use a dedicated service there.

Reference: [Midway CRUD](https://midwayjs.org/docs/extensions/crud)

---

### 8.5 Handle File Uploads with @midwayjs/busboy

**Impact: HIGH** — "Secure multipart parsing with whitelist and size limits"

Prefer `@midwayjs/busboy` (since v3.17.0, replaces `@midwayjs/upload`) for file uploads. Unlike the legacy upload component, its `UploadMiddleware` is **not** auto-loaded — apply it per-route (`{ middleware: [UploadMiddleware] }`) so only upload endpoints parse multipart bodies. Always set `whitelist`, `mimeTypeWhiteList`, and `limits.fileSize` to prevent temp-dir abuse. Use `@Files()`/`@Fields()` to receive files and form fields.

**Incorrect (auto-parsing every request, no whitelist, no size limit):**

```typescript
// ❌ legacy upload auto-parses ALL requests — fills temp dir on any multipart POST
import * as upload from '@midwayjs/upload';
@Configuration({ imports: [upload] })

@Controller('/')
export class HomeController {
  @Inject() ctx: Context;
  @Post('/upload')
  async upload() {
    const file = this.ctx.files[0];
    // ❌ no whitelist (WebShell risk), no size limit, no cleanup
  }
}
```

**Correct (busboy route-scoped middleware + whitelist + limits + cleanup):**

```typescript
import * as busboy from '@midwayjs/busboy';
import { UploadMiddleware, uploadWhiteList } from '@midwayjs/busboy';
import { Configuration } from '@midwayjs/core';
@Configuration({ imports: [busboy] })

// config.default.ts
export default {
  busboy: {
    mode: 'file',                          // 'file' | 'stream' | 'asyncIterator'
    whitelist: uploadWhiteList,            // allowed extensions; null = WebShell risk!
    mimeTypeWhiteList: undefined,          // add for MIME checking (file mode only)
    tmpdir: join(tmpdir(), 'midway-busboy-files'),
    cleanTimeout: 5 * 60 * 1000,
    limits: { fileSize: 10 * 1024 * 1024 }, // 10MB hard limit
  },
} as MidwayConfig;

// route-scoped — only /upload parses multipart
import { Files, Fields } from '@midwayjs/core';
@Controller('/upload')
export class UploadController {
  @Inject() ctx: Context;

  @Post('/', { middleware: [UploadMiddleware] })
  async upload(@Files() files, @Fields() fields) {
    // file mode: each file = { filename, data: <tempPath>, fieldname, mimeType }
    const saved = files.map(f => ({ name: f.filename, path: f.data }));
    await this.ctx.cleanupRequestFiles();   // ✓ clean temp files when done
    return saved;
  }

  // asyncIterator mode for streaming large files without buffering
  @Post('/stream', { middleware: [UploadMiddleware] })
  async stream(@Files() fileIter: AsyncGenerator<UploadStreamFileInfo>) {
    for await (const file of fileIter) {
      file.data.pipe(createWriteStream(join(tmpdir(), file.filename)));
    }
    return { ok: true };
  }
}
```

Reference: [Midway Busboy](https://midwayjs.org/docs/extensions/busboy), [Midway Upload](https://midwayjs.org/docs/extensions/upload)

---

### 8.6 Build GraphQL APIs with @midwayjs/apollo

**Impact: MEDIUM** — "Apollo Server with Midway DI and typed resolvers"

`@midwayjs/apollo` bundles Apollo Server with Midway DI. Define the schema via `typeDefs` (SDL) or `typePaths` (glob), and resolvers as `@Resolver()` classes using decorators re-exported from the component (`@Query`, `@Args`, `@Context`, `@Parent`, `@Info`). Use `contextFactory` to inject request-scoped data. The resolver's `@Context()` is the Midway-enhanced context (has `requestContext`, `logger`) — no need to unwrap.

**Incorrect (importing GraphQL decorators from wrong packages):**

```typescript
// ❌ decorators from type-graphql or @nestjs/graphql — not the Midway component
import { Resolver, Query } from 'type-graphql';
// ❌ manual context unwrapping
async user(@Ctx() ctx) { const midwayCtx = ctx.ctx; /* ... */ }
```

**Correct (Midway apollo resolvers + SDL + contextFactory):**

```typescript
import * as apollo from '@midwayjs/apollo';
@Configuration({ imports: [koa, apollo] })

// config.default.ts
export default {
  apollo: {
    path: '/graphql',
    typePaths: ['./schema/**/*.graphql'],   // OR typeDefs: `type Query { user(id: ID!): String }`
    graphiql: process.env.NODE_ENV === 'local',
    methods: ['GET', 'POST'],
    apollo: { introspection: true },         // Apollo Server options
    contextFactory: async (ctx) => ({ currentUserId: ctx.headers['x-user-id'] }),
  },
} as MidwayConfig;

// resolver — decorators from @midwayjs/apollo
import { Resolver, Query, Args, Context } from '@midwayjs/apollo';
@Resolver()
export class UserResolver {
  @Inject() userService: UserService;

  @Query('userName')
  async userName(@Args('id') id: string, @Context() context) {
    // context is Midway-enhanced: has requestContext, logger, getApp()
    context.logger.info('query user %s', id);
    return (await this.userService.findById(id))?.name;
  }
}
```

Subscriptions use `graphql-ws` (`subscriptions: true | { path }`) returning `AsyncIterable`. Get request-scoped beans in resolvers via `context.requestContext.getAsync(Service)`.

Reference: [Midway Apollo (GraphQL)](https://midwayjs.org/docs/extensions/apollo)

---

### 8.7 Use the HTTP Client for Outbound Requests

**Impact: MEDIUM** — "DI-managed axios: instances, interceptors, streams, retry, error mapping"

Midway offers two ways to make outbound HTTP requests. For simple one-off calls (no retries, no interceptors, no shared config), use the built-in `makeHttpRequest`/`HttpClient` from `@midwayjs/core` (no extra dependency). For anything more — shared baseURL/headers/timeouts, multiple instances, global interceptors, typed responses, streaming, retry — use the `@midwayjs/axios` component and inject `HttpService`. In v4 the bare `axios` export was removed; always inject `HttpService`. Configure instances declaratively in config and inject named instances with `@InjectClient(HttpServiceFactory, 'name')`.

**Incorrect (raw axios import, manual instance creation, no DI, no error handling):**

```typescript
import axios from 'axios';          // ❌ bypasses the component lifecycle & config

@Provide()
export class WeatherService {
  async getWeather(city: string) {
    // ❌ manual client, no shared config, no timeout, no retry, hard to test
    const res = await axios.get(`https://api.weather.com/${city}`);
    return res.data;
  }
}

// ❌ v3-style bare axios export (removed in v4)
import { axios } from '@midwayjs/axios';
@Provide()
export class Svc {
  @Inject() http: axios;            // ❌ removed in v4, use HttpService
}

// ❌ unhandled rejection — a failing upstream crashes the request
async callUpstream() {
  const { data } = await this.httpService.get('/flaky');   // ❌ no try/catch, raw axios error leaks
  return data;
}
```

**Correct (built-in for simple cases + @midwayjs/axios HttpService for real integrations):**

```typescript
// === Option A: built-in client (no dependency) for simple calls ===
import { makeHttpRequest } from '@midwayjs/core';

async fetchConfig() {
  const result = await makeHttpRequest('https://api.example.com/config', {
    method: 'GET',
    dataType: 'json',     // 'json' | 'text' | (default Buffer)
    timeout: 5000,
  });
  return result.data;     // NOTE: never return the raw `result` object (circular)
}

// === Option B: @midwayjs/axios HttpService (preferred for integrations) ===
// configuration.ts
import * as axios from '@midwayjs/axios';
@Configuration({ imports: [axios] })

// config.default.ts — declarative instances (axios.create config shape)
export default {
  axios: {
    default: {                        // shared by all instances
      timeout: 5000,
    },
    clients: {
      default: {                      // default instance
        baseURL: 'https://api.example.com',
        headers: { 'X-Requested-With': 'XMLHttpRequest' },
      },
      payment: {                      // a second named instance
        baseURL: 'https://pay.example.com',
        timeout: 3000,
      },
    },
  },
} as MidwayConfig;

// service — inject the default instance with typed responses
import { HttpService } from '@midwayjs/axios';
import { AxiosResponse } from 'axios';

interface WeatherDTO { city: string; temp: number; }

@Provide()
export class WeatherService {
  @Inject() httpService: HttpService;

  async getWeather(city: string): Promise<WeatherDTO> {
    // type the generic so `data` is typed — never `any`
    const { data } = await this.httpService.get<WeatherDTO>(`/weather/${city}`);
    return data;
  }
}

// inject a NAMED instance via the service factory (no magic strings)
import { HttpServiceFactory } from '@midwayjs/axios';
import { InjectClient } from '@midwayjs/core';

@Provide()
export class PaymentService {
  @InjectClient(HttpServiceFactory, 'payment')
  paymentHttp: HttpService;

  async charge(amount: number) {
    const { data } = await this.paymentHttp.post('/charge', { amount });
    return data;
  }
}
```

### Map Axios Errors to Midway Errors

Axios rejects with an `AxiosError` whose shape differs from Midway's error model. Always catch and map it to a `MidwayHttpError` (or a custom domain error) so the `@Catch` filters produce a consistent response. Inspect `error.response` (server replied with non-2xx), `error.request` (no reply), or neither (setup failure).

**Incorrect (let raw AxiosError bubble — clients see an inconsistent 500):**

```typescript
@Provide()
export class UserService {
  @Inject() httpService: HttpService;

  async fetchProfile(id: number) {
    const { data } = await this.httpService.get(`/users/${id}`);  // ❌ raw AxiosError on failure
    return data;
  }
}
```

**Correct (catch + map to structured Midway errors):**

```typescript
import { MidwayHttpError, httpError, HttpStatus } from '@midwayjs/core';
import { AxiosError } from 'axios';

// a typed domain error for upstream failures
export class UpstreamServiceError extends MidwayHttpError {
  constructor(service: string, cause: Error) {
    super(`${service} request failed`, HttpStatus.BAD_GATEWAY, { cause });
  }
}

@Provide()
export class UserService {
  @Inject() httpService: HttpService;
  @Inject() logger: ILogger;

  async fetchProfile(id: number) {
    try {
      const { data } = await this.httpService.get(`/users/${id}`);
      return data;
    } catch (err) {
      const e = err as AxiosError;
      // server replied with an error status → map it through
      if (e.response) {
        const status = e.response.status;
        if (status === 404) throw new httpError.NotFoundError(`user ${id} not found`);
        if (status === 401) throw new httpError.UnauthorizedError();
        // propagate the upstream status as a Bad Gateway with the original cause
        throw new UpstreamServiceError('user-service', e);
      }
      // no response (timeout / network) → Service Unavailable
      if (e.request) {
        this.logger.error('user-service unreachable', e.message);
        throw new httpError.ServiceUnavailableError('user-service unavailable');
      }
      // request setup error
      throw new UpstreamServiceError('user-service', e);
    }
  }
}
```

### Configure Global Interceptors (auth, logging, retry)

Interceptors are best configured **once** in `onReady` on the resolved `HttpService` (or a named instance via `HttpServiceFactory.get('name')`). Extend `AxiosRequestConfig` via declaration merging to carry custom fields (like `retry`) in a type-safe way — no magic strings. The request interceptor runs before the call; the response interceptor handles both success and errors.

**Correct (auth interceptor + timing/log interceptor + retry interceptor):**

```typescript
// src/interface.ts — type-safe custom config fields (declaration merging)
import '@midwayjs/axios';
declare module '@midwayjs/axios/dist/interface' {
  interface AxiosRequestConfig {
    retry?: number;        // max retries on 5xx/network errors
    retryDelay?: number;   // base backoff in ms
    traceId?: string;      // propagates request tracing
  }
}

// configuration.ts
import { Configuration, IMidwayContainer, ILogger, Logger } from '@midwayjs/core';
import * as axios from '@midwayjs/axios';
import { AxiosError } from 'axios';

@Configuration({ imports: [axios] })
export class MainConfiguration {
  @Logger() logger: ILogger;

  async onReady(container: IMidwayContainer) {
    const httpService = await container.getAsync(axios.HttpService);

    // 1) auth + trace-id injection
    httpService.interceptors.request.use((config) => {
      config.headers = config.headers ?? {};
      config.headers.Authorization = `Bearer ${getToken()}`;
      config.headers['x-trace-id'] = config.traceId ?? randomUUID();
      return config;
    });

    // 2) timing/log interceptor (success + error)
    httpService.interceptors.request.use((config) => {
      (config as any).__startedAt = Date.now();
      return config;
    });
    httpService.interceptors.response.use(
      (response) => {
        const ms = Date.now() - ((response.config as any).__startedAt ?? Date.now());
        this.logger.info('http %s %s %d %dms', response.config.method, response.config.url, response.status, ms);
        return response;
      },
      (error: AxiosError) => {
        const cfg = error.config ?? {};
        const ms = Date.now() - ((cfg as any).__startedAt ?? Date.now());
        this.logger.error('http %s %s failed %dms %s', cfg.method, cfg.url, ms, error.message);
        return Promise.reject(error);
      },
    );

    // 3) retry interceptor with exponential backoff (idempotent calls only)
    httpService.interceptors.response.use(undefined, async (error: AxiosError) => {
      const config = error.config;
      if (!config) return Promise.reject(error);
      const maxRetries = config.retry ?? 0;
      const attempt = ((config as any).__retryCount ?? 0) + 1;
      const retryable =
        maxRetries > 0 &&
        attempt <= maxRetries &&
        (isNetworkError(error) || (error.response?.status ?? 0) >= 500);

      if (!retryable) return Promise.reject(error);

      const delay = (config.retryDelay ?? 500) * Math.pow(2, attempt - 1);
      this.logger.warn('http retry attempt %d/%d in %dms', attempt, maxRetries, delay);
      await new Promise((r) => setTimeout(r, delay));
      (config as any).__retryCount = attempt;
      return httpService.request(config);   // replay
    });

    // configure a NAMED instance instead:
    // const factory = await container.getAsync(axios.HttpServiceFactory);
    // const pay = factory.get('payment');
    // pay.interceptors.request.use(...);
  }
}

function isNetworkError(error: AxiosError): boolean {
  return !error.response && !!error.request;   // no server reply
}
```

> Retry only idempotent methods (GET, HEAD, PUT, DELETE). Retrying POST can duplicate side effects — gate on `config.method`.

### Stream Downloads and Uploads

For large payloads, use `responseType: 'stream'` to pipe a download without buffering the whole body, and `FormData` for multipart uploads. Always await stream completion (e.g. `stream/promises.finished`) and handle backpressure.

**Correct (stream download + multipart upload):**

```typescript
import { createWriteStream } from 'fs';
import { finished } from 'stream/promises';
import { FormData } from '@midwayjs/axios';   // or 'form-data'

@Provide()
export class FileTransferService {
  @Inject() httpService: HttpService;

  // download a large file to disk without buffering in memory
  async download(url: string, dest: string): Promise<void> {
    const response = await this.httpService.get<NodeJS.ReadableStream>(url, {
      responseType: 'stream',
    });
    const writer = createWriteStream(dest);
    response.data.pipe(writer);
    await finished(writer);          // ✓ resolves on end, rejects on error
  }

  // multipart upload via FormData
  async upload(filePath: string, meta: Record<string, string>) {
    const form = new FormData();
    form.append('file', createReadStream(filePath));
    for (const [k, v] of Object.entries(meta)) form.append(k, v);
    const { data } = await this.httpService.post('/upload', form, {
      headers: form.getHeaders(),     // multipart boundary headers
      maxContentLength: Infinity,
      maxBodyLength: Infinity,
    });
    return data;
  }
}
```

For non-app code (scripts/tests) where DI is unavailable, the raw axios instance is exported as `Axios`: `import { Axios } from '@midwayjs/axios'`.

### Request Cancellation and Timeouts

Use `signal` (AbortController) for explicit cancellation and `timeout` for hard deadlines. The v4 `HttpService` forwards these to axios. Cancel in-flight requests on shutdown or when a user aborts.

**Correct (AbortController + per-request timeout):**

```typescript
@Provide()
export class SearchService {
  @Inject() httpService: HttpService;

  // caller can cancel by calling controller.abort()
  async search(query: string, controller = new AbortController()) {
    const { data } = await this.httpService.get('/search', {
      params: { q: query },
      timeout: 3000,                 // hard timeout (ms)
      signal: controller.signal,     // explicit cancellation
    });
    return data;
  }
}

// graceful shutdown: cancel pending searches in onStop
```

Reference: [Midway HTTP Request (axios)](https://midwayjs.org/docs/extensions/axios), [axios Interceptors](https://github.com/axios/axios#interceptors)

---

### 8.8 Proxy Requests with @midwayjs/http-proxy

**Impact: LOW-MEDIUM** — "Reverse proxy for external/static resources"

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

---

### 8.9 Internationalize with @midwayjs/i18n

**Impact: MEDIUM** — "Request-aware multi-language text resolution"

`@midwayjs/i18n` provides request-aware translation via `MidwayI18nService.translate()`. Store translations in JSON files under `src/locales/`, register them in `i18n.localeTable` (locale → group → keys). Locale is auto-resolved from query → cookie → `Accept-Language` header → `defaultLocale`, with `writeCookie` caching the choice. Use groups to isolate component/module translations. Dynamically add translations from a DB via `addLocale()` in `onReady`.

**Incorrect (hardcoded strings, manual locale detection):**

```typescript
@Get('/')
async index(@Query('lang') lang: string) {
  // ❌ hardcoded strings, no fallback, no parameter interpolation
  if (lang === 'zh') return '你好';
  return 'Hello';
}
```

**Correct (locale files + translate with args + group isolation):**

```typescript
import * as i18n from '@midwayjs/i18n';
@Configuration({ imports: [i18n] })

// src/locales/en_US.json: { "hello": "Hello {username}" }
// src/locales/zh_CN.json: { "hello": "你好 {username}" }

// config.default.ts
export default {
  i18n: {
    defaultLocale: 'en_US',
    localeTable: {
      en_US: { default: require('../locales/en_US'), user: require('../locales/user_en_US') },
      zh_CN: { default: require('../locales/zh_CN'), user: require('../locales/user_zh_CN') },
    },
    fallbacks: { 'en_*': 'en_US' },   // wildcard fallback mapping
    writeCookie: true,                 // cache locale choice in cookie
    resolver: { queryField: 'locale', cookieField: { fieldName: 'locale' } },
    missingKeyHandler: (msg, opts) => `[missing:${msg}]`,   // log/guard missing keys
  },
} as MidwayConfig;

// service — translate with parameter interpolation
import { MidwayI18nService } from '@midwayjs/i18n';
@Controller('/')
export class HomeController {
  @Inject() i18nService: MidwayI18nService;

  @Get('/')
  async index(@Query('username') username: string) {
    // auto-resolves locale from query/cookie/header; explicit override via { locale }
    return this.i18nService.translate('hello', { args: { username } });
    // non-default group: this.i18nService.translate('user.hello', { args: [username], group: 'user' })
  }
}

// dynamic translations from DB in onReady
async onReady() {
  this.i18nService.addLocale('zh_TW', { hello: '你好，{username}' });
}
```

Locale priority (high→low): explicit `locale` in `translate()` → `saveRequestLocale()` → query → cookie → `Accept-Language` → `defaultLocale`. Internally all locales are normalized to lowercase-hyphen (`en_US` → `en-us`).

Reference: [Midway i18n](https://midwayjs.org/docs/extensions/i18n)

---

### 8.10 Apply Middleware at the Right Level with Match/Ignore

**Impact: MEDIUM-HIGH** — "Correct scoping prevents auth/logging bypass and perf waste"

Midway middleware uses the **onion model** (Koa/Egg) — code runs both before AND after the controller, and `await next()` returns the downstream result. Apply middleware at three levels: route (`@Get('/', { middleware: [...] })`), controller (`@Controller('/', { middleware: [...] })`), and global (`app.useMiddleware(...)`). Use `match(ctx)` or `ignore(ctx)` (only one takes effect; strings/regex/arrays/functions) to scope execution — essential for auth/login routes. Order global middleware with `app.getMiddleware().insertBefore/After`.

**Incorrect (auth middleware on every route including login, manual next handling):**

```typescript
async onReady(_, app) {
  app.useMiddleware(JwtMiddleware);   // ❌ blocks /login, /register, /captcha
}

@Middleware()
export class JwtMiddleware implements IMiddleware<Context, NextFunction> {
  resolve() {
    return async (ctx, next) => {
      const token = verifyToken(ctx);   // ❌ throws on public routes
      ctx.user = token;
      next();                            // ❌ result of next() discarded, breaks onion
    };
  }
}
```

**Correct (ignore public routes, propagate result, ordered global registration):**

```typescript
@Middleware()
export class JwtMiddleware implements IMiddleware<Context, NextFunction> {
  // skip auth on public routes (only one of match/ignore is used)
  ignore(ctx: Context) {
    return /\/login|\/register|\/captcha|\/health/.test(ctx.path);
  }

  resolve() {
    return async (ctx: Context, next: NextFunction) => {
      const token = extractToken(ctx);
      if (token) ctx.user = verifyToken(token);
      const result = await next();     // ✓ await + return preserves onion model
      return result;
    };
  }
  static getName(): string { return 'jwt'; }
}

// configuration.ts — order matters: cors before auth
async onReady(_, app) {
  app.getMiddleware().insertFirst(CorsMiddleware);
  app.useMiddleware(JwtMiddleware);
  // or: app.getMiddleware().insertBefore(JwtMiddleware, 'cors');
}

// reuse a middleware with different options
app.useMiddleware(createMiddleware(ReportMiddleware, { text: 'x' }, 'report2'));
```

Note: returning `null` from a Koa handler sets status **204**. To return 200, set `ctx.status = 200` explicitly.

Reference: [Midway Middleware](https://midwayjs.org/docs/middleware)

---

### 8.11 Use Pipes for Input Transformation

**Impact: MEDIUM** — "Clean, validated data reaches handlers"

Pipes run **after** param decorators and handle validation/transformation. Use built-in pipes (`ParseIntPipe`, `ParseFloatPipe`, `ParseBoolPipe`, `DefaultValuePipe` from `@midwayjs/validation`) on route params for automatic type conversion with errors. Create custom pipes (`@Pipe` + `PipeTransform`) for business-specific transforms. For whole-body validation, use a validated DTO instead of per-field pipes.

**Incorrect (manual parsing and validation in every handler):**

```typescript
@Get('/:id')
async findOne(@Param('id') id: string) {
  const num = Number(id);          // ❌ NaN if invalid, no error
  if (isNaN(num)) throw new Error('bad id');
  return this.service.findById(num);
}

@Get('/')
async list(@Query('page') page: string) {
  const pageNum = parseInt(page) || 1;   // ❌ repeated manual parsing
  return this.service.list(pageNum);
}
```

**Correct (built-in pipes + custom pipe):**

```typescript
import { DefaultValuePipe } from '@midwayjs/validation';
import { ParseIntPipe } from '@midwayjs/validation';

@Controller('/user')
export class UserController {
  // built-in pipes: default + int conversion with proper errors
  @Get('/')
  async list(
    @Query('page', new DefaultValuePipe(1), ParseIntPipe) page: number,
    @Query('limit', new DefaultValuePipe(20), ParseIntPipe) limit: number,
  ) {
    return this.service.list(page, limit);
  }

  @Get('/:id')
  async findOne(@Param('id', ParseIntPipe) id: number) {
    return this.service.findById(id);
  }
}

// src/pipe/parse-date.pipe.ts — custom pipe
import { Pipe, PipeTransform, TransformOptions } from '@midwayjs/core';
import { httpError } from '@midwayjs/core';

@Pipe()
export class ParseDatePipe implements PipeTransform<string, Date> {
  transform(value: string, options: TransformOptions): Date {
    const date = new Date(value);
    if (isNaN(date.getTime())) throw new httpError.BadRequestError('invalid date');
    return date;
  }
}

// usage
@Get('/report')
async report(@Query('from', ParseDatePipe) from: Date) {
  return this.service.report(from);
}
```

Reference: [Midway Pipes](https://midwayjs.org/docs/pipe)

---

### 8.12 Standardize Responses with HttpServerResponse

**Impact: MEDIUM** — "Consistent success/fail envelope and SSE support"

For a uniform API envelope and advanced response types (SSE, file, stream), use `HttpServerResponse` (since v3.17.0). It provides chainable `.success()`/`.fail()` with overridable JSON/TEXT/BLOB templates, plus `.sse()` for AI streaming with upstream forwarding. Subclass it for per-scenario formats without changing global defaults. Avoid hand-rolling `{ code, msg, data }` shapes in every handler.

**Incorrect (hand-rolled, inconsistent response shapes):**

```typescript
@Get('/:id')
async findOne(@Param('id') id: number) {
  const user = await this.userService.findById(id);
  if (!user) {
    // ❌ inconsistent shape, manual status plumbing
    this.ctx.status = 404;
    return { code: 404, msg: 'not found', data: null };
  }
  return { code: 0, msg: 'ok', data: user };   // ❌ different envelope than above
}
```

**Correct (HttpServerResponse chain + custom subclass + SSE):**

```typescript
import { HttpServerResponse } from '@midwayjs/core';

@Get('/:id')
async findOne(@Param('id') id: number) {
  const user = await this.userService.findById(id);
  if (!user) {
    // fail() → { success: 'false', message: {} }
    return new HttpServerResponse(this.ctx).fail().json({ id, reason: 'not found' });
  }
  // success() → { success: 'true', data: {} }; .json() must be called last
  return new HttpServerResponse(this.ctx).success().json(user);
}

// override the global templates via static properties (success/fail are on the base ServerResponse class)
HttpServerResponse.JSON_TPL = (data, isSuccess, ctx) =>
  JSON.stringify({ code: isSuccess ? 0 : 1, data, message: isSuccess ? 'ok' : data });

// per-scenario custom format: override the static *_TPL properties on a subclass
// (do NOT look for a getJsonTpl() method — it does not exist; use JSON_TPL/TEXT_TPL/BLOB_TPL)
class ApiGatewayResponse extends HttpServerResponse {
  static JSON_TPL = (data, isSuccess, ctx) =>
    JSON.stringify({ ok: isSuccess, payload: data });
}

// SSE with upstream AI forwarding
@Get('/chat')
async chat(@Query('q') q: string) {
  const sse = new HttpServerResponse(this.ctx).sse();
  await sse.forward(upstreamStream, { protocol: 'openai' });  // forwards OpenAI chunks
  sse.sendEnd();
}
```

Reference: [Midway Data Response](https://midwayjs.org/docs/data_response)

---

### 8.13 Serve Static Files with @midwayjs/static-file

**Impact: MEDIUM** — "Config-driven static asset hosting"

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

---

### 8.14 Generate API Docs with @midwayjs/swagger

**Impact: MEDIUM** — "Auto-generated OpenAPI 3 docs from controllers and DTOs"

`@midwayjs/swagger` auto-generates OpenAPI 3.0.3 docs by reading controllers (`@Body`/`@Query`/`@Param`), DTO types, and `@ApiProperty` decorators. Gate it to dev with `enabledEnvironment`. Decorate DTOs with `@ApiProperty({ example, description, type })` and routes with `@ApiOperation`/`@ApiTags` for rich docs. Enable `useValidationSchema: true` (default) to auto-derive fields from `@midwayjs/validation` DTOs — `@ApiProperty` wins on conflicts.

**Incorrect (no API docs, manual maintenance):**

```typescript
// ❌ no swagger — API consumers have no machine-readable contract
@Controller('/user')
export class UserController {
  @Post('/')
  async create(@Body() dto: CreateUserDTO) { return this.userService.create(dto); }
}
// CreateUserDTO has no field descriptions
```

**Correct (env-gated swagger + @ApiProperty DTOs + validation integration):**

```typescript
import * as swagger from '@midwayjs/swagger';
@Configuration({
  imports: [
    { component: swagger, enabledEnvironment: ['local'] },   // dev only
  ],
})

// config.default.ts
export default {
  swagger: {
    title: 'My API', version: '1.0.0',
    description: 'API documentation',
    swaggerPath: '/swagger-ui',     // UI at /swagger-ui/index.html
    auth: { authType: 'bearer', type: 'http', scheme: 'bearer' },
    useValidationSchema: true,     // auto-derive from @midwayjs/validation @Rule
  },
} as MidwayConfig;

// DTO with ApiProperty — explicit wins over validation-derived
import { ApiProperty } from '@midwayjs/swagger';
export class CreateUserDTO {
  @ApiProperty({ example: 'John', description: 'user name' })
  name: string;

  @ApiProperty({ type: 'array', items: { type: 'string' } })
  tags: string[];

  @ApiProperty({ enum: ['active', 'inactive'] })
  status: string;

  @ApiProperty({ type: () => Profile })   // lazy for circular refs
  profile: Profile;
}

// controller with operation metadata
import { ApiOperation, ApiTags, ApiBearerAuth } from '@midwayjs/swagger';
@ApiTags('user')
@ApiBearerAuth()
@Controller('/user')
export class UserController {
  @ApiOperation({ summary: 'Create user', description: 'Creates a new user account' })
  @Post('/')
  async create(@Body() dto: CreateUserDTO) { return this.userService.create(dto); }
}
```

File-upload docs need explicit `@ApiBody({ contentType: BodyContentType.Multipart })`.

Reference: [Midway Swagger](https://midwayjs.org/docs/extensions/swagger)

---

### 8.15 Implement Multi-Tenancy with @midwayjs/tenant

**Impact: MEDIUM-HIGH** — "Request-scoped tenant isolation visible to Singletons"

`@midwayjs/tenant` provides request-scoped tenant isolation via `TenantManager`. Set the current tenant in middleware (`setCurrentTenant`), then read it anywhere — including from `@Singleton` services — via `getCurrentTenant()`. This avoids the scope-downgrade problem: Singletons stay Singleton while still seeing per-request tenant data. Define a typed `TenantInfo` interface for your tenant shape.

**Incorrect (passing tenantId through every method, or shared mutable state):**

```typescript
// ❌ tenantId threaded through every signature — invasive, error-prone
async createOrder(userId: number, tenantId: number, dto: any) { /* ... */ }
async findOrder(id: number, tenantId: number) { /* ... */ }

// ❌ shared mutable tenant on a Singleton — clobbered by concurrent requests
@Singleton()
export class TenantHolder { private id: number; set(id) { this.id = id; } }
```

**Correct (middleware sets, any scope reads via TenantManager):**

```typescript
import * as tenant from '@midwayjs/tenant';
import { TenantManager } from '@midwayjs/tenant';
@Configuration({ imports: [tenant] })

// typed tenant info
interface TenantInfo { id: string; name: string; plan: string; }

// middleware extracts tenant from the request and sets it for the chain
@Middleware()
export class TenantMiddleware implements IMiddleware<Context, NextFunction> {
  @Inject() tenantManager: TenantManager;

  resolve() {
    return async (ctx: Context, next: NextFunction) => {
      // extract from header/subdomain/JWT — your strategy
      const info: TenantInfo = { id: ctx.headers['x-tenant-id'], name: '...', plan: 'pro' };
      await this.tenantManager.setCurrentTenant(info);
      return next();
    };
  }
}

// a Singleton reads the current tenant without being Request-scoped
@Singleton()
export class OrderService {
  @Inject() tenantManager: TenantManager;

  async create(dto: any) {
    const tenant = await this.tenantManager.getCurrentTenant<TenantInfo>();
    return this.repo.save({ ...dto, tenantId: tenant.id });
  }
}

// register globally
async onReady(_, app) { app.useMiddleware(TenantMiddleware); }
```

Tenant data is always tied to a request — set it inside middleware per request. Both Request and Singleton scopes see only the current request's tenant.

Reference: [Midway Tenant](https://midwayjs.org/docs/extensions/tenant)

---

### 8.16 Version APIs for Backward Compatibility

**Impact: MEDIUM** — "Evolve APIs without breaking existing clients"

Midway supports route versioning (URI/HEADER/MEDIA_TYPE/CUSTOM) so old clients keep working while new clients use updated endpoints. Enable it under the framework config key (`koa.versioning`), then set `version` on `@Controller` or individual routes. Use simple numeric versions (`'1'`, `'2'`), never semver strings. Access the request version via `ctx.apiVersion`. Not supported in Serverless.

**Incorrect (breaking changes to existing routes, manual v1/v2 path prefixes):**

```typescript
// ❌ breaking change to /users/:id response shape — old clients break
@Controller('/users')
export class UserController {
  @Get('/:id')
  async findOne() { return { firstName, lastName }; }   // was { name }
}

// ❌ manual path-based versioning — error-prone
@Controller('/v1/users') // ...
@Controller('/v2/users') // ...
```

**Correct (URI versioning + declarative @Controller version):**

```typescript
// config.default.ts
export default {
  koa: {
    versioning: {
      enabled: true,
      type: 'URI',            // /v1/users, /v2/users (recommended — cache-friendly)
      prefix: 'v',
      defaultVersion: '1',
      // alternatives:
      // type: 'HEADER', header: 'x-api-version',
      // type: 'MEDIA_TYPE', mediaTypeParam: 'version',  // Accept: application/json;version=1
      // type: 'CUSTOM', extractVersionFn: (ctx) => ctx.query.version,
    },
  },
} as MidwayConfig;

// v1 controller — old response shape
@Controller('/users', { version: '1' })
export class UserV1Controller {
  @Get('/:id')
  async findOne(@Param('id') id: string) {
    const u = await this.userService.findById(id);
    return { name: `${u.firstName} ${u.lastName}` };
  }
}

// v2 controller — new response shape (same path, different version)
@Controller('/users', { version: '2' })
export class UserV2Controller {
  @Get('/:id')
  async findOne(@Param('id') id: string) {
    const u = await this.userService.findById(id);
    return { firstName: u.firstName, lastName: u.lastName };
  }
}
```

For deprecation, set response headers (`X-API-Deprecated`, `X-API-Sunset-Date`).

Reference: [Midway Versioning](https://midwayjs.org/docs/versioning)

---

### 8.17 Render Views with Template Engines

**Impact: MEDIUM** — "Server-side rendering with EJS/Nunjucks"

Use `@midwayjs/view-ejs` or `@midwayjs/view-nunjucks` for server-side rendering. Configure the `view` key with `mapping` (extension → engine), `defaultViewEngine`, and `rootDir` (multi-directory, mergeable). Render via `ctx.render(name, locals)`. Register custom Nunjucks filters in `onReady`. In egg (`@midwayjs/web`), disable the built-in view plugin to avoid conflicts.

**Incorrect (string concatenation, no template engine):**

```typescript
@Get('/page')
async page(@Query('name') name: string) {
  // ❌ manual HTML string building — error-prone, no escaping, unmaintainable
  return `<html><body><h1>Hello ${name}</h1></body></html>`;
}
```

**Correct (EJS engine + config + render):**

```typescript
import * as view from '@midwayjs/view-ejs';
@Configuration({ imports: [koa, view] })

// config.default.ts
export default {
  view: {
    defaultExtension: '.ejs',
    defaultViewEngine: 'ejs',
    mapping: { '.ejs': 'ejs' },
    rootDir: { default: join(appInfo.appDir, 'view') },
  },
} as MidwayConfig;

// view/hello.ejs:  <h1>Hello <%= data %></h1>
@Controller('/')
export class HomeController {
  @Inject() ctx: Context;

  @Get('/')
  async index() {
    await this.ctx.render('hello', { data: 'world' });
    // or with defaultExtension: this.ctx.render('hello', { data: 'world' })
  }
}
```

Nunjucks custom filters and custom engine (for Nunjucks, import from `@midwayjs/view-nunjucks`):
```typescript
// register filter in onReady (Nunjucks only — import from @midwayjs/view-nunjucks, NOT view-ejs)
import * as nunjucks from '@midwayjs/view-nunjucks';
@Inject() env: nunjucks.NunjucksEnvironment;
async onReady() {
  this.env.addFilter('upper', (s: string) => s.toUpperCase());
}

// custom engine implements IViewEngine, registered via viewManager.use('name', Engine)
```

Reference: [Midway View (EJS)](https://midwayjs.org/docs/extensions/render)

---

## 9. Microservices & Messaging

**Section Impact: MEDIUM**

### 9.1 Use @midwayjs/cron for Local Scheduled Tasks

**Impact: MEDIUM** — "Simple per-process cron jobs without Redis"

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

---

### 9.2 Use @OnEvent for Decoupled Event-Driven Processing

**Impact: MEDIUM-HIGH** — "Decouples modules without direct service dependencies"

Use the `@midwayjs/event-emitter` component (built on eventemitter2) for in-process pub/sub. Emitters fire events; listeners react without a direct dependency. **Critical:** `@OnEvent` handlers must be on a `@Singleton()` class. Emit synchronously with `emit()` or asynchronously with `emitAsync()`. **Best practice:** put `@OnEvent` handlers on a `@Singleton()` class (listeners lack request context; Singleton avoids scope issues)

**Incorrect (tight coupling — service knows every downstream consumer):**

```typescript
@Provide()
export class UserService {
  @Inject() emailService: EmailService;
  @Inject() analyticsService: AnalyticsService;
  @Inject() loyaltyService: LoyaltyService;

  async createUser(dto: CreateUserDTO) {
    const user = await this.repo.save(dto);
    // ❌ every new behavior requires editing this service
    await this.emailService.sendWelcome(user);
    await this.analyticsService.track('signup', user);
    await this.loyaltyService.addPoints(user.id, 100);
    return user;
  }
}
```

**Correct (emit event; listeners in separate modules react):**

```typescript
import { Provide, Singleton, Inject } from '@midwayjs/core';
import { OnEvent, EventEmitterService } from '@midwayjs/event-emitter';

@Provide()
export class UserService {
  @Inject() eventEmitter: EventEmitterService;

  async createUser(dto: CreateUserDTO) {
    const user = await this.repo.save(dto);
    // emit — no knowledge of consumers
    await this.eventEmitter.emitAsync('user.created', user);
    return user;
  }
}

// each listener in its own module — @Singleton is REQUIRED for @OnEvent
@Singleton()
export class EmailListener {
  @OnEvent('user.created', { suppressErrors: false })
  async sendWelcome(user: User) { await this.mailer.send(user.email); }
}

@Singleton()
export class AnalyticsListener {
  @OnEvent('user.created')   // wildcard 'user.*' also supported when config.wildcard=true
  async track(user: User) { await this.analytics.track('signup', user); }
}

// config.default.ts
export default {
  eventEmitter: { wildcard: true, delimiter: '.', maxListeners: 10 },
} as MidwayConfig;
```

Reference: [Midway Events](https://midwayjs.org/docs/extensions/events)

---

### 9.3 Build MCP Servers with @midwayjs/mcp

**Impact: MEDIUM** — "Expose Tools/Prompts/Resources to AI clients"

`@midwayjs/mcp` implements the Model Context Protocol for exposing Tools, Prompts, and Resources to AI clients (Cursor, Claude Desktop, etc.). Choose a transport: `stdio` (CLI, no HTTP needed), `stream-http` (recommended for web), or `sse` (deprecated). For HTTP transports you **must** also import a web framework (`@midwayjs/koa`/`express`). Define capabilities via `@Tool`/`@Prompt`/`@Resource` classes with `zod` schemas. Return `{ content, isError: true }` instead of throwing to surface errors to the AI client.

**Incorrect (incorrect transport config, throwing instead of structured error):**

```typescript
// ❌ stream-http without a web framework — nothing serves the endpoint
@Configuration({ imports: [mcp] })

@Tool('fetch', { inputSchema: { url: z.string() } })
export class FetchTool implements IMcpTool {
  async execute(args) {
    throw new Error('failed');   // ❌ crashes; AI client gets no structured error
  }
}
```

**Correct (HTTP transport + zod schemas + structured results + JWT auth):**

```typescript
import * as mcp from '@midwayjs/mcp';
import * as koa from '@midwayjs/koa';
@Configuration({ imports: [koa, mcp] })

// config.default.ts
export default {
  koa: { port: 3000 },
  mcp: {
    serverInfo: { name: 'my-mcp-server', version: '1.0.0' },
    transportType: 'stream-http',        // recommended; endpoints default to /mcp
    enableJwtAuthHelper: true,           // built-in JWT auth → ctx.authInfo
  },
} as MidwayConfig;

// Tool — zod input schema, CallToolResult output
import { Tool, IMcpTool, ToolConfig } from '@midwayjs/mcp';
import { CallToolResult } from '@modelcontextprotocol/sdk/types.js';
import { z } from 'zod';

const cfg: ToolConfig<{ city: z.ZodString }> = {
  description: 'Get weather for a city',
  inputSchema: { city: z.string().describe('City name') },
};
@Tool('get_weather', cfg)
export class WeatherTool implements IMcpTool {
  async execute(args: { city: string }): Promise<CallToolResult> {
    try {
      const temp = await this.weatherApi(args.city);
      return { content: [{ type: 'text', text: `${args.city}: ${temp}°C` }] };
    } catch (err) {
      return { content: [{ type: 'text', text: err.message }], isError: true };  // ✓ structured error
    }
  }
}

// Prompt — returns messages; Resource — uri template + mimeType
// @Prompt('greet', { argsSchema: { name: z.string() } }) → GetPromptResult
// @Resource('db://users/{id}', { uri: '...', mimeType: 'application/json' })

// dynamic registration at runtime
@Inject() mcpFramework: mcp.MidwayMCPFramework;
async onReady() {
  const server = this.mcpFramework.getServer();
  server.registerTool('dynamic_tool', { /* config */ }, async (args) => ({ /* result */ }));
}
```

One transport type per app instance. `ctx.authInfo` holds JWT-derived fields (`clientId`, `scopes`, `expiresAt`) when the JWT helper is enabled.

Reference: [Midway MCP](https://midwayjs.org/docs/extensions/mcp)

---

### 9.4 Use MQTT for IoT Messaging

**Impact: MEDIUM** — "Correct subscriber/publisher patterns with separate configs"

`@midwayjs/mqtt` splits into **subscribers** (`@MqttSubscriber(name)` + `IMqttSubscriber`) and **publishers** (service-factory pattern via `DefaultMqttProducer`). They are independent — use either alone. Subscribers are keyed by name matching the config; publishers support multi-instance via `mqtt.pub.clients`. The subscriber context (`ctx`) carries `topic`, `message` (Buffer), and `packet`. Serverless supports **publish only**.

**Incorrect (raw mqtt client, no lifecycle, no DI):**

```typescript
import mqtt from 'mqtt';   // ❌ bypasses the component
const client = mqtt.connect('mqtt://broker');
client.on('message', (topic, message) => { /* ❌ no DI, no graceful shutdown */ });
```

**Correct (Midway subscriber + publisher factory):**

```typescript
import * as mqtt from '@midwayjs/mqtt';
@Configuration({ imports: [mqtt] })

// config.default.ts — separate sub/pub configs
export default {
  mqtt: {
    sub: {
      sub1: { connectOptions: { host: 'broker.hivemq.com', port: 1883 }, subscribeOptions: { topicObject: 'sensors/temp' } },
    },
    pub: {
      clients: { default: { host: 'broker.hivemq.com', port: 1883 } },
    },
  },
} as MidwayConfig;

// subscriber — name must match config key
import { MqttSubscriber, IMqttSubscriber, Context } from '@midwayjs/mqtt';
@MqttSubscriber('sub1')
export class TempSubscriber implements IMqttSubscriber {
  @Inject() ctx: Context;
  async subscribe() {
    const temp = JSON.parse(this.ctx.message.toString()).value;  // message is Buffer
    // process reading
  }
}

// publisher — default or named instance
import { DefaultMqttProducer, MqttProducerFactory } from '@midwayjs/mqtt';
import { InjectClient } from '@midwayjs/core';
@Provide()
export class CommandPublisher {
  @Inject() producer: DefaultMqttProducer;                       // default
  @InjectClient(MqttProducerFactory, 'default') named: DefaultMqttProducer;
  async send(topic: string, msg: string) {
    await this.producer.publishAsync(topic, msg, { qos: 1 });
  }
}
```

Reference: [Midway MQTT](https://midwayjs.org/docs/extensions/mqtt)

---

### 9.5 Use the Correct Microservice Provider/Consumer Pattern

**Impact: MEDIUM** — "gRPC/RabbitMQ/Kafka each have specific decorator conventions"

Each Midway transport has distinct decorators. gRPC uses `@Provider(MSProviderType.GRPC)` + `@GrpcMethod()`. RabbitMQ/Kafka use `@Consumer(MSListenerType.X)` + `@RabbitMQListener`/`@KafkaConsumer`. Kafka producers use the service-factory pattern (`@InjectClient(KafkaProducerFactory, 'name')`). Each transport has its own `Context` type, framework, and (for most) an independent logger. Match the decorator to the transport — do not mix conventions.

**Incorrect (wrong decorator for the transport, ignoring context typing):**

```typescript
// ❌ using @Provide for a gRPC service (should be @Provider)
@Provide()
export class Greeter { async sayHello(req) { return {}; } }

// ❌ RabbitMQ consumer that never acks the message (redelivered forever)
@Consumer(MSListenerType.RABBITMQ)
export class UserConsumer {
  @RabbitMQListener('tasks')
  async gotData(msg: ConsumeMessage) {
    await this.process(msg);   // ❌ missing ctx.channel.ack(msg)
  }
}
```

**Correct (transport-specific decorators + typed context + ack):**

```typescript
// === gRPC Provider (server side) ===
import { MSProviderType, Provider, GrpcMethod, Inject } from '@midwayjs/core';
import { Context } from '@midwayjs/grpc';

@Provider(MSProviderType.GRPC, { package: 'helloworld' })
export class Greeter implements helloworld.Greeter {
  @Inject() ctx: Context;
  @GrpcMethod()
  async sayHello(request: helloworld.HelloRequest) {
    return { message: 'Hello ' + request.name };
  }
}
// gRPC client: this.grpcClients.getService<helloworld.GreeterClient>('helloworld.Greeter')

// === RabbitMQ Consumer (must ack!) ===
import { Consumer, MSListenerType, RabbitMQListener } from '@midwayjs/core';
import { Context } from '@midwayjs/rabbitmq';
import { ConsumeMessage } from 'amqplib';

@Consumer(MSListenerType.RABBITMQ)
export class TaskConsumer {
  @Inject() ctx: Context;   // { channel, requestContext }
  @RabbitMQListener('tasks', {
    exchange: 'logs',
    exchangeOptions: { type: 'fanout', durable: false },
    consumeOptions: { noAck: false },
  })
  async gotData(msg: ConsumeMessage) {
    try {
      await this.process(msg.content.toString());
    } finally {
      this.ctx.channel.ack(msg);   // ✓ always ack (or nack for retry)
    }
  }
}

// === Kafka Consumer + Producer ===
import { KafkaConsumer, IKafkaConsumer } from '@midwayjs/kafka';
import { KafkaJS } from '@midwayjs/kafka';   // KafkaJS namespace re-exports kafkajs types

@KafkaConsumer('sub1')
export class OrderConsumer implements IKafkaConsumer {
  async eachMessage(payload: KafkaJS.EachMessagePayload) { /* ... */ }
}

// Kafka producer via service-factory
import { InjectClient } from '@midwayjs/core';
import { KafkaProducerFactory } from '@midwayjs/kafka';
@Provide()
export class OrderProducer {
  @InjectClient(KafkaProducerFactory, 'pub1') producer: KafkaJS.Producer;
  async publish(topic: string, key: string, value: string) {
    await this.producer.send({ topic, messages: [{ key, value }] });
  }
}
```

WebSocket: `@WSController(namespace)` + `@OnWSMessage`/`@WSEmit` (socket.io) or `@WSController()` + `@OnWSMessage`/`@WSBroadCast` (ws).

Reference: [Midway gRPC](https://midwayjs.org/docs/extensions/grpc), [RabbitMQ](https://midwayjs.org/docs/extensions/rabbitmq), [Kafka](https://midwayjs.org/docs/extensions/kafka)

---

### 9.6 Use BullMQ for Reliable Background Job Processing

**Impact: MEDIUM-HIGH** — "Reliable, retryable, distributed background jobs"

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

---

## 10. DevOps & Deployment

**Section Impact: LOW-MEDIUM**

### 10.1 Use the Midway Toolchain (mwtsc, mwts, create-midway)

**Impact: MEDIUM** — "Correct dev/build/lint/scaffold tooling for v4"

v4 standardizes its toolchain: `create-midway` for scaffolding, `mwtsc` for dev/build (a `tsc` wrapper with `--run` for auto-restart), `mwts` for lint/format (ESLint Flat Config + Prettier), and `tsc` for production builds. The legacy `@midwayjs/cli` (`midway-bin`) is deprecated — prefer `mwtsc`/`tsc`/`jest`/`mocha`. Always use `npm init midway@latest` (the `@latest` tag is required to get current templates).

**Incorrect (deprecated CLI, manual tsc watch without restart, missing lint):**

```bash
# ❌ deprecated midway-bin dev (use mwtsc)
midway-bin dev --ts

# ❌ tsc watch without restarting the server on change
tsc --watch   # compiles but never restarts the app

# ❌ no linting configured
```

**Correct (mwtsc dev + tsc build + mwts lint + create-midway scaffold):**

```bash
# scaffold a v4 project (always use @latest)
npm init midway@latest -- --type=koa-v4
# other v4 templates: koa-v4-esm, react-functional-v4, vue-functional-v4
```

```json
// package.json
{
  "scripts": {
    "dev": "cross-env NODE_ENV=local mwtsc --watch --run @midwayjs/mock/app --port 7001",
    "build": "cross-env rm -rf dist && tsc",
    "lint": "mwts check",
    "lint:fix": "mwts fix",
    "test": "jest --runInBand"
  }
}
```

```typescript
// mwtsc --run: recompiles + restarts on code change (the @midwayjs/mock/app starts the framework)
// Serverless dev: mwtsc --watch --run @midwayjs/mock/function

// mwts — ESLint Flat Config (eslint.config.js)
const mwtsConfig = require('mwts/eslint.config.js');
module.exports = [
  { ignores: ['**/node_modules', '**/dist'] },
  ...mwtsConfig,
];
// migrate from mwts 1.x: npx mwts migrate
```

Key `mwtsc` flags: `--watch` (watch mode), `--run <file>` (restart on change — must be **last**), `--port` (override HTTP port), `--ssl` (HTTPS test cert). Key `mwts` commands: `mwts check`, `mwts fix`, `mwts init`, `mwts migrate`.

Reference: [Midway mwtsc](https://midwayjs.org/docs/tool/mwtsc), [mwts](https://midwayjs.org/docs/tool/mwts), [create-midway](https://midwayjs.org/docs/tool/create_midway)

---

### 10.2 Build CLI Tools with @midwayjs/commander

**Impact: LOW-MEDIUM** — "DI-powered command-line tools with full Midway context"

`@midwayjs/commander` (built on commander.js) lets you build CLI tools with full Midway DI. Each command is a `@Command()` class implementing `CommandRunner.run()`, with `@Option` for flags and `EnquirerService` for interactive prompts. Run via `node bootstrap.js <command> [args]`. Each command execution creates a request context, so `@Inject`/`@Config` work normally. Test via `framework.runCommand(...)` rather than mocking `process.argv`.

**Incorrect (plain script with no DI, manual argv parsing):**

```typescript
// ❌ no DI, manual parsing, no Midway services available
const args = process.argv.slice(2);
const name = args[0];
console.log(`hello ${name}`);
```

**Correct (DI-powered command + options + structured output):**

```typescript
import * as commander from '@midwayjs/commander';
@Configuration({ imports: [commander] })

// src/command/hello.command.ts
import { Command, CommandRunner, Option } from '@midwayjs/commander';
import { Inject, ILogger } from '@midwayjs/core';

@Command({ name: 'hello', description: 'Greet a user', arguments: '<name>', aliases: ['hi'] })
export class HelloCommand implements CommandRunner {
  @Inject() logger: ILogger;
  @Inject() userService: UserService;   // ✓ full DI available

  @Option({ flags: '-l, --lang [lang]', description: 'language', defaultValue: 'en' })
  parseLang(val: string) { return val; }

  async run(passedParams: string[], options?: { lang?: string }) {
    const name = passedParams[0];
    const greeting = options?.lang === 'zh' ? `你好 ${name}` : `Hello ${name}`;
    this.logger.info(greeting);
    return greeting;   // string→stdout, object→JSON, Readable→pipe
  }
}

// run: node bootstrap.js hello world --lang zh
// aliases: node bootstrap.js hi world

// interactive prompts
import { QuestionSet, Question, EnquirerService } from '@midwayjs/commander';
@QuestionSet()
export class DeployQuestions {
  @Question({ type: 'input', name: 'env', message: 'Target environment?' })
  env: string;
}
// const answers = await this.enquirer.prompt(DeployQuestions);
```

Reference: [Midway Commander](https://midwayjs.org/docs/extensions/commander)

---

### 10.3 Use Config Centers and Service Discovery (Consul/etcd)

**Impact: MEDIUM** — "Centralized config and distributed service registration"

`@midwayjs/consul` and `@midwayjs/etcd` both use the Service Factory pattern (`client`/`clients`) and provide a unified v4 **Service Discovery** abstraction (`XxxServiceDiscovery`). For config, poll the KV store in a `@Singleton` service with `@Init()` + `setInterval` — never call `kv.get` per request (it hammers the server). For service registration, call `createClient()` in `onServerReady`, then `register()` (idempotent) and `getInstances()`.

**Incorrect (per-request KV lookup, no registration caching):**

```typescript
// ❌ KV read on every request — overwhelms Consul
@Provide()
export class ConfigService {
  @Inject() consul: ConsulService;
  async getValue(key: string) {
    return (await this.consul.kv.get(key)).Value;   // ❌ network call per request
  }
}
```

**Correct (singleton-cached config + service registration + discovery):**

```typescript
import * as consul from '@midwayjs/consul';
@Configuration({ imports: [consul] })

// config.default.ts
export default {
  consul: {
    client: { host: '127.0.0.1', port: 8500 },
    serviceDiscovery: {},
  },
} as MidwayConfig;

// config polling — Singleton caches the value
import { Provide, Scope, ScopeEnum, Init, Inject } from '@midwayjs/core';
import { ConsulService } from '@midwayjs/consul';
@Singleton()
export class DynamicConfigService {
  @Inject() consul: ConsulService;
  private config: Record<string, any>;

  @Init()
  async init() {
    await this.refresh();
    setInterval(() => this.refresh(), 5000);   // poll every 5s, not per request
  }
  private async refresh() { this.config = (await this.consul.kv.get('app/config')).Value; }
  get<T>(key: string): T { return this.config?.[key]; }
}

// service registration in onServerReady
import { ConsulServiceDiscovery } from '@midwayjs/consul';
@Configuration({})
export class MainConfiguration {
  @Inject() discovery: ConsulServiceDiscovery;
  async onServerReady() {
    const client = this.discovery.createClient();
    await client.register({                    // idempotent
      id: 'order-1', name: 'order-service',
      address: '10.0.0.1', port: 7001,
      check: { ttl: '10s' },                   // health-check TTL
      meta: { version: '1.0.0' },
    });
  }
}

// discover instances
const instances = await this.discovery.getInstances({ service: 'order-service' });
```

etcd follows the same pattern (`ETCDService`, `EtcdServiceDiscovery`, `etcd.client.host: [...]`). Consul `getInstance` takes `{ service }` (object); etcd takes a bare string — check each type.

### The General ServiceDiscovery Abstraction

Midway provides a unified abstraction in `@midwayjs/core` for implementing custom service discovery against any registry. Extend `ServiceDiscoveryClient` (instance lifecycle: `register`/`deregister`/`online`/`offline`) and `ServiceDiscovery` (entry: `createClient`/`getInstances`/`getInstance` with built-in load balancing). Built-in load balancers: `LoadBalancerType.RANDOM`, `ROUND_ROBIN`. Health checks via `ServiceDiscoveryHealthCheckFactory.create('ttl'|'http'|'tcp', opts)`.

```typescript
import { ServiceDiscovery, ServiceDiscoveryClient, LoadBalancerType } from '@midwayjs/core';

// 1. implement the client (talks to your registry)
class MySDClient extends ServiceDiscoveryClient<any, MyOptions, MyInstance> {
  async register(inst: MyInstance) { /* write to registry */ }
  async deregister() { /* remove */ }
  async online() { /* mark available */ }
  async offline() { /* mark unavailable */ }
  async beforeStop() { /* stop heartbeat/timers */ }
}

// 2. implement the entry class
class MyServiceDiscovery extends ServiceDiscovery<any, MyOptions, MyInstance, MyInstance, string> {
  protected getServiceClient() { return {}; }
  protected createServiceDiscoverClientImpl(opts: MyOptions) { return new MySDClient(this.getServiceClient(), opts); }
  protected getDefaultServiceDiscoveryOptions(): MyOptions { return { ttl: 30 }; }
  public async getInstances(serviceName: string): Promise<MyInstance[]> { /* query registry */ return []; }
}

// 3. use
const discovery = await container.getAsync(MyServiceDiscovery);
discovery.setLoadBalancer(LoadBalancerType.ROUND_ROBIN);
const client = discovery.createClient({ ttl: 20 });
await client.register({ serviceName: 'order', id: 'order-1' });
const instances = await discovery.getInstances('order');
const one = await discovery.getInstance('order');   // load-balanced pick
// on shutdown: ServiceDiscovery.destroy() calls stop() on all clients → deregister
```

Reference: [Midway Service Discovery](https://midwayjs.org/docs/service_discovery), [Consul](https://midwayjs.org/docs/extensions/consul), [etcd](https://midwayjs.org/docs/extensions/etcd)

---

### 10.4 Debug with Request Tracing (@midwayjs/code-dye)

**Impact: LOW-MEDIUM** — "Per-method timing and call-chain tracing — dev only"

`@midwayjs/code-dye` traces per-method execution time, call chains, args, and return values for a single request. **Gate it to dev only** via `{ component: codeDye, enabledEnvironment: ['local'] }` — it instruments every method and has a strong performance cost. Trigger tracing per-request via a query param (`?codeDyeXXX=html`) or header; report types are `html` (inline report), `json`, or `log` (no response modification).

**Incorrect (enabling in production, or always-on instrumentation):**

```typescript
// ❌ enabled in ALL environments — production perf disaster
@Configuration({ imports: [codeDye] })
```

**Correct (env-gated + opt-in per-request trigger):**

```typescript
import * as codeDye from '@midwayjs/code-dye';

// configuration.ts — dev only
@Configuration({
  imports: [
    { component: codeDye, enabledEnvironment: ['local', 'qa'] },   // never prod
  ],
})

// config.local.ts
export default {
  codeDye: {
    matchQueryKey: 'codeDyeABC',    // trigger: ?codeDyeABC=<reportType>
    matchHeaderKey: 'codeDyeHeader',
  },
} as MidwayConfig;

// Request: GET /api/users?codeDyeABC=html → HTML trace report in response
//          GET /api/users?codeDyeABC=json → JSON trace in response
//          GET /api/users?codeDyeABC=log  → trace to logs, normal response
```

Only dyed requests (carrying the trigger param) pay the instrumentation cost.

Reference: [Midway Code Dye](https://midwayjs.org/docs/extensions/code_dye)

---

### 10.5 Implement Graceful Shutdown and Deployment Correctly

**Impact: MEDIUM-HIGH** — "Zero-downtime deployments and clean resource release"

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

---

### 10.6 Deploy FaaS to Aliyun FC and AWS Lambda

**Impact: MEDIUM** — "Correct build pipeline and platform-specific packaging"

Aliyun FC and AWS Lambda use **different** deployment paths. Aliyun FC pure-function mode uses `@midwayjs/fc-starter` + `f.yml` + `s.yaml` (Serverless Devs), auto-generating entry files with `@midwayjs/serverless-yaml-generator`. AWS Lambda uses a **standard Midway app** packaged via SAM (`template.yaml`), not FaaS decorators. In both cases: build with `tsc`, set the platform port only in `bootstrap.js` (FC=9000, Lambda=8080) to avoid breaking local dev, and install production deps in the deploy artifact.

**Incorrect (wrong platform tooling, port in config files):**

```typescript
// ❌ using f.yml/s.yaml for AWS Lambda (Aliyun-only)
// ❌ hardcoding the platform port in config.default.ts (breaks local dev on 7001)
export default { koa: { port: 9000 } } as MidwayConfig;
```

**Correct (Aliyun FC pure-function + AWS Lambda SAM):**

```typescript
// === Aliyun FC — bootstrap.js sets port 9000 ONLY for the platform ===
const { Bootstrap } = require('@midwayjs/bootstrap');
Bootstrap.configure({ globalConfig: { koa: { port: 9000 } } }).run();

// f.yml (minimal — platform + starter)
// provider:
//   name: aliyun
//   starter: '@midwayjs/fc-starter'
```

```bash
# Aliyun FC deploy pipeline (deploy.sh)
npm i
tsc
serverless-yaml-generator        # auto-generates s.yaml + entry files from @ServerlessTrigger
mkdir .serverless
cp -r dist *.json *.yaml *.js .serverless/
cd .serverless
npm install --production
s deploy                         # Serverless Devs CLI
```

```yaml
# AWS Lambda — template.yaml (SAM), standard app (no FaaS decorators)
AWSTemplateFormatVersion: "2010-09-09"
Transform: AWS::Serverless-2016-10-31
Resources:
  ApiFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: ./dist.zip
      Handler: dist/index/index.handler
      Runtime: nodejs20.x
      Timeout: 900
      Events:
        ApiEvent:
          Type: Api
          Properties:
            Path: /{any+}
            Method: ANY
```

```bash
# AWS Lambda deploy
# bootstrap.js → Bootstrap.configure({ globalConfig: { koa: { port: 8080 } } }).run();
cd sam && sam build && sam deploy
```

Error handling: production returns "Internal Server Error" (never leaks stacks); override with `SERVERLESS_OUTPUT_ERROR_STACK=true` for debugging. POST body parsing differs per gateway (base64 JSON vs urlencoded) — see `serverless_post_difference` doc.

Reference: [Midway Aliyun FC](https://midwayjs.org/docs/serverless/aliyun_faas), [AWS Lambda](https://midwayjs.org/docs/serverless/aws_lambda)

---

### 10.7 Use Structured Logging via midwayLogger

**Impact: MEDIUM-HIGH** — "Structured logging enables effective debugging and monitoring"

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

---

### 10.8 Expose Metrics with @midwayjs/prometheus

**Impact: MEDIUM** — "Auto-exposed /metrics for Prometheus scraping"

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

---

### 10.9 Use Multi-Environment Configuration with Validation

**Impact: HIGH** — "Proper configuration prevents deployment failures"

Midway loads config by environment: `config.default.ts` (all environments) + `config.{env}.ts` (env-specific), merged via `extend2` (arrays are overwritten, not merged). Map configs explicitly by environment key in `importConfigs`. Type every config file as `MidwayConfig` (and component-specific via declaration merging). Inject values with `@Config('dotted.path')` or `@AllConfig()` (v4 replaces `@Config(ALL)`). Never read `@Config` in a constructor — use `@Init`. For runtime/remote config, return it from `onConfigLoad`.

**Incorrect (raw process.env scattered, untyped config, accessing config in constructor):**

```typescript
@Provide()
export class DatabaseService {
  constructor() {
    // ❌ process.env scattered, NaN on missing, untyped
    this.pool = mysql.createPool({
      host: process.env.DB_HOST,
      port: parseInt(process.env.DB_PORT),    // NaN if missing
    });
  }
}

// ❌ untyped config object
export default { koa: { port: 7001 } };
```

**Correct (typed MidwayConfig + explicit env mapping + @Config injection):**

```typescript
// config/config.default.ts — loaded in ALL environments
import { MidwayConfig } from '@midwayjs/core';
export default {
  keys: process.env.APP_KEYS,
  koa: { port: 7001, globalPrefix: '/api' },
  typeorm: { dataSource: { default: {
    type: 'mysql', host: process.env.DB_HOST, port: Number(process.env.DB_PORT),
  } } },
} as MidwayConfig;

// config/config.local.ts — dev only
import { MidwayConfig } from '@midwayjs/core';
export default {
  typeorm: { dataSource: { default: { synchronize: true } } },  // dev-only sync
} as MidwayConfig;

// config/config.prod.ts — prod only
import { MidwayConfig } from '@midwayjs/core';
export default {
  typeorm: { dataSource: { default: { synchronize: false } } }, // NEVER sync in prod
} as MidwayConfig;

// configuration.ts — explicit environment mapping
@Configuration({
  importConfigs: [{ default: DefaultConfig, local: LocalConfig, prod: ProdConfig }],
})

// service — inject typed config (NOT in constructor)
import { Provide, Config, Init } from '@midwayjs/core';
@Provide()
export class DatabaseService {
  @Config('typeorm.dataSource.default') dbConfig: any;

  @Init()
  async init() { this.pool = mysql.createPool(this.dbConfig); }  // ✓ available in @Init
}

// remote/dynamic config via onConfigLoad (return value is merged)
async onConfigLoad(container: IMidwayContainer) {
  return await fetchRemoteConfig();
}
```

Config load order: component `default` → app `default` → component `{env}` → app `{env}`. The function form `(appInfo: MidwayAppInfo) => MidwayConfig` gives access to `appDir`, `baseDir`, `HOME`, etc. for path resolution.

Reference: [Midway Environment Config](https://midwayjs.org/docs/env_config)

---

### 10.10 Manage Processes for Production (PM2/cfork)

**Impact: MEDIUM** — "Cluster-mode deployment and zero-downtime restarts"

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

---

### 10.11 Enable Distributed Tracing with OpenTelemetry

**Impact: MEDIUM-HIGH** — "v4 merged tracing into core — trace across services"

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

---

## References

- https://midwayjs.org
- https://github.com/midwayjs/midway
- https://github.com/cool-team-official/cool-admin-midway
- https://www.midwayjs.org/docs/upgrade_v4
- https://www.midwayjs.org/docs/container

---

*Generated by build-agents.ts on 2026-06-29*
