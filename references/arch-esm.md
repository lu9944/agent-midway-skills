---
title: Enable ESM Support Correctly
impact: MEDIUM
impactDescription: "v4 first-class ESM requires specific config changes"
tags: architecture, esm, modules, v4, toolchain
---

## Enable ESM Support Correctly

v4 provides a first-class ESM scaffold (`koa-v4-esm`). ESM requires: `"type": "module"` in `package.json`, `moduleResolution: Node16/NodeNext` in `tsconfig.json`, imports **must include `.js` extension** (even for `.ts` source), no `require`/`__dirname`/`module.exports`, and the `ESModuleFileDetector` in `@Configuration`. Config loading must use the object form (`importConfigs: [{ default, local }]`). Not supported: alias paths (use Node subpath exports), build-time non-JS copying.

**Incorrect (CJS patterns in an ESM project):**

```typescript
// ❌ missing .js extension (breaks in ESM)
import { UserService } from './service/user';

// ❌ CJS-only APIs
const dir = __dirname;                    // undefined in ESM
module.exports = { foo };                 // syntax error

// ❌ directory-based config loading (unsupported in ESM)
@Configuration({ importConfigs: [join(__dirname, './config')] })
```

**Correct (ESM conventions + ESModuleFileDetector + object config form):**

```json
// package.json
{ "type": "module" }
// tsconfig.json
{ "compilerOptions": { "module": "ESNext", "moduleResolution": "Node16", "target": "ESNext" } }
```

```typescript
// src/configuration.ts
import { Configuration, ESModuleFileDetector } from '@midwayjs/core';
import DefaultConfig from './config/config.default.js';   // ✓ .js extension
import LocalConfig from './config/config.local.js';

@Configuration({
  imports: [/* ... */],
  detector: new ESModuleFileDetector(),                   // ✓ ESM detector
  importConfigs: [{ default: DefaultConfig, local: LocalConfig }],  // ✓ object form
})
export class MainConfiguration {}

// __dirname replacement in ESM
import { dirname } from 'node:path';
import { fileURLToPath } from 'node:url';
const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);
```

Scaffold: `npm init midway@latest -- --type=koa-v4-esm`. Dev/build use `mwtsc`; test with `mocha + ts-node`.

Reference: [Midway ESM](https://midwayjs.org/docs/esm)
