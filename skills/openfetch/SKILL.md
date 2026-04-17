---
name: openfetch
description: Use when the user works with @hamdymohamedak/openfetch, openFetch, or a fetch-only HTTP client with interceptors, middleware, plugins (retry, timeout, hooks, debug), memory cache (TTL, stale-while-revalidate), fluent chaining (sugar), raw native Response, idempotency keys, or SSR/RSC-safe fetch in Node, browsers, Bun, Deno, or Workers. Use instead of guessing axios or XHR APIs.
---

# openFetch agent skill

**Library version this skill targets:** `v0.2.9` (`@hamdymohamedak/openfetch`). Prefer facts here and in [quick reference](./references/quick-reference.md) over guessing other clients’ APIs.

## Do not assume axios

- Transport is **`fetch` only** (no XHR). No upload progress bar via XHR.
- Package is **ESM-only** (`import` from `"@hamdymohamedak/openfetch"`).
- Runtime: **Node 18+** or any environment with `fetch` + `AbortController`.

## Install & links

- **npm:** `@hamdymohamedak/openfetch`
- **Source:** https://github.com/openfetch-js/OpenFetch  
- **Human docs:** https://openfetch-js.github.io/openfetch-docs/

## Imports

**Main entry** (many symbols are also re-exported here; bundlers can still tree-shake with `"sideEffects": false`):

```ts
import openFetch, {
  create,
  createClient,
  createRetryMiddleware,
  createCacheMiddleware,
  MemoryCacheStore,
  appendCacheKeyVaryHeaders,
  OpenFetchError,
  isOpenFetchError,
  assertSafeHttpUrl,
  generateIdempotencyKey,
  hasIdempotencyKeyHeader,
  ensureIdempotencyKeyHeader,
  maskHeaderValues,
  cloneResponse,
  redactSensitiveUrlQuery,
  // Same as @hamdymohamedak/openfetch/plugins:
  retry,
  timeout,
  hooks,
  debug,
  strictFetch,
  // Same as @hamdymohamedak/openfetch/sugar:
  createFluentClient,
} from "@hamdymohamedak/openfetch";
```

**Subpaths** (explicit splits for tooling that respects `package.json` `exports`):

```ts
import { retry, timeout, hooks, debug, strictFetch } from "@hamdymohamedak/openfetch/plugins";
import { createFluentClient } from "@hamdymohamedak/openfetch/sugar";
```

- Default export `openFetch` = `createClient()` with no initial defaults.
- **`create`** is an alias of **`createClient`**.

## Client & responses

```ts
const api = createClient({
  baseURL: "https://api.example.com",
  headers: { Authorization: "Bearer …" },
  timeout: 10_000,
  unwrapResponse: true,
  assertSafeUrl: true, // runs assertSafeHttpUrl on the resolved URL before fetch
  middlewares: [createRetryMiddleware()],
});
```

- **`unwrapResponse: false` (default):** helpers return `OpenFetchResponse<T>` → `{ data, status, statusText, headers, config }`.
- **`unwrapResponse: true`:** helpers return **`T`** (body only).
- **`rawResponse: true`:** `data` is the native **`Response`** with an **unread** body; adapter skips parse + **`transformResponse`**. Request interceptors already ran; **response** interceptors still run (`data` is that `Response`). Middleware that assumes parsed JSON in **`ctx.response.data`** will not see transforms until you read the body. Use **`cloneResponse`** from the package if two independent reads are needed.

Merged config must resolve a **`url`** (absolute, or `baseURL` + relative path). Otherwise the client throws an `Error` whose message includes ``openfetch: `url` is required``.

## HTTP helpers

| Call | Notes |
|------|--------|
| `get(url, config?)` | Optional `params` object → query string |
| `post(url, data?, config?)` | Plain objects → JSON + `content-type` unless overridden; `FormData` / `URLSearchParams` / `Blob` / binary views are not auto-JSON |
| `put`, `patch` | Same body rules as `post` |
| `delete(url, config?)` | No `data` argument; use `config.data` / `config.body` if the API needs a body |
| `head`, `options` | Same style as `get` |
| `request(urlOrConfig, config?)` | Full `method`, `responseType`, `rawResponse`, etc. |

## Plugins (`retry`, `timeout`, `hooks`, `debug`, `strictFetch`)

Shorthand factories that return **middleware**; register with **`client.use(fn)`** (same as `createRetryMiddleware`, with small DX differences).

```ts
const client = createClient({ baseURL: "https://api.example.com" })
  .use(retry({ attempts: 3, baseDelayMs: 400 })) // register retry before timeout
  .use(timeout(12_000))
  .use(debug({ includeRequestHeaders: true, maskStrategy: "partial" }));
```

- **`retry(options)`** — `attempts` aliases **`maxAttempts`**; supports **`timeoutTotalMs`**, **`enforceTotalTimeout`**, **`timeoutPerAttemptMs`**, **`retryNonIdempotentMethods`**, etc. (see retry section).
- **`timeout(ms)`** — request timeout middleware.
- **`hooks({ onRequest, onResponse, onError })`** — lifecycle logging/tracing; for transforms prefer **interceptors**.
- **`debug({ … })`** — structured logs; masked headers via **`maskStrategy`** (`full` | `partial` | `hash`); optional masked request headers; URL query redaction aligned with **`redactSensitiveUrlQuery`**.
- **`strictFetch()`** — if **`redirect`** is unset, sets **`redirect: 'error'`** so redirects are not followed silently.

