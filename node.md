# Node.js Interview Guide — Trade-offs & Concepts

A simple, easy-to-understand collection of Node.js interview questions with answers.
Focus: **trade-off questions** (when to pick what) and **conceptual questions** (how things work).

---

## Part 1: Trade-off Questions

Trade-off questions test whether you understand *why* you choose one option over another.
There is rarely a "correct" answer — the interviewer wants your reasoning.

### 1. Node.js vs traditional multi-threaded servers (Java, .NET)

| | Node.js | Multi-threaded (Java/.NET) |
|---|---|---|
| Model | Single-threaded event loop | One thread per request |
| Best for | I/O-heavy apps (APIs, chat, streaming) | CPU-heavy apps (image/video processing, heavy math) |
| Memory | Low (few threads) | Higher (many threads) |
| Weakness | CPU work blocks everything | Thread management overhead |

**Simple answer:** Choose Node.js when your app spends most of its time *waiting* (database, network, files).
Choose threads when your app spends most of its time *calculating*.

---

### 2. Callbacks vs Promises vs async/await

- **Callbacks** — oldest style. Simple for one task, but nesting many leads to "callback hell" (pyramid of doom). Error handling is manual and repetitive.
- **Promises** — flatten the nesting with `.then()/.catch()`. Easier chaining and error handling.
- **async/await** — syntactic sugar over Promises. Reads like normal top-to-bottom code, easiest to debug.

**Trade-off:** async/await is the cleanest, but you must wrap it in `try/catch` and remember that
sequential `await` calls run one-by-one. Use `Promise.all()` when tasks are independent and can run in parallel.

```js
// Slow — runs one after another (sequential)
const a = await getUser();
const b = await getOrders();

// Fast — runs together (parallel)
const [a, b] = await Promise.all([getUser(), getOrders()]);
```

---

### 3. `process.nextTick()` vs `setImmediate()` vs `setTimeout()`

- **`process.nextTick(cb)`** — runs *before* the event loop continues, right after the current operation. Highest priority.
- **`setImmediate(cb)`** — runs on the next loop iteration, in the "check" phase.
- **`setTimeout(cb, 0)`** — runs after at least the given delay, in the "timers" phase.

**Trade-off:** `process.nextTick` can *starve* the event loop if used in a loop (I/O never gets a chance).
Prefer `setImmediate` when you want to yield to I/O but still run soon.

---

### 4. Vertical scaling vs Horizontal scaling (Cluster / PM2 / containers)

- **Vertical** — bigger machine (more CPU/RAM). Simple, but has a hard limit.
- **Horizontal** — more instances. Node's single thread only uses one CPU core, so on a
  multi-core machine you run multiple processes.

**Trade-off:** Use the **Cluster module** or **PM2** to fork one worker per CPU core.
Horizontal scaling needs a load balancer and stateless design (store sessions in Redis, not memory).

```js
// Cluster: use all CPU cores
const cluster = require('node:cluster');
const os = require('node:os');
if (cluster.isPrimary) {
  os.cpus().forEach(() => cluster.fork());
} else {
  require('./server'); // each worker runs the server
}
```

---

### 5. Streams vs reading the whole file into memory

- **`fs.readFile`** — loads the entire file into memory, then gives it to you. Simple, but a 2 GB file needs 2 GB of RAM.
- **Streams** — process data in small chunks as they arrive. Constant, low memory usage.

**Trade-off:** Streams are more code and harder to reason about, but essential for large files,
video, or piping data (e.g. `readStream.pipe(res)`). Use `readFile` only for small files.

---

### 6. `worker_threads` vs `child_process` vs Cluster

- **`worker_threads`** — real threads *inside* one process. Share memory. Best for CPU-heavy work (parsing, encryption).
- **`child_process`** — spawns a separate process (can run any program/script). Isolated memory, heavier.
- **`cluster`** — a special use of child_process to run multiple copies of your server for load balancing.

**Simple rule:**
- CPU-heavy computation in your JS → `worker_threads`
- Run an external command/script → `child_process`
- Handle more web traffic → `cluster`

---

### 7. Monolith vs Microservices in Node.js

- **Monolith** — one app, one deploy. Easy to start, easy to test, faster development early on.
- **Microservices** — many small services. Independent scaling and deployment, but adds network calls, complexity, and DevOps overhead.

**Trade-off:** Start with a monolith. Split into microservices only when teams/features grow and
specific parts need independent scaling. Don't add complexity you don't need yet.

---

### 8. SQL vs NoSQL with Node.js

- **SQL (PostgreSQL, MySQL)** — structured data, strong relationships, ACID transactions. Great when data integrity matters (payments, orders).
- **NoSQL (MongoDB)** — flexible schema, horizontal scaling, fast for unstructured/changing data.

**Trade-off:** Node pairs well with MongoDB (JSON everywhere), but "JSON is easy" is not a good
reason to skip SQL. Choose based on your *data*, not the language.

---

### 9. ORM (Sequelize/Prisma/Mongoose) vs raw queries

- **ORM** — write JS objects instead of SQL. Faster development, safer from SQL injection, portable.
- **Raw queries** — full control, best performance, complex queries are easier to express.

**Trade-off:** ORMs can generate slow queries and hide what's happening. Use an ORM for typical
CRUD, drop to raw SQL for complex reports and performance-critical paths.

---

### 10. Monolithic error handling: throw vs return error

- **Throwing** — clean for synchronous and async/await code (`try/catch`), but an uncaught throw in async code can crash the process.
- **Returning errors** (Go-style `[err, data]`) — explicit, but verbose and easy to forget to check.

**Best practice:** Use `try/catch` with async/await, a central Express error-handling middleware,
and always handle Promise rejections. Never swallow errors silently.

---

### 11. JWT vs Session-based authentication

| | JWT (stateless) | Sessions (stateful) |
|---|---|---|
| Storage | Client holds token | Server stores session (Redis/DB) |
| Scaling | Easy (no server state) | Needs shared session store |
| Revoke | Hard (valid until expiry) | Easy (delete session) |
| Size | Sent on every request | Just a session ID |

**Trade-off:** JWT scales better but you can't easily log someone out. Sessions give control
but need a shared store. Common fix: short-lived JWT + refresh tokens.

---

### 12. Caching: in-memory vs Redis

