---
title: Use Prisma ORM in Fullstack Projects
impact: MEDIUM
impactDescription: "Schema-first ORM with generated typed client"
tags: database, prisma, orm, hooks, fullstack, type-safe
---

## Use Prisma ORM in Fullstack Projects

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
