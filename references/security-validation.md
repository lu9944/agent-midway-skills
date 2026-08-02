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

### Pluggable validator adapters (v4)

The component registers **one validator adapter at a time** via `validation.validators` — v4 ships `@midwayjs/validation-joi`, `@midwayjs/validation-zod`, `@midwayjs/validation-zod4`, and `@midwayjs/validation-class-validator`. The adapter defines which schema API `@Rule(...)` accepts and owns error formatting (e.g. the zod4 adapter formats messages through `zod-validation-error` + `@midwayjs/i18n`, so locale-aware error text follows the i18n config). Pick one per app — `defaultValidator` is inferred from the registered adapter; do not register two for the same `@Rule` syntax.

```typescript
// config.default.ts — joi adapter (v3-like syntax)
import joi from '@midwayjs/validation-joi';
export default {
  validation: {
    validators: { joi },
    // defaultValidator: 'joi',   // optional; first registered becomes default
  },
} as MidwayConfig;

// config.default.ts — zod4 adapter (native z.* schemas, i18n error messages)
import zod4 from '@midwayjs/validation-zod4';
export default {
  validation: { validators: { zod4 } },
  i18n: { defaultLocale: 'zh-CN' },   // error messages follow locale via zod-i18n-map
} as MidwayConfig;
```

Matching `@Rule` usage: joi → `@Rule(Joi.string().required())`; zod4 → `@Rule(z.string().min(1))` with `import { z } from 'zod'`; class-validator → `@Rule(IsString() as any)`. Functional-API validation (`input({ body: z.object(...) })`) is zod-native and independent of this adapter choice.

Reference: [Midway Validation (v4)](https://midwayjs.org/docs/extensions/validation)
