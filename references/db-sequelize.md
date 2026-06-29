---
title: Follow Sequelize v4 Patterns (@Table replaces @BaseTable)
impact: HIGH
impactDescription: "v4 removed @BaseTable and changed config form"
tags: database, sequelize, v4, model, repository
---

## Follow Sequelize v4 Patterns (@Table replaces @BaseTable)

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
