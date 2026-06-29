---
title: Follow TypeORM v4 Datasource Patterns
impact: HIGH
impactDescription: "v4 changed the package, config key, and removed EntityModel"
tags: database, typeorm, v4, entity, repository
---

## Follow TypeORM v4 Datasource Patterns

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
