---
title: Deploy FaaS to Aliyun FC and AWS Lambda
impact: MEDIUM
impactDescription: "Correct build pipeline and platform-specific packaging"
tags: devops, serverless, faas, deployment, aliyun-fc, aws-lambda, sam
---

## Deploy FaaS to Aliyun FC and AWS Lambda

Aliyun FC and AWS Lambda use **different** deployment paths. Aliyun FC pure-function mode uses `@midwayjs/fc-starter` + `f.yml` + `s.yaml` (Serverless Devs), auto-generating entry files with `@midwayjs/serverless-yaml-generator`. AWS Lambda uses a **standard Midway app** packaged via SAM (`template.yaml`), not FaaS decorators. In both cases: build with `tsc`, set the platform port only in `bootstrap.js` (FC=9000, Lambda=8080) to avoid breaking local dev, and install production deps in the deploy artifact.

**Incorrect (wrong platform tooling, port in config files):**

```typescript
// ❌ using f.yml/s.yaml for AWS Lambda (Aliyun-only)
// ❌ hardcoding the platform port in config.default.ts (breaks local dev on 7001)
export default { koa: { port: 9000 } } as MidwayConfig;
```

**Correct (Aliyun FC pure-function + AWS Lambda SAM):**

```typescript
// === Aliyun FC — bootstrap.js sets port 9000 ONLY for the platform ===
const { Bootstrap } = require('@midwayjs/bootstrap');
Bootstrap.configure({ globalConfig: { koa: { port: 9000 } } }).run();

// f.yml (minimal — platform + starter)
// provider:
//   name: aliyun
//   starter: '@midwayjs/fc-starter'
```

```bash
# Aliyun FC deploy pipeline (deploy.sh)
npm i
tsc
serverless-yaml-generator        # auto-generates s.yaml + entry files from @ServerlessTrigger
mkdir .serverless
cp -r dist *.json *.yaml *.js .serverless/
cd .serverless
npm install --production
s deploy                         # Serverless Devs CLI
```

```yaml
# AWS Lambda — template.yaml (SAM), standard app (no FaaS decorators)
AWSTemplateFormatVersion: "2010-09-09"
Transform: AWS::Serverless-2016-10-31
Resources:
  ApiFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: ./dist.zip
      Handler: dist/index/index.handler
      Runtime: nodejs20.x
      Timeout: 900
      Events:
        ApiEvent:
          Type: Api
          Properties:
            Path: /{any+}
            Method: ANY
```

```bash
# AWS Lambda deploy
# bootstrap.js → Bootstrap.configure({ globalConfig: { koa: { port: 8080 } } }).run();
cd sam && sam build && sam deploy
```

Error handling: production returns "Internal Server Error" (never leaks stacks); override with `SERVERLESS_OUTPUT_ERROR_STACK=true` for debugging. POST body parsing differs per gateway (base64 JSON vs urlencoded) — see `serverless_post_difference` doc.

Reference: [Midway Aliyun FC](https://midwayjs.org/docs/serverless/aliyun_faas), [AWS Lambda](https://midwayjs.org/docs/serverless/aws_lambda)
