---
title: Implement Captcha for Anti-Bot Protection
impact: MEDIUM
impactDescription: "Prevents brute-force and automated abuse"
tags: security, captcha, anti-bot, cache
---

## Implement Captcha for Anti-Bot Protection

Use `@midwayjs/captcha` for image, math-formula, and text captchas. It stores verification state via `@midwayjs/cache-manager` (default memory store; swap to Redis for multi-instance). The component does **not** send SMS/email — `text()` returns a code you deliver yourself. Always verify via `check(id, answer)` before processing sensitive actions.

**Incorrect (no captcha on login/forgot-password, raw verification):**

```typescript
@Post('/login')
async login(@Body() dto: LoginDTO) {
  // ❌ no captcha → brute-force attack surface
  return this.authService.login(dto);
}
```

**Correct (image captcha + verification + Redis store):**

```typescript
import * as captcha from '@midwayjs/captcha';
import * as cacheManager from '@midwayjs/cache-manager';
@Configuration({ imports: [captcha, cacheManager] })

// config.default.ts — use Redis for multi-instance consistency
export default {
  captcha: {
    default: { size: 4, noise: 1, width: 120, height: 40 },
    image: { type: 'mixed' },     // 'mixed' | 'letter' | 'number'
    expirationTime: 300,          // 5 min (seconds)
  },
  cacheManager: { clients: { captcha: { store: createRedisStore('default') } } },
} as MidwayConfig;

// controller
import { CaptchaService } from '@midwayjs/captcha';
@Controller('/auth')
export class AuthController {
  @Inject() captchaService: CaptchaService;

  @Get('/captcha')
  async getCaptcha() {
    const { id, imageBase64 } = await this.captchaService.image();
    return { id, imageBase64 };
  }

  @Post('/login')
  async login(@Body('captchaId') captchaId: string, @Body('captchaCode') code: string) {
    const passed = await this.captchaService.check(captchaId, code);
    if (!passed) throw new httpError.BadRequestError('invalid captcha');
    return this.authService.login(/* ... */);
  }

  // SMS/email code: component returns the code, you send it
  @Get('/sms-code')
  async sendSms(@Query('phone') phone: string) {
    const { id, text } = await this.captchaService.text({ type: 'number' });
    await this.smsService.send(phone, text);   // you deliver the code
    return { id };
  }
}
```

Reference: [Midway Captcha](https://midwayjs.org/docs/extensions/captcha)
