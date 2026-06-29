---
title: Organize by Feature Modules with Directory Conventions
impact: CRITICAL
impactDescription: "3-5x faster onboarding and feature development"
tags: architecture, modules, directory, organization
---

## Organize by Feature Modules with Directory Conventions

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
