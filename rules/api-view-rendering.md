---
title: Render Views with Template Engines
impact: MEDIUM
impactDescription: "Server-side rendering with EJS/Nunjucks"
tags: api, render, view, ejs, nunjucks, ssr
---

## Render Views with Template Engines

Use `@midwayjs/view-ejs` or `@midwayjs/view-nunjucks` for server-side rendering. Configure the `view` key with `mapping` (extension → engine), `defaultViewEngine`, and `rootDir` (multi-directory, mergeable). Render via `ctx.render(name, locals)`. Register custom Nunjucks filters in `onReady`. In egg (`@midwayjs/web`), disable the built-in view plugin to avoid conflicts.

**Incorrect (string concatenation, no template engine):**

```typescript
@Get('/page')
async page(@Query('name') name: string) {
  // ❌ manual HTML string building — error-prone, no escaping, unmaintainable
  return `<html><body><h1>Hello ${name}</h1></body></html>`;
}
```

**Correct (EJS engine + config + render):**

```typescript
import * as view from '@midwayjs/view-ejs';
@Configuration({ imports: [koa, view] })

// config.default.ts
export default {
  view: {
    defaultExtension: '.ejs',
    defaultViewEngine: 'ejs',
    mapping: { '.ejs': 'ejs' },
    rootDir: { default: join(appInfo.appDir, 'view') },
  },
} as MidwayConfig;

// view/hello.ejs:  <h1>Hello <%= data %></h1>
@Controller('/')
export class HomeController {
  @Inject() ctx: Context;

  @Get('/')
  async index() {
    await this.ctx.render('hello', { data: 'world' });
    // or with defaultExtension: this.ctx.render('hello', { data: 'world' })
  }
}
```

Nunjucks custom filters and custom engine (for Nunjucks, import from `@midwayjs/view-nunjucks`):
```typescript
// register filter in onReady (Nunjucks only — import from @midwayjs/view-nunjucks, NOT view-ejs)
import * as nunjucks from '@midwayjs/view-nunjucks';
@Inject() env: nunjucks.NunjucksEnvironment;
async onReady() {
  this.env.addFilter('upper', (s: string) => s.toUpperCase());
}

// custom engine implements IViewEngine, registered via viewManager.use('name', Engine)
```

Reference: [Midway View (EJS)](https://midwayjs.org/docs/extensions/render)
