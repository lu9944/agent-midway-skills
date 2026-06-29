---
title: Validate All Input with @midwayjs/validation (v4)
impact: HIGH
impactDescription: "First line of defense — and v4 requires the component for auto DTO conversion"
tags: security, validation, dto, v4, joi
---

## Validate All Input with @midwayjs/validation (v4)

In v4, automatic request-to-DTO conversion only happens when a validation component is enabled. Use `@midwayjs/validation` (the v4 successor to `@midwayjs/validate`) — it supports pluggable joi/zod/class-validator. Define DTOs with `@Rule(schema)` properties, and apply `@Validate` on handlers. The v4 component removed `RuleType`; use the validator's native API directly (`Joi.*`, `z.*`). Never trust raw `@Body`/`@Query` without a validated DTO.

**Incorrect (v3-style unvalidated input, deprecated RuleType):**

```typescript
// ❌ raw body with no validation — SQL injection / type confusion risk
@Post('/')
async create(@Body() body: any) {
  return this.userService.create(body);
}

// ❌ v3 deprecated package + RuleType (removed in v4 validation component)
import { Rule, RuleType } from '@midwayjs/validate';
export class UserDTO {
  @Rule(RuleType.string().required()) name: string;
}

// ❌ expecting auto-DTO conversion without a validation component (v4 disabled this)
async create(@Body() user: CreateUserDTO) { /* user is a plain object, not instance */ }
```

**Correct (v4 @midwayjs/validation with joi + validated DTO):**

```typescript
// configuration.ts
import * as validation from '@midwayjs/validation';
@Configuration({ imports: [koa, validation] })

// config.default.ts — register the joi validator
import joi from '@midwayjs/validation-joi';   // registers the validator adapter (NOT the Joi API itself)
export default {
  validation: { validators: { joi }, defaultValidator: 'joi' },
} as MidwayConfig;

// src/dto/user.dto.ts — @Rule uses Joi's native schema directly (import Joi from 'joi')
import { Rule } from '@midwayjs/validation';
import { getSchema } from '@midwayjs/validation';
import Joi from 'joi';   // import Joi from the 'joi' package, NOT from '@midwayjs/validation-joi'

export class CreateUserDTO {
  @Rule(Joi.string().min(2).max(100).required())
  name: string;

  @Rule(Joi.string().email().required())
  email: string;

  @Rule(Joi.number().integer().min(0).max(150))
  age: number;
}

// src/controller/user.controller.ts
import { Validate } from '@midwayjs/validation';

@Controller('/user')
export class UserController {
  @Post('/')
  @Validate()                          // triggers validation + DTO conversion
  async create(@Body() dto: CreateUserDTO) {
    // dto is a validated CreateUserDTO instance
    return this.userService.create(dto);
  }
}
```

For cascading/nested DTOs, use the arrow-function form because the validator isn't registered at class-eval time: `@Rule(() => getSchema(AddressDTO).required())`. Validation failures throw `MidwayValidationError` — catch it with a `@Catch(MidwayValidationError)` filter.

Reference: [Midway Validation (v4)](https://midwayjs.org/docs/extensions/validation)
