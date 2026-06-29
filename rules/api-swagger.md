---
title: Generate API Docs with @midwayjs/swagger
impact: MEDIUM
impactDescription: "Auto-generated OpenAPI 3 docs from controllers and DTOs"
tags: api, swagger, openapi, documentation, api-docs
---

## Generate API Docs with @midwayjs/swagger

`@midwayjs/swagger` auto-generates OpenAPI 3.0.3 docs by reading controllers (`@Body`/`@Query`/`@Param`), DTO types, and `@ApiProperty` decorators. Gate it to dev with `enabledEnvironment`. Decorate DTOs with `@ApiProperty({ example, description, type })` and routes with `@ApiOperation`/`@ApiTags` for rich docs. Enable `useValidationSchema: true` (default) to auto-derive fields from `@midwayjs/validation` DTOs — `@ApiProperty` wins on conflicts.

**Incorrect (no API docs, manual maintenance):**

```typescript
// ❌ no swagger — API consumers have no machine-readable contract
@Controller('/user')
export class UserController {
  @Post('/')
  async create(@Body() dto: CreateUserDTO) { return this.userService.create(dto); }
}
// CreateUserDTO has no field descriptions
```

**Correct (env-gated swagger + @ApiProperty DTOs + validation integration):**

```typescript
import * as swagger from '@midwayjs/swagger';
@Configuration({
  imports: [
    { component: swagger, enabledEnvironment: ['local'] },   // dev only
  ],
})

// config.default.ts
export default {
  swagger: {
    title: 'My API', version: '1.0.0',
    description: 'API documentation',
    swaggerPath: '/swagger-ui',     // UI at /swagger-ui/index.html
    auth: { authType: 'bearer', type: 'http', scheme: 'bearer' },
    useValidationSchema: true,     // auto-derive from @midwayjs/validation @Rule
  },
} as MidwayConfig;

// DTO with ApiProperty — explicit wins over validation-derived
import { ApiProperty } from '@midwayjs/swagger';
export class CreateUserDTO {
  @ApiProperty({ example: 'John', description: 'user name' })
  name: string;

  @ApiProperty({ type: 'array', items: { type: 'string' } })
  tags: string[];

  @ApiProperty({ enum: ['active', 'inactive'] })
  status: string;

  @ApiProperty({ type: () => Profile })   // lazy for circular refs
  profile: Profile;
}

// controller with operation metadata
import { ApiOperation, ApiTags, ApiBearerAuth } from '@midwayjs/swagger';
@ApiTags('user')
@ApiBearerAuth()
@Controller('/user')
export class UserController {
  @ApiOperation({ summary: 'Create user', description: 'Creates a new user account' })
  @Post('/')
  async create(@Body() dto: CreateUserDTO) { return this.userService.create(dto); }
}
```

File-upload docs need explicit `@ApiBody({ contentType: BodyContentType.Multipart })`.

Reference: [Midway Swagger](https://midwayjs.org/docs/extensions/swagger)
