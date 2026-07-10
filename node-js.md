# Node.js — Deep Conceptual Guide

> A concepts-first reference. Each section explains *how* Node works underneath, *why* it behaves that way, and the trade-offs that matter in production.

## Table of Contents

1. [Asynchronous Programming](#1-asynchronous-programming)
2. [The Event Loop & Timing](#2-the-event-loop--timing)
3. [Streams & Buffers](#3-streams--buffers)
4. [HTTP, Networking & APIs](#4-http-networking--apis)
5. [Express & Web Frameworks](#5-express--web-frameworks)
6. [Authentication & Security](#6-authentication--security)
7. [Multi-processing & Scaling](#7-multi-processing--scaling)
8. [Performance & Optimization](#8-performance--optimization)
9. [Error Handling](#9-error-handling)
10. [Architecture & Design Patterns](#10-architecture--design-patterns)
11. [Modern Node.js Features](#11-modern-nodejs-features)

---

## 1. Asynchronous Programming

### The core mental model

Node.js runs your JavaScript on a **single thread**. That one thread executes your code, and it must never block, because while it is blocked *nothing else can happen* — no new connections accepted, no timers fired, no callbacks run. Asynchronous programming is the set of techniques Node uses to start a long-running operation (disk read, network call, DNS lookup), hand it off to the system, and get *notified later* when it finishes — all without the main thread waiting.

The key insight: **Node is single-threaded for your JavaScript, but multi-threaded underneath.** libuv (Node's C library) maintains a thread pool and uses OS async primitives (epoll on Linux, kqueue on macOS, IOCP on Windows). When you call `fs.readFile`, the actual read happens off-thread; your callback is queued back onto the main thread only when the data is ready.

### The three generations of async

**1. Callbacks (the original)**

```js
fs.readFile('/path', (err, data) => {
  if (err) return handleError(err);
  process(data);
});
```

The **error-first callback** convention: the first argument is always the error (or `null`), the rest are results. This is a *convention*, not enforced by the language.

Problems: **callback hell** (nested pyramids), no composition, error handling is manual and easy to forget, and you cannot `try/catch` across the async boundary.

**2. Promises (ES2015)**

A Promise is an object representing a value that may not exist yet. It has three states: **pending → fulfilled** or **pending → rejected**. Once settled, it is immutable.

```js
readFilePromise('/path')
  .then(process)
  .catch(handleError);
```

Critical properties:
- **`.then` callbacks always run asynchronously**, even if the promise is already resolved. They are scheduled as microtasks (more in §2).
- Promises **compose**: `.then` returns a new promise, so chains flatten. Returning a promise from within `.then` waits for it.
- A rejection propagates down the chain until a `.catch`. An **unhandled rejection** is a common bug source.

**3. async/await (ES2017)**

Syntactic sugar over promises. `await` pauses the *async function* (not the thread) until the promise settles, then resumes.

```js
async function load() {
  try {
    const data = await readFilePromise('/path');
    return process(data);
  } catch (err) {
    handleError(err);
  }
}
```

`await` gives you synchronous-looking control flow with real `try/catch`, while remaining non-blocking. An `async` function *always* returns a promise.

### Concurrency patterns that matter

**Sequential vs parallel** — a frequent performance bug:

```js
// SLOW — each await waits for the previous (sequential)
const a = await fetchA();
const b = await fetchB();

// FAST — both start immediately, then we await both (parallel)
const [a, b] = await Promise.all([fetchA(), fetchB()]);
```

Promise combinators:
- **`Promise.all`** — resolves when all resolve; rejects fast on the *first* rejection. Other work keeps running but its results are discarded.
- **`Promise.allSettled`** — waits for all, never rejects; returns `{status, value|reason}` for each. Use when you want *every* outcome.
- **`Promise.race`** — settles as soon as the first promise settles (fulfilled *or* rejected). Useful for timeouts.
- **`Promise.any`** — resolves on the first *fulfillment*; rejects only if all reject (with an `AggregateError`).

**Controlling concurrency** — `Promise.all` on 10,000 items opens 10,000 connections at once and can crash the process or the downstream service. Use a concurrency limiter (e.g. `p-limit`) or batch:

```js
import pLimit from 'p-limit';
const limit = pLimit(10);           // max 10 in flight
await Promise.all(urls.map(u => limit(() => fetch(u))));
```

### Common async pitfalls

- **`forEach` does not await.** `array.forEach(async ...)` fires all callbacks and returns immediately without waiting. Use `for...of` with `await` (sequential) or `Promise.all(map(...))` (parallel).
- **Forgetting to return a promise** inside `.then` breaks the chain's error propagation and sequencing.
- **Mixing callbacks and promises** — wrap callback APIs once with `util.promisify` rather than manually.
- **Floating promises** — calling an async function without `await` or `.catch` means errors vanish into unhandled rejections.

```js
import { promisify } from 'util';
const readFileAsync = promisify(fs.readFile);
```

Modern Node also exposes promise-native APIs directly: `fs.promises`, `dns.promises`, `timers/promises`, etc.

---

## 2. The Event Loop & Timing

This is the single most important internal to understand. Get this right and most "weird ordering" bugs become obvious.

### What the event loop is

The event loop is a C-level loop (in libuv) that runs as long as there is work to do. Each turn ("tick") it processes a set of **phases** in a fixed order. Each phase has a FIFO queue of callbacks. Node runs callbacks in the current phase until the queue empties or a system limit is hit, then moves on.

### The phases (in order)

```
   ┌───────────────────────────┐
┌─>│           timers          │  setTimeout, setInterval callbacks
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │  some system ops (e.g. TCP errors)
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │  internal only
│  └─────────────┬─────────────┘      ┌───────────────┐
│  ┌─────────────┴─────────────┐      │   incoming:   │
│  │           poll            │<─────┤  connections, │  waits for I/O; runs I/O callbacks
│  └─────────────┬─────────────┘      │   data, etc.  │
│  ┌─────────────┴─────────────┐      └───────────────┘
│  │           check           │  setImmediate callbacks
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │  socket.on('close'), etc.
   └───────────────────────────┘
```

- **timers** — executes callbacks scheduled by `setTimeout`/`setInterval` whose threshold has elapsed.
- **poll** — the heart. Retrieves new I/O events and executes their callbacks. If there are no timers pending, the loop can *block here* waiting for I/O. This is where the process "sleeps" efficiently instead of spinning.
- **check** — runs `setImmediate` callbacks. `setImmediate` is designed to run *right after* the poll phase completes.
- **close** — emits `'close'` events.

### Microtasks vs macrotasks — the crucial distinction

There are two special queues that are **NOT** phases and are drained **between every callback**:

1. **`process.nextTick` queue** — highest priority. Drained after the current operation completes, *before* any other microtask.
2. **Microtask queue** (Promises, `queueMicrotask`) — drained after the nextTick queue.

The rule: **after each individual macrotask callback (and after each phase transition), Node fully drains the nextTick queue, then fully drains the promise microtask queue, before continuing.**

This is why:

```js
console.log('1: sync');
setTimeout(() => console.log('2: timeout'), 0);
setImmediate(() => console.log('3: immediate'));
Promise.resolve().then(() => console.log('4: promise'));
process.nextTick(() => console.log('5: nextTick'));
console.log('6: sync');

// Output:
// 1: sync
// 6: sync
// 5: nextTick      <- microtasks drain before the loop continues
// 4: promise
// 2: timeout       <- timers phase (2 vs 3 order at top level is not guaranteed)
// 3: immediate
```

Sync code runs first (it's the current macrotask). Then before the event loop advances, `nextTick` drains, then promises drain. Only then do timer/immediate callbacks run.

### `setTimeout(fn, 0)` vs `setImmediate` vs `nextTick`

- **`process.nextTick`** — runs before the loop continues *at all*. Can **starve the event loop** if you recurse — a `nextTick` that schedules another `nextTick` never lets I/O run. Use sparingly.
- **`setImmediate`** — runs in the check phase, *after* poll. Guaranteed to run after I/O callbacks of the current iteration.
- **`setTimeout(fn, 0)`** — actually a minimum of ~1ms; runs in the timers phase.

**Inside an I/O callback, `setImmediate` always beats `setTimeout(fn,0)`** deterministically, because after the poll phase the check phase runs immediately, whereas timers wait for the next loop iteration:

```js
fs.readFile(__filename, () => {
  setTimeout(() => console.log('timeout'), 0);
  setImmediate(() => console.log('immediate'));
});
// Always: immediate, then timeout
```

### Blocking the event loop — the cardinal sin

Because everything shares one thread, a synchronous CPU-bound operation freezes the *entire* server:

```js
// This blocks ALL other requests for however long the loop runs
app.get('/bad', (req, res) => {
  let sum = 0;
  for (let i = 0; i < 1e10; i++) sum += i;   // seconds of blocking
  res.json({ sum });
});
```

Also blocking: large synchronous JSON.parse/stringify, sync `fs` calls (`readFileSync`), synchronous crypto (`pbkdf2Sync`), and complex regex (**ReDoS** — catastrophic backtracking).

Solutions: offload to **Worker Threads** (§7), break work into chunks yielded via `setImmediate`, use streaming, or push to a separate service/queue.

### Timers are not precise

`setTimeout(fn, 100)` means "run *no earlier than* 100ms." If the loop is busy or a timer callback is long, the actual delay is larger. Never rely on timers for precise timing or high-frequency scheduling.

### `timers/promises`

Modern Node provides promisified timers, useful with async/await:

```js
import { setTimeout as sleep } from 'timers/promises';
await sleep(1000);   // non-blocking pause
```

---

## 3. Streams & Buffers

### Why streams exist

Reading a 2GB file with `fs.readFile` loads all 2GB into memory before you can touch it. Streams let you process data **piece by piece** as it arrives, using bounded memory. This is fundamental to Node's scalability — HTTP requests/responses, file I/O, compression, and crypto are all streams.

### Buffers — the raw bytes

A `Buffer` is a fixed-length chunk of memory *outside* the V8 heap, representing raw binary data. JavaScript strings are UTF-16 and immutable; Buffers are for bytes.

```js
const buf = Buffer.from('héllo', 'utf8');   // bytes of the string
buf.length;                                  // 6 (é is 2 bytes in UTF-8)
buf.toString('utf8');                        // back to string
Buffer.alloc(10);                            // zero-filled, safe
Buffer.allocUnsafe(10);                      // faster, but may contain old memory — must overwrite
```

Key points:
- `Buffer.allocUnsafe` reuses memory that may contain **sensitive leftover data** — always fully overwrite it or use `alloc`.
- **Encoding matters.** A multi-byte character (UTF-8, emoji) can be **split across chunk boundaries**. Never `buf.toString()` on individual stream chunks if you need correct text — use `string_decoder`'s `StringDecoder`, which buffers incomplete sequences.

```js
import { StringDecoder } from 'string_decoder';
const decoder = new StringDecoder('utf8');
stream.on('data', chunk => output += decoder.write(chunk)); // handles split chars
```

### The four stream types

| Type | Direction | Examples |
|------|-----------|----------|
| **Readable** | source you read from | `fs.createReadStream`, HTTP request, `process.stdin` |
| **Writable** | sink you write to | `fs.createWriteStream`, HTTP response, `process.stdout` |
| **Duplex** | both, independent | TCP socket |
| **Transform** | Duplex where output is derived from input | `zlib.createGzip`, crypto cipher |

### Flowing vs paused mode

A Readable stream operates in one of two modes:

- **Paused (default)** — you must explicitly call `.read()` to pull data.
- **Flowing** — data is pushed to you via `'data'` events as fast as it arrives.

Attaching a `'data'` listener or calling `.pipe()` switches to flowing. In flowing mode with no backpressure handling, a fast source can overwhelm a slow consumer.

### Backpressure — the concept that makes streams safe

When you write to a Writable faster than it can drain (e.g. slow disk, slow network), data queues in memory. **Backpressure** is the mechanism that signals "slow down."

`writable.write(chunk)` returns `false` when its internal buffer exceeds `highWaterMark`. You should stop writing and wait for the `'drain'` event:

```js
function writeData(writable, data) {
  if (!writable.write(data)) {
    // buffer full — wait before writing more
    writable.once('drain', () => writeData(writable, nextData));
  }
}
```

**`.pipe()` and `pipeline()` handle backpressure automatically** — this is the main reason to use them instead of manual `data`/`write` loops.

```js
// pipe: simple, but does NOT forward errors or clean up on failure
readable.pipe(gzip).pipe(writable);

// pipeline: handles backpressure AND errors AND cleanup — PREFER THIS
import { pipeline } from 'stream/promises';
await pipeline(readable, gzip, writable);
```

The classic `.pipe()` bug: if the source errors mid-stream, the destination is **not** closed, leaking file descriptors. `pipeline` destroys all streams on any error. Always prefer `pipeline`.

### `highWaterMark`

The threshold (in bytes for binary, or number of objects in objectMode) at which a stream considers its buffer "full." Default 16KB for byte streams, 16 objects for object mode. Tuning it trades memory for fewer, larger operations.

### Practical example — streaming an HTTP download to disk with compression

```js
import { pipeline } from 'stream/promises';
import { createWriteStream } from 'fs';
import { createGzip } from 'zlib';
import { Readable } from 'stream';

const res = await fetch(url);
await pipeline(
  Readable.fromWeb(res.body),   // web ReadableStream → Node Readable
  createGzip(),                 // Transform
  createWriteStream('out.gz')
);
// Bounded memory regardless of file size; errors propagate; FDs cleaned up.
```

### Async iteration over streams (modern, clean)

```js
for await (const chunk of readable) {
  process(chunk);   // backpressure handled automatically by the loop
}
```

This is often the most readable way to consume a stream and integrates with `try/catch`.

---

## 4. HTTP, Networking & APIs

### The layers

Node's networking is layered: `net` (raw TCP) → `http`/`https` (HTTP/1.1) → `http2` → your framework. There's also `dgram` (UDP) and `tls` (encrypted sockets).

### The `http` module — request & response are streams

```js
import http from 'http';

const server = http.createServer((req, res) => {
  // req is a Readable stream (the request body)
  // res is a Writable stream (the response)
  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({ ok: true }));
});
server.listen(3000);
```

Because `req` is a stream, the body **does not arrive all at once**. To read a JSON body manually you must collect chunks (frameworks do this for you):

```js
let body = '';
req.on('data', c => body += c);
req.on('end', () => {
  const data = JSON.parse(body);   // guard with try/catch + size limit!
});
```

**Always cap body size** — an unbounded `body +=` is a memory-exhaustion DoS vector.

### Keep-Alive and connection reuse

HTTP/1.1 defaults to persistent connections. Establishing a TCP (and TLS) connection is expensive (round trips + handshake). **Keep-Alive** reuses a connection for multiple requests. On the client side, use an `Agent` with `keepAlive: true` — this dramatically cuts latency for repeated calls to the same host.

### `fetch` is built in (Node 18+)

Node ships a global `fetch` implemented on **undici** (a modern HTTP client). No more `node-fetch` or `axios` needed for basic use.

```js
const res = await fetch('https://api.example.com/data', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(payload),
  signal: AbortSignal.timeout(5000),   // built-in timeout
});
if (!res.ok) throw new Error(`HTTP ${res.status}`);
const data = await res.json();
```

Gotchas:
- `fetch` **does not reject on 4xx/5xx** — only on network failure. Check `res.ok`.
- The body can be consumed **once** — pick `.json()` OR `.text()`, not both.
- Use `AbortController`/`AbortSignal.timeout` for cancellation and timeouts (there's no built-in default timeout — a hung server can hang you forever).

### Timeouts everywhere

Every network call needs a timeout. Without one, a slow or dead peer ties up resources indefinitely. Set them on servers (`server.requestTimeout`, `server.headersTimeout`) and clients (`AbortSignal.timeout`).

### HTTP/2 and HTTP/3

- **HTTP/2** (`http2` module) — multiplexes many streams over one connection (no head-of-line blocking at the HTTP layer), header compression (HPACK), server push. Big win for many small assets.
- **HTTP/3** — runs over QUIC (UDP), eliminating TCP head-of-line blocking. Support in Node is still evolving.

### REST vs alternatives

- **REST** — resource-oriented, HTTP verbs, stateless. Ubiquitous, cacheable, simple.
- **GraphQL** — single endpoint, client specifies exactly the fields it wants; solves over/under-fetching but adds complexity and caching challenges.
- **gRPC** — HTTP/2 + Protocol Buffers (binary). Fast, strongly typed, great for internal service-to-service; not browser-native without a proxy.
- **WebSockets** (`ws` library, or built-in client in recent Node) — full-duplex persistent connection for real-time (chat, live updates). Server-Sent Events (SSE) is a simpler one-directional alternative.

### DNS caching

Node does **not** cache DNS by default; every connection may trigger a lookup. Under load this adds latency and can exhaust the libuv thread pool (DNS lookups via `getaddrinfo` use it). Consider a DNS cache or `dns.setDefaultResultOrder` tuning for high-throughput clients.

---

## 5. Express & Web Frameworks

### The middleware model

Express is fundamentally a **middleware pipeline**. A request flows through an ordered chain of functions, each with the signature `(req, res, next)`. Each middleware can:
- inspect/modify `req`/`res`,
- end the response (`res.send`), or
- call `next()` to pass control to the next middleware.

```js
app.use((req, res, next) => {
  req.startTime = Date.now();
  next();                       // MUST call next() or the request hangs
});
```

**Order matters absolutely.** Middleware runs top to bottom. Body parsers must come before routes that read the body; auth middleware before protected routes; error handlers last.

### The request lifecycle

```
incoming request
  → app-level middleware (logging, cors, body-parser)
  → router matching
  → route-specific middleware (auth, validation)
  → route handler
  → response sent
  (any thrown/next(err) jumps to) → error-handling middleware
```

### Error-handling middleware — the four-argument signature

Express identifies error handlers by their **arity**: they take *four* arguments `(err, req, res, next)`. It must be registered **last**.

```js
app.use((err, req, res, next) => {
  console.error(err);
  res.status(err.status || 500).json({ error: err.message });
});
```

**Critical caveat:** in Express 4, **async errors are not caught automatically.** A rejected promise in an async handler does not reach your error middleware — you must catch it and call `next(err)`, or wrap handlers:

```js
const asyncHandler = fn => (req, res, next) => Promise.resolve(fn(req, res, next)).catch(next);
app.get('/x', asyncHandler(async (req, res) => { /* throw is now caught */ }));
```

**Express 5** fixes this — it automatically forwards rejected promises from async handlers to the error middleware.

### Routing

```js
const router = express.Router();          // modular, mountable
router.get('/:id', handler);              // :id is req.params.id
app.use('/users', router);                // mounts at /users
```

Route params, query strings (`req.query`), and body (`req.body`, needs a parser) are the three input sources — validate all of them.

### Essential middleware

- `express.json()` / `express.urlencoded()` — body parsing (set a `limit` to prevent large-payload DoS).
- `helmet` — sets security headers (CSP, HSTS, X-Frame-Options...).
- `cors` — controls cross-origin access; **do not** blindly `origin: '*'` with credentials.
- `compression` — gzip responses.
- `express-rate-limit` — throttle abusive clients.
- `morgan` — request logging.

### The framework landscape

- **Express** — minimal, unopinionated, massive ecosystem. The default choice; v5 modernizes async handling.
- **Fastify** — schema-based (JSON Schema) validation and serialization, significantly faster, built-in logging (pino), plugin encapsulation. Great when performance and structure matter.
- **Koa** — from the Express team; async/await-native middleware via `ctx` and an onion-model (middleware wraps downstream via `await next()`). Tiny core.
- **NestJS** — opinionated, TypeScript-first, Angular-style DI, modules, decorators. Structure for large teams/apps; heavier.
- **Hono** — ultralight, edge/runtime-agnostic (Workers, Deno, Bun, Node), Web-standard APIs.

### Why Fastify's approach is faster

Fastify compiles JSON schemas into optimized serialization functions ahead of time, avoiding the reflection cost of generic `JSON.stringify` and validating input at the boundary. This "define your shapes" model is both faster and safer.

---

## 6. Authentication & Security

### Authentication vs authorization

- **Authentication (AuthN)** — *who are you?* (login).
- **Authorization (AuthZ)** — *what are you allowed to do?* (permissions/roles).

Keep them distinct in your code.

### Password storage — never reversible

Never store plaintext or encrypted (reversible) passwords. **Hash** them with a slow, salted, memory-hard algorithm:

- **bcrypt** — battle-tested, adaptive cost factor.
- **scrypt** (built into `crypto`) — memory-hard.
- **argon2** — modern winner of the Password Hashing Competition; preferred for new systems.

```js
import argon2 from 'argon2';
const hash = await argon2.hash(password);      // salt handled internally
const ok   = await argon2.verify(hash, input);
```

The **salt** (unique random per password) defeats rainbow tables; the **cost factor** makes brute force expensive and is tunable as hardware improves. Never use fast hashes (MD5, SHA-256) for passwords.

### Sessions vs tokens (JWT)

**Session-based (stateful):**
- Server stores session data (in Redis/DB); client holds an opaque session ID in a cookie.
- Easy to revoke (delete the session). Requires server-side storage.

**Token-based / JWT (stateless):**
- A signed token (`header.payload.signature`) carries claims; the server verifies the signature without a lookup.
- Scales horizontally with no shared session store, but **hard to revoke** before expiry.

**JWT deep points:**
- A JWT is **signed, not encrypted** — the payload is base64, readable by anyone. Never put secrets in it.
- Signature algorithms: **HS256** (shared secret) vs **RS256** (private/public key — verifiers only need the public key). Prefer RS256 for multi-service setups.
- The infamous **`alg: none`** and **algorithm-confusion** attacks: always pin the expected algorithm on verification; never trust the token's own header.
- Use **short-lived access tokens** + a **refresh token** (stored securely, revocable) to mitigate the non-revocation problem.

### Cookies — secure by default

```js
res.cookie('sid', id, {
  httpOnly: true,   // JS can't read it → mitigates XSS token theft
  secure: true,     // HTTPS only
  sameSite: 'lax',  // mitigates CSRF; 'strict' for sensitive
  maxAge: 3600_000,
});
```

- **`httpOnly`** — cookie invisible to `document.cookie`, so XSS can't steal it.
- **`secure`** — never sent over plain HTTP.
- **`sameSite`** — restricts cross-site sending, the primary modern CSRF defense.

### The OWASP Top 10 mindset (Node specifics)

- **Injection (SQL/NoSQL/command):** Use parameterized queries / prepared statements. Never string-concatenate user input into queries. For NoSQL, beware operator injection (`{ $gt: '' }`) — validate types. Never pass user input to `exec`/`eval`/`child_process` shell.
- **XSS:** Escape/encode output; set a **Content-Security-Policy**; use `httpOnly` cookies. Frameworks like React escape by default.
- **CSRF:** `sameSite` cookies + anti-CSRF tokens for state-changing requests.
- **Broken auth:** rate-limit login, enforce strong passwords, lock/backoff on repeated failures, use MFA.
- **Sensitive data exposure:** TLS everywhere, encrypt at rest, never log secrets/PII.
- **Security misconfiguration:** `helmet`, disable `x-powered-by`, don't leak stack traces to clients, keep dependencies patched.
- **Vulnerable dependencies:** `npm audit`, lockfiles, minimize the dependency tree (supply-chain risk).
- **SSRF:** validate/allowlist outbound URLs; block requests to internal metadata endpoints (`169.254.169.254`).

### Cryptography essentials

- Use the built-in `crypto` module; don't roll your own.
- **`crypto.randomBytes` / `crypto.randomUUID`** for tokens/IDs — never `Math.random()` (not cryptographically secure).
- **Timing attacks:** compare secrets with `crypto.timingSafeEqual`, not `===`.
- **Encryption:** use authenticated modes (AES-256-**GCM**), unique IVs per message.
- Store secrets in **environment variables / a secrets manager**, never in code or git.

### Input validation & sanitization

Validate at the boundary with a schema library (**Zod**, **Joi**, **Yup**, or Fastify's JSON Schema). Reject unexpected shapes early. Distinguish validation (is it well-formed?) from sanitization (strip/escape dangerous content).

### Rate limiting & DoS protection

Limit requests per IP/user, cap request body sizes, set timeouts, and put a reverse proxy (nginx) or CDN/WAF in front. Guard against **ReDoS** (evil regex) and algorithmic complexity attacks.

---

## 7. Multi-processing & Scaling

### The problem

One Node process uses **one CPU core** for JavaScript. On an 8-core machine, a single process leaves 7 cores idle. Scaling means either running multiple processes (to use all cores / multiple machines) or offloading CPU work to threads.

### The `cluster` module — multiple processes, shared port

`cluster` forks the master into N worker processes that **share the same listening socket**. The OS (or Node's round-robin, default on non-Windows) distributes incoming connections among workers.

```js
import cluster from 'cluster';
import os from 'os';
import http from 'http';

if (cluster.isPrimary) {
  for (let i = 0; i < os.availableParallelism(); i++) cluster.fork();
  cluster.on('exit', () => cluster.fork());   // restart crashed workers
} else {
  http.createServer(handler).listen(3000);    // each worker listens
}
```

Key facts:
- Workers are **separate processes with separate memory** — no shared variables. Share state via Redis/DB, not in-process.
- A crash in one worker doesn't kill the others; the primary can respawn it.
- In production, a process manager like **PM2** does this for you (`pm2 start app.js -i max`) with monitoring, zero-downtime reloads, and log management.

### Worker Threads — CPU-bound work, shared memory

`cluster` is for scaling I/O-bound servers across cores. **Worker Threads** (`worker_threads`) are for **CPU-bound tasks** (image processing, encryption, heavy computation) that would otherwise block the event loop.

```js
import { Worker } from 'worker_threads';
const worker = new Worker('./heavy.js', { workerData: input });
worker.on('message', result => { /* got it, main thread never blocked */ });
```

Differences from cluster:
- Threads share the **same process** and can share memory via **`SharedArrayBuffer`** (with `Atomics` for safe access) — far cheaper than IPC message copying.
- Communication via `postMessage` uses **structured clone** (copy) by default, but `ArrayBuffer`s can be **transferred** (ownership moves, zero-copy).
- Use a **worker pool** (e.g. `piscina`) rather than spawning a thread per task — thread creation has overhead.

**Rule of thumb:** I/O-bound → single process (or cluster for cores) is fine; CPU-bound → worker threads.

### `child_process` — spawning external programs

`spawn` (streams, for long-running/large output), `exec` (buffers output, convenient for short commands — **shell injection risk** with user input), `fork` (spawn another Node process with an IPC channel).

### Horizontal scaling & statelessness

To scale across *machines*, make your app **stateless**: no in-memory sessions, no local file uploads. Externalize state:
- Sessions → **Redis**.
- Uploads → **object storage** (S3).
- Cache → **Redis/Memcached**.
- Pub/sub across instances → Redis/Kafka.

Then put a **load balancer** (nginx, HAProxy, cloud LB) in front. Sticky sessions become unnecessary when state is external.

### Graceful shutdown

When scaling and deploying, processes get killed (SIGTERM). Handle it: stop accepting new connections, finish in-flight requests, close DB pools, then exit. Otherwise you drop requests on every deploy.

```js
process.on('SIGTERM', async () => {
  server.close(() => {});      // stop accepting new connections
  await db.end();              // drain resources
  process.exit(0);
});
```

---

## 8. Performance & Optimization

### Measure first

Never optimize blind. Tools:
- **`--prof`** / **`--cpu-prof`** — V8 CPU profiling → flame graphs.
- **`clinic.js`** (`doctor`, `flame`, `bubbleprof`) — diagnoses event-loop delays, I/O, CPU.
- **`--inspect`** + Chrome DevTools — live profiling and heap snapshots.
- **`perf_hooks`** — `performance.now()`, `PerformanceObserver`, and **event loop lag** monitoring (`monitorEventLoopDelay`).
- **autocannon** / **k6** — load testing.

### The number-one rule: don't block the event loop

Event loop lag is the primary Node performance killer. Symptoms: latency spikes, timeouts under load. Causes: sync I/O, big JSON, heavy crypto, ReDoS, huge synchronous loops. Fixes: async APIs, streaming, worker threads, chunking work with `setImmediate`.

Monitor it:

```js
import { monitorEventLoopDelay } from 'perf_hooks';
const h = monitorEventLoopDelay(); h.enable();
// later: h.mean, h.max, h.percentile(99)  — in nanoseconds
```

### V8 and the JIT

Node runs on **V8**, which JIT-compiles hot functions. To keep code fast:
- Keep object shapes ("hidden classes") **monomorphic** — initialize all properties in the constructor, in the same order; don't add/delete properties later. Polymorphic shapes deoptimize.
- Prefer arrays of one type; avoid sparse/holey arrays.
- Keep hot-path functions small and simple so V8 can inline and optimize them.

### Memory management & GC

V8 uses a **generational garbage collector**: short-lived objects live in the "young generation" (cheap, frequent scavenges), and survivors are promoted to the "old generation" (expensive mark-sweep-compact). Implications:
- Allocating lots of short-lived objects is fine; V8 is optimized for it.
- **Memory leaks** are the real danger: growing global arrays/maps, unremoved event listeners, closures capturing large scopes, unbounded caches. Symptom: old-generation heap grows without bound → eventually GC thrashing or OOM crash.
- Diagnose with **heap snapshots** (compare over time to find retained objects) and `--max-old-space-size` to control the heap cap.

Common leak: adding listeners without removing them. Node warns at 10+ listeners on an emitter (`MaxListenersExceededWarning`) — usually a real bug.

### Caching

- **In-process cache** — fastest, but per-process (doesn't scale across the cluster) and counts against heap. Bound it (LRU).
- **Distributed cache (Redis)** — shared across instances, survives restarts.
- **HTTP caching** — `Cache-Control`, `ETag`, CDN. Cache at the edge whenever possible.
- **Memoization** for pure expensive functions.

### Concrete wins

- **Connection pooling** for DBs — never open a connection per request.
- **Keep-Alive** for outbound HTTP (§4).
- **Stream** large payloads instead of buffering.
- **Compress** responses (gzip/brotli) — but cache the compressed output.
- **Avoid `JSON.parse`/`stringify` on huge objects** on the hot path; consider streaming JSON parsers or a faster serializer (Fastify's compiled serializers).
- **Batch** DB queries; avoid **N+1** query patterns (use `DataLoader` for GraphQL).
- **Index** your database — most "Node is slow" is actually a slow query.
- **Pin CPU work to worker threads.**

### The libuv thread pool

`fs`, DNS (`getaddrinfo`), and some crypto run on libuv's thread pool, default size **4** (`UV_THREADPOOL_SIZE`). Heavy concurrent file/crypto/DNS work can saturate it, causing queuing even though the event loop is idle. Increase it (up to 1024) for such workloads — but measure.

---

## 9. Error Handling

### Two categories of error

1. **Operational errors** — expected runtime failures: file not found, network timeout, invalid input, DB down. These are *not bugs*; handle them gracefully and recover.
2. **Programmer errors** — bugs: `undefined is not a function`, passing wrong types, unhandled edge cases. You **cannot** recover from these reliably; the correct response is often to log, alert, and **crash/restart** (let a process manager bring you back to a clean state).

Conflating the two — e.g. try/catching a bug and continuing — leaves the process in a corrupt, unpredictable state.

### Error propagation across async boundaries

- **Synchronous / async-await:** `try/catch` works.
- **Callbacks:** error-first convention (`(err, data) => ...`) — you must check `err` every time.
- **Promises:** `.catch()` or `try/catch` around `await`.
- **Event emitters:** errors surface as an `'error'` event. **An `'error'` event with no listener throws and crashes the process** — always attach one (e.g. on streams, sockets, servers).

```js
stream.on('error', err => cleanup(err));   // or the FD/socket leaks and process may crash
```

### The two process-level safety nets

```js
process.on('unhandledRejection', (reason) => {
  log.error(reason);
  // A promise rejected with no .catch. Log, then usually exit for a clean restart.
});

process.on('uncaughtException', (err) => {
  log.error(err);
  // The event loop reached an error nothing caught. State is unknown.
  process.exit(1);   // DO exit — do not resume as if nothing happened
});
```

These are **last resorts for logging and graceful shutdown**, not for keeping a broken process alive. After an `uncaughtException`, the process is in an undefined state; the only safe move is to shut down cleanly and let your supervisor (PM2/systemd/Kubernetes) restart a fresh instance.

### Custom error classes

Model your domain errors so callers can branch on type, and so you carry HTTP status / error codes:

```js
class AppError extends Error {
  constructor(message, { status = 500, code, isOperational = true } = {}) {
    super(message);
    this.name = this.constructor.name;
    this.status = status;
    this.code = code;
    this.isOperational = isOperational;
    Error.captureStackTrace(this, this.constructor);
  }
}
class NotFoundError extends AppError {
  constructor(msg = 'Not found') { super(msg, { status: 404, code: 'NOT_FOUND' }); }
}
```

The `isOperational` flag lets your global handler decide: operational → respond with a clean error; non-operational → log and restart.

### Centralized handling

Don't scatter ad-hoc error responses. Funnel everything to one error-handling layer (Express error middleware, a wrapper, or a domain error handler) that decides logging, response shape, and severity. Return **consistent, safe** error responses to clients — never leak stack traces or internal details in production.

### Async error patterns

- Wrap async route handlers so rejections reach your handler (Express 4; automatic in Express 5).
- Use `Promise.allSettled` when partial failure is acceptable.
- Add **timeouts** to every external call so a hang becomes a catchable error.
- Preserve context: log with request IDs (correlation IDs) so you can trace an error across services. `AsyncLocalStorage` (§11) is ideal for propagating a request ID without threading it through every function.

### The `cause` chain and `AggregateError`

Modern JS lets you preserve the original error:

```js
throw new Error('Failed to load user', { cause: dbError });   // ES2022
```

`AggregateError` (from `Promise.any`) bundles multiple errors.

---

## 10. Architecture & Design Patterns

### Layered / hexagonal architecture

Separate concerns into layers so business logic doesn't depend on frameworks or databases:

```
Routes/Controllers   ← HTTP concerns (parse, validate, respond)
      │
Services / Use-cases ← business logic (framework-agnostic, testable)
      │
Repositories / DAL   ← data access (hides DB details behind an interface)
      │
Models / Entities    ← domain data
```

The goal (**dependency inversion**): high-level business logic depends on *abstractions*, not on Express or Postgres. You can swap the web framework or database without touching the core. **Hexagonal / ports-and-adapters** formalizes this — the domain sits in the center, adapters plug in at the edges.

### Core Node patterns

- **Module pattern** — each file is a module with explicit exports; encapsulation via closures. ESM (`import`/`export`) is now standard over CommonJS (`require`).
- **Singleton** — Node module caching means a module's exports are shared across all importers, giving you a de facto singleton (e.g. a DB connection pool). Beware: this makes global state; keep it intentional.
- **Factory** — functions that create configured objects, hiding construction details.
- **Dependency Injection** — pass dependencies in rather than importing them directly inside a module. Makes code testable (inject mocks) and decoupled. NestJS builds this in; elsewhere you can do it manually or with a container (awilix, tsyringe).
- **Observer / EventEmitter** — Node's `EventEmitter` is the built-in pub/sub. Decouples producers from consumers. Foundation of streams, HTTP servers, etc.
- **Middleware / chain of responsibility** — the Express model (§5); also applies to generic pipelines.
- **Repository** — abstract data access behind a collection-like interface.

### EventEmitter — the backbone

```js
import { EventEmitter } from 'events';
class Orders extends EventEmitter {}
const orders = new Orders();
orders.on('created', order => sendEmail(order));   // decoupled consumer
orders.emit('created', newOrder);
```

Caveats: emitters are **synchronous** by default (listeners run in order, on the same tick); a throwing listener can disrupt others. Remove listeners to avoid leaks. Handle `'error'`.

### Application architectures

- **Monolith** — one deployable. Simplest to build/deploy/debug; the right default for most projects. A modular monolith keeps clean internal boundaries.
- **Microservices** — independently deployable services communicating over the network (REST/gRPC/message queue). Enables independent scaling and team autonomy, but adds distributed-systems complexity: network failures, eventual consistency, distributed tracing, deployment overhead. Don't start here without a reason.
- **Event-driven / message queues** — services communicate asynchronously via **Kafka/RabbitMQ/SQS**. Decouples producers and consumers, smooths load spikes, enables retries and replay. Patterns: pub/sub, work queues, event sourcing, CQRS, saga (for distributed transactions).
- **Serverless** — functions (AWS Lambda) that scale to zero. Great for spiky/irregular loads; watch cold starts and statelessness.

### Cross-cutting concerns

- **Configuration** — via environment variables (12-factor), validated at startup. Never hardcode.
- **Logging** — structured JSON logs (**pino**, winston), with levels and correlation IDs. Log to stdout; let the platform aggregate.
- **Observability** — the three pillars: **logs, metrics, traces**. OpenTelemetry is the standard for distributed tracing.
- **Validation at boundaries** — validate all external input (HTTP, queue messages) with schemas.
- **Graceful degradation** — circuit breakers (open the circuit after repeated failures to a dependency), retries with **exponential backoff + jitter**, bulkheads, timeouts.

### Project structure

Organize by **feature/domain**, not by technical type, as the app grows:

```
src/
  users/       users.routes.js  users.service.js  users.repo.js
  orders/      orders.routes.js  ...
  shared/      db.js  logger.js  errors.js
```

This keeps related code together and scales better than a top-level `controllers/ services/ models/` split for large apps.

---

## 11. Modern Node.js Features

Node evolves fast. Recent releases (18 → 20 → 22 → 24) have absorbed many things that previously required libraries. Highlights:

### ES Modules are first-class

`import`/`export` work natively. Set `"type": "module"` in `package.json` or use `.mjs`. Key differences from CommonJS:
- No `__dirname`/`__filename` — use `import.meta.dirname` / `import.meta.url` (`dirname` added in Node 20.11+).
- **Top-level `await`** — you can `await` at the top of a module.
- ESM is asynchronously loaded and statically analyzable (enables tree-shaking).
- Interop: ESM can `import` CommonJS; recent Node (22+) can even **`require()` an ESM module** synchronously in many cases, easing migration.

### Built-in `fetch`, `Request`, `Response`, `Headers`, `FormData` (18+)

Web-standard HTTP client globally available, on undici. Also a **`WebSocket`** client is now built in. Less need for `node-fetch`, `axios`, `ws` for clients.

### Built-in test runner (`node:test`) + `--test`

No Jest/Mocha needed for many projects:

```js
import { test } from 'node:test';
import assert from 'node:assert';
test('adds', () => assert.strictEqual(1 + 1, 2));
```

Run with `node --test`. Includes mocking, coverage, watch mode, and TAP/spec reporters.

### Watch mode — `node --watch`

Built-in restart-on-change; replaces `nodemon` for basic use. `--env-file=.env` loads environment variables natively (replaces `dotenv` for simple cases).

### Native TypeScript execution

Recent Node can **run `.ts` files directly** via type-stripping (`--experimental-strip-types`, increasingly on by default in Node 23/24). It strips types (no type-checking, no transpiling of TS-only syntax like enums by default) so you often no longer need `ts-node`/`tsx` just to execute TypeScript. You still run `tsc` separately for type *checking*.

### `AsyncLocalStorage` (async context)

Solves "how do I carry a request ID / user context through a deep async call chain without passing it as a parameter everywhere?" It provides per-async-execution-context storage — the async analog of thread-local storage.

```js
import { AsyncLocalStorage } from 'async_hooks';
const context = new AsyncLocalStorage();

app.use((req, res, next) => {
  context.run({ requestId: crypto.randomUUID() }, next);
});
// anywhere deep in the call stack:
const { requestId } = context.getStore();   // no parameter threading
```

Foundational for logging, tracing, and multi-tenant context.

### Web-standard APIs on the global

- **`AbortController` / `AbortSignal`** — standard cancellation for fetch, timers, streams, and your own async ops. `AbortSignal.timeout(ms)` and `AbortSignal.any([...])`.
- **Web Streams API** (`ReadableStream`/`WritableStream`/`TransformStream`) alongside Node streams, with interop (`Readable.fromWeb`/`toWeb`).
- **`structuredClone()`** — deep clone globally.
- **`crypto.randomUUID()`**, Web Crypto (`crypto.subtle`).
- **`URL`/`URLPattern`**, `TextEncoder`/`TextDecoder`.

### Permission Model (experimental)

`--permission` lets you restrict a process's access to the filesystem, child processes, and worker threads — a capability model (`--allow-fs-read`, `--allow-fs-write`, `--allow-child-process`). Useful for running untrusted code with reduced privilege.

### Single Executable Applications (SEA)

Bundle your app + the Node runtime into **one distributable binary**, so users don't need Node installed. Still maturing but usable for CLIs.

### `node:` protocol imports

Import built-ins with an explicit prefix — `import fs from 'node:fs'` — which is unambiguous (can't be shadowed by an npm package named `fs`) and required for some newer built-in modules. Prefer it in new code.

### Diagnostics & observability improvements

- **`--cpu-prof` / `--heap-prof`** flags produce profiles without extra tooling.
- **Diagnostics Channel** (`diagnostics_channel`) — a built-in pub/sub for instrumentation, used by APM tools.
- **`node --inspect`** and heap snapshot APIs (`v8.writeHeapSnapshot`).
- Improved source-map support (`--enable-source-maps`) for accurate stack traces from transpiled/TS code.

### New JS language features shipping with newer V8

Top-level await, `Array.fromAsync`, `Object.groupBy`, the `using` / `await using` declarations for explicit resource management (`Symbol.dispose`/`Symbol.asyncDispose`), `Promise.withResolvers`, plus faster startup and JIT (Sparkplug/Maglev tiers) and GC improvements.

---

## Quick reference: the mental models that unlock everything

1. **One thread runs your JS; libuv does I/O off-thread and queues callbacks back.** Never block that thread.
2. **The event loop has phases; microtasks (nextTick > promises) drain between every callback.** This explains all ordering questions.
3. **Streams + backpressure = bounded memory.** Use `pipeline`, not `pipe`.
4. **Scale I/O with cluster/processes; scale CPU with worker threads. Keep state external to stay stateless.**
5. **Distinguish operational errors (handle) from bugs (crash & restart).** Never resume after `uncaughtException`.
6. **Validate at the boundary, hash passwords slowly, sign/verify tokens carefully, secure your cookies, time out every network call.**
7. **Measure before optimizing; event-loop lag is usually the culprit.**
8. **Depend on abstractions, not frameworks/DBs, so the core stays testable and portable.**

---
---

# Part II — Interview Mastery

> Part I teaches the concepts. Part II teaches you to **perform** them under interview pressure. The gap between a Senior and a Staff/Architect offer is rarely *knowledge* — it's the ability to answer the follow-up, reason about failure modes out loud, and defend trade-offs. Each question below is tagged by the level it screens for, with a **model answer**, the **follow-up trap** interviewers spring, and the **weak-vs-strong** tell.

## How to use this section

- **Answer out loud before reading the model answer.** Reading feels like learning; it isn't. Recall is.
- **Interviewers score the follow-up, not the first answer.** Anyone can recite "the event loop has phases." The signal is whether you can explain *why `setImmediate` beats `setTimeout` inside an I/O callback but not at the top level.*
- **Say the trade-off unprompted.** Staff+ candidates volunteer "here's what this costs" before being asked. That's the single strongest tell.
- **Structure every answer:** (1) direct answer in one sentence, (2) the mechanism/why, (3) the trade-off or failure mode. Rambling reads as uncertainty.

---

## A. Interview Q&A by Topic

### A1. Async Programming

**Q1 (Senior). "Walk me through what actually happens when you `await` a promise."**

*Model answer:* `await` suspends the async function and registers a continuation as a **microtask** on the promise. Control returns to the caller immediately — the thread is *not* blocked. When the promise settles, its continuation is queued on the microtask queue and runs after the current synchronous stack unwinds and before the event loop advances to the next phase. The async function resumes exactly where it left off, with the resolved value or a thrown rejection.

*Follow-up trap:* "So does `await` add latency even if the promise is already resolved?"
→ **Yes.** `await Promise.resolve(5)` still defers the continuation to a microtask — it does not run synchronously. This is by design (consistency: `.then` is always async). It's one microtask tick of latency. Strong candidates know this; it explains subtle ordering bugs.

*Weak vs strong:* Weak says "await pauses execution." Strong says "await pauses *this async function* by scheduling a microtask continuation — the thread keeps running other work."

---

**Q2 (Senior/Staff). "You have 10,000 URLs to fetch. `await Promise.all(urls.map(fetch))` — what's wrong?"**

*Model answer:* It opens 10,000 concurrent connections at once. That will (a) exhaust file descriptors / ephemeral ports on your box, (b) hammer the downstream service into rate-limiting or collapse, and (c) if any single fetch rejects, `Promise.all` rejects immediately while the other 9,999 keep running in the background wasting resources. The fix is **bounded concurrency** — a pool of N in-flight (via `p-limit`, a semaphore, or manual batching) — plus `allSettled` if partial failure is acceptable.

*Follow-up trap:* "Implement the concurrency limiter without a library."
→ You should be able to write this on a whiteboard:

```js
async function mapWithConcurrency(items, limit, fn) {
  const results = new Array(items.length);
  let i = 0;
  async function worker() {
    while (i < items.length) {
      const idx = i++;                 // claim an index atomically (single-threaded, safe)
      results[idx] = await fn(items[idx], idx);
    }
  }
  await Promise.all(Array.from({ length: limit }, worker));
  return results;
}
```

*Why it works:* N "worker" loops run concurrently, each pulling the next index. Because JS is single-threaded, `i++` needs no lock. This is the pattern to reach for; know it cold.

---

**Q3 (Staff). "`Promise.all` vs `allSettled` vs `race` vs `any` — give me a real production scenario for each."**

| Combinator | Scenario | Failure semantics |
|---|---|---|
| `all` | Fan-out to services you *all* need (user + prefs + billing to render a page) | Fail fast — one down = whole request fails, correct here |
| `allSettled` | Sending 1,000 notifications; some may bounce | Never rejects; you report per-item success/failure |
| `race` | Timeout: `race([work(), timeout(5000)])` | First to settle wins (including rejection) |
| `any` | Query 3 replica regions, take the fastest healthy one | Rejects only if *all* fail (`AggregateError`) |

*Follow-up trap:* "With `race` for a timeout, what leaks?"
→ The losing promise **keeps running** — `race` doesn't cancel it. The `work()` call still holds its socket/DB connection until it finishes. For true cancellation you need `AbortController` wired into the actual operation, not just `race`. This distinction (settling vs cancelling) is a classic Staff-level probe.

---

### A2. Event Loop & Timing

**Q4 (Senior). "Predict the output. Explain each line."**

```js
async function f() {
  console.log('A');
  await null;
  console.log('B');
}
console.log('C');
f();
console.log('D');
Promise.resolve().then(() => console.log('E'));
```

*Answer:* `C, A, D, B, E`.
- `C` — sync.
- `f()` runs synchronously until the `await` → prints `A`, then `await null` suspends f (schedules continuation as microtask #1).
- `D` — sync, after `f()` returned control.
- Sync stack done. Drain microtasks: continuation of `f` (`B`) was queued first, then `E`. → `B`, `E`.

*Follow-up trap:* "What if I change `await null` to `await Promise.resolve()`?" → Same output here, but a bare `await promise` can add an *extra* microtask tick vs `await null`/`await value` in some engine versions — historically this was a real ordering difference (the "await optimization" in V8). Knowing the history signals depth; for current V8 they're equivalent.

---

**Q5 (Staff). "Why does `setImmediate` fire before `setTimeout(fn, 0)` inside an I/O callback, but the order is non-deterministic at the top level?"**

*Model answer:* At the **top level**, the order depends on how long process startup takes to reach the first event-loop iteration. If ≥1ms has elapsed, the timer is "ready" and the timers phase (which comes first) runs it before check. If <1ms, the timer isn't ready yet, so check (`setImmediate`) runs first. It's a race against the 1ms timer floor — hence non-deterministic.

Inside an **I/O callback**, you're already in the **poll phase**. The very next phase is **check** (`setImmediate`), which runs *this same iteration*. The timers phase is at the *top* of the loop — it won't come around again until the *next* iteration. So `setImmediate` deterministically wins.

*Follow-up trap:* "Can `process.nextTick` starve `setImmediate`?" → **Yes.** The nextTick queue drains completely between phases. A `nextTick` that recursively schedules another `nextTick` never yields to the loop — timers, I/O, and immediates all starve. `setImmediate` recursion does *not* starve, because it yields one iteration each time. This is why the guidance is "prefer `setImmediate` for deferring work; reserve `nextTick` for edge cases like emitting an event after the constructor returns."

---

**Q6 (Staff/Architect). "Your p99 latency spikes to 800ms under load but CPU sits at 40%. Diagnose."**

*Model answer:* Classic **event-loop lag with an idle-looking CPU**. Something is intermittently blocking the single JS thread — sync work between async operations. The 40% average hides bursts of 100% for tens of ms. Candidates:
- A large synchronous `JSON.parse`/`stringify` on a hot path.
- Synchronous crypto (`pbkdf2Sync`, sync bcrypt).
- A ReDoS-prone regex on user input.
- libuv **thread-pool saturation** (4 threads) from heavy `fs`/DNS/crypto — the loop is idle but callbacks queue behind a full pool. CPU looks low because threads are *waiting*, not computing.

*How I'd confirm:* `monitorEventLoopDelay()` (p99 in ns), `clinic doctor`, and check `UV_THREADPOOL_SIZE` utilization. Fix depends on cause: move CPU work to worker threads, chunk with `setImmediate`, raise the pool size for I/O-heavy pools, or cache the expensive computation.

*Weak vs strong:* Weak jumps to "add more instances." Strong distinguishes CPU-bound blocking from thread-pool saturation from GC pauses — three different causes with the same symptom, each with a different fix.

---

### A3. Streams & Buffers

**Q7 (Senior). "What is backpressure and what breaks without it?"**

*Model answer:* Backpressure is the flow-control signal from a slow consumer to a fast producer. `writable.write()` returns `false` when the internal buffer exceeds `highWaterMark`; the producer should pause until `'drain'`. Without it, a fast source (reading a file, receiving a request) outruns a slow sink (slow disk, slow client), and unbounded data queues **in memory** — the heap grows until GC thrashes and the process OOM-crashes. `.pipe()` and `pipeline()` implement this automatically.

*Follow-up trap:* "A user reports the server OOMs when clients on slow mobile connections download large files. Why, and the fix?"
→ The response Writable (the socket) drains slowly for slow clients, but if you're pushing data with `res.write()` in a loop without honoring the `false` return, or reading the whole file into memory first, buffered data piles up per-connection. Multiply by many slow clients = OOM. Fix: `pipeline(fs.createReadStream(file), res)` — backpressure ties read speed to each client's download speed. This is a *very* common real incident.

---

**Q8 (Staff). "Why prefer `pipeline` over `pipe`? Show the bug `pipe` has."**

*Model answer:* `.pipe()` does not forward errors and does not clean up on failure. If the source stream errors mid-transfer, the destination is left **open** — leaking a file descriptor / socket. Over time you exhaust FDs and the process can't accept connections.

```js
// BUG: source error leaves writable open, FD leaks
readable.pipe(transform).pipe(writable);
readable.on('error', ...);  // even if you catch here, writable isn't destroyed

// CORRECT: pipeline destroys ALL streams on any error, and gives one error callback
import { pipeline } from 'stream/promises';
try {
  await pipeline(readable, transform, writable);
} catch (err) {
  // every stream already destroyed; FDs released
}
```

*Follow-up trap:* "Write a Transform that uppercases text but handles multi-byte UTF-8 split across chunks." → This tests whether you know chunk boundaries can split a character:

```js
import { Transform } from 'stream';
import { StringDecoder } from 'string_decoder';

const upper = new Transform({
  construct(cb) { this.decoder = new StringDecoder('utf8'); cb(); },
  transform(chunk, _enc, cb) {
    cb(null, this.decoder.write(chunk).toUpperCase());
  },
  flush(cb) { cb(null, this.decoder.end().toUpperCase()); },
});
```

The `StringDecoder` buffers an incomplete trailing byte sequence until the next chunk completes it. Naive `chunk.toString()` would corrupt multi-byte chars at boundaries.

---

### A4. HTTP & Networking

**Q9 (Senior). "`fetch` returned a 404 but didn't throw. Explain, and what else bites people."**

*Model answer:* `fetch` only rejects on *network-level* failure (DNS, connection refused, timeout). An HTTP 4xx/5xx is a *successful* HTTP exchange, so the promise resolves — you must check `res.ok` / `res.status` yourself. Other gotchas: the body is a one-shot stream (`.json()` OR `.text()`, not both); there is **no default timeout**, so a hung server hangs you forever unless you pass `AbortSignal.timeout(ms)`; and response bodies must be consumed or explicitly cancelled or you leak the connection back to the undici pool.

*Follow-up trap:* "How does connection reuse work and why does it matter?" → undici keeps a keep-alive connection pool per origin. Reusing a warm connection skips the TCP + TLS handshake (multiple round trips). For a service making many calls to the same host, this is often a 2–5× latency win. On the *server*, tune `server.keepAliveTimeout` vs your load balancer's idle timeout — if the LB's is longer, you get sporadic 502s from races on connection close (a famous AWS ALB + Node gotcha).

---

**Q10 (Architect). "Design the timeout and retry strategy for a service that calls 3 downstream APIs."**

*Model answer:* Layered:
- **Per-call timeout** (`AbortSignal.timeout`) shorter than your own SLA — if your SLA is 1s and you call 3 in parallel, each gets ~800ms, not 1s.
- **Retries only on idempotent operations and transient errors** (5xx, timeouts, connection resets) — never blindly retry a non-idempotent POST.
- **Exponential backoff + jitter** to avoid retry storms / thundering herd when a dependency recovers.
- **Circuit breaker** per dependency — after N consecutive failures, open the circuit and fail fast for a cooldown, so you don't pile requests onto a dying service.
- **Budget the retries** — a retry multiplies load; cap total attempts and respect an overall deadline.

*Follow-up trap:* "Why jitter?" → Without jitter, all clients that failed at time T retry at exactly T+backoff, re-synchronizing the stampede. Jitter (randomize the delay) spreads them out. This is the AWS "exponential backoff and jitter" paper — name-dropping it signals seniority.

---

### A5. Express & Frameworks

**Q11 (Senior). "In Express 4, why don't errors thrown in an async route reach the error middleware, and how do you fix it?"**

*Model answer:* Express 4's router only catches *synchronous* throws. An `async` handler returns a rejected promise that Express 4 never awaits, so the rejection becomes an unhandled rejection and the request hangs until timeout. Fix: wrap handlers so rejections are forwarded to `next()`:

```js
const asyncHandler = fn => (req, res, next) => Promise.resolve(fn(req, res, next)).catch(next);
```

Express 5 fixes this natively (awaits handler promises).

*Follow-up trap:* "How does Express know a function is error-handling middleware?" → **Arity.** A middleware with exactly **four** parameters `(err, req, res, next)` is treated as an error handler. Omit the 4th and it's a normal middleware — a subtle bug: `(err, req, res) => …` is *not* an error handler. Must be registered last.

---

**Q12 (Staff). "Fastify claims to be faster than Express. Mechanically, why?"**

*Model answer:* Three main reasons: (1) It compiles your JSON **response schema** into a specialized serializer ahead of time, avoiding generic `JSON.stringify` reflection on every response. (2) It compiles **validation** schemas similarly at the boundary. (3) Its router and plugin encapsulation avoid the per-request middleware-array overhead Express incurs. The philosophy is "declare your shapes, pay the cost once at boot." The trade-off: more upfront structure, less of Express's anything-goes flexibility.

*Follow-up trap:* "Is the framework usually your bottleneck?" → Honest Staff answer: **rarely.** In real apps, DB queries, downstream calls, and serialization dominate; the framework's per-request overhead is microseconds. Choose Fastify for its schema/validation/plugin model and structure, not because you'll notice the raw routing speed. Volunteering this shows judgment over hype.

---

### A6. Auth & Security

**Q13 (Senior). "Sessions vs JWT — when do you pick which?"**

*Model answer:* **Sessions (stateful):** opaque ID in an httpOnly cookie, data in Redis/DB. Trivially revocable, small cookie, but needs a shared session store. Best for classic web apps and anywhere instant revocation matters (banking, admin). **JWT (stateless):** signed claims verified without a lookup — scales across services with no shared store, but **can't be revoked before expiry**. Best for short-lived access tokens in distributed/microservice systems. The mature answer is usually **both**: short-lived JWT access token + long-lived, revocable refresh token stored server-side.

*Follow-up trap:* "How do you revoke a JWT before it expires?" → You can't revoke the token itself; you add state back — a denylist of revoked token IDs (jti) in Redis checked on each request (partially defeats statelessness), or keep access tokens *very* short (minutes) so revoking the refresh token quickly cuts off access. There's no free lunch; naming the trade-off is the point.

---

**Q14 (Staff). "Name two JWT attacks and how you prevent them."**

*Model answer:*
1. **`alg: none`** — attacker sets the header algorithm to `none` and strips the signature; a naive verifier accepts an unsigned token. Prevent by **pinning the expected algorithm** on verify — never trust the token's own `alg` header.
2. **Algorithm confusion (RS256 → HS256)** — a service that verifies with RS256 (public key) is tricked into HS256, where the attacker uses the *public* key as the HMAC secret to forge a valid signature. Prevent by explicitly restricting `algorithms: ['RS256']` in the verify call.

Bonus: a JWT is **signed, not encrypted** — the payload is base64 and world-readable. Never put secrets in it. Also validate `exp`, `aud`, `iss`.

*Follow-up trap:* "Where do you store the JWT in a browser?" → Not `localStorage` (readable by any XSS). Prefer an **httpOnly, Secure, SameSite cookie** so JS can't exfiltrate it — but then you must add CSRF defense (SameSite + anti-CSRF token). There's a genuine XSS-vs-CSRF trade-off; a strong candidate names both sides.

---

**Q15 (Architect). "A password reset endpoint — walk me through securing it end to end."**

*Model answer:* Generate a **cryptographically random** token (`crypto.randomBytes`), store only its **hash** in the DB with a short expiry and single-use flag (so a DB leak doesn't expose live reset links). Email the raw token. On use: constant-time compare (`crypto.timingSafeEqual`), check expiry + unused, then rotate the password hash (argon2/bcrypt) and **invalidate existing sessions**. Rate-limit the request-reset endpoint per account and per IP. Return the *same* generic response whether or not the email exists (no account enumeration). Log the event for audit.

*Follow-up trap:* "Why hash the reset token if it's already random and short-lived?" → Defense in depth: if the DB is read (SQLi, backup leak), stored *raw* tokens are immediately usable to hijack accounts; stored *hashes* are not reversible. Same reasoning as password hashing.

---

### A7. Scaling & Concurrency

**Q16 (Senior). "cluster vs worker_threads — when each?"**

*Model answer:* **cluster** forks N *processes* sharing one listening socket to use all CPU cores for an **I/O-bound** server — separate memory, isolation, a crash takes down one worker not all. **worker_threads** run *inside one process*, sharing memory (`SharedArrayBuffer` + `Atomics`), for **CPU-bound** work (image resize, compression, crypto) that would otherwise block the event loop. Rule of thumb: scale I/O with processes/cluster; offload CPU with threads. In production, PM2/Kubernetes usually replaces manual `cluster`.

*Follow-up trap:* "You have a CPU-bound endpoint. Why not just add more cluster workers?" → You could, but each worker is a full process (~30–50MB+ V8 heap each) and blocking one still blocks every request routed to it until it finishes. A **worker-thread pool** (piscina) keeps the main event loop responsive per process and is lighter than N processes. Also, message passing to threads can transfer `ArrayBuffer` ownership (zero-copy) vs cluster's IPC serialization.

---

**Q17 (Architect). "How do you make a Node service horizontally scalable across many machines?"**

*Model answer:* Make it **stateless** — no in-memory sessions, no local file storage, no in-process caches that must be consistent. Externalize every piece of state: sessions and cache → Redis, uploads → object storage (S3), cross-instance events → Redis pub/sub or Kafka, coordination/locks → Redis/etcd. Then any instance can serve any request behind a load balancer; no sticky sessions needed. Add **graceful shutdown** (drain on SIGTERM) so rolling deploys and autoscaling don't drop requests, and health/readiness probes so the LB only routes to ready instances.

*Follow-up trap:* "You use WebSockets — that breaks statelessness. Now what?" → WebSocket connections are inherently stateful (a client is pinned to one instance for the connection's life). Handle it with: a shared pub/sub backplane (Redis adapter for socket.io) so a message published on instance A reaches a socket held on instance B, sticky routing at the LB for the initial upgrade, and connection-draining on shutdown that tells clients to reconnect (which re-balances them).

---

### A8. Performance

**Q18 (Staff). "How does V8 make your JS fast, and how do you accidentally deoptimize it?"**

*Model answer:* V8 runs a tiered pipeline: **Ignition** (bytecode interpreter) runs everything first, and hot functions get JIT-compiled to machine code by **Sparkplug** (fast baseline), then **Maglev** (mid-tier), then **TurboFan** (optimizing). Optimization relies on **hidden classes** (shapes) and inline caches: if objects at a call site always have the same shape (monomorphic), V8 inlines property access to a direct offset. You deoptimize by making call sites **polymorphic/megamorphic** — objects with different property sets or insertion orders — or by changing an object's shape after creation (adding/deleting properties), mixing types in an array, or triggering deopt bailouts (e.g. `arguments` leaks, `try/catch` in old versions). Fix: initialize all fields in the constructor in a consistent order; keep arrays monotyped and dense; keep hot functions small.

*Follow-up trap:* "Does that ever actually matter?" → Rarely on the app hot path vs DB/network, but *very* much in tight numeric loops, serializers, parsers, and libraries. Know it, but lead with "measure first — most 'Node is slow' is a missing DB index or a blocking call, not V8."

---

**Q19 (Staff/Architect). "Production memory grows steadily until OOM every ~6 hours. Find the leak."**

*Model answer:* Methodology:
1. Confirm it's the JS heap vs RSS (native/buffers): `process.memoryUsage()` — watch `heapUsed` vs `rss`. Growing `rss` but flat `heapUsed` points to Buffers/native (e.g. a stream not draining).
2. Take **heap snapshots** at intervals (`v8.writeHeapSnapshot()` or DevTools) and **diff** them — the retained objects that grow between snapshots are the leak.
3. Usual suspects: an ever-growing module-level `Map`/array (unbounded cache), event listeners added per request but never removed (watch for `MaxListenersExceededWarning`), closures capturing large scopes held by long-lived timers, or `AsyncLocalStorage`/context maps not cleared.
4. Fix and verify the sawtooth returns (heap drops after GC) instead of a monotonic climb.

*Follow-up trap:* "`--max-old-space-size` — does raising it fix a leak?" → No. It only delays the OOM (and lengthens GC pauses). It's a band-aid for buying time or right-sizing a legitimately large heap, never a fix for an actual leak.

---

### A9. Error Handling

**Q20 (Senior). "Operational vs programmer errors — why does the distinction matter operationally?"**

*Model answer:* **Operational errors** are expected failures of a correct program (network timeout, file missing, bad user input, DB down) — you handle and recover: retry, return a 4xx/5xx, degrade. **Programmer errors** are bugs (`undefined is not a function`, wrong type) — the process state is now unknown, so the *only* safe response is log + crash + restart clean via your supervisor. The danger is conflating them: swallowing a bug in a broad `try/catch` and continuing leaves corrupted state that causes weirder failures later. The `isOperational` flag on custom errors lets a central handler branch: operational → clean response; non-operational → alert and exit.

*Follow-up trap:* "So should you use `uncaughtException` to keep the server alive?" → No — that's the anti-pattern. After `uncaughtException` the state is undefined; use the handler only to **log and shut down gracefully**, then let PM2/Kubernetes restart a fresh process. Same for `unhandledRejection`. Trying to "resume" is how you get zombie processes serving corrupt data.

---

**Q21 (Staff). "An EventEmitter emits `'error'` with no listener. What happens and why is it special?"**

*Model answer:* Node **throws** the error and, if uncaught, **crashes the process**. `'error'` is the one event with this special behavior — it's deliberate: an unheard error usually means something is genuinely broken (a dead socket, a failed stream), and silently dropping it would hide serious faults. So you must attach `'error'` listeners on every stream, socket, and server. This is also why streams that error without a handler crash the app — a common production surprise.

*Follow-up trap:* "How does `AsyncLocalStorage` help error handling in a distributed system?" → It carries a **correlation/request ID** through the entire async call chain without threading it as a parameter, so every log line and error for one request shares an ID. Combined with distributed tracing (OpenTelemetry), you can follow one request's error across service boundaries. It's the async analog of thread-local storage.

---

### A10. Architecture & Patterns

**Q22 (Architect). "When would you NOT use microservices?"**

*Model answer:* Almost always at the start. Microservices trade in-process simplicity for distributed-systems complexity: network failures, partial failures, eventual consistency, distributed tracing, versioned contracts, orchestration, and per-service ops. You take that on when you have a *concrete* forcing function — independent scaling of a hot component, independent deploy cadence, team autonomy at scale, or fault isolation. Otherwise a **modular monolith** (clean internal boundaries, one deployable) gives you most of the organizational benefit without the distributed tax, and you can extract a service later once the seams are proven. "Don't distribute until you must" is the mature default.

*Follow-up trap:* "You split into services and now a user action must update three of them consistently. How?" → You can't have a distributed ACID transaction cheaply. Use a **saga** (a sequence of local transactions with compensating actions on failure) or the **outbox pattern** (write the state change and an event atomically in one DB transaction, publish the event reliably from the outbox). Accept **eventual consistency** and design idempotent consumers. This is the heart of distributed data management.

---

**Q23 (Staff). "Module-level singletons in Node — feature or footgun?"**

*Model answer:* Both. Because Node **caches modules** after first load, a module's exports are shared across all importers — a natural singleton for things like a DB pool or logger. That's good for shared resources. The footgun is that it's **global mutable state**: it makes testing harder (can't easily inject a mock), couples code to a concrete instance, and can cause surprises across worker threads (each thread has its own module cache — the singleton is *not* shared across threads). Prefer **dependency injection** for testability; use the module-singleton deliberately for genuinely process-wide resources.

*Follow-up trap:* "Is the singleton shared across cluster workers?" → No. Each cluster worker is a separate process with its own memory and module cache — so an in-memory singleton cache is *per worker*, not global. This is exactly why cross-instance state must live in Redis. Same for worker_threads (separate module registries).

---

### A11. Modern Node

**Q24 (Senior). "CommonJS vs ESM — the differences that actually bite in production."**

*Model answer:* CJS is **synchronous** (`require` blocks and returns immediately, resolved at runtime) and gives you `__dirname`/`module.exports`. ESM is **asynchronous and statically analyzable** (`import` is hoisted, enabling tree-shaking), supports **top-level await**, uses `import.meta.url`/`import.meta.dirname` instead of `__dirname`, and is strict-mode by default. Gotchas: you can't `require()` arbitrary ESM historically (though recent Node 22+ allows `require()` of ESM without top-level await); mixing the two causes "Cannot use import statement" / "require is not defined" errors; `"type": "module"` in package.json flips the default for `.js` files; and named imports from a CJS module rely on static analysis that sometimes fails, forcing default-import-then-destructure.

*Follow-up trap:* "Top-level await in ESM — any risk?" → It makes module *loading* asynchronous, so a slow top-level await delays everything importing that module — effectively a startup bottleneck if you `await` a slow network call at module scope. Keep top-level await for genuinely load-time-critical initialization.

---

## B. System Design Drills (Node-Specific)

> Format for each: **Requirements → Approach → Node specifics → Failure modes → Scaling.** In an interview, always start by clarifying requirements and stating assumptions — jumping to a solution is the #1 mistake.

### B1. Distributed Rate Limiter

**Clarify first:** Per-user or per-IP? Global limit or per-endpoint? Hard limit or smooth? Tolerable to occasionally allow a few over (soft) or must be exact (hard)? Single region or multi?

**Approach — Token Bucket in Redis** (allows bursts, smooth refill; the usual best default):
- Each key (`ratelimit:{userId}`) holds tokens + last-refill timestamp.
- On request: refill tokens based on elapsed time, if ≥1 token, decrement and allow; else reject with `429` + `Retry-After`.

**Node specifics — atomicity is the whole game.** Read-modify-write across the network races: two concurrent requests both read 1 token, both allow. Solve with a **Lua script** (Redis executes it atomically) or `INCR` + `EXPIRE` for the simpler fixed-window:

```js
// Sliding-window-ish counter, atomic via Lua
const LUA = `
  local current = redis.call('INCR', KEYS[1])
  if current == 1 then redis.call('PEXPIRE', KEYS[1], ARGV[1]) end
  return current`;
const count = await redis.eval(LUA, 1, `rl:${userId}`, windowMs);
if (count > limit) return res.status(429).set('Retry-After', ...).end();
```

**Algorithm trade-offs:**
- **Fixed window** — simplest, but allows 2× burst at the window boundary (100 at 0:59, 100 at 1:00).
- **Sliding window log** — exact, but stores every timestamp (memory-heavy).
- **Sliding window counter** — weighted blend of two windows; good accuracy/memory balance.
- **Token bucket** — allows controlled bursts, smooth; usually the best UX.

**Failure modes:** Redis down → decide **fail-open** (allow, prioritize availability) vs **fail-closed** (reject, prioritize protection) — a real decision, state it. Clock skew across app servers → use Redis server time in the Lua script, not app-server time. Hot key (one huge tenant) → shard the key or use a local pre-filter.

**Scaling:** A per-instance in-memory limiter is fast but each of N instances allows the full limit → effective limit ×N. For a global limit you need the shared Redis. Hybrid: coarse local limiter + periodic sync to reduce Redis load.

---

### B2. Background Job / Task Queue

**Clarify:** At-least-once or exactly-once? Ordering required? Retry policy? Delayed/scheduled jobs? Throughput? Job duration (seconds vs hours)?

**Approach:** Producers push jobs to a durable queue; a pool of workers pulls and processes. Use **BullMQ (Redis)** for most Node systems, or SQS/RabbitMQ/Kafka for larger scale.

**Node specifics:**
- Workers are separate processes (`worker_threads` only if CPU-bound *within* a job).
- Concurrency per worker is bounded (don't pull 1,000 jobs into one event loop).
- **Idempotency is mandatory** — at-least-once delivery means a job *will* occasionally run twice (worker crashes after doing the work but before ack). Make handlers idempotent via a dedup key / "already processed" check.

**Reliability pattern:**
- Job moves `waiting → active → completed/failed`. A **visibility timeout / lock** means if a worker dies mid-job, the job is re-queued after the lock expires (hence at-least-once).
- **Retries with exponential backoff + jitter**; after N failures → **dead-letter queue** for inspection, never silent drop.
- For scheduled/delayed jobs, use a delayed set (BullMQ) or the queue's native delay.

**Failure modes:** Poison message (always fails) endlessly retrying → cap retries + DLQ. Worker OOM on a huge job → cap payload, stream large data by reference (S3 pointer, not inline). Thundering herd when many delayed jobs fire at once → jitter the schedule.

**Exactly-once caveat:** True exactly-once *delivery* is effectively impossible in distributed systems; you achieve exactly-once *effect* via idempotent processing + dedup. Say this — interviewers probe for the myth.

---

### B3. Real-Time Notification / Feed Service

**Clarify:** Push (WebSocket/SSE) or poll? Delivery guarantee? Online-only or store-and-forward for offline users? Fan-out size (a celebrity with 10M followers)? Ordering?

**Approach:** WebSocket (bidirectional) or **SSE** (server→client only, simpler, auto-reconnect, works over plain HTTP) for delivery. A pub/sub backbone (Redis/Kafka) decouples event producers from the connection-holding gateway.

**Node specifics:**
- Node handles **many idle connections well** (event-driven, low per-connection cost) — good fit for WebSocket gateways.
- Connections are **stateful and pinned** to one instance → you need a **backplane**: publish an event on instance A, every instance's subscriber receives it and pushes to whichever local sockets belong to the target users (Redis pub/sub adapter).
- Track `userId → set of socket/instance` in Redis for targeted delivery.

**Fan-out models:**
- **Fan-out on write** (push to each follower's feed on post) — fast reads, expensive for celebrities (10M writes per post).
- **Fan-out on read** (build feed at read time) — cheap writes, expensive reads.
- **Hybrid** — fan-out on write for normal users, fan-out on read for celebrity accounts. This is the real-world answer (Twitter's approach).

**Failure modes:** Slow consumer / backpressure — a client that can't keep up must not balloon server memory; drop/coalesce or disconnect. Reconnection storms after a deploy → connection draining + jittered reconnect. Missed messages while offline → store-and-forward with a per-user inbox + "since" cursor on reconnect.

**Scaling:** Sticky LB routing for the upgrade, horizontal gateway instances behind the Redis backplane, and separate the *delivery* tier from the *business* tier.

---

### B4. URL Shortener (tests ID generation + caching + read scale)

**Clarify:** Traffic (reads ≫ writes)? Custom aliases? Analytics? Expiry? Latency target?

**Approach:** `POST /shorten` stores `{shortCode → longURL}`; `GET /:code` looks up and 301/302-redirects. It's a **read-heavy** system — optimize reads.

**ID generation (the interesting part):**
- **Auto-increment + base62 encode** — short codes, but sequential/guessable and couples to one DB.
- **Random base62** — unguessable, but needs a collision check (retry on conflict).
- **Hash of URL (truncated)** — dedups identical URLs but collisions need handling.
- **Distributed ID (Snowflake-style)** — 64-bit: timestamp + machine ID + sequence, generated without coordination across instances. Reach for this at scale.

**Node specifics:** The redirect path must be sub-millisecond → **cache-aside with Redis** in front of the DB; the working set of hot URLs lives in cache. Optionally an in-process LRU as L1 in front of Redis (L2). Use `301` (permanent, cacheable by browsers/CDN) vs `302` (if you need analytics on every hit).

**Failure modes:** Cache stampede on a viral link expiring simultaneously across instances → use a lock / `SETNX` so one instance repopulates while others wait, or probabilistic early expiry. Hot key → the CDN/browser cache absorbs most of it for 301s.

**Scaling:** CDN in front, read replicas for the DB, Redis for hot lookups, and because reads dominate, this scales almost linearly with caching.

---

### B5. File Upload Service (tests streams + backpressure + storage)

**Clarify:** Max size? Direct-to-storage or through the app? Resumable? Virus scan / processing? Auth?

**Approach:** For large files, **don't buffer in the app** — either (a) **stream** through the app to storage with `pipeline`, or (b) far better at scale, issue a **pre-signed S3 URL** so the client uploads *directly* to object storage, bypassing your Node process entirely.

**Node specifics:** If streaming through the app, `pipeline(req, transform?, s3UploadStream)` ties the client's upload speed to the storage write speed via backpressure — bounded memory regardless of file size. Cap the size (`Content-Length` check + hard byte limit) to prevent DoS. Never `req.on('data', d => buf += d)` for files.

**Failure modes:** Unbounded body = memory-exhaustion DoS → enforce limits at the parser and reverse proxy. Multi-byte/multipart parsing edge cases → use a battle-tested parser (busboy). Partial upload on disconnect → resumable uploads (S3 multipart) or cleanup of orphaned parts.

**Scaling:** Pre-signed direct-to-S3 removes your app from the data path entirely (your Node process only signs URLs — trivial to scale). Post-process (thumbnails, scanning) asynchronously via the job queue (B2), triggered by an S3 event.

---

### B6. Caching Layer (the cross-cutting design)

**Patterns:** **Cache-aside** (app checks cache, on miss reads DB and populates — most common), **read-through/write-through** (cache sits inline), **write-behind** (async flush, risks loss on crash).

**The hard problems (name these unprompted):**
- **Invalidation** — "one of the two hard things." TTL is the pragmatic default; event-based invalidation is precise but complex.
- **Stampede / thundering herd** — a hot key expires and 10,000 requests hit the DB simultaneously. Fixes: a mutex/lock so one request repopulates, probabilistic early recomputation, or serving stale-while-revalidate.
- **Consistency** — cache and DB diverge on concurrent writes. Prefer *invalidate* over *update* on write; accept a small staleness window.
- **Hot keys** — one key overwhelms one Redis shard → local L1 cache in front, or key replication.

**Node specifics:** Multi-tier — in-process LRU (L1, nanoseconds, per-instance, bounded to protect heap) + Redis (L2, shared). Remember L1 is per-process → not consistent across instances; only cache immutable or staleness-tolerant data there.

---

## C. Deeper Internals

### C1. V8 Execution Pipeline

V8 is a **multi-tier JIT**, not a simple interpreter:

1. **Parser** → AST → **Ignition** compiles to **bytecode** (compact, quick to produce). All code starts here.
2. Ignition collects **type feedback** (what shapes/types flow through each operation) via inline caches.
3. Hot functions tier up:
   - **Sparkplug** — a fast, non-optimizing *baseline* compiler: bytecode → machine code with almost no analysis. Cheap to produce, ~effortless speedup.
   - **Maglev** — a mid-tier optimizing compiler (newer): uses type feedback for solid optimization at moderate compile cost. Fills the gap between Sparkplug and TurboFan.
   - **TurboFan** — the heavy optimizing compiler: aggressive inlining, escape analysis, using the collected type feedback to make **speculative optimizations**.
4. **Deoptimization**: TurboFan's optimizations are *speculative* — "this call site always saw shape X." If a new shape appears, the assumption is invalidated and V8 **bails out** back to bytecode (deopt), which is expensive if it happens repeatedly ("deopt loop").

**Interview-relevant implication:** keep types/shapes stable so speculation holds. Monomorphic code stays in TurboFan; megamorphic code gets deoptimized or never optimized.

### C2. Hidden Classes (Shapes) & Inline Caches

V8 doesn't use dictionaries for object properties (too slow). It assigns each object a **hidden class** describing its layout (property → memory offset). Objects created the same way share a hidden class.

```js
function Point(x, y) { this.x = x; this.y = y; }   // all Points share one hidden class
const a = new Point(1, 2);
const b = new Point(3, 4);   // same shape → fast

a.z = 5;   // a TRANSITIONS to a new hidden class; a and b now differ → slower
```

**Inline caches (ICs):** at a property-access site (`obj.x`), V8 caches "for hidden class H, `x` is at offset 2." Next time, if `obj` has class H, it's a direct memory read (monomorphic IC — fast). If the site sees 2–4 shapes it's polymorphic; more, **megamorphic** — the IC gives up and does a slow generic lookup.

**Rules that follow:** initialize all properties in the constructor, same order, every time; don't add/delete properties after creation; don't use objects as random-keyed maps (use `Map`); keep arrays monotyped (`PACKED_SMI`/`PACKED_DOUBLE` are fast; mixing or creating holes → `HOLEY` elements kind, slower).

### C3. Garbage Collection Internals

V8's heap is **generational**, exploiting "most objects die young":

- **Young generation (nursery)** — small, split into two semi-spaces. Collected by **Scavenge** (Cheney's copying algorithm): live objects are copied to the other semi-space, dead ones abandoned wholesale. Fast, frequent, pauses are short. Survivors of two scavenges get **promoted** to old space.
- **Old generation** — larger, collected by **Mark-Sweep-Compact**: mark reachable objects, sweep the dead, compact to reduce fragmentation. More expensive.

**Pause reduction techniques:** V8 does much GC work **concurrently** (on background threads) and **incrementally** (in small steps interleaved with JS) so stop-the-world pauses shrink. **Orinoco** is the umbrella project name for these parallel/concurrent/incremental improvements.

**Operational implications:**
- Allocating many short-lived objects is *cheap* (scavenge is fast) — don't contort code to avoid all allocation.
- The killer is **promoting garbage to old space** — objects that live just long enough to be promoted but are actually leaks. Old-space GC is what causes noticeable pauses and, when it can't reclaim, OOM.
- `--max-old-space-size=N` (MB) caps old space. Default is ~2GB+ on 64-bit modern Node; raise for legitimately large heaps, but a rising old-space + rising GC frequency = leak, not undersizing.
- Watch **GC pause** as a latency source: long p99s can be GC stop-the-world, distinct from event-loop blocking and thread-pool saturation.

### C4. libuv & the Event Loop at the Source Level

libuv is the C library that *is* the event loop. Key structures:

- **Handles** (long-lived: a TCP server, a timer) and **requests** (short-lived: a single write, a getaddrinfo).
- The loop runs the phases (§2). The **poll** phase is where it blocks on the OS multiplexer:
  - Linux: **epoll**, macOS/BSD: **kqueue**, Windows: **IOCP**, Solaris: event ports.
- **Network I/O is kernel-async** — the socket is registered with epoll/kqueue and the kernel notifies libuv when readable/writable. **No thread-pool thread is used** for network sockets.
- **The thread pool (default 4)** handles operations with *no* portable async kernel API: **file system** ops (there's no reliable async file I/O across all OSes), **DNS `getaddrinfo`** (`dns.lookup`), `dns.lookupService`, and CPU-bound crypto (`pbkdf2`, `scrypt`, `randomBytes` for large sizes) and `zlib`.

**The distinction that trips people:** `dns.lookup` (used implicitly by `http`/`net` when you connect by hostname) uses the **thread pool** → 4 concurrent slow DNS resolutions can block file reads and each other. `dns.resolve*` uses the async network resolver (c-ares) and does **not** use the pool. Under high connection churn to hostnames, DNS can silently saturate the pool — a subtle production bug. Mitigations: raise `UV_THREADPOOL_SIZE`, cache DNS, or use `dns.resolve`.

**`UV_THREADPOOL_SIZE`** must be set *before* the loop starts (it's read once at first use). Max 1024. Sizing: for I/O-pool-bound work, roughly match expected concurrent fs/DNS/crypto ops, but benchmark — more threads = more context switching and memory.

### C5. The C++/JS Boundary & N-API

Node's built-ins (`fs`, `crypto`, `net`) are thin JS wrappers over **C++ bindings** that call libuv/OpenSSL/etc. Crossing the JS↔C++ boundary has a cost, so hot paths minimize crossings (e.g. batch operations). Native addons use **N-API / node-addon-api** — an ABI-stable interface so a compiled addon works across Node versions without recompiling (unlike raw V8 APIs, which change). Relevant when someone asks "how would you integrate a C++ library / speed up a hotspot beyond what JS allows" → worker threads first, native addon (N-API) as a last resort, or WASM as a portable middle ground.

---

## D. The 60-Second Answers (rapid-fire recall)

Interviewers often machine-gun short questions. Have crisp one-liners ready:

- **Why is Node good for I/O-bound, bad for CPU-bound?** One JS thread; I/O is offloaded (kernel/pool) so the thread orchestrates thousands of connections, but CPU work occupies the thread and blocks everyone.
- **`process.nextTick` vs `setImmediate`?** nextTick drains before the loop continues (can starve it); setImmediate runs in the check phase, one per iteration (won't starve).
- **Why can't you revoke a JWT?** It's self-contained and stateless — verification needs no server lookup, so there's nothing to invalidate short of adding state back (denylist) or short expiry.
- **`pipe` vs `pipeline`?** pipeline forwards errors and destroys all streams on failure; pipe leaks FDs on error.
- **Cluster vs worker_threads?** Processes for I/O across cores (isolated memory); threads for CPU work (shared memory).
- **Why `Buffer.allocUnsafe`?** Faster (skips zero-fill) but may expose old memory — only if you immediately overwrite the whole buffer.
- **What saturates the libuv thread pool?** fs, `dns.lookup`, and some crypto/zlib — default 4 threads.
- **`Promise.all` vs `allSettled`?** all rejects on first failure; allSettled always resolves with every outcome.
- **How do you not block the event loop with CPU work?** Worker threads, chunk with `setImmediate`, or offload to a service/queue.
- **After `uncaughtException`, what do you do?** Log, shut down gracefully, let the supervisor restart — never resume.
- **Stateless service — why?** Any instance serves any request → horizontal scale, no sticky sessions; externalize state to Redis/S3.
- **N+1 query — what and fix?** One query per item in a loop; fix by batching (`IN (...)`, joins, DataLoader).

---

## E. What Each Level Is Really Testing

Calibrate your answers to the bar:

- **Senior (SWE II→Senior):** Correctness and depth on the mechanics. Can you explain the event loop, write a concurrency limiter, secure an endpoint, debug a leak? You *know* the platform.
- **Lead:** The above + judgment on trade-offs and the ability to justify a technical direction to a team. You make defensible *choices* and can mentor.
- **Staff:** Cross-cutting systems thinking. You reason about failure modes, blast radius, observability, and second-order effects; you volunteer trade-offs; you know when *not* to use the fancy thing. You *shape* how a system behaves under stress.
- **Architect:** Business-context-driven design. You weigh cost, org structure (Conway's law), migration paths, and long-term evolution; you decide monolith-vs-services with a forcing function, not fashion; you design for the team and the decade, not the demo. You *own* the trade-offs and their consequences.

The common thread from Senior → Architect: **less "what is it" and more "why, when not, and what does it cost."** Practice ending every answer with the trade-off.
