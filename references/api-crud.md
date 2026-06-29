---
title: Generate CRUD Routes with @midwayjs/crud
impact: MEDIUM-HIGH
impactDescription: "Declarative REST CRUD with query/filter/sort out of the box"
tags: api, crud, rest, typeorm, sequelize, mikro, mongoose
---

## Generate CRUD Routes with @midwayjs/crud

`@midwayjs/crud` generates standard REST endpoints (list/detail/create/update/delete) from a `@Crud()` declaration, with built-in pagination, filtering, sorting, search, and soft-delete. Pick a DB adapter via subpath imports (`@midwayjs/crud/typeorm`, `/mikro`, `/sequelize`, `/mongoose`). The `@Crud()` decorator only **declares** routes; the injected `CrudService` does the work. Put complex business logic in a separate service, not in CRUD overrides.

**Incorrect (hand-writing identical CRUD controllers for every entity):**

```typescript
@Controller('/users')
export class UserController {
  @Inject() userRepo: Repository<User>;
  @Get('/') async list(@Query() q) { /* pagination logic, repeated... */ }
  @Get('/:id') async detail(@Param('id') id) { /* ... */ }
  @Post('/') async create(@Body() b) { /* ... */ }
  @Patch('/:id') async update() { /* ... */ }
  @Del('/:id') async delete() { /* ... */ }
}
// ❌ same boilerplate for orders, products, etc.
```

**Correct (declarative @Crud with TypeORM adapter + query protocol):**

```typescript
import * as crud from '@midwayjs/crud';
import { TypeOrmCrudService } from '@midwayjs/crud/typeorm';
@Configuration({ imports: [koa, typeorm, crud, validation] })

// service extends the adapter base
@Provide()
export class UserCrudService extends TypeOrmCrudService<UserEntity> {
  @InjectEntityModel(UserEntity) repo: Repository<UserEntity>;
}

// controller — declarative routes + query options
import { Crud } from '@midwayjs/crud';
@Crud({
  model: UserEntity,
  service: UserCrudService,
  dto: { create: CreateUserDTO, update: UpdateUserDTO },
  query: {
    defaultLimit: 20, maxLimit: 100,
    sortable: ['id', 'createdAt'],        // whitelist sort fields
    filterable: ['status'],               // whitelist filter fields
    searchable: ['name', 'email'],        // full-text search fields
    join: ['profile'],                     // allowed relations
  },
  routes: { only: ['list', 'detail', 'create'] },  // or exclude
  delete: { mode: 'soft' },               // opt-in soft delete
})
@Controller('/users')
export class UserController {
  @Inject() crudService: UserCrudService;   // still required — does the work
  // override a default route by redefining the same path, or add custom routes
}

// list endpoint query protocol: ?page=1&limit=20&sort=id:DESC&filter=status||eq||active&search=john
// returns: { data, meta: { page, limit, total, pageCount, hasNext, hasPrev } }
```

DTO validation requires `@midwayjs/validation` installed. CRUD routes auto-emit Swagger metadata. Not suitable for transaction-heavy/workflow modules — use a dedicated service there.

Reference: [Midway CRUD](https://midwayjs.org/docs/extensions/crud)
