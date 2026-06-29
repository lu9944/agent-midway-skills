---
title: Use the HTTP Client for Outbound Requests
impact: MEDIUM
impactDescription: "DI-managed axios: instances, interceptors, streams, retry, error mapping"
tags: api, http-client, axios, outbound-request, integration, retry, stream
---

## Use the HTTP Client for Outbound Requests

Midway offers two ways to make outbound HTTP requests. For simple one-off calls (no retries, no interceptors, no shared config), use the built-in `makeHttpRequest`/`HttpClient` from `@midwayjs/core` (no extra dependency). For anything more — shared baseURL/headers/timeouts, multiple instances, global interceptors, typed responses, streaming, retry — use the `@midwayjs/axios` component and inject `HttpService`. In v4 the bare `axios` export was removed; always inject `HttpService`. Configure instances declaratively in config and inject named instances with `@InjectClient(HttpServiceFactory, 'name')`.

**Incorrect (raw axios import, manual instance creation, no DI, no error handling):**

```typescript
import axios from 'axios';          // ❌ bypasses the component lifecycle & config

@Provide()
export class WeatherService {
  async getWeather(city: string) {
    // ❌ manual client, no shared config, no timeout, no retry, hard to test
    const res = await axios.get(`https://api.weather.com/${city}`);
    return res.data;
  }
}

// ❌ v3-style bare axios export (removed in v4)
import { axios } from '@midwayjs/axios';
@Provide()
export class Svc {
  @Inject() http: axios;            // ❌ removed in v4, use HttpService
}

// ❌ unhandled rejection — a failing upstream crashes the request
async callUpstream() {
  const { data } = await this.httpService.get('/flaky');   // ❌ no try/catch, raw axios error leaks
  return data;
}
```

**Correct (built-in for simple cases + @midwayjs/axios HttpService for real integrations):**

```typescript
// === Option A: built-in client (no dependency) for simple calls ===
import { makeHttpRequest } from '@midwayjs/core';

async fetchConfig() {
  const result = await makeHttpRequest('https://api.example.com/config', {
    method: 'GET',
    dataType: 'json',     // 'json' | 'text' | (default Buffer)
    timeout: 5000,
  });
  return result.data;     // NOTE: never return the raw `result` object (circular)
}

// === Option B: @midwayjs/axios HttpService (preferred for integrations) ===
// configuration.ts
import * as axios from '@midwayjs/axios';
@Configuration({ imports: [axios] })

// config.default.ts — declarative instances (axios.create config shape)
export default {
  axios: {
    default: {                        // shared by all instances
      timeout: 5000,
    },
    clients: {
      default: {                      // default instance
        baseURL: 'https://api.example.com',
        headers: { 'X-Requested-With': 'XMLHttpRequest' },
      },
      payment: {                      // a second named instance
        baseURL: 'https://pay.example.com',
        timeout: 3000,
      },
    },
  },
} as MidwayConfig;

// service — inject the default instance with typed responses
import { HttpService } from '@midwayjs/axios';
import { AxiosResponse } from 'axios';

interface WeatherDTO { city: string; temp: number; }

@Provide()
export class WeatherService {
  @Inject() httpService: HttpService;

  async getWeather(city: string): Promise<WeatherDTO> {
    // type the generic so `data` is typed — never `any`
    const { data } = await this.httpService.get<WeatherDTO>(`/weather/${city}`);
    return data;
  }
}

// inject a NAMED instance via the service factory (no magic strings)
import { HttpServiceFactory } from '@midwayjs/axios';
import { InjectClient } from '@midwayjs/core';

@Provide()
export class PaymentService {
  @InjectClient(HttpServiceFactory, 'payment')
  paymentHttp: HttpService;

