---
title: Write Controllers with Declarative Routing and Param Decorators
impact: MEDIUM
impactDescription: "Clean, typed, swagger-friendly route definitions"
tags: api, controller, routing, decorators
---

## Write Controllers with Declarative Routing and Param Decorators

Use `@Controller(prefix?)` to group routes, HTTP-method decorators (`@Get`, `@Post`, `@Del`, ...) to bind paths, and parameter decorators (`@Body`, `@Query`, `@Param`, `@Headers`, `@Session`) to extract typed request data. Compose multiple param decorators on one handler. Use `@SetHeader`, `@HttpCode`, `@ContentType` for response shaping, and `{ summary: '...' }` in route options for Swagger. Delegate business logic to injected services — controllers should stay thin.

**Incorrect (manual ctx parsing, manual response writing, fat controller):**

```typescript
@Controller('/user')
export class UserController {
  @Inject() ctx: Context;
  @Inject() userService: UserService;

  @Get('/:id')
  async findOne() {
    // ❌ manually parsing everything from ctx
    const id = this.ctx.params.id;
    const fields = this.ctx.query.fields;
    const user = await this.userService.findById(Number(id));
    this.ctx.body = user;           // ❌ manual response writing
  }
}
```

**Correct (declarative decorators, thin controller, typed params):**

```typescript
import { Controller, Get, Post, Body, Query, Param, Provide, Inject, HttpCode } from '@midwayjs/core';
import { DefaultValuePipe, ParseIntPipe } from '@midwayjs/validation';
import { UserService } from '../service/user.service';
import { CreateUserDTO } from '../dto/user.dto';

@Controller('/user')
export class UserController {
  @Inject() userService: UserService;

  // compose multiple param decorators; { summary } for swagger docs
  @Get('/')
  async findAll(
    @Query('page', new DefaultValuePipe(1), ParseIntPipe) page: number,
    @Query('limit', new DefaultValuePipe(20), ParseIntPipe) limit: number,
  ) {
    return this.userService.findAll(page, limit);
  }

  @Get('/:id')
  async findOne(@Param('id', ParseIntPipe) id: number) {   // ParseIntPipe converts string→number
    return this.userService.findById(id);
  }

  @Post('/')
  @HttpCode(201)
  async create(@Body() dto: CreateUserDTO) {  // validated DTO
    return this.userService.create(dto);
  }
}

// global route prefix via config:
// config.default.ts → koa: { globalPrefix: '/v1' }
// opt out per-controller: @Controller('/health', { ignoreGlobalPrefix: true })
```

### Dynamic route registration and introspection (MidwayWebRouterService)

For runtime-driven routes (plugins, dynamic APIs) or route introspection in middleware/guards, inject `MidwayWebRouterService` (available at `onReady` and after):

```typescript
import { MidwayWebRouterService } from '@midwayjs/core';

@Configuration({})
export class MainConfiguration {
  @Inject() webRouterService: MidwayWebRouterService;

  async onReady() {
    // introspect: get all routes (sorted by priority)
    const table = this.webRouterService.getFlattenRouterTable();
    const priorityList = this.webRouterService.getRoutePriorityList();

    // find the matched route for a URL (useful in middleware/guards for authz)
    const matched = this.webRouterService.getMatchedRouterInfo('/user/1', 'get');

    // dynamically register a controller WITHOUT @Controller decorator
    this.webRouterService.addController(DynamicController, {
      prefix: '/plugin',
      routerOptions: { middleware: [AuthMiddleware] },
    });

    // dynamically add a raw route handler
    this.webRouterService.addRouter(
      async (ctx) => { ctx.body = 'dynamic'; },
      { url: '/dynamic', requestMethod: 'GET' },
    );
  }
}
```

Reference: [Midway Controller](https://midwayjs.org/docs/controller), [Router Table](https://midwayjs.org/docs/router_table)
