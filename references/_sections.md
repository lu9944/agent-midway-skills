# Sections

This file defines all sections, their ordering, impact levels, and descriptions.
The section ID (in parentheses) is the filename prefix used to group rules.

---

## 1. Architecture (arch)

**Impact:** CRITICAL
**Description:** The `@Configuration` entry point, component composition, directory conventions, and v4's explicit file detector are the foundation of maintainable Midway applications. Correct module organization and component reuse prevent the most common structural failures.

## 2. Dependency Injection (di)

**Impact:** CRITICAL
**Description:** Midway's IoC container is the heart of the framework. Understanding scopes (Singleton/Request/Prototype), the `@Provide`/`@Inject` pairing, `@Init` lifecycle, and circular-dependency resolution is essential for testable and correct code. Most runtime crashes trace back to DI misuse.

## 3. Error Handling (error)

**Impact:** HIGH
**Description:** Midway provides a structured error system built on `MidwayError`/`MidwayHttpError`, registered error codes, and `@Catch` filters. Centralized exception filters ensure uniform error responses and reliable HTTP status mapping.

## 4. Security (security)

**Impact:** HIGH
**Description:** Guards for route-level authorization, JWT authentication, the v4 `@midwayjs/validation` component, and captcha protect your application. Input validation and access control are non-negotiable for production readiness.

## 5. Performance (perf)

**Impact:** HIGH
**Description:** Async lifecycle hooks, the v4 cache-manager component, and the async context manager directly impact application responsiveness, startup time, and scalability. Correct scope usage also prevents memory leaks and data races.

## 6. Testing (test)

**Impact:** MEDIUM-HIGH
**Description:** The `@midwayjs/mock` toolkit (`createApp`, `createLightApp`, `createHttpRequest`) enables comprehensive unit and e2e coverage with proper dependency isolation and context mocking.

## 7. Database & ORM (db)

**Impact:** MEDIUM-HIGH
**Description:** TypeORM (v4 datasource form), Sequelize (`@Table` instead of `BaseTable`), multi-datasource management, transactions via `DataSourceManager`, and the Redis service factory are crucial for data-intensive applications.

## 8. API Design (api)

**Impact:** MEDIUM
**Description:** Controllers, routing, the singleton-scoped middleware model, pipes, AOP aspects, DTO validation, and the standardized `HttpServerResponse` improve API usability, consistency, and maintainability.

## 9. Microservices & Messaging (micro)

**Impact:** MEDIUM
**Description:** Building distributed systems requires the event emitter, BullMQ task queues, cron jobs, gRPC/RabbitMQ/Kafka providers and consumers, and WebSocket controllers. Each has its own framework, context, and decorator conventions.

## 10. DevOps & Deployment (devops)

**Impact:** LOW-MEDIUM
**Description:** Multi-environment configuration, structured logging via `midwayLogger`, graceful shutdown via lifecycle hooks, and containerized deployment ensure production readiness and zero-downtime operations.
