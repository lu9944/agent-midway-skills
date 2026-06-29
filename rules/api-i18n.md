---
title: Internationalize with @midwayjs/i18n
impact: MEDIUM
impactDescription: "Request-aware multi-language text resolution"
tags: api, i18n, internationalization, localization, locale
---

## Internationalize with @midwayjs/i18n

`@midwayjs/i18n` provides request-aware translation via `MidwayI18nService.translate()`. Store translations in JSON files under `src/locales/`, register them in `i18n.localeTable` (locale → group → keys). Locale is auto-resolved from query → cookie → `Accept-Language` header → `defaultLocale`, with `writeCookie` caching the choice. Use groups to isolate component/module translations. Dynamically add translations from a DB via `addLocale()` in `onReady`.

**Incorrect (hardcoded strings, manual locale detection):**

```typescript
@Get('/')
async index(@Query('lang') lang: string) {
  // ❌ hardcoded strings, no fallback, no parameter interpolation
  if (lang === 'zh') return '你好';
  return 'Hello';
}
```

**Correct (locale files + translate with args + group isolation):**

```typescript
import * as i18n from '@midwayjs/i18n';
@Configuration({ imports: [i18n] })

// src/locales/en_US.json: { "hello": "Hello {username}" }
// src/locales/zh_CN.json: { "hello": "你好 {username}" }

// config.default.ts
export default {
  i18n: {
    defaultLocale: 'en_US',
    localeTable: {
      en_US: { default: require('../locales/en_US'), user: require('../locales/user_en_US') },
      zh_CN: { default: require('../locales/zh_CN'), user: require('../locales/user_zh_CN') },
    },
    fallbacks: { 'en_*': 'en_US' },   // wildcard fallback mapping
    writeCookie: true,                 // cache locale choice in cookie
    resolver: { queryField: 'locale', cookieField: { fieldName: 'locale' } },
    missingKeyHandler: (msg, opts) => `[missing:${msg}]`,   // log/guard missing keys
  },
} as MidwayConfig;

// service — translate with parameter interpolation
import { MidwayI18nService } from '@midwayjs/i18n';
@Controller('/')
export class HomeController {
  @Inject() i18nService: MidwayI18nService;

  @Get('/')
  async index(@Query('username') username: string) {
    // auto-resolves locale from query/cookie/header; explicit override via { locale }
    return this.i18nService.translate('hello', { args: { username } });
    // non-default group: this.i18nService.translate('user.hello', { args: [username], group: 'user' })
  }
}

// dynamic translations from DB in onReady
async onReady() {
  this.i18nService.addLocale('zh_TW', { hello: '你好，{username}' });
}
```

Locale priority (high→low): explicit `locale` in `translate()` → `saveRequestLocale()` → query → cookie → `Accept-Language` → `defaultLocale`. Internally all locales are normalized to lowercase-hyphen (`en_US` → `en-us`).

Reference: [Midway i18n](https://midwayjs.org/docs/extensions/i18n)
