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