- **In-memory (a JS `Map` or `node-cache`)** — fastest, zero setup, but lost on restart and not shared across instances.
- **Redis** — shared across all instances, survives restarts, supports expiry/pub-sub. Adds a network hop and infrastructure.

**Trade-off:** For a single instance and small data, in-memory is fine. The moment you run
multiple instances (horizontal scaling), you need Redis so all instances see the same cache.

---

## Part 2: Conceptual Questions

These test whether you understand *how Node.js actually works* under the hood.

### 1. What is Node.js?

Node.js is a **runtime** that lets you run JavaScript outside the browser, built on Chrome's
**V8 engine**. It uses a **non-blocking, event-driven** model, making it great for scalable
network applications. It is *not* a language and *not* a framework.

---

### 2. Is Node.js single-threaded? Then how does it handle concurrency?

Your JavaScript runs on a **single thread** (the event loop). But Node isn't *only* one thread —
under the hood **libuv** provides a **thread pool** (default 4 threads) for expensive operations
like file I/O, DNS, and crypto.

**Simple picture:** JS gives a task to Node, Node hands slow work to the OS or thread pool,
and when it's done, the callback is queued back to the single thread. So one thread stays busy
*coordinating* instead of *waiting*.

---

### 3. Explain the Event Loop (the most asked question)

The event loop is what lets single-threaded Node do many things "at once". It runs in **phases**, repeatedly:

1. **Timers** — runs `setTimeout` / `setInterval` callbacks.
2. **Pending callbacks** — some system-level callbacks.
3. **Poll** — retrieves new I/O events, runs I/O callbacks (the main phase).
4. **Check** — runs `setImmediate` callbacks.
5. **Close** — runs close events like `socket.on('close')`.

Between *every* phase, Node drains two special queues:
- **`process.nextTick` queue** (highest priority)
- **microtask/Promise queue** (`.then`, `await`)

**One-line answer:** The event loop continuously checks queues of finished work and runs their
callbacks on the single main thread, so nothing has to block and wait.

---

### 4. Blocking vs Non-blocking code

- **Blocking** — code that stops everything until it finishes (`fs.readFileSync`). Nothing else runs.
- **Non-blocking** — code that starts a task and moves on, using a callback/Promise when done (`fs.readFile`).

**Rule:** Never use `...Sync` functions or run heavy CPU loops in a request handler — they
freeze the entire server for *all* users.

---

### 5. What is libuv?

**libuv** is the C library that powers Node's async magic. It provides:
- the **event loop**,
- the **thread pool** for file system and CPU-bound tasks,
- cross-platform handling of async network I/O.

In short, libuv is *how* Node does non-blocking I/O across Windows, Mac, and Linux.

---

### 6. Microtasks vs Macrotasks

- **Macrotasks** — `setTimeout`, `setInterval`, `setImmediate`, I/O callbacks.
- **Microtasks** — Promise callbacks (`.then`, `await`), `queueMicrotask`, `process.nextTick`.

**Key rule:** After each macrotask, Node empties the *entire* microtask queue before moving on.
So microtasks always run before the next timer/I/O callback.

```js
console.log('1');
setTimeout(() => console.log('2'), 0);   // macrotask
Promise.resolve().then(() => console.log('3')); // microtask
console.log('4');
// Output: 1, 4, 3, 2
```

---

### 7. What is the difference between `require` (CommonJS) and `import` (ES Modules)?

- **`require` (CommonJS)** — synchronous, loaded at runtime, the classic Node style. You can call it conditionally.
- **`import` (ES Modules)** — the modern JS standard, asynchronous, statically analyzable (better tree-shaking), supports top-level `await`.

**Trade-off:** ESM is the future and works across browser + Node, but many old packages are
CommonJS. Use `"type": "module"` in package.json to opt into ESM.

---

### 8. What is middleware in Express?

Middleware is a function with access to the **request (`req`)**, **response (`res`)**, and the
**`next`** function. It runs *in order* and can:
- modify req/res,
- end the response, or
- pass control with `next()`.

Used for logging, authentication, parsing bodies, and error handling.

```js
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next(); // pass to the next middleware
});
```

---

### 9. What are Streams and their types?

Streams process data in chunks instead of all at once. Four types:
- **Readable** — you read from it (`fs.createReadStream`).
- **Writable** — you write to it (`fs.createWriteStream`).
- **Duplex** — both read and write (a TCP socket).
- **Transform** — modifies data as it passes through (gzip compression).

`pipe()` connects them: `readStream.pipe(gzip).pipe(writeStream)`.

---

### 10. What is the Buffer?

A **Buffer** is a chunk of raw binary memory outside V8's heap. It's used to handle binary data
(files, network packets, images) since JavaScript strings can't represent raw bytes well.

```js
const buf = Buffer.from('Hello');
console.log(buf);            // <Buffer 48 65 6c 6c 6f>
console.log(buf.toString()); // Hello
```

---

### 11. What is the EventEmitter?

The `EventEmitter` class is the backbone of Node's event-driven design. You emit named events
and register listeners for them — the publisher/subscriber pattern.

```js
const EventEmitter = require('node:events');
const bus = new EventEmitter();
bus.on('order', (id) => console.log(`Order ${id} received`));
bus.emit('order', 42); // Order 42 received
```

Many core objects (streams, HTTP servers) are EventEmitters.

---

### 12. How does Node handle CPU-intensive tasks?

Since heavy CPU work blocks the single event-loop thread, you should:
1. Offload it to **`worker_threads`**, or
2. Run it in a separate **`child_process`**, or
3. Break it into smaller chunks with `setImmediate` to let the loop breathe, or
4. Move it to a background job queue (e.g. BullMQ + Redis).

**Never** run a long `for` loop directly in a request handler.

---

### 13. What is the difference between `exports` and `module.exports`?

`module.exports` is the *actual* object that gets returned by `require`.
`exports` is just a shortcut variable pointing to the same object.

```js
exports.add = (a, b) => a + b;      // works (adding a property)
module.exports = { add };            // works (replacing the object)
exports = { add };                   // does NOT work (breaks the reference)
```

**Rule:** Assign a whole new object to `module.exports`, not `exports`.

---

### 14. What is a memory leak in Node and how do you find one?

