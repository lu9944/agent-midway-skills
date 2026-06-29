---
name: midway-best-practices
description: Midway.js v4 best practices and architecture patterns for building production-ready applications. This skill should be used when writing, reviewing, or refactoring Midway code to ensure proper patterns for configuration composition, dependency injection, scopes, validation, error handling, TypeORM v4, and microservices.
license: MIT
tags: midway, midwayjs, v4, ioc, dependency-injection, typeorm, serverless, faas, nodejs, typescript
metadata:
  author: agent-midway-skills
  version: "1.0.0"
---

# Midway Best Practices

Comprehensive best practices guide for **Midway.js v4** applications. Contains 78 rules across 10 categories, prioritized by impact to guide automated refactoring and code generation. All examples use v4 APIs (`@midwayjs/core`, explicit `CommonJSFileDetector`, `@midwayjs/validation`, `@midwayjs/typeorm`, `@AllConfig()`, etc.) and reflect the real-world patterns from the `cool-admin-midway` reference project. Covers every component in the `@midwayjs/*` v4 extension catalog plus Serverless (FaaS) deployment.

## When to Apply

Reference these guidelines when:

- Writing new Midway controllers, services, entities, or configuration
- Composing `@Configuration` and wiring components
- Implementing authentication (guards, JWT, passport) or input validation
- Using the IoC container (scopes, injection identifiers, circular dependencies)
- Working with TypeORM/Sequelize/Redis/MongoDB/MikroORM in v4 datasource form
- Building microservices (gRPC, RabbitMQ, Kafka, MQTT), event emitters, or task queues
- Handling file uploads, i18n, static files, Swagger docs, or GraphQL
- Using object storage (COS/OSS), config centers (Consul/etcd), or monitoring (Prometheus)
- Reviewing or refactoring existing Midway codebases
- Setting up multi-environment config, logging, multi-tenancy, or deployment

## When NOT to Apply

Do **not** use this skill when:

- **The project uses NestJS** — NestJS has different decorators (`@Module`, `@Injectable`), DI, and patterns; use a NestJS-specific skill instead
- **The project uses Midway v3 or earlier** — v4 has breaking changes (removed `@midwayjs/decorator`, `CommonJSFileDetector`, `@AllConfig()`, `@EntityModel`); this skill targets v4 only
- **The project is a pure frontend app** (React/Vue without a Midway backend) — no Midway server-side code to apply patterns to
- **The task is raw Express/Koa without Midway** — this skill assumes the Midway IoC container, `@Configuration`, and component system
- **The task is about the Midway framework's internal source code** (contributing to `midwayjs/midway`) — this skill covers application-level usage, not framework internals development

## v4-Specific Notes (Critical)

- **Imports**: everything comes from `@midwayjs/core` (the `@midwayjs/decorator` package was removed)
- **Detector**: v4 removed implicit auto-scan — declare `new CommonJSFileDetector()` in `@Configuration`
- **Config**: `@AllConfig()` replaces `@Config(ALL)`; `@MainApp()` replaces empty `@App()`
- **Validation**: prefer `@midwayjs/validation` (joi/zod/class-validator) over the deprecated `@midwayjs/validate`
- **TypeORM**: use `@midwayjs/typeorm` (not `@midwayjs/orm`), datasource config form, native `@Entity` (no `@EntityModel`)
- **Sequelize**: `@Table` from `sequelize-typescript` replaces the removed `@BaseTable`
- **Node.js**: v4 requires Node v20+

## Rule Categories by Priority

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
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

## Quick Reference

### 1. Architecture (CRITICAL)

- `arch-configuration-composition` - Use @Configuration with explicit component composition + detector
- `arch-module-structure` - Organize by feature modules with directory conventions
- `arch-file-detector` - Declare the v4 file detector explicitly
- `arch-component-development` - Develop reusable components and custom frameworks
- `arch-faas-functions` - Write FaaS functions with @ServerlessTrigger (Serverless)
- `arch-functional-api` - Use the Functional API for declarative routes
- `arch-hooks-fullstack` - Build fullstack apps with Midway Hooks
- `arch-esm` - Enable ESM support correctly
- `arch-context-extension` - Extend the Context with typed custom properties
- `arch-custom-decorator` - Build custom decorators with DecoratorManager and MetadataManager
- `arch-service-factory` - Extend ServiceFactory for multi-instance services
- `arch-autoload` - Use @Autoload for self-initializing classes

### 2. Dependency Injection (CRITICAL)

- `di-property-injection` - Use property injection with @Provide/@Inject pairing
- `di-scope-awareness` - Understand Singleton, Request, and Prototype scopes
- `di-no-constructor-access` - Never access injected deps in the constructor
- `di-circular-dependency` - Resolve circular dependencies with @LazyInject
- `di-request-context-resolution` - Resolve Request-scoped services in Singleton middleware
- `di-injection-identifiers` - Use defined injection identifiers, not magic strings

### 3. Error Handling (HIGH)