  async charge(amount: number) {
    const { data } = await this.paymentHttp.post('/charge', { amount });
    return data;
  }
}
```

### Map Axios Errors to Midway Errors

Axios rejects with an `AxiosError` whose shape differs from Midway's error model. Always catch and map it to a `MidwayHttpError` (or a custom domain error) so the `@Catch` filters produce a consistent response. Inspect `error.response` (server replied with non-2xx), `error.request` (no reply), or neither (setup failure).

**Incorrect (let raw AxiosError bubble — clients see an inconsistent 500):**

```typescript
@Provide()
export class UserService {
  @Inject() httpService: HttpService;

  async fetchProfile(id: number) {
    const { data } = await this.httpService.get(`/users/${id}`);  // ❌ raw AxiosError on failure
    return data;
  }
}
```

**Correct (catch + map to structured Midway errors):**

```typescript
import { MidwayHttpError, httpError, HttpStatus } from '@midwayjs/core';
import { AxiosError } from 'axios';

// a typed domain error for upstream failures
export class UpstreamServiceError extends MidwayHttpError {
  constructor(service: string, cause: Error) {
    super(`${service} request failed`, HttpStatus.BAD_GATEWAY, { cause });
  }
}

@Provide()
export class UserService {
  @Inject() httpService: HttpService;
  @Inject() logger: ILogger;

  async fetchProfile(id: number) {
    try {
      const { data } = await this.httpService.get(`/users/${id}`);
      return data;
    } catch (err) {
      const e = err as AxiosError;
      // server replied with an error status → map it through
      if (e.response) {
        const status = e.response.status;
        if (status === 404) throw new httpError.NotFoundError(`user ${id} not found`);
        if (status === 401) throw new httpError.UnauthorizedError();
        // propagate the upstream status as a Bad Gateway with the original cause
        throw new UpstreamServiceError('user-service', e);
      }
      // no response (timeout / network) → Service Unavailable
      if (e.request) {
        this.logger.error('user-service unreachable', e.message);
        throw new httpError.ServiceUnavailableError('user-service unavailable');
      }
      // request setup error
      throw new UpstreamServiceError('user-service', e);
    }
  }
}
```

### Configure Global Interceptors (auth, logging, retry)

Interceptors are best configured **once** in `onReady` on the resolved `HttpService` (or a named instance via `HttpServiceFactory.get('name')`). Extend `AxiosRequestConfig` via declaration merging to carry custom fields (like `retry`) in a type-safe way — no magic strings. The request interceptor runs before the call; the response interceptor handles both success and errors.

**Correct (auth interceptor + timing/log interceptor + retry interceptor):**

```typescript
// src/interface.ts — type-safe custom config fields (declaration merging)
import '@midwayjs/axios';
declare module '@midwayjs/axios/dist/interface' {
  interface AxiosRequestConfig {
    retry?: number;        // max retries on 5xx/network errors
    retryDelay?: number;   // base backoff in ms
    traceId?: string;      // propagates request tracing
  }
}

// configuration.ts
import { Configuration, IMidwayContainer, ILogger, Logger } from '@midwayjs/core';
import * as axios from '@midwayjs/axios';
import { AxiosError } from 'axios';

@Configuration({ imports: [axios] })
export class MainConfiguration {
  @Logger() logger: ILogger;

