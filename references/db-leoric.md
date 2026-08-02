---
title: Use @midwayjs/leoric for a Lightweight MySQL/PostgreSQL ORM
impact: MEDIUM
impactDescription: "Alternative to TypeORM — expressive Model API, datasource-style config"
tags: database, orm, leoric, mysql, postgresql
---

## Use @midwayjs/leoric for a Lightweight MySQL/PostgreSQL ORM

`@midwayjs/leoric` (v4.2.1+) wraps the [Leoric](https://github.com/cyjake/leoric) ORM — an active-record style, zero-config alternative to TypeORM with an expressive chainable Model API (`order`, `findOne`, `create`, `update`, pagination, hooks). Configuration mirrors the v4 datasource form used by TypeORM/MikroORM: `leoric.dataSource.<name>` with `models` globs (files ending in `*.ts`/`*.js` under `src/model/`). Inject models with `@InjectModel(Model)`. Use it when you prefer a lightweight Model-first API over decorator-based entities; both can coexist with `@midwayjs/typeorm` in the same app (different config namespaces).

**Incorrect (hand-rolled SQL, missing model registration):**

```typescript
// ❌ raw connection + string SQL — no type safety, no model layer, no migrations
@Provide()
export class UserService {
  async findByName(name: string) {
    const conn = await mysql.createConnection({ /* ... */ });
    return (await conn.query(`SELECT * FROM user WHERE name = '${name}'`))[0];  // ❌ injection risk
  }
}

// ❌ model file not under the configured models glob — never loaded
// src/model/user.ts  vs  config models: ['**/models/*{.ts,.js}']
```

**Correct (datasource config + Model + @InjectModel):**

```typescript
// configuration.ts
import { Configuration } from '@midwayjs/core';
import * as leoric from '@midwayjs/leoric';

@Configuration({ imports: [leoric] })

// config.default.ts — same shape as the v4 typeorm/mikro datasource form
import { join } from 'path';
export default {
  leoric: {
    dataSource: {
      default: {
        dialect: 'mysql',
        host: '127.0.0.1',
        port: 3306,
        database: 'admin_db',
        username: 'root',
        password: '123456',
        models: [join(__dirname, '../model/*{.ts,.js}')],   // model globs
      },
    },
  },
} as MidwayConfig;

// src/model/user.ts — model definitions are plain classes (active-record style)
import { Model } from '@midwayjs/leoric';

export class User extends Model {
  static tableName = 'user';            // optional, defaults to class name
  id!: number;
  name!: string;
  email!: string;
  status!: number;
}

// src/modules/user/service/user.service.ts — inject and query
import { Provide } from '@midwayjs/core';
import { InjectModel } from '@midwayjs/leoric';
import { User } from '../../../model/user';

@Provide()
export class UserService {
  @InjectModel(User)
  User: typeof User;

  async list({ page, size, name }) {
    const where = name ? { name } : undefined;
    return {
      list: await this.User.order('id', 'desc').limit(size).offset((page - 1) * size).findAll(where),
      count: await this.User.count(where),
    };
  }

  async create(dto) {
    return await this.User.create(dto);
  }

  async update(id: number, dto) {
    await this.User.where({ id }).update(dto);
    return this.User.findByPk(id);
  }
}
```

Supported model instance methods and hooks (Leoric): `findByPk`, `findOne`, `findAll`, `where`/`order`/`limit`/`offset` chains, `create`, `update`, `remove`, `save`, `beforeCreate`/`afterCreate`/`beforeUpdate`/`afterUpdate` hooks. For transactional workflows, prefer `@midwayjs/typeorm`'s `DataSourceManager`; for plain CRUD with minimal ceremony, Leoric is the lighter fit.

Reference: [Midway Leoric](https://www.midwayjs.org/docs/extensions/leoric) (package: `@midwayjs/leoric`)
