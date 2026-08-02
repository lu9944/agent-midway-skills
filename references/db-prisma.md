---
title: Use Prisma ORM (Schema-First, with Functional API)
impact: MEDIUM
impactDescription: "Schema-first ORM with generated typed client"
tags: database, prisma, orm, functional, fullstack, type-safe
---

## Use Prisma ORM (Schema-First, with Functional API)

Prisma fits Midway **Functional API** (`defineApi`) fullstack projects — it provides a schema-first workflow (`schema.prisma`), a generated fully-typed client (`@prisma/client`), and end-to-end type safety when combined with typed API contracts. Initialize the client as a module-level singleton, then use it directly inside `defineApi` handlers (or via `useInject` for IoC-managed usage) — no Midway component wrapper exists (there is no `@midwayjs/prisma` package). Set the engine mirror for poor-network regions.

> **Note:** earlier docs tied Prisma to Midway Hooks (`@midwayjs/hooks`); that stack is deprecated (see `arch-hooks-fullstack.md`). Use the Functional API instead.

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
// src/api/prisma.ts — singleton client (module-level)
import { PrismaClient } from '@prisma/client';
export const prisma = new PrismaClient();

// src/server/api/user.api.ts — use inside a Functional API handler with zod
import { defineApi } from '@midwayjs/core/functional';
import { z } from 'zod';
import { prisma } from './prisma';

export const userApi = defineApi('/users', (api) => ({
  list: api
    .get('/')
    .output(z.array(z.object({ id: z.number(), name: z.string().nullable() })))
    .handle(async () => {
      return prisma.user.findMany({});   // ✓ fully typed result
    }),

  signUp: api
    .post('/')
    .input({ body: z.object({ name: z.string(), email: z.string().email() }) })
    .handle(async ({ input }) => {
      return prisma.user.create({ data: input.body });
    }),
}));

// frontend: typed call via createClient
// import { api } from '../client';
// const user = await api.user.signUp({ body: { name: 'John', email: 't@t.com' } });
```

For poor network, set the engine mirror before install:
```bash
PRISMA_ENGINES_MIRROR=https://registry.npmmirror.com/-/binary/prisma/
```

Prisma works best in the Functional-API/typed fullstack context. For class-based standard projects, prefer TypeORM/Sequelize/MikroORM/Leoric (which have dedicated Midway component wrappers).

Reference: [Midway Hooks + Prisma](https://midwayjs.org/docs/hooks/prisma), [Prisma Docs](https://www.prisma.io/)
