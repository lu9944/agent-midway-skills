---
title: Use the Midway Toolchain (mwtsc, mwts, create-midway)
impact: MEDIUM
impactDescription: "Correct dev/build/lint/scaffold tooling for v4"
tags: devops, tooling, mwtsc, mwts, cli, scaffold, lint
---

## Use the Midway Toolchain (mwtsc, mwts, create-midway)

v4 standardizes its toolchain: `create-midway` for scaffolding, `mwtsc` for dev/build (a `tsc` wrapper with `--run` for auto-restart), `mwts` for lint/format (ESLint Flat Config + Prettier), and `tsc` for production builds. The legacy `@midwayjs/cli` (`midway-bin`) is deprecated — prefer `mwtsc`/`tsc`/`jest`/`mocha`. Always use `npm init midway@latest` (the `@latest` tag is required to get current templates).

**Incorrect (deprecated CLI, manual tsc watch without restart, missing lint):**

```bash
# ❌ deprecated midway-bin dev (use mwtsc)
midway-bin dev --ts

# ❌ tsc watch without restarting the server on change
tsc --watch   # compiles but never restarts the app

# ❌ no linting configured
```

**Correct (mwtsc dev + tsc build + mwts lint + create-midway scaffold):**

```bash
# scaffold a v4 project (always use @latest)
npm init midway@latest -- --type=koa-v4
# other v4 templates: koa-v4-esm, react-functional-v4, vue-functional-v4
```

```json
// package.json
{
  "scripts": {
    "dev": "cross-env NODE_ENV=local mwtsc --watch --run @midwayjs/mock/app --port 7001",
    "build": "cross-env rm -rf dist && tsc",
    "lint": "mwts check",
    "lint:fix": "mwts fix",
    "test": "jest --runInBand"
  }
}
```

```typescript
// mwtsc --run: recompiles + restarts on code change (the @midwayjs/mock/app starts the framework)
// Serverless dev: mwtsc --watch --run @midwayjs/mock/function

// mwts — ESLint Flat Config (eslint.config.js)
const mwtsConfig = require('mwts/eslint.config.js');
module.exports = [
  { ignores: ['**/node_modules', '**/dist'] },
  ...mwtsConfig,
];
// migrate from mwts 1.x: npx mwts migrate
```

Key `mwtsc` flags: `--watch` (watch mode), `--run <file>` (restart on change — must be **last**), `--port` (override HTTP port), `--ssl` (HTTPS test cert). Key `mwts` commands: `mwts check`, `mwts fix`, `mwts init`, `mwts migrate`.

Reference: [Midway mwtsc](https://midwayjs.org/docs/tool/mwtsc), [mwts](https://midwayjs.org/docs/tool/mwts), [create-midway](https://midwayjs.org/docs/tool/create_midway)
