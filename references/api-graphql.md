---
title: Build GraphQL APIs with @midwayjs/apollo
impact: MEDIUM
impactDescription: "Apollo Server with Midway DI and typed resolvers"
tags: api, graphql, apollo, resolver, api-design
---

## Build GraphQL APIs with @midwayjs/apollo

`@midwayjs/apollo` bundles Apollo Server with Midway DI. Define the schema via `typeDefs` (SDL) or `typePaths` (glob), and resolvers as `@Resolver()` classes using decorators re-exported from the component (`@Query`, `@Args`, `@Context`, `@Parent`, `@Info`). Use `contextFactory` to inject request-scoped data. The resolver's `@Context()` is the Midway-enhanced context (has `requestContext`, `logger`) — no need to unwrap.

**Incorrect (importing GraphQL decorators from wrong packages):**

```typescript
// ❌ decorators from type-graphql or @nestjs/graphql — not the Midway component
import { Resolver, Query } from 'type-graphql';
// ❌ manual context unwrapping
async user(@Ctx() ctx) { const midwayCtx = ctx.ctx; /* ... */ }
```

**Correct (Midway apollo resolvers + SDL + contextFactory):**

```typescript
import * as apollo from '@midwayjs/apollo';
@Configuration({ imports: [koa, apollo] })

// config.default.ts
export default {
  apollo: {
    path: '/graphql',
    typePaths: ['./schema/**/*.graphql'],   // OR typeDefs: `type Query { user(id: ID!): String }`
    graphiql: process.env.NODE_ENV === 'local',
    methods: ['GET', 'POST'],
    apollo: { introspection: true },         // Apollo Server options
    contextFactory: async (ctx) => ({ currentUserId: ctx.headers['x-user-id'] }),
  },
} as MidwayConfig;

// resolver — decorators from @midwayjs/apollo
import { Resolver, Query, Args, Context } from '@midwayjs/apollo';
@Resolver()
export class UserResolver {
  @Inject() userService: UserService;

  @Query('userName')
  async userName(@Args('id') id: string, @Context() context) {
    // context is Midway-enhanced: has requestContext, logger, getApp()
    context.logger.info('query user %s', id);
    return (await this.userService.findById(id))?.name;
  }
}
```

Subscriptions use `graphql-ws` (`subscriptions: true | { path }`) returning `AsyncIterable`. Get request-scoped beans in resolvers via `context.requestContext.getAsync(Service)`.

Reference: [Midway Apollo (GraphQL)](https://midwayjs.org/docs/extensions/apollo)
