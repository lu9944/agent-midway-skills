---
title: Use MikroORM with the Correct v6/v7 Patterns
impact: MEDIUM
impactDescription: "v6 removed Repository.persist; v7 is a separate package"
tags: database, mikro, mikro-orm, v4, repository
---

## Use MikroORM with the Correct v6/v7 Patterns

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