**Order:** register **`retry` before `timeout`** so the timeout middleware wraps attempts the way the docs describe.

## Fluent client (`createFluentClient`)

Callable URL + method chaining (Wretch-style): **`fluent("/path").get().json()`**, **`fluent("/x").post(body).send()`**.

- **Terminals** (`.json()`, `.text()`, `.blob()`, `.arrayBuffer()`, `.stream()`, `.send()`, `.raw()`) each start **one** HTTP request unless **`.memo()`** is used on that chain.
- **`.memo()`** — one round-trip; body buffered once as `ArrayBuffer`; multiple terminals reuse it (this is **not** HTTP cache).
- **`.raw()`** — same semantics as **`rawResponse: true`**.

## Interceptors vs middleware

- **Interceptors:** `client.interceptors.request.use` / `response.use` — transform config or response; request stack is **LIFO**, response **FIFO**.
- **Middleware:** `client.use(fn)` — async `(ctx, next) => …` around the real `fetch`; use for retry, cache, logging. **Order matters** (e.g. cache vs retry).

## Retry (`createRetryMiddleware` / `retry()`)

Exponential backoff, jitter, configurable HTTP statuses and network/parse retries.

- **By default**, retries for network / parse / retryable HTTP errors apply only to **GET, HEAD, OPTIONS, TRACE** — **not** POST/PUT/PATCH/DELETE (avoid duplicate side effects).
- Opt in for mutating methods: **`retry: { retryNonIdempotentMethods: true }`** on the client or per request.
- When **`retryNonIdempotentMethods`** is true, **`maxAttempts > 1`**, method is **POST**, and no idempotency header exists, middleware adds a stable **`Idempotency-Key`** (unless **`autoIdempotencyKey: false`**). Use **`generateIdempotencyKey`**, **`hasIdempotencyKeyHeader`**, **`ensureIdempotencyKeyHeader`** for custom flows.
- **`timeoutTotalMs`** — monotonic budget for the **whole** retry sequence (including backoff); on expiry: **`OpenFetchError`** code **`ERR_RETRY_TIMEOUT`** (not retried). **`enforceTotalTimeout`** (default true) merges a deadline into **`signal`** per attempt.
- **`timeoutPerAttemptMs`** — overrides per-attempt **`request.timeout`** inside the retry middleware.
- **`ERR_CANCELED`** (user/per-attempt timeout abort) is **not** retried.

## Memory cache

- `new MemoryCacheStore({ maxEntries })` + **`createCacheMiddleware(store, { ttlMs, staleWhileRevalidateMs, methods, key, varyHeaderNames, … })`**.
- Default cache key includes method + full URL; **`authorization`** and **`cookie`** are **always** folded into the key **unless** **`varyHeaderNames`** is explicitly **`[]`** (dangerous for shared auth — see package warning / use **`suppressAuthCacheKeyWarning`** only when intentional). Prefer omitting `varyHeaderNames` or listing extra headers; use **`appendCacheKeyVaryHeaders`** for custom keys.
- **`staleWhileRevalidateMs`:** can serve stale after TTL and refresh in background; that background path calls **`dispatch` directly** — not every **`client.use()`** middleware runs for it (see human docs architecture note).

## Errors & logging

- Guard with **`isOpenFetchError(err)`**.
- **`error.toShape()`** / **`toJSON()`** — by default **omits** response **`data`** and **`headers`**; pass **`includeResponseData: true`** / **`includeResponseHeaders: true`** only for trusted diagnostics. **`redactSensitiveUrlQuery`** defaults on for the serialized URL in the shape.
- The live error may still hold full **`config`** (including secrets) — never forward raw instances to clients.

## Security

- **`assertSafeHttpUrl(url)`** — optional guard for untrusted URLs (`http`/`https` only; blocks many literal private/loopback IPs). **Does not** stop DNS rebinding; pair with hostname allowlists / egress controls when relevant.
- **`assertSafeUrl: true`** on the client — runs **`assertSafeHttpUrl`** on the **fully resolved** URL before **`fetch`**.

## Framework usage (minimal)

- **React:** `useEffect` + `await openFetch.get(url)`; use `res.data` unless `unwrapResponse` is true on the client.
- **RSC / server:** `await api.get(…)` in an async component is fine; keep secrets server-side.
- **Vue:** `onMounted` + assign `res.data` to a `ref`.

## When to refuse or redirect

- Need **XHR upload progress** → say `fetch` does not expose it; suggest multipart + server progress, or a different tool.
- User asks for **openFetch docs** verbatim → point to the docs URL above or read [quick reference](./references/quick-reference.md) in this folder.

## Repo layout (for code navigation)

- `src/runtime/client.ts` — client, verb helpers  
- `src/transport/dispatch.ts` — `fetch`, parse, `validateStatus`, `rawResponse`  
- `src/runtime/middleware.ts`, `interceptors.ts`, `retry.ts`, `cache.ts`  
- `src/domain/error.ts` — `OpenFetchError`, `toShape`  
- `src/plugins/` — `retry`, `timeout`, `hooks`, `debug`, `strictFetch`  
- `src/sugar/fluent.ts` — `createFluentClient`, `.memo()`, `.raw()`
