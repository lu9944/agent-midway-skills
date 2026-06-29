---
title: Access Object Storage via Service Factories (COS/OSS/TableStore)
impact: MEDIUM
impactDescription: "Correct DI-managed multi-instance cloud storage"
tags: database, cos, oss, tablestore, object-storage, cloud
---

## Access Object Storage via Service Factories (COS/OSS/TableStore)

Cloud storage components (`@midwayjs/cos`, `@midwayjs/oss`, `@midwayjs/tablestore`) all follow the Service Factory pattern: configure via `<name>.client` (single) or `<name>.clients` (multi), inject the default `XxxService`, and use `XxxServiceFactory.get('name')` for named instances. Never hardcode SDK credentials — put them in config (env-driven). OSS supports cluster and STS modes; TableStore re-exports SDK types directly.

**Incorrect (raw SDK, manual client, hardcoded credentials):**

```typescript
import OSS from 'ali-oss';   // ❌ bypasses DI, no lifecycle, hardcoded
const client = new OSS({ accessKeyId: 'xxx', accessKeySecret: 'yyy' });
await client.put('file', buffer);
```

**Correct (DI-managed service factory + multi-instance config):**

```typescript
// === Aliyun OSS ===
import * as oss from '@midwayjs/oss';
@Configuration({ imports: [oss] })
// config.default.ts
export default {
  oss: {
    default: { timeout: 5000 },
    clients: {
      default: { bucket: 'app-bucket', region: 'oss-cn-hangzhou', accessKeyId: process.env.OSS_KEY, accessKeySecret: process.env.OSS_SECRET },
      backup: { bucket: 'backup-bucket', region: 'oss-cn-hangzhou', accessKeyId: process.env.OSS_KEY, accessKeySecret: process.env.OSS_SECRET },
    },
  },
} as MidwayConfig;

import { OSSService, OSSServiceFactory } from '@midwayjs/oss';
@Provide()
export class FileService {
  @Inject() ossService: OSSService;                          // default
  @Inject() ossFactory: OSSServiceFactory;
  async upload(buf: Buffer, key: string) { return this.ossService.put(key, buf); }
  async backup(buf: Buffer, key: string) { return this.ossFactory.get('backup').put(key, buf); }
  // STS mode: inject OSSSTSService, call assumeRole(roleArn)
}

// === Tencent COS ===
import * as cos from '@midwayjs/cos';
// config: cos.clients: { default: { SecretId, SecretKey } }
import { COSService } from '@midwayjs/cos';
@Provide()
export class CosService {
  @Inject() cosService: COSService;
  async upload() { return this.cosService.sliceUploadFile({ Bucket, Region, Key, FilePath }); }
}

// === Aliyun TableStore ===
import * as tablestore from '@midwayjs/tablestore';
import { TableStoreService, Long } from '@midwayjs/tablestore';
@Provide()
export class TsService {
  @Inject() tsService: TableStoreService;
  async get(id: number) {
    return this.tsService.getRow({ tableName: 't', primaryKey: [{ gid: Long.fromNumber(id) }] });
  }
}
```

Dynamic instance creation: `factory.createInstance(config, 'bucket3')`.

Reference: [Midway OSS](https://midwayjs.org/docs/extensions/oss), [COS](https://midwayjs.org/docs/extensions/cos), [TableStore](https://midwayjs.org/docs/extensions/tablestore)
