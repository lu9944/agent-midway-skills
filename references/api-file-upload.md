---
title: Handle File Uploads with @midwayjs/busboy
impact: HIGH
impactDescription: "Secure multipart parsing with whitelist and size limits"
tags: api, upload, multipart, file, busboy
---

## Handle File Uploads with @midwayjs/busboy

Prefer `@midwayjs/busboy` (since v3.17.0, replaces `@midwayjs/upload`) for file uploads. Unlike the legacy upload component, its `UploadMiddleware` is **not** auto-loaded — apply it per-route (`{ middleware: [UploadMiddleware] }`) so only upload endpoints parse multipart bodies. Always set `whitelist`, `mimeTypeWhiteList`, and `limits.fileSize` to prevent temp-dir abuse. Use `@Files()`/`@Fields()` to receive files and form fields.

**Incorrect (auto-parsing every request, no whitelist, no size limit):**

```typescript
// ❌ legacy upload auto-parses ALL requests — fills temp dir on any multipart POST
import * as upload from '@midwayjs/upload';
@Configuration({ imports: [upload] })

@Controller('/')
export class HomeController {
  @Inject() ctx: Context;
  @Post('/upload')
  async upload() {
    const file = this.ctx.files[0];
    // ❌ no whitelist (WebShell risk), no size limit, no cleanup
  }
}
```

**Correct (busboy route-scoped middleware + whitelist + limits + cleanup):**

```typescript
import * as busboy from '@midwayjs/busboy';
import { UploadMiddleware, uploadWhiteList } from '@midwayjs/busboy';
import { Configuration } from '@midwayjs/core';
@Configuration({ imports: [busboy] })

// config.default.ts
export default {
  busboy: {
    mode: 'file',                          // 'file' | 'stream' | 'asyncIterator'
    whitelist: uploadWhiteList,            // allowed extensions; null = WebShell risk!
    mimeTypeWhiteList: undefined,          // add for MIME checking (file mode only)
    tmpdir: join(tmpdir(), 'midway-busboy-files'),
    cleanTimeout: 5 * 60 * 1000,
    limits: { fileSize: 10 * 1024 * 1024 }, // 10MB hard limit
  },
} as MidwayConfig;

// route-scoped — only /upload parses multipart
import { Files, Fields } from '@midwayjs/core';
@Controller('/upload')
export class UploadController {
  @Inject() ctx: Context;

  @Post('/', { middleware: [UploadMiddleware] })
  async upload(@Files() files, @Fields() fields) {
    // file mode: each file = { filename, data: <tempPath>, fieldname, mimeType }
    const saved = files.map(f => ({ name: f.filename, path: f.data }));
    await this.ctx.cleanupRequestFiles();   // ✓ clean temp files when done
    return saved;
  }

  // asyncIterator mode for streaming large files without buffering
  @Post('/stream', { middleware: [UploadMiddleware] })
  async stream(@Files() fileIter: AsyncGenerator<UploadStreamFileInfo>) {
    for await (const file of fileIter) {
      file.data.pipe(createWriteStream(join(tmpdir(), file.filename)));
    }
    return { ok: true };
  }
}
```

Reference: [Midway Busboy](https://midwayjs.org/docs/extensions/busboy), [Midway Upload](https://midwayjs.org/docs/extensions/upload)
