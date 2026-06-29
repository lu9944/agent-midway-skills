---
title: Subscribe to Changing Data with DataListener
impact: MEDIUM
impactDescription: "Caches remote/volatile data behind a stable read API"
tags: performance, data-listener, cache, subscribe, singleton
---

## Subscribe to Changing Data with DataListener

`DataListener<T>` (from `@midwayjs/core`) is a base class for the "data subscription" pattern: a Singleton holds a piece of data that updates periodically (from a remote API, DB, or event stream), while consumers read the latest value via a stable `getData()` API without worrying about freshness. Implement `initData()` (initial/async fetch) and `onData(setData)` (periodic update); clean up timers/connections in `destroyListener()`.

**Incorrect (every consumer fetches remote data independently, or stale cached value):**

```typescript
@Provide()
export class UserService {
  async getConfig() {
    return await this.httpService.get('/remote-config');  // ❌ network call every request
  }
}

// OR a stale singleton with no update mechanism
@Singleton()
export class ConfigCache {
  private data = {};   // ❌ never refreshes after first load
}
```

**Correct (DataListener Singleton + periodic update + cleanup):**

```typescript
import { Provide, Scope, ScopeEnum, DataListener, Inject } from '@midwayjs/core';

@Provide()
@Scope(ScopeEnum.Singleton)
export class RemoteConfigListener extends DataListener<Record<string, any>> {
  @Inject() httpService: HttpService;
  private timer: NodeJS.Timeout;

  // initial data (can be async)
  async initData() {
    const { data } = await this.httpService.get('/remote-config');
    return data;
  }

  // periodic update — call setData() to refresh the cached value
  onData(setData: (data: Record<string, any>) => void) {
    this.timer = setInterval(async () => {
      const { data } = await this.httpService.get('/remote-config');
      setData(data);   // ✓ updates the internal cache; consumers see it instantly
    }, 60_000);         // refresh every minute
  }

  // clean up resources on shutdown
  async destroyListener() {
    clearInterval(this.timer);
  }
}

// consumer — always reads the latest value, no network calls
@Provide()
export class UserService {
  @Inject() remoteConfig: RemoteConfigListener;

  async getFeatureFlags() {
    return this.remoteConfig.getData();   // ✓ instant read, always fresh
  }
}
```

The pattern hides volatility behind a stable API: the listener manages freshness (timers, subscriptions), while business code simply calls `getData()`. Pair with `@Autoload` if the listener should self-start without `onReady` wiring.

Reference: [Midway Data Listener](https://midwayjs.org/docs/data_listener)
