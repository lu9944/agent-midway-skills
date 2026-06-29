---
title: Build CLI Tools with @midwayjs/commander
impact: LOW-MEDIUM
impactDescription: "DI-powered command-line tools with full Midway context"
tags: devops, cli, commander, command, tool
---

## Build CLI Tools with @midwayjs/commander

`@midwayjs/commander` (built on commander.js) lets you build CLI tools with full Midway DI. Each command is a `@Command()` class implementing `CommandRunner.run()`, with `@Option` for flags and `EnquirerService` for interactive prompts. Run via `node bootstrap.js <command> [args]`. Each command execution creates a request context, so `@Inject`/`@Config` work normally. Test via `framework.runCommand(...)` rather than mocking `process.argv`.

**Incorrect (plain script with no DI, manual argv parsing):**

```typescript
// ❌ no DI, manual parsing, no Midway services available
const args = process.argv.slice(2);
const name = args[0];
console.log(`hello ${name}`);
```

**Correct (DI-powered command + options + structured output):**

```typescript
import * as commander from '@midwayjs/commander';
@Configuration({ imports: [commander] })

// src/command/hello.command.ts
import { Command, CommandRunner, Option } from '@midwayjs/commander';
import { Inject, ILogger } from '@midwayjs/core';

@Command({ name: 'hello', description: 'Greet a user', arguments: '<name>', aliases: ['hi'] })
export class HelloCommand implements CommandRunner {
  @Inject() logger: ILogger;
  @Inject() userService: UserService;   // ✓ full DI available

  @Option({ flags: '-l, --lang [lang]', description: 'language', defaultValue: 'en' })
  parseLang(val: string) { return val; }

  async run(passedParams: string[], options?: { lang?: string }) {
    const name = passedParams[0];
    const greeting = options?.lang === 'zh' ? `你好 ${name}` : `Hello ${name}`;
    this.logger.info(greeting);
    return greeting;   // string→stdout, object→JSON, Readable→pipe
  }
}

// run: node bootstrap.js hello world --lang zh
// aliases: node bootstrap.js hi world

// interactive prompts
import { QuestionSet, Question, EnquirerService } from '@midwayjs/commander';
@QuestionSet()
export class DeployQuestions {
  @Question({ type: 'input', name: 'env', message: 'Target environment?' })
  env: string;
}
// const answers = await this.enquirer.prompt(DeployQuestions);
```

Reference: [Midway Commander](https://midwayjs.org/docs/extensions/commander)
