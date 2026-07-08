# MERN Stack Senior Engineer Interview Guide

> **Principal Engineer · Technical Interview Series**
> Experience: 7+ years | Target: Senior → Staff → Architect
> Companies: Amazon · Google · Microsoft · Atlassian · Adobe · Uber · Airbnb · Salesforce · Walmart
> Questions: 9 / 60+ covered

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

- Event Loop (Node)
- Streams
- Clustering
- Memory Leaks

### Express / MongoDB

- Middleware
- Indexing
- Aggregation
- Sharding

### System Design

- URL Shortener
- Chat Application
- Notifications
- Micro Frontends

### AWS / Security / Patterns

- EC2 / S3 / CloudFront
- XSS / CSRF / JWT
- Design Patterns
- Leadership
