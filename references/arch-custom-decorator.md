---
title: Build Custom Decorators with DecoratorManager and MetadataManager
impact: MEDIUM-HIGH
impactDescription: "v4 metadata system for class/property/method/parameter decorators"
tags: architecture, decorator, custom-decorator, DecoratorManager, MetadataManager, AOP
---

## Build Custom Decorators with DecoratorManager and MetadataManager

v4 replaced `reflect-metadata` with two static utility classes: `DecoratorManager` (registers/saves modules, creates custom decorators) and `MetadataManager` (read/write metadata with prototype-chain awareness). To make a decorator **functional** (not just metadata-carrying), register an implementation handler on `MidwayDecoratorService` (`registerPropertyHandler`/`registerMethodHandler`/`registerParameterHandler`) in `@Init` or `onReady`. Method decorators require the decorated method to be **async**.

### Custom property decorator (e.g. a `@MemoryCache` injector)

```typescript
import { Provide, Scope, ScopeEnum, Init, Inject } from '@midwayjs/core';
import { DecoratorManager, MidwayDecoratorService } from '@midwayjs/core';

const MEMORY_CACHE_KEY = 'decorator:memoryCache';

// 1. define the decorator
export function MemoryCache(key?: string): PropertyDecorator {
  return DecoratorManager.createCustomPropertyDecorator(MEMORY_CACHE_KEY, { key });
}

// 2. implement the handler in a Singleton service
@Provide()
@Scope(ScopeEnum.Singleton)
export class DecoratorInitService {
  @Inject() decoratorService: MidwayDecoratorService;
  private store = new Map<string, any>();

  @Init()
  async init() {
    this.decoratorService.registerPropertyHandler(MEMORY_CACHE_KEY, (propertyName, meta) => {
      return this.store.get(meta.key);   // returns the injected value
    });
  }
}

// 3. usage — the property gets the handler's return value
@Provide()
export class ConfigService {
  @MemoryCache('appConfig')
  config: Record<string, any>;   // injected from store.get('appConfig')
}
```

### Custom method decorator (e.g. `@LoggingTime` — must be async)

```typescript
const LOGGING_KEY = 'decorator:loggingTime';

export function LoggingTime(unit: 'ms' | 's' = 'ms'): MethodDecorator {
  return DecoratorManager.createCustomMethodDecorator(LOGGING_KEY, { unit });
}

// implement with registerMethodHandler — returns an { around } join point
@Init()
async init() {
  this.decoratorService.registerMethodHandler(LOGGING_KEY, (options) => {
    return {
      around: async (joinPoint: JoinPoint) => {
        const start = Date.now();
        const result = await joinPoint.proceed(...joinPoint.args);  // call original
        this.logger.info(`${joinPoint.methodName} took ${Date.now() - start}${options.metadata.unit}`);
        return result;
      },
    };
  });
}

// usage — the decorated method MUST be async
@Provide()
export class ReportService {
  @LoggingTime('ms')
  async generate() { /* ... */ }   // ✓ async required
}
```

For a **no-impl** method decorator (metadata only, no interception), pass `false`: `createCustomMethodDecorator(KEY, {}, false)`.

### Custom parameter decorator (e.g. `@CurrentUser()`)

```typescript
import { REQUEST_OBJ_CTX_KEY } from '@midwayjs/core';

const USER_KEY = 'decorator:currentUser';

export function CurrentUser(): ParameterDecorator {
  return DecoratorManager.createCustomParamDecorator(USER_KEY, {});
}

// implement — options has { target, propertyName, metadata, originArgs, parameterIndex }
this.decoratorService.registerParameterHandler(USER_KEY, (options) => {
  // if it throws, the framework falls back to the original arg (no exception propagated)
  const ctx = options.target[REQUEST_OBJ_CTX_KEY];
  return ctx?.user ?? {};
});

// usage
@Controller('/user')
export class UserController {
  @Get('/me')
  async me(@CurrentUser() user: User) {   // user injected from ctx
    return user;
  }
}
```

### Custom class decorator + metadata (e.g. register a category of classes)

```typescript
import { DecoratorManager, MetadataManager, Scope, ScopeEnum, Provide } from '@midwayjs/core';

export const MODEL_KEY = 'decorator:model';

export function Model(): ClassDecorator {
  return (target: any) => {
    DecoratorManager.saveModule(MODEL_KEY, target);     // register for later listModule()
    MetadataManager.defineMetadata(MODEL_KEY, { test: 'abc' }, target);
    Scope(ScopeEnum.Request)(target);
    Provide()(target);   // make it injectable
  };
}

// in onReady — discover all @Model classes
const models = DecoratorManager.listModule(MODEL_KEY, (m) => m.name.includes('User'));
```

### MetadataManager API reference

```typescript
MetadataManager.defineMetadata(key, value, target, propertyName?);   // set
MetadataManager.attachMetadata(key, value, target, propertyName?);   // append to array
MetadataManager.getMetadata(key, target, propertyName?);             // prototype-chain lookup
MetadataManager.getOwnMetadata(key, target, propertyName?);          // own only
MetadataManager.copyMetadata(source, target);                        // includes prototype chain
MetadataManager.getMethodParamTypes(target, methodName);             // param types
```

Reference: [Midway Custom Decorator](https://midwayjs.org/docs/custom_decorator)
