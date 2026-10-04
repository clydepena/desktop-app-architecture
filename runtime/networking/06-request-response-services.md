# Request/response services

Most desktop applications use the network this way, without any real-time session: accounts and sign-in, update
checks, cloud saves and sync, downloads and asset stores, telemetry and crash reports, license checks. Each is "send
a request, get a response", usually over HTTPS.

## Asynchronous requests

```
http.get(url)
    .timeout(10 seconds)
    .then(response -> ...)    // on the main thread
    .onError(error -> ...)    // on the main thread
    .cancelIfClosed(panel);   // the panel that asked was closed: drop the result
```

- Requests run on the network thread or as jobs ([Jobs](../06-jobs.md)); results are delivered on the main thread.
- Every request has a **timeout**. "Waiting forever" is a bug.
- Results can arrive after the thing that asked is gone: tie requests to a scope and cancel them when it closes
  ([Services and scopes](../02-services-and-scopes.md)).

## Failure is normal

- **Retries with exponential backoff:** wait 1 s, then 2 s, 4 s, 8 s..., plus random "jitter" so thousands of
  clients don't retry at the same instant after an outage.
- **Retry only what is safe to retry.** A request is **idempotent** if repeating it has the same effect as doing it
  once ("set my display name to X"). For requests that aren't ("buy item"), send an **idempotency key**: the server
  recognizes the repeat and doesn't charge twice.
- **Circuit breaker:** after repeated failures, stop calling a service for a while instead of hammering it, then
  try again carefully.
- **Tell the user what failed and what will happen:** "Couldn't sync, will retry in 30 s", not a silent failure.

## Authentication tokens

- Sign-in returns a short-lived **access token** and a longer-lived **refresh token**.
- The access token is refreshed **before** it expires, in the background.
- Tokens are stored in the operating system's **secure storage** (keychain, credential manager), never in plain
  settings files or logs.

## Offline mode

The application keeps working without a connection:

- changes are queued locally and **synced later**
- the UI shows the state honestly ("offline, 3 changes waiting")
- **conflicts** (the same cloud save changed on two machines while offline) need a rule: newest wins, keep both,
  merge, or ask the user

## Caching and limits

- **Cache** responses that don't change often, with validation (`ETag` / `If-None-Match`), so unchanged data costs
  almost nothing to re-check.
- **Respect rate limits** sent by services, and apply your own on clients (one update check per day is plenty).

## Telemetry and crash reports

Send in the background, batched, compressed, and **only with the user's consent**. Never block startup or shutdown
on them; if sending fails, keep a small queue for next time and drop the oldest when it grows.