  async onReady(container: IMidwayContainer) {
    const httpService = await container.getAsync(axios.HttpService);

    // 1) auth + trace-id injection
    httpService.interceptors.request.use((config) => {
      config.headers = config.headers ?? {};
      config.headers.Authorization = `Bearer ${getToken()}`;
      config.headers['x-trace-id'] = config.traceId ?? randomUUID();
      return config;
    });

    // 2) timing/log interceptor (success + error)
    httpService.interceptors.request.use((config) => {
      (config as any).__startedAt = Date.now();
      return config;
    });
    httpService.interceptors.response.use(
      (response) => {
        const ms = Date.now() - ((response.config as any).__startedAt ?? Date.now());
        this.logger.info('http %s %s %d %dms', response.config.method, response.config.url, response.status, ms);
        return response;
      },
      (error: AxiosError) => {
        const cfg = error.config ?? {};
        const ms = Date.now() - ((cfg as any).__startedAt ?? Date.now());
        this.logger.error('http %s %s failed %dms %s', cfg.method, cfg.url, ms, error.message);
        return Promise.reject(error);
      },
    );

    // 3) retry interceptor with exponential backoff (idempotent calls only)
    httpService.interceptors.response.use(undefined, async (error: AxiosError) => {
      const config = error.config;
      if (!config) return Promise.reject(error);
      const maxRetries = config.retry ?? 0;
      const attempt = ((config as any).__retryCount ?? 0) + 1;
      const retryable =
        maxRetries > 0 &&
        attempt <= maxRetries &&
        (isNetworkError(error) || (error.response?.status ?? 0) >= 500);

      if (!retryable) return Promise.reject(error);

      const delay = (config.retryDelay ?? 500) * Math.pow(2, attempt - 1);
      this.logger.warn('http retry attempt %d/%d in %dms', attempt, maxRetries, delay);
      await new Promise((r) => setTimeout(r, delay));
      (config as any).__retryCount = attempt;
      return httpService.request(config);   // replay
    });

    // configure a NAMED instance instead:
    // const factory = await container.getAsync(axios.HttpServiceFactory);
    // const pay = factory.get('payment');
    // pay.interceptors.request.use(...);
  }
}

function isNetworkError(error: AxiosError): boolean {
  return !error.response && !!error.request;   // no server reply
}
```

> Retry only idempotent methods (GET, HEAD, PUT, DELETE). Retrying POST can duplicate side effects — gate on `config.method`.

### Stream Downloads and Uploads

For large payloads, use `responseType: 'stream'` to pipe a download without buffering the whole body, and `FormData` for multipart uploads. Always await stream completion (e.g. `stream/promises.finished`) and handle backpressure.

**Correct (stream download + multipart upload):**

```typescript
import { createWriteStream } from 'fs';
import { finished } from 'stream/promises';
import { FormData } from '@midwayjs/axios';   // or 'form-data'

@Provide()
export class FileTransferService {
  @Inject() httpService: HttpService;

  // download a large file to disk without buffering in memory
  async download(url: string, dest: string): Promise<void> {
    const response = await this.httpService.get<NodeJS.ReadableStream>(url, {
      responseType: 'stream',
    });
    const writer = createWriteStream(dest);
    response.data.pipe(writer);
    await finished(writer);          // ✓ resolves on end, rejects on error
  }

  // multipart upload via FormData
  async upload(filePath: string, meta: Record<string, string>) {
    const form = new FormData();
    form.append('file', createReadStream(filePath));
    for (const [k, v] of Object.entries(meta)) form.append(k, v);
    const { data } = await this.httpService.post('/upload', form, {
      headers: form.getHeaders(),     // multipart boundary headers
      maxContentLength: Infinity,
      maxBodyLength: Infinity,
    });
    return data;
  }
}
```

For non-app code (scripts/tests) where DI is unavailable, the raw axios instance is exported as `Axios`: `import { Axios } from '@midwayjs/axios'`.

### Request Cancellation and Timeouts

Use `signal` (AbortController) for explicit cancellation and `timeout` for hard deadlines. The v4 `HttpService` forwards these to axios. Cancel in-flight requests on shutdown or when a user aborts.

**Correct (AbortController + per-request timeout):**

```typescript
@Provide()
export class SearchService {
  @Inject() httpService: HttpService;

  // caller can cancel by calling controller.abort()
  async search(query: string, controller = new AbortController()) {
    const { data } = await this.httpService.get('/search', {
      params: { q: query },
      timeout: 3000,                 // hard timeout (ms)
      signal: controller.signal,     // explicit cancellation
    });
    return data;
  }
}

// graceful shutdown: cancel pending searches in onStop
```

Reference: [Midway HTTP Request (axios)](https://midwayjs.org/docs/extensions/axios), [axios Interceptors](https://github.com/axios/axios#interceptors)
