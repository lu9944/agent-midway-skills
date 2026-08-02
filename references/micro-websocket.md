---
title: Choose the Right WebSocket Stack (socket.io vs ws)
impact: MEDIUM-HIGH
impactDescription: "Real-time push features — pick socket.io for browser fallback, ws for raw protocol"
tags: microservices, websocket, socketio, ws, realtime, push
---

## Choose the Right WebSocket Stack (socket.io vs ws)

Midway v4 ships two WebSocket frameworks: `@midwayjs/socketio` (socket.io v4 — namespaces, rooms, auto-reconnect, HTTP long-polling fallback; use for browser clients) and `@midwayjs/ws` (raw WebSocket protocol, no fallback, lowest overhead; use for servers/CLI/self-built protocol). Both reuse the same decorators: `@WSController(namespace)` + `@OnWSConnection()` / `@OnWSMessage(event)` / `@OnWSDisConnection()`. Response differs: socket.io uses `@WSEmit(event)` (send to the caller) and `@WSBroadCast()` (send to all in the namespace); ws uses `@WSBroadCast()` and a plain return value. They can be mounted as the main framework or alongside an HTTP framework (e.g. `@midwayjs/koa`) in the same `@Configuration` — the typical admin-dashboard pattern.

**Incorrect (WebSocket used for one-way polling, mixing frameworks, no lifecycle handling):**

```typescript
// ❌ polling instead of push — wastes connections and adds latency
@Controller('/api/notify')
export class NotifyController {
  @Get('/poll')
  async poll() {
    // ❌ client polls every 2s; a WebSocket would push instantly
    return await this.notifyService.getUnread(this.ctx.admin?.userId);
  }
}

// ❌ no disconnect handling — ghost connections leak memory
@WSController('/notify')
export class NotifySocketController {
  @Inject() ctx: Context;
  @OnWSMessage('subscribe')
  async subscribe() { /* ... */ }   // ❌ nobody cleans up on disconnect
}
```

**Correct (socket.io pushed over koa — cool-admin style admin notification module):**

```typescript
// src/configuration.ts — attach socket.io to the existing koa app (same port)
import { Configuration } from '@midwayjs/core';
import * as koa from '@midwayjs/koa';
import * as socketio from '@midwayjs/socketio';
import * as orm from '@midwayjs/typeorm';

@Configuration({
  imports: [koa, socketio, orm],
})
export class MainConfiguration {}

// src/modules/base/socket/notify.socket.ts — admin notification push
import { WSController, OnWSConnection, OnWSMessage, WSEmit, Inject, App } from '@midwayjs/core';
import { Context, Application } from '@midwayjs/socketio';

@WSController('/notify')          // namespace: /notify (string or RegExp)
export class NotifySocketController {
  @Inject()
  ctx: Context;                   // ctx IS the socket instance (ctx.id = socket id)

  @App('socketIO')
  socketApp: Application;         // server instance for cross-controller broadcast

  @OnWSConnection()
  async onConnection() {
    // e.g. track online users in Redis
    this.ctx.join('online');      // room membership
  }

  @OnWSMessage('subscribe')
  @WSEmit('notify')               // result is emitted to the CALLER on event 'notify'
  async subscribe(userId: string) {
    this.ctx.data.userId = userId;
    return { ok: true };
  }

  @OnWSDisConnection()
  async onDisconnect() { /* cleanup */ }
}

// Other controllers can broadcast outside the socket lifecycle:
// @Provide() class NotifyService {
//   @App('socketIO') socketApp: Application;
//   async pushToUser(userId: string, msg: string) {
//     this.socketApp.to(`user:${userId}`).emit('notify', msg);   // socket.io broadcast
//   }
// }
```

**Correct (ws for raw protocol + server heartbeat):**

```typescript
// src/socket/echo.socket.ts — @midwayjs/ws (no namespace arg on @WSController)
import { WSController, OnWSMessage, WSBroadCast, OnWSDisConnection, Inject } from '@midwayjs/core';
import { Context } from '@midwayjs/ws';

@WSController()
export class EchoSocketController {
  @Inject() ctx: Context;

  @OnWSMessage('message')
  @WSBroadCast()                 // send the return value to ALL connected clients
  async gotMessage(data: string) {
    return { echo: data };
  }

  @OnWSDisConnection()
  async onDisconnect(id: number) { /* cleanup */ }
}

// config.default.ts — detect dead connections (critical for long-lived ws)
export default {
  webSocket: {
    enableServerHeartbeatCheck: true,   // ping/pong, default check 30s
    serverHeartbeatInterval: 30000,
  },
} as MidwayConfig;
```

**Testing — socket.io needs a fixed port in tests (HTTP app tests use random ports):**

```typescript
// src/config/config.unittest.ts
export default {
  koa: { port: null },        // disable http in tests
  socketIO: { port: 3000 },   // explicit ws port
} as MidwayConfig;
// then createHttpRequest(app) as usual; use socket.io-client to connect on :3000
```

**Decision guide:**

| Requirement | Choice |
|-------------|--------|
| Browser chat/dashboard push, auto-reconnect, long-poll fallback | `@midwayjs/socketio` (namespaces + `@WSEmit`/`to().emit()`) |
| Low-overhead raw protocol, game server, server-to-server | `@midwayjs/ws` (no fallback, `@WSBroadCast`) |
| Realtime metrics in Prometheus | add `@midwayjs/prometheus-socket-io` alongside the metrics config |

Reference: [Midway Socket.IO](https://midwayjs.org/docs/extensions/socketio), [Midway WebSocket (ws)](https://midwayjs.org/docs/extensions/ws)
