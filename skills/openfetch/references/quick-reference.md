# openFetch — quick reference (v0.2.9)

## Config highlights (`OpenFetchConfig`)

`url`, `baseURL`, `method`, `params`, `paramsSerializer`, `headers`, `data`, `body`, `timeout`, `signal`, `auth` (Basic), `withCredentials`, `responseType` (`json` | `text` | `blob` | `arraybuffer` | `stream`), **`rawResponse`**, `validateStatus`, `transformRequest` / `transformResponse` (arrays, concatenated with defaults), `middlewares`, `retry`, `memoryCache`, `unwrapResponse`, **`assertSafeUrl`**, plus `RequestInit` passthrough (`cache`, `credentials`, `mode`, `redirect`, …).

## Subpath exports

| Import path | Purpose |
|-------------|---------|
| `@hamdymohamedak/openfetch` | Core client, middleware factories, errors, helpers, re-exported plugins + fluent |
| `@hamdymohamedak/openfetch/plugins` | `retry`, `timeout`, `hooks`, `debug`, `strictFetch` only |
| `@hamdymohamedak/openfetch/sugar` | `createFluentClient` only |

## Plugins (middleware factories)

| Export | Role |
|--------|------|
| `retry(opts)` | Same family as `createRetryMiddleware`; `attempts` → `maxAttempts` |
| `timeout(ms)` | Per-request timeout middleware |
| `hooks({ onRequest, onResponse, onError })` | Lifecycle side effects |
| `debug({ … })` | Logging with `maskStrategy`, optional masked request headers |
| `strictFetch()` | Default `redirect: 'error'` when `redirect` was unset |

Register **`retry` before `timeout`** when using both.

## Fluent (`createFluentClient`)

- Call: `fluent(url, config?)` → chain `.get()` / `.post(data)` / … → terminal.
- Terminals: `.json()`, `.text()`, `.blob()`, `.arrayBuffer()`, `.stream()`, `.send()`, `.raw()` — each starts a **new** request unless **`.memo()`** is on the chain.
- **`.memo()`** — one HTTP call; body buffered once (not HTTP caching).
- **`.raw()`** — native `Response`, unread body; same core behavior as `rawResponse: true`.

## `OpenFetchError` codes (common)

| Code | Meaning |
|------|--------|
| `ERR_BAD_RESPONSE` | Status failed `validateStatus` |
| `ERR_NETWORK` | Network / throw from `fetch` |
| `ERR_PARSE` | Body parse failed |
| `ERR_CANCELED` | Aborted (timeout or external signal); **not** retried |
| `ERR_RETRY_TIMEOUT` | `retry.timeoutTotalMs` budget exhausted; **not** retried |

## Retry options (resolved with `ctx.request.retry`)

Not exhaustive — see types / docs: `maxAttempts`, `baseDelayMs`, `maxDelayMs`, `factor`, `retryOnStatus`, `retryOnNetworkError`, `retryNonIdempotentMethods`, `shouldRetry`, **`timeoutTotalMs`**, **`enforceTotalTimeout`**, **`timeoutPerAttemptMs`**, **`autoIdempotencyKey`**, etc.

## Cache middleware

- Methods default **`GET`**, **`HEAD`** only.
- Keys: method + URL; **`authorization`** + **`cookie`** always vary **unless** `varyHeaderNames` is explicitly `[]` (see main skill / package warnings).
- **`memoryCache`**: per-request `ttlMs`, `staleWhileRevalidateMs`, `skip`.
- **`appendCacheKeyVaryHeaders(baseKey, headers, names)`** — helper for custom `key`.

## Merge rules (`mergeConfig`)

Later wins for top-level keys. `headers` shallow-merged. `middlewares`, `transformRequest`, `transformResponse` **concatenated**. `retry` / `memoryCache` shallow-merged. Prototype-pollution keys stripped from merge targets.

## Useful package exports (logging / tooling)

`maskHeaderValues`, `cloneResponse`, `redactSensitiveUrlQuery`, `DEFAULT_SENSITIVE_QUERY_PARAM_NAMES`, idempotency helpers (see main skill).
