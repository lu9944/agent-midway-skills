---
title: Use Pipes for Input Transformation
impact: MEDIUM
impactDescription: "Clean, validated data reaches handlers"
tags: api, pipe, transformation, validation
---

## Use Pipes for Input Transformation

Pipes run **after** param decorators and handle validation/transformation. Use built-in pipes (`ParseIntPipe`, `ParseFloatPipe`, `ParseBoolPipe`, `DefaultValuePipe` from `@midwayjs/validation`) on route params for automatic type conversion with errors. Create custom pipes (`@Pipe` + `PipeTransform`) for business-specific transforms. For whole-body validation, use a validated DTO instead of per-field pipes.

**Incorrect (manual parsing and validation in every handler):**

```typescript
@Get('/:id')
async findOne(@Param('id') id: string) {
  const num = Number(id);          // ❌ NaN if invalid, no error
  if (isNaN(num)) throw new Error('bad id');
  return this.service.findById(num);
}

@Get('/')
async list(@Query('page') page: string) {
  const pageNum = parseInt(page) || 1;   // ❌ repeated manual parsing
  return this.service.list(pageNum);
}
```

**Correct (built-in pipes + custom pipe):**

```typescript
import { DefaultValuePipe } from '@midwayjs/validation';
import { ParseIntPipe } from '@midwayjs/validation';

@Controller('/user')
export class UserController {
  // built-in pipes: default + int conversion with proper errors
  @Get('/')
  async list(
    @Query('page', new DefaultValuePipe(1), ParseIntPipe) page: number,
    @Query('limit', new DefaultValuePipe(20), ParseIntPipe) limit: number,
  ) {
    return this.service.list(page, limit);
  }

  @Get('/:id')
  async findOne(@Param('id', ParseIntPipe) id: number) {
    return this.service.findById(id);
  }
}

// src/pipe/parse-date.pipe.ts — custom pipe
import { Pipe, PipeTransform, TransformOptions } from '@midwayjs/core';
import { httpError } from '@midwayjs/core';

@Pipe()
export class ParseDatePipe implements PipeTransform<string, Date> {
  transform(value: string, options: TransformOptions): Date {
    const date = new Date(value);
    if (isNaN(date.getTime())) throw new httpError.BadRequestError('invalid date');
    return date;
  }
}

// usage
@Get('/report')
async report(@Query('from', ParseDatePipe) from: Date) {
  return this.service.report(from);
}
```

Reference: [Midway Pipes](https://midwayjs.org/docs/pipe)
