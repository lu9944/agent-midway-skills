---
name: midway-best-practices
description: >
  Midway.js v4 best practices and architecture patterns for building production-ready
  applications. Use when writing, reviewing, or refactoring Midway code: configuration
  composition, dependency injection, scopes, validation, error handling, TypeORM v4,
  microservices, FaaS/Serverless, security, testing, and deployment. 82 rules across
  10 categories, each in a separate reference file loaded on demand.
license: MIT
compatibility: Designed for opencode, Claude Code, Codex, Cursor, and any agent supporting the Agent Skills standard
metadata:
  author: agent-midway-skills
  version: "1.0.0"
  midway-version: "v4.2.1"
  rule-count: "82"
---

# Midway.js v4 Best Practices

Comprehensive best-practice skill for **Midway.js v4** (tested against **v4.2.1**).
82 rules across 10 categories, each in its own reference file under `references/`.

Load only the rules relevant to your task — each file is self-contained with
incorrect vs. correct examples and v4-specific guidance.

## When to Use

- Writing new Midway controllers, services, entities, or configuration
- Composing `@Configuration` and wiring components
- Implementing authentication (guards, JWT, passport) or input validation
- Using the IoC container (scopes, injection identifiers, circular dependencies)
- Working with TypeORM / Sequelize / Redis / MongoDB / MikroORM / Leoric / Prisma
- Building microservices (gRPC, RabbitMQ, Kafka, MQTT), events, task queues, or WebSocket push
- Handling file uploads, i18n, Swagger docs, or GraphQL
- Deploying FaaS functions to Aliyun FC or AWS Lambda
- Reviewing or refactoring existing Midway codebases

## When NOT to Use

- **NestJS** — different decorators and DI; use a NestJS skill instead
- **Midway v3 or earlier** — v4 has breaking API changes; this skill targets v4 only
- **Pure frontend apps** without a Midway backend
- **Raw Express/Koa** without the Midway IoC container
- **Framework internals development** (contributing to `midwayjs/midway` repo)

## v4 Quick Reference

| Concern | v4 Change |
|---------|-----------|
| Imports | Everything from `@midwayjs/core` (`@midwayjs/decorator` removed) |
| File detection | Declare `new CommonJSFileDetector()` explicitly — no implicit auto-scan |
| Config | `@AllConfig()` replaces `@Config(ALL)`; `@MainApp()` replaces empty `@App()` |
| Validation | `@midwayjs/validation` (not deprecated `@midwayjs/validate`) |
| TypeORM | `@midwayjs/typeorm` with datasource config form, native `@Entity` (no `@EntityModel`) |
| Sequelize | `@Table` from `sequelize-typescript` replaces removed `@BaseTable` |
| Node.js | Requires Node v20+ |

---

## Task-to-Rule Decision Guide

When a task matches, load the corresponding reference file via `Read references/<file>`.

### Starting a new project or module

| Task | Reference |
|------|-----------|
| Setting up `@Configuration` with components | `references/arch-configuration-composition.md` |
| Organizing directory structure by feature | `references/arch-module-structure.md` |
| Declaring the file detector (v4 required) | `references/arch-file-detector.md` |
| Enabling ESM support | `references/arch-esm.md` |

### Dependency injection issues

| Task | Reference |
|------|-----------|
| Using `@Provide` / `@Inject` correctly | `references/di-property-injection.md` |
| Choosing Singleton vs Request vs Prototype scope | `references/di-scope-awareness.md` |
| Why constructor injection fails | `references/di-no-constructor-access.md` |
| Resolving circular dependencies | `references/di-circular-dependency.md` |
| Request-scoped service in Singleton middleware | `references/di-request-context-resolution.md` |
| Using proper injection identifiers | `references/di-injection-identifiers.md` |

### Error handling

| Task | Reference |
|------|-----------|
| Throwing structured errors | `references/error-midway-error.md` |
| Centralized error catching | `references/error-filters.md` |

### Security

| Task | Reference |
|------|-----------|
| Route-level authorization with `@Guard` | `references/security-guards.md` |
| JWT authentication | `references/security-jwt.md` |
| Input validation with `@midwayjs/validation` | `references/security-validation.md` |
| CORS and security headers | `references/security-cors-headers.md` |
| OAuth / strategy-based auth with Passport | `references/security-passport.md` |
| Anti-bot captcha | `references/security-captcha.md` |
| Fine-grained RBAC/ABAC with Casbin | `references/security-casbin.md` |

### Performance

| Task | Reference |
|------|-----------|
| Async lifecycle hooks (`@Init`, `@Destroy`) | `references/perf-lifecycle-async.md` |
| Caching with `@midwayjs/cache-manager` | `references/perf-caching.md` |
| Async context (request-scoped data) | `references/perf-async-context.md` |
| CPU-bound work in thread pools | `references/perf-thread-pool.md` |
| Retry patterns | `references/perf-retry.md` |
| Reactive data subscriptions | `references/perf-data-listener.md` |

### Testing

| Task | Reference |
|------|-----------|
| App bootstrap testing with `@midwayjs/mock` | `references/test-mock-framework.md` |
| HTTP E2E testing with Supertest | `references/test-http-supertest.md` |
| FaaS function testing | `references/test-faas-testing.md` |

### Database & ORM