- `error-midway-error` - Extend MidwayError and MidwayHttpError
- `error-filters` - Use @Catch error filters for centralized handling

### 4. Security (HIGH)

- `security-guards` - Use @Guard for route-level authorization
- `security-jwt` - Implement secure JWT authentication
- `security-validation` - Validate all input with @midwayjs/validation (v4)
- `security-cors-headers` - Configure CORS and security headers
- `security-passport` - Use Passport for OAuth and strategy-based authentication
- `security-captcha` - Implement captcha for anti-bot protection
- `security-casbin` - Use Casbin for fine-grained RBAC/ABAC authorization

### 5. Performance (HIGH)

- `perf-lifecycle-async` - Use async lifecycle hooks correctly
- `perf-caching` - Use @midwayjs/cache-manager for strategic caching
- `perf-async-context` - Enable the async context manager for request context
- `perf-thread-pool` - Offload CPU work to thread pools (@midwayjs/piscina)
- `perf-retry` - Retry fallible operations with retryWithAsync
- `perf-data-listener` - Subscribe to changing data with DataListener

### 6. Testing (MEDIUM-HIGH)

- `test-mock-framework` - Use @midwayjs/mock for app bootstrap testing
- `test-http-supertest` - Use Supertest for HTTP E2E testing
- `test-faas-testing` - Test FaaS functions with createFunctionApp (Serverless)

### 7. Database & ORM (MEDIUM-HIGH)

- `db-typeorm-v4` - Follow TypeORM v4 datasource patterns
- `db-multi-datasource` - Manage multiple datasources and transactions
- `db-redis` - Use the Redis service factory correctly
- `db-sequelize` - Follow Sequelize v4 patterns (@Table replaces @BaseTable)
- `db-mongodb` - Use MongoDB with Typegoose/Mongoose datasource form
- `db-mikro` - Use MikroORM with the correct v6/v7 patterns
- `db-object-storage` - Access object storage via service factories (COS/OSS/TableStore)
- `db-prisma` - Use Prisma ORM in fullstack projects

### 8. API Design (MEDIUM)

- `api-controllers` - Write controllers with declarative routing and param decorators
- `api-middleware-scoping` - Apply middleware at the right level with match/ignore
- `api-pipes` - Use pipes for input transformation
- `api-aspects` - Use @Aspect for cross-cutting AOP interception
- `api-response-format` - Standardize responses with HttpServerResponse
- `api-http-client` - Use the HTTP client for outbound requests (axios HttpService)
- `api-file-upload` - Handle file uploads with @midwayjs/busboy
- `api-static-files` - Serve static files with @midwayjs/static-file
- `api-swagger` - Generate API docs with @midwayjs/swagger
- `api-view-rendering` - Render views with template engines
- `api-crud` - Generate CRUD routes with @midwayjs/crud
- `api-http-proxy` - Proxy requests with @midwayjs/http-proxy
- `api-i18n` - Internationalize with @midwayjs/i18n
- `api-tenant` - Implement multi-tenancy with @midwayjs/tenant
- `api-graphql` - Build GraphQL APIs with @midwayjs/apollo
- `api-versioning` - Version APIs for backward compatibility
- `api-cookie-session` - Manage cookies and sessions correctly

### 9. Microservices & Messaging (MEDIUM)

- `micro-events` - Use @OnEvent for decoupled event-driven processing
- `micro-task-queues` - Use BullMQ for reliable background job processing
- `micro-cron` - Use @midwayjs/cron for local scheduled tasks
- `micro-provider-consumer` - Use the correct microservice provider/consumer pattern
- `micro-mqtt` - Use MQTT for IoT messaging
- `micro-mcp` - Build MCP servers with @midwayjs/mcp

### 10. DevOps & Deployment (LOW-MEDIUM)

- `devops-multi-env-config` - Use multi-environment configuration with validation
- `devops-logging` - Use structured logging via midwayLogger
- `devops-deployment` - Implement graceful shutdown and deployment correctly
- `devops-metrics` - Expose metrics with @midwayjs/prometheus
- `devops-config-discovery` - Use config centers and service discovery (Consul/etcd)
- `devops-process-management` - Manage processes for production (PM2/cfork)
- `devops-debug-tracing` - Debug with request tracing (@midwayjs/code-dye)
- `devops-cli` - Build CLI tools with @midwayjs/commander
- `devops-faas-deployment` - Deploy FaaS to Aliyun FC and AWS Lambda (Serverless)
- `devops-tracing` - Enable distributed tracing with OpenTelemetry
- `devops-cli-tools` - Use the Midway toolchain (mwtsc, mwts, create-midway)

## How to Use

Read individual rule files for detailed explanations and code examples:

```
rules/di-scope-awareness.md
rules/security-validation.md
rules/_sections.md
```

Each rule file contains:
- Brief explanation of why it matters
- Incorrect code example with explanation
- Correct code example with explanation
- Midway documentation reference

## Full Compiled Document

For the complete guide with all rules expanded: `AGENTS.md`