A memory leak is memory that's no longer needed but never freed, so RAM keeps growing.
Common causes: global variables, forgotten timers, unremoved event listeners, and growing caches.

**How to find:** Watch memory with `process.memoryUsage()`, take **heap snapshots** in Chrome
DevTools (via `--inspect`), and compare them over time to see what keeps growing.

---

### 15. What happens when you run `node app.js`? (lifecycle)

1. Node initializes V8 and the event loop.
2. It reads and executes your file top-to-bottom (synchronous code first).
3. Async operations are registered and offloaded.
4. When the main script finishes, the event loop keeps running as long as there is pending work
   (timers, I/O, open sockets).
5. When there's nothing left to do, the process exits.

---

### 16. Error handling: `uncaughtException` and `unhandledRejection`

- **`uncaughtException`** — a synchronous error nobody caught. The app is in an unknown state; log it and **restart the process** (don't just resume).
- **`unhandledRejection`** — a Promise that rejected with no `.catch()`. In modern Node this can also crash the process.

**Best practice:** Handle errors locally with `try/catch`, use these global handlers only as a
last-resort log-and-exit safety net (let PM2/Docker restart you).

---

## Quick Rapid-Fire Answers

- **Why is Node fast for I/O?** Non-blocking event loop + libuv thread pool means it never sits idle waiting.
- **Default libuv thread pool size?** 4 (change with `UV_THREADPOOL_SIZE`).
- **Is `setTimeout(fn, 0)` immediate?** No — it waits for the timers phase, and microtasks run first.
- **`npm` vs `npx`?** `npm` installs/manages packages; `npx` executes a package binary without installing it globally.
- **`dependencies` vs `devDependencies`?** Runtime needs vs development-only tools (test, lint, build).
- **What is `package-lock.json`?** Locks exact versions so every install is identical and reproducible.
- **How to keep a Node app alive in production?** PM2, systemd, or a container orchestrator (Docker/Kubernetes) that auto-restarts on crash.

---

*Tip for interviews:* For trade-off questions, always say **"it depends on X"** and name the deciding factor
(I/O vs CPU, scale, team size, data shape). That shows engineering judgment, which is what they're really testing.

---

## Part 3: More Important Concepts (Deep Dive)

These are powerful concepts that make you stand out — they show you understand Node beyond the basics.

### 1. Global objects (`__dirname`, `__filename`, `process`, `global`)

Node gives you a few globals available everywhere without `require`:
- **`__dirname`** — absolute path of the current file's *folder*.
- **`__filename`** — absolute path of the current *file*.
- **`process`** — info and control over the running Node process.
- **`global`** — the global namespace (like `window` in the browser).

```js
console.log(__dirname);   // D:\learning
console.log(__filename);  // D:\learning\app.js
```

**Note:** In ES Modules, `__dirname` and `__filename` don't exist — use
`import.meta.url` with `fileURLToPath` instead.

---

### 2. The `process` object & environment variables

`process` is your window into the running program:
- **`process.env`** — environment variables (config, secrets).
- **`process.argv`** — command-line arguments.
- **`process.cwd()`** — current working directory.
- **`process.exit(code)`** — stop the process.
- **`process.on('SIGINT', ...)`** — react to signals (Ctrl+C).

```js
// Never hard-code secrets — read from the environment
const dbUrl = process.env.DATABASE_URL;
const port = process.env.PORT || 3000;
```

Use a `.env` file with the `dotenv` package in development, and real environment
variables in production. **Never commit secrets to git.**

---

### 3. Graceful shutdown

When your server is stopped (deploy, crash, Ctrl+C), you shouldn't cut off active requests.
A **graceful shutdown** stops accepting new requests, finishes the current ones, closes DB
connections, then exits.

```js
process.on('SIGTERM', async () => {
  console.log('Shutting down gracefully...');
  server.close();          // stop accepting new connections
  await db.disconnect();   // clean up resources
  process.exit(0);
});
```

**Why it matters:** Without it, in-flight requests get dropped and data can be left half-written.

---

### 4. Backpressure in streams

When a fast source produces data quicker than a slow destination can handle it, memory fills up.
**Backpressure** is the mechanism that tells the source to *slow down*.

`pipe()` handles backpressure for you automatically — that's why you should prefer it over
manually reading and writing.

```js
// Good: pipe manages backpressure — reads only as fast as it can write
readStream.pipe(writeStream);

// Risky: manual write can overflow memory if you ignore the return value
readStream.on('data', (chunk) => {
  const ok = writeStream.write(chunk);
  if (!ok) readStream.pause(); // resume on 'drain'
});
```

---

### 5. Advanced Promise combinators

Beyond `Promise.all`, know these four:
- **`Promise.all`** — waits for all; rejects if *any* one fails. (all-or-nothing)
- **`Promise.allSettled`** — waits for all; never rejects, gives you each result/error.
- **`Promise.race`** — resolves/rejects with the *first* one to finish (win or lose).
- **`Promise.any`** — resolves with the first *success*; rejects only if all fail.

```js
// Useful for timeouts: whichever finishes first wins
const data = await Promise.race([
  fetchData(),
  new Promise((_, rej) => setTimeout(() => rej(new Error('timeout')), 5000)),
]);
```

---

### 6. AbortController — cancelling async work

Modern Node lets you cancel operations (fetch, timers, streams) with an `AbortController`.

```js
const controller = new AbortController();
setTimeout(() => controller.abort(), 5000); // cancel after 5s

const res = await fetch(url, { signal: controller.signal });
```

**Why it matters:** Prevents wasted work and hanging requests when a client disconnects
or a timeout hits.

---

### 7. Connection pooling

Opening a new database connection for every request is slow and expensive. A **connection pool**
keeps a set of open connections and reuses them.

```js
const pool = new Pool({ max: 20 }); // reuse up to 20 connections
const result = await pool.query('SELECT * FROM users');
```

**Concept:** Borrow a connection → use it → return it to the pool. Almost every DB library
(pg, mysql2, mongoose) does this. Tuning pool size is a common performance win.

---

### 8. The Reactor Pattern (the idea behind Node)

Node is built on the **Reactor Pattern**:
1. Your code submits an I/O request with a handler (callback).
2. Node registers it and *immediately continues* (non-blocking).
3. When the I/O completes, an event is fired.
4. The event loop picks it up and runs your handler.

**One-liner:** "Give me the work and a callback; I'll call you back when it's done."
This is *why* one thread can handle thousands of connections.

---

### 9. Job queues & background workers

For slow tasks (send email, generate PDF, process video), don't make the user wait.
Push the task to a **queue** and process it in the background.

```
Request → add job to queue (Redis) → respond instantly
Worker  → picks up job later → does the heavy work
```

Popular tool: **BullMQ** (Redis-based). Benefits: retries, scheduling, and your API stays fast.

---

### 10. Rate limiting

Protect your API from abuse and overload by limiting how many requests a client can make.

```js
const rateLimit = require('express-rate-limit');
app.use(rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100,                 // max 100 requests per IP per window
}));
```

**Concept:** Common algorithms are *fixed window*, *sliding window*, and *token bucket*.
For multiple instances, store counters in Redis so the limit is shared.

---

### 11. Clustering + load balancing (using all cores)

One Node process = one CPU core. To use a 4-core machine fully, run 4 workers.
The `cluster` module (or PM2) forks workers that **share the same port**, and the OS/Node
load-balances incoming connections across them.

```bash
pm2 start app.js -i max   # one worker per CPU core
```

**Remember:** Workers don't share memory, so keep shared state (sessions, cache) in Redis.

---

### 12. Garbage collection & the V8 heap (simple version)

V8 automatically frees memory you no longer use (garbage collection). It splits memory into:
- **New space** — short-lived objects, cleaned often and fast.
- **Old space** — long-lived objects, cleaned less often (slower).

**What you should know:** GC pauses can cause latency spikes. Avoid keeping huge objects
alive unnecessarily. Default heap limit is ~2 GB (raise with `--max-old-space-size`).

---

### 13. Semantic Versioning (SemVer) & lockfiles

Package versions look like **`MAJOR.MINOR.PATCH`** (e.g. `4.18.2`):
- **MAJOR** — breaking changes.
- **MINOR** — new features, backward-compatible.
- **PATCH** — bug fixes.

Range symbols in package.json:
- `^4.18.2` → allow minor + patch updates (`4.x.x`).
- `~4.18.2` → allow patch updates only (`4.18.x`).
- `4.18.2` → exact version.

**`package-lock.json`** freezes the *exact* resolved versions so every machine installs
identical dependencies. Always commit it.

---

### 14. Security essentials

Common Node/Express security wins interviewers love to hear:
- **`helmet`** — sets safe HTTP headers automatically.
- **Validate input** — use `zod` / `joi`; never trust user data.
- **Prevent injection** — use parameterized queries or an ORM (never string-concat SQL).
- **Rate limiting** — block brute-force attacks.
- **Hash passwords** — with `bcrypt` / `argon2`, never store plain text.
- **Keep secrets in env vars**, not in code.
- **`npm audit`** — check dependencies for known vulnerabilities.

---

### 15. Debugging & profiling

- **`node --inspect app.js`** — attach Chrome DevTools to set breakpoints and inspect memory.
- **`console.time()` / `console.timeEnd()`** — quick timing of a code block.
- **`process.memoryUsage()`** — check heap/RSS usage.
- **Heap snapshots** — compare over time to find memory leaks.
- **`--prof`** — generate a CPU profile to find slow functions.

---

### 16. Caching strategies (the concept)

Caching means storing computed/fetched results to avoid redoing work. Common patterns:
- **Cache-aside** — check cache first; on miss, read DB and fill the cache. (most common)
- **Write-through** — write to cache and DB together (cache always fresh).
- **TTL (time-to-live)** — auto-expire entries so stale data clears itself.

**Key challenge:** *cache invalidation* — knowing when cached data is out of date.
"There are only two hard things in CS: cache invalidation and naming things."

---

*Final tip:* Don't just memorize definitions. For each concept, be ready to answer
**"why does this exist?"** and **"when would you use it?"** — that's what separates a
junior answer from a senior one.

---

## Part 4: Complete Conceptual Topics (A–Z Reference)

A categorized map of *every* major concept area in Node.js. Use this as a checklist —
if you can explain each line in one or two sentences, you're interview-ready.

### A. Runtime & Architecture

- **V8 engine** — Google's JS engine that compiles JavaScript to machine code (JIT compilation). Node embeds it.
- **Node architecture** — three layers: your **JS code** → **Node bindings (C++)** → **V8 + libuv**. libuv provides the event loop and thread pool.
- **JIT compilation** — V8 first interprets code, then compiles "hot" (frequently used) code to fast machine code.
- **REPL** — Read-Eval-Print-Loop; type `node` in a terminal to run JS interactively.
- **Event-driven architecture** — the whole platform reacts to events (requests, I/O completion) rather than polling.
- **The C++ addons** — you can write native modules in C++ for performance-critical code (`node-gyp`).

### B. Module System

- **CommonJS** — `require` / `module.exports`; synchronous; default in older Node.
- **ES Modules (ESM)** — `import` / `export`; the modern standard; async; enable with `"type": "module"`.
- **Module wrapper** — Node wraps every file in a function giving you `exports, require, module, __dirname, __filename`.
- **Module resolution** — how Node finds a module: core module → `node_modules` (walking up folders) → relative path.
- **Module caching** — a module is loaded and executed **once**; later `require`s return the cached copy (great for singletons).
- **Three module types** — **core** (`fs`, `http`), **local** (`./myFile`), **third-party** (from `node_modules`).

### C. npm & Package Management

- **npm** — the default package manager and registry.
- **`package.json`** — project manifest: name, version, scripts, dependencies.
- **`package-lock.json`** — locks exact dependency versions for reproducible installs.
- **npm scripts** — custom commands (`npm run build`); `start`/`test` are special.
- **`npx`** — run a package binary without installing it globally.
- **Semantic versioning (`^`, `~`)** — MAJOR.MINOR.PATCH rules for updates.
- **Workspaces / monorepo** — manage multiple packages in one repo.
- **Alternatives** — `yarn`, `pnpm` (faster, disk-efficient).

### D. Core Built-in Modules (must-know)

- **`fs`** — file system (read/write files); sync, callback, and `fs/promises` versions.
- **`path`** — build/normalize file paths cross-platform (`path.join`, `path.resolve`).
- **`http` / `https`** — create servers and make requests without a framework.
- **`url`** — parse and format URLs; `URLSearchParams` for query strings.
- **`os`** — system info (CPUs, memory, platform).
- **`events`** — the `EventEmitter` class.
- **`stream`** — streaming data interfaces.
- **`crypto`** — hashing, encryption, random bytes.
- **`util`** — helpers like `util.promisify`.
- **`buffer`** — raw binary data.
- **`process`** — the current process (env, argv, exit).
- **`child_process` / `cluster` / `worker_threads`** — multi-processing/threading.

### E. Asynchronous Programming

- **Error-first callbacks** — the classic Node pattern: `callback(err, data)`.
- **Callback hell** — deeply nested callbacks; solved by Promises/async-await.
- **Promises** — objects representing a future value (pending → fulfilled/rejected).
- **async/await** — write async code that reads synchronously.
- **`util.promisify`** — convert a callback function into a Promise-returning one.
- **Promise combinators** — `all`, `allSettled`, `race`, `any`.
- **Async iterators / generators** — `for await...of` to loop over async data (great with streams).
- **`async_hooks`** — track the lifecycle of async resources (advanced, used by tracing tools).
- **AbortController** — cancel async operations.

### F. The Event Loop & Timing

- **Event loop phases** — timers → pending → poll → check → close.
- **`process.nextTick`** — runs before the loop continues (highest priority).
- **Microtasks vs macrotasks** — Promises (micro) run before timers/I/O (macro).
- **`setTimeout` / `setInterval` / `setImmediate`** — scheduling callbacks.
- **Event loop starvation** — heavy sync code or nextTick loops block everything.
- **Blocking vs non-blocking** — never block the single thread.

### G. Streams & Buffers

- **Buffer** — fixed-size raw binary memory.
- **Stream types** — Readable, Writable, Duplex, Transform.
- **`pipe()` and `pipeline()`** — connect streams; `pipeline` adds error handling.
- **Backpressure** — slowing the source when the destination is busy.
- **Object mode streams** — streams that carry JS objects, not just bytes.

### H. HTTP, Networking & APIs

- **Creating an HTTP server** — `http.createServer()`.
- **Request/response lifecycle** — how a request flows through your app.
- **REST API principles** — resources, HTTP verbs (GET/POST/PUT/PATCH/DELETE), statelessness.
- **HTTP status codes** — 2xx success, 3xx redirect, 4xx client error, 5xx server error.
- **Routing** — mapping URLs+methods to handlers.
- **CORS** — controlling which origins can call your API.
- **WebSockets** — full-duplex real-time communication (`ws`, `socket.io`).
- **GraphQL** — alternative to REST: client asks for exactly the data it needs.
- **gRPC** — high-performance RPC using Protocol Buffers.
- **Content negotiation / body parsing** — JSON, form-data, multipart (file uploads).

### I. Express & Web Frameworks

- **Middleware** — functions in the request pipeline (`req, res, next`).
- **Middleware types** — application, router, built-in, third-party, error-handling.
- **Routing & route params** — `/users/:id`, query strings.
- **Error-handling middleware** — 4-argument `(err, req, res, next)`.
- **Static file serving** — `express.static`.
- **Template engines** — EJS, Pug, Handlebars (server-side rendering).
- **Framework choices** — Express (classic), Fastify (fast), NestJS (structured/TypeScript), Koa, Hapi.

### J. Databases & Data Access

- **SQL vs NoSQL** — structured/relational vs flexible/document.
- **ORM / ODM** — Sequelize, Prisma, TypeORM (SQL); Mongoose (MongoDB).
- **Connection pooling** — reuse DB connections.
- **Transactions** — all-or-nothing groups of operations (ACID).
- **Migrations** — versioned schema changes.
- **Indexing** — speed up queries (concept, not Node-specific).
- **Query builders** — Knex.js (between raw SQL and full ORM).

### K. Authentication & Security

- **Authentication vs Authorization** — who you are vs what you can do.
- **JWT vs sessions** — stateless token vs server-side session.
- **Password hashing** — `bcrypt`, `argon2` (never store plain text).
- **OAuth / OpenID / SSO** — third-party login (Google, GitHub).
- **`helmet`** — secure HTTP headers.
- **Input validation** — `zod`, `joi`.
- **Common attacks** — SQL injection, XSS, CSRF, and how to prevent them.
- **Rate limiting & throttling** — prevent abuse.
- **HTTPS/TLS** — encrypt traffic.
- **`npm audit`** — scan dependencies for vulnerabilities.

### L. Multi-processing & Scaling

- **`cluster`** — fork workers to use all CPU cores.
- **`worker_threads`** — real threads for CPU-heavy JS.
- **`child_process`** — `spawn`, `exec`, `execFile`, `fork`.
- **IPC (inter-process communication)** — messaging between processes.
- **Load balancing** — spread requests across instances.
- **Stateless design** — keep state in Redis/DB so any instance can serve any request.
- **Horizontal vs vertical scaling** — more machines vs bigger machine.

### M. Performance & Optimization

- **Caching** — in-memory, Redis; cache-aside, write-through, TTL.
- **Profiling** — `--inspect`, `--prof`, Chrome DevTools.
- **Memory management** — V8 heap, garbage collection, avoiding leaks.
- **Avoid blocking the event loop** — offload CPU work.
- **Compression** — gzip/brotli responses.
- **Load testing** — `autocannon`, `k6`.
- **Lazy loading & pagination** — don't fetch everything at once.

### N. Error Handling

- **Error-first callbacks / try-catch / `.catch()`** — three ways depending on style.
- **Custom error classes** — extend `Error` for typed errors.
- **Centralized error middleware** — one place to handle Express errors.
- **`uncaughtException` / `unhandledRejection`** — global safety nets (log & restart).
- **Operational vs programmer errors** — expected failures (bad input) vs bugs.

### O. Testing

- **Test types** — unit, integration, end-to-end (E2E).
- **Frameworks** — Jest, Mocha+Chai, and the built-in `node:test`.
- **Mocking / stubbing / spies** — fake dependencies (Sinon, Jest mocks).
- **Test doubles** — replace real DB/HTTP in tests.
- **Coverage** — how much code your tests exercise.
- **TDD / BDD** — write tests first / behavior-focused tests.
- **API testing** — `supertest` for HTTP endpoints.

### P. Deployment & DevOps

- **Environment variables & config** — `dotenv`, 12-factor app.
- **Process managers** — PM2 (restart, cluster, logs), systemd.
- **Docker** — containerize the app for consistent deployment.
- **CI/CD** — automated test + deploy pipelines (GitHub Actions, Jenkins).
- **Logging** — `winston`, `pino`, `morgan`; structured logs.
- **Monitoring & observability** — metrics, tracing, health checks (Prometheus, Grafana, OpenTelemetry).
- **Graceful shutdown** — finish in-flight requests before exiting.
- **Reverse proxy** — Nginx in front of Node for TLS, static files, load balancing.

### Q. Architecture & Design Patterns

- **MVC** — Model-View-Controller separation.
- **Layered architecture** — routes → controllers → services → repositories.
- **Repository pattern** — abstract data access.
- **Dependency injection** — pass dependencies in instead of hard-coding them.
- **Singleton** — module caching gives you singletons for free.
- **Factory / Observer / Strategy** — classic GoF patterns applied in Node.
- **Middleware pattern** — chain of processing functions.
- **Monolith vs microservices** — single app vs distributed services.
- **Message queues / event-driven** — RabbitMQ, Kafka, BullMQ for decoupling.
- **API gateway** — single entry point routing to microservices.

### R. Modern Node.js Features (recent versions)

- **Built-in `fetch`** — HTTP requests without axios/node-fetch (stable in Node 21+).
- **Built-in test runner** — `node:test` + `node --test`.
- **Watch mode** — `node --watch app.js` auto-restarts on changes (no nodemon needed).
- **`.env` file support** — `node --env-file=.env` (no dotenv needed).
- **Top-level await** — use `await` outside async functions in ESM.
- **Permission model** — restrict file/network access with `--permission`.
- **Diagnostics channel** — publish/subscribe for diagnostics data.
- **Single executable apps** — bundle Node + your app into one binary.

---

## How to Use This Guide

1. **First pass** — read every heading and check the ones you can already explain out loud.
2. **Second pass** — for each unchecked item, learn it until you can give a *one-sentence* answer plus *when to use it*.
3. **Practice** — turn concepts into sentences: *"I'd use `worker_threads` here because the task is CPU-bound and would block the event loop."*
4. **Depth on demand** — interviewers drill into whatever you mention, so only bring up what you can defend.

*Golden rule:* Understanding **the event loop, async patterns, and scaling trade-offs**
deeply matters more than memorizing every module name.

---

## Part 5: Deep Dive — 4 Key Areas

Detailed exploration of the four areas that come up most in senior interviews.

---

# 🔐 A. Authentication & Security (Deep Dive)

### 1. Authentication vs Authorization

- **Authentication (AuthN)** — *"Who are you?"* Verifying identity (login with password, OTP, etc.).
- **Authorization (AuthZ)** — *"What are you allowed to do?"* Checking permissions/roles after login.

**Simple example:** Logging in = authentication. Being blocked from `/admin` because you're a normal user = authorization.

---

### 2. Password hashing (never store plain text)

Store a **hash**, not the password. Hashing is one-way — you can't reverse it.

```js
const bcrypt = require('bcrypt');

// Sign up: hash before saving
const hash = await bcrypt.hash(plainPassword, 10); // 10 = salt rounds

// Login: compare
const isMatch = await bcrypt.compare(plainPassword, hash);
```

**Key ideas:**
- **Salt** — random data added to each password so two identical passwords get different hashes (defeats rainbow tables). bcrypt salts automatically.
- **Cost factor / rounds** — makes hashing intentionally slow to resist brute force.
- **`bcrypt` vs `argon2`** — argon2 is newer and won the Password Hashing Competition; both are good. Never use plain MD5/SHA-256 for passwords (too fast).

---

### 3. JWT (JSON Web Token) explained fully

A JWT is a **self-contained, signed token** with three parts separated by dots:

```
header.payload.signature
eyJhbGc...  .  eyJ1c2Vy...  .  SflKxwRJ...
```

- **Header** — algorithm + type.
- **Payload** — claims (userId, role, expiry). *Readable by anyone — never put secrets here!*
- **Signature** — proves the token wasn't tampered with (signed with a secret key).

```js
const jwt = require('jsonwebtoken');

// Create on login
const token = jwt.sign({ userId: 1, role: 'admin' }, process.env.JWT_SECRET, {
  expiresIn: '15m',
});

// Verify on each request (middleware)
function auth(req, res, next) {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ error: 'No token' });
  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET);
    next();
  } catch {
    return res.status(403).json({ error: 'Invalid token' });
  }
}
```

**The refresh token pattern (important!):**
- **Access token** — short-lived (~15 min), sent on every request.
- **Refresh token** — long-lived (~7 days), stored securely (httpOnly cookie), used only to get a new access token.
- **Why:** limits damage if an access token leaks, while keeping users logged in.

---

### 4. Sessions vs JWT (when to use which)

| | JWT | Session |
|---|---|---|
| State | Stateless (server stores nothing) | Stateful (server stores session) |
| Logout / revoke | Hard (valid until expiry) | Easy (delete session) |
| Scaling | Great (no shared store needed) | Needs shared store (Redis) |
| Best for | APIs, mobile, microservices | Traditional web apps needing instant revoke |

**Interview answer:** "JWT for stateless APIs and microservices; sessions when I need instant logout/revocation. To fix JWT's revoke weakness, I use short expiry + refresh tokens, or a token blocklist in Redis."

---

### 5. OWASP Top 10 attacks (know these!)

- **SQL / NoSQL Injection** — attacker injects malicious query. **Fix:** parameterized queries / ORM, never string-concat user input.
- **XSS (Cross-Site Scripting)** — attacker injects `<script>` into your page. **Fix:** escape/sanitize output, Content-Security-Policy, `helmet`.
- **CSRF (Cross-Site Request Forgery)** — trick a logged-in user into a request. **Fix:** CSRF tokens, `SameSite` cookies.
- **Broken authentication** — weak passwords, no rate limit. **Fix:** strong hashing, rate limiting, MFA.
- **Sensitive data exposure** — secrets in code/logs. **Fix:** env vars, HTTPS, don't log tokens.
- **Security misconfiguration** — default settings, verbose errors. **Fix:** `helmet`, hide stack traces in production.

---

### 6. Practical security checklist

```js
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');

app.use(helmet());                       // secure headers
app.use(express.json({ limit: '10kb' })); // limit body size (DoS protection)
app.use(rateLimit({ windowMs: 15*60*1000, max: 100 }));
```

- ✅ Validate every input (`zod` / `joi`).
- ✅ Use HTTPS/TLS everywhere.
- ✅ Store secrets in env vars, not code.
- ✅ Set cookies `httpOnly`, `secure`, `sameSite`.
- ✅ Run `npm audit` and keep dependencies updated.
- ✅ Hide error details in production (no stack traces to users).
- ✅ Use CORS to whitelist trusted origins only.

---

# ⚙️ B. Multi-processing & Scaling (Deep Dive)

### 1. The core problem

Node runs your JavaScript on **one thread**. On a machine with 8 CPU cores, a single Node
process uses only **1 core** — the other 7 sit idle. Scaling means fixing this.

---

### 2. `cluster` module — use all CPU cores

The `cluster` module forks multiple worker processes that **share the same port**.
The primary process load-balances incoming connections across workers (round-robin).

```js
const cluster = require('node:cluster');
const os = require('node:os');

if (cluster.isPrimary) {
  const cpus = os.cpus().length;
  for (let i = 0; i < cpus; i++) cluster.fork();

  cluster.on('exit', (worker) => {
    console.log(`Worker ${worker.process.pid} died, restarting...`);
    cluster.fork(); // auto-restart crashed workers
  });
} else {
  require('./server'); // each worker runs the HTTP server
}
```

**Key point:** Workers **don't share memory**. In-memory data (sessions, cache, counters)
must move to a shared store like **Redis**, or each worker will have its own copy.

---

### 3. `worker_threads` — for CPU-heavy work

Unlike cluster (separate processes), worker threads run **inside one process** and can
share memory via `SharedArrayBuffer`. Use them for heavy computation, not for scaling HTTP.

```js
const { Worker } = require('node:worker_threads');

function runHeavyTask(data) {
  return new Promise((resolve, reject) => {
    const worker = new Worker('./heavy-worker.js', { workerData: data });
    worker.on('message', resolve);
    worker.on('error', reject);
  });
}
// The event loop stays free while the worker crunches numbers.
```

**cluster vs worker_threads:**
- **cluster** → scale *network/HTTP throughput* across cores.
- **worker_threads** → offload *CPU-bound computation* without blocking the event loop.

---

### 4. `child_process` — run separate programs

`spawn`, `exec`, `execFile`, `fork` let you run external commands or other Node scripts.

- **`spawn`** — streams output; good for large/continuous output.
- **`exec`** — buffers output; good for short commands (careful: shell injection risk).
- **`fork`** — special spawn for Node scripts with a built-in messaging channel (IPC).

```js
const { spawn } = require('node:child_process');
const ls = spawn('ls', ['-la']);
ls.stdout.on('data', (data) => console.log(`${data}`));
```

---

### 5. Horizontal vs vertical scaling

- **Vertical** — bigger server (more CPU/RAM). Simple but has a ceiling and single point of failure.
- **Horizontal** — more servers/instances behind a **load balancer** (Nginx, cloud LB). Scales almost infinitely and adds redundancy.

**Requirement for horizontal scaling → stateless design:** any instance must be able to
handle any request. So move state out of the process:
- Sessions/cache → **Redis**
- Files/uploads → **object storage (S3)**
- Background jobs → **message queue (BullMQ, RabbitMQ, Kafka)**

---

### 6. Load balancing strategies

- **Round-robin** — requests distributed in turn (default).
- **Least connections** — send to the least busy instance.
- **IP hash / sticky sessions** — same client always hits the same instance (needed if you *must* keep state in memory — but avoid this).

---

### 7. Scaling architecture (the big picture)

```
                 ┌──────────────┐
   Clients  →    │ Load Balancer │  (Nginx / Cloud LB)
                 └──────┬───────┘
          ┌────────────┼────────────┐
     ┌────▼───┐   ┌────▼───┐   ┌────▼───┐
     │ Node 1 │   │ Node 2 │   │ Node 3 │   ← stateless instances (each = cluster of workers)
     └────┬───┘   └────┬───┘   └────┬───┘
          └────────────┼────────────┘
              ┌─────────▼─────────┐
              │ Redis  +  Database │  ← shared state
              └───────────────────┘
```

**Interview one-liner:** "Cluster within a box to use all cores, then run multiple stateless
instances behind a load balancer, with Redis and the DB holding shared state."

---

# 🚀 C. Performance & Optimization (Deep Dive)

### 1. The #1 rule: never block the event loop

One slow synchronous operation freezes the server for **every** user. Avoid:
- Sync FS calls (`readFileSync`) in request handlers.
- Heavy loops / big JSON parsing on the main thread → use `worker_threads`.
- Complex synchronous regex (ReDoS).

```js
// ❌ Blocks everyone
app.get('/report', (req, res) => {
  const data = fs.readFileSync('huge.csv'); // freezes the loop
  res.send(process(data));
});

// ✅ Non-blocking
app.get('/report', async (req, res) => {
  const data = await fs.promises.readFile('huge.csv');
  res.send(process(data));
});
```

---

### 2. Caching (biggest performance lever)

Store results so you don't redo expensive work.

```js
// Cache-aside pattern with Redis
async function getUser(id) {
  const cached = await redis.get(`user:${id}`);
  if (cached) return JSON.parse(cached);        // cache hit

  const user = await db.findUser(id);           // cache miss
  await redis.set(`user:${id}`, JSON.stringify(user), 'EX', 3600); // TTL 1h
  return user;
}
```

**Strategies:** cache-aside (most common), write-through, write-back.
**Levels:** in-memory (fastest, per-instance) → Redis (shared) → CDN (static assets).
**Hard part:** cache invalidation — use TTLs and clear cache on writes.

---

### 3. Database optimization

- **Indexing** — the single biggest DB win; index columns you filter/sort on.
- **Connection pooling** — reuse connections instead of opening new ones per request.
- **Avoid N+1 queries** — don't query in a loop; use joins or `IN (...)` / `.populate()`.
- **Pagination** — never `SELECT *` millions of rows; use `LIMIT`/`OFFSET` or cursors.
- **Select only needed columns** — don't fetch data you won't use.

```js
// ❌ N+1: 1 query for posts + 1 query per post's author
const posts = await Post.find();
for (const p of posts) p.author = await User.findById(p.authorId);

// ✅ One query with a join/populate
const posts = await Post.find().populate('author');
```

---

### 4. Response optimization

- **Compression** — gzip/brotli shrinks responses (`compression` middleware).
- **Pagination & filtering** — send less data.
- **HTTP caching headers** — `ETag`, `Cache-Control` let clients skip re-downloading.
- **Streaming** — stream large responses instead of buffering in memory.

```js
const compression = require('compression');
app.use(compression()); // gzip all responses
```

---

### 5. Profiling & finding bottlenecks

*Measure before optimizing — don't guess.*

- **`node --inspect`** — Chrome DevTools: breakpoints, CPU profile, heap snapshots.
- **`node --prof`** — generate a V8 CPU profile.
- **`console.time()` / `timeEnd()`** — quick timing.
- **`process.memoryUsage()`** — heap/RSS tracking.
- **Load testing** — `autocannon`, `k6`, `artillery` to find limits under load.
- **APM tools** — New Relic, Datadog, Clinic.js for production insight.

---

### 6. Memory & garbage collection

- V8 heap default limit ≈ **2 GB** (raise with `--max-old-space-size=4096`).
- **GC pauses** cause latency spikes — avoid keeping huge objects alive.
- **Common leaks:** global variables, uncleared timers/intervals, un-removed event listeners, ever-growing caches/arrays.
- **Find leaks:** take heap snapshots over time and compare what keeps growing.

---

### 7. Quick performance checklist

- ✅ Offload CPU work to worker threads.
- ✅ Cache hot data (Redis).
- ✅ Add DB indexes + connection pooling.
- ✅ Enable gzip compression.
- ✅ Use pagination.
- ✅ Cluster across CPU cores.
- ✅ Use a CDN for static assets.
- ✅ Profile before and after every change.

---

# 🛡️ D. Error Handling (Deep Dive)

### 1. Two categories of errors (very important distinction)

- **Operational errors** — *expected* runtime problems: invalid input, DB down, file not found, network timeout. **Handle these gracefully** and keep running.
- **Programmer errors** — *bugs*: `undefined is not a function`, wrong types. **Don't try to recover** — fix the code; let the process crash and restart.

**Interview gold:** "I handle operational errors gracefully but let programmer errors crash the process, because after a bug the app is in an unknown state — restarting is safer than continuing."

---

### 2. Handling errors in each async style

```js
// Callbacks (error-first)
fs.readFile('f.txt', (err, data) => {
  if (err) return handle(err);
  use(data);
});

// Promises
doThing().then(use).catch(handle);

// async/await (cleanest)
try {
  const data = await doThing();
  use(data);
} catch (err) {
  handle(err);
}
```

---

### 3. Custom error classes

Create typed errors so you can handle them differently and set correct HTTP status codes.

```js
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true; // it's an expected error
    Error.captureStackTrace(this, this.constructor);
  }
}

class NotFoundError extends AppError {
  constructor(msg = 'Not found') { super(msg, 404); }
}

// Usage
if (!user) throw new NotFoundError('User not found');
```

---

### 4. Centralized error-handling middleware (Express)

Instead of handling errors in every route, funnel them to **one** place.
Express error middleware has **four** arguments `(err, req, res, next)`.

```js
// Routes just throw or call next(err)
app.get('/user/:id', async (req, res, next) => {
  try {
    const user = await getUser(req.params.id);
    if (!user) throw new NotFoundError();
    res.json(user);
  } catch (err) {
    next(err); // pass to the error handler
  }
});

// One central handler at the very end
app.use((err, req, res, next) => {
  const status = err.statusCode || 500;
  console.error(err); // log full error server-side
  res.status(status).json({
    error: err.isOperational ? err.message : 'Internal server error', // hide bug details
  });
});
```

**Tip:** Wrap async routes in a helper so you don't repeat try/catch:

```js
const asyncHandler = (fn) => (req, res, next) =>
  Promise.resolve(fn(req, res, next)).catch(next);

app.get('/user/:id', asyncHandler(async (req, res) => {
  const user = await getUser(req.params.id);
  res.json(user);
}));
```

---

### 5. Global safety nets (last resort)

```js
process.on('unhandledRejection', (reason) => {
  console.error('Unhandled Rejection:', reason);
  // log it; consider a graceful shutdown
});

process.on('uncaughtException', (err) => {
  console.error('Uncaught Exception:', err);
  // App state is now unreliable → log, then exit and let PM2/Docker restart
  process.exit(1);
});
```

**Rule:** These are safety nets, **not** your primary strategy. Handle errors locally.
On `uncaughtException`, log and **restart** — don't resume.

---

### 6. Graceful error recovery patterns

- **Retry with backoff** — for transient failures (network blips), retry a few times with increasing delay.
- **Circuit breaker** — stop calling a failing service for a while to let it recover (`opossum` library).
- **Timeouts** — never wait forever; abort slow calls (`AbortController` / `Promise.race`).
- **Fallbacks** — serve cached/default data when a dependency is down.

```js
// Simple retry with exponential backoff
async function retry(fn, attempts = 3, delay = 200) {
  for (let i = 0; i < attempts; i++) {
    try { return await fn(); }
    catch (err) {
      if (i === attempts - 1) throw err;
      await new Promise(r => setTimeout(r, delay * 2 ** i)); // 200ms, 400ms, 800ms
    }
  }
}
```

---

### 7. Error-handling best practices summary

- ✅ Distinguish operational vs programmer errors.
- ✅ Always `catch` Promise rejections / use try-catch with async-await.
- ✅ Use custom error classes with status codes.
- ✅ Centralize handling in one Express middleware.
- ✅ Log full errors server-side; return safe messages to clients.
- ✅ Never leak stack traces to users in production.
- ✅ Use global handlers only to log + restart.
- ✅ Add retries, timeouts, and circuit breakers for external calls.