| Task | Reference |
|------|-----------|
| TypeORM v4 datasource patterns | `references/db-typeorm-v4.md` |
| Multiple datasources & transactions | `references/db-multi-datasource.md` |
| Redis service factory | `references/db-redis.md` |
| Sequelize v4 (`@Table` replaces `@BaseTable`) | `references/db-sequelize.md` |
| MongoDB with Typegoose/Mongoose | `references/db-mongodb.md` |
| MikroORM v6/v7 | `references/db-mikro.md` |
| Leoric ORM (lightweight Model API) | `references/db-leoric.md` |
| Object storage (COS/OSS/TableStore) | `references/db-object-storage.md` |
| Prisma ORM | `references/db-prisma.md` |

### API design

| Task | Reference |
|------|-----------|
| Controllers & routing decorators | `references/api-controllers.md` |
| Middleware scoping (match/ignore) | `references/api-middleware-scoping.md` |
| Pipes for input transformation | `references/api-pipes.md` |
| AOP with `@Aspect` | `references/api-aspects.md` |
| Standardized response format | `references/api-response-format.md` |
| HTTP client (axios HttpService) | `references/api-http-client.md` |
| File uploads with `@midwayjs/busboy` | `references/api-file-upload.md` |
| Static file serving | `references/api-static-files.md` |
| Swagger API docs | `references/api-swagger.md` |
| View/template rendering | `references/api-view-rendering.md` |
| CRUD generation with `@midwayjs/crud` | `references/api-crud.md` |
| HTTP proxy | `references/api-http-proxy.md` |
| Internationalization (i18n) | `references/api-i18n.md` |
| Multi-tenancy | `references/api-tenant.md` |
| GraphQL with `@midwayjs/apollo` | `references/api-graphql.md` |
| API versioning | `references/api-versioning.md` |
| Cookies & sessions | `references/api-cookie-session.md` |

### Microservices & messaging

| Task | Reference |
|------|-----------|
| Event-driven processing with `@OnEvent` | `references/micro-events.md` |
| Background jobs with BullMQ | `references/micro-task-queues.md` |
| Scheduled tasks with `@midwayjs/cron` | `references/micro-cron.md` |
| Microservice provider/consumer pattern | `references/micro-provider-consumer.md` |
| MQTT for IoT | `references/micro-mqtt.md` |
| WebSocket (socket.io vs ws) | `references/micro-websocket.md` |
| MCP servers with `@midwayjs/mcp` | `references/micro-mcp.md` |

### DevOps & deployment

| Task | Reference |
|------|-----------|
| Multi-environment configuration | `references/devops-multi-env-config.md` |
| Structured logging | `references/devops-logging.md` |
| Graceful shutdown & deployment | `references/devops-deployment.md` |
| Prometheus metrics | `references/devops-metrics.md` |
| Config centers & service discovery (Consul/etcd) | `references/devops-config-discovery.md` |
| Process management (PM2/cfork) | `references/devops-process-management.md` |
| Request tracing debug (`@midwayjs/code-dye`) | `references/devops-debug-tracing.md` |
| CLI tools with `@midwayjs/commander` | `references/devops-cli.md` |
| FaaS deployment (Aliyun FC / AWS Lambda) | `references/devops-faas-deployment.md` |
| Distributed tracing (OpenTelemetry) | `references/devops-tracing.md` |
| Midway toolchain (mwtsc, mwts) | `references/devops-cli-tools.md` |
| One-time scripts with `@midwayjs/one-shot` | `references/devops-one-shot.md` |
| Official skill bundle (`@midwayjs/skill-midway`) | `references/devops-skill-midway.md` |

### Advanced architecture

| Task | Reference |
|------|-----------|
| Building reusable components/frameworks | `references/arch-component-development.md` |
| FaaS functions with `@ServerlessTrigger` | `references/arch-faas-functions.md` |
| Functional API (declarative routes) | `references/arch-functional-api.md` |
| Fullstack with Midway Hooks (⚠️ deprecated → use Functional API) | `references/arch-hooks-fullstack.md` |
| Extending Context with typed properties | `references/arch-context-extension.md` |
| Custom decorators (DecoratorManager) | `references/arch-custom-decorator.md` |
| ServiceFactory for multi-instance services | `references/arch-service-factory.md` |
| `@Autoload` for self-initializing classes | `references/arch-autoload.md` |

---

## How to Use This Skill

1. **Identify the task** — match against the decision guide above
2. **Read the reference file** — `Read references/<file>.md`
3. **Apply the pattern** — each file has incorrect vs correct examples with explanations

### Rule file format

Each reference file contains:
- YAML frontmatter (`title`, `impact`, `impactDescription`, `tags`)
- Brief explanation of why the rule matters
- Incorrect code example with explanation
- Correct code example with explanation
- Midway documentation link

## Category Priority

| Priority | Category | Impact | File Prefix |
|----------|----------|--------|-------------|
| 1 | Architecture | CRITICAL | `arch-` |
| 2 | Dependency Injection | CRITICAL | `di-` |
| 3 | Error Handling | HIGH | `error-` |
| 4 | Security | HIGH | `security-` |
| 5 | Performance | HIGH | `perf-` |
| 6 | Testing | MEDIUM-HIGH | `test-` |
| 7 | Database & ORM | MEDIUM-HIGH | `db-` |
| 8 | API Design | MEDIUM | `api-` |
| 9 | Microservices & Messaging | MEDIUM | `micro-` |
| 10 | DevOps & Deployment | LOW-MEDIUM | `devops-` |
