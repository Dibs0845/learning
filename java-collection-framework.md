# Java Collections Framework — Interview Questions & Answers

A complete reference covering fundamentals, internals, comparisons, and real-world scenario questions for Java Collections Framework (JCF) interviews.

---

## Table of Contents
1. [Basics & Hierarchy](#1-basics--hierarchy)
2. [List Interface](#2-list-interface)
3. [Set Interface](#3-set-interface)
4. [Map Interface](#4-map-interface)
5. [Queue & Deque](#5-queue--deque)
6. [Iterators & Fail-Fast vs Fail-Safe](#6-iterators--fail-fast-vs-fail-safe)
7. [Comparable vs Comparator](#7-comparable-vs-comparator)
8. [Concurrent Collections](#8-concurrent-collections)
9. [Internal Working (Deep Dive)](#9-internal-working-deep-dive)
10. [Generics & Collections Utility](#10-generics--collections-utility)
11. [Performance & Complexity](#11-performance--complexity)
12. [Scenario-Based Questions](#12-scenario-based-questions)

---

## 1. Basics & Hierarchy

### Q1. What is the Java Collections Framework?
A unified architecture (interfaces, implementations, algorithms) for storing and manipulating groups of objects. Root interfaces: `Collection` (List, Set, Queue) and `Map` (not a `Collection` subtype, but part of the framework).

```
Iterable
   └── Collection
         ├── List (ArrayList, LinkedList, Vector, Stack)
         ├── Set (HashSet, LinkedHashSet, TreeSet)
         └── Queue (PriorityQueue, ArrayDeque, LinkedList)
                └── Deque

Map (separate hierarchy)
   ├── HashMap
   ├── LinkedHashMap
   ├── TreeMap (SortedMap → NavigableMap)
   └── Hashtable / ConcurrentHashMap
```

### Q2. Why doesn't `Map` extend `Collection`?
- Conceptually, `Collection` holds single elements; `Map` holds key-value pairs — a fundamentally different contract.
- Methods like `add(Object)` don't make sense for a Map (you need `put(key, value)`).
- If `Map` extended `Collection`, iterating would be ambiguous — over keys, values, or entries?

### Q3. Array vs Collection?
| Array | Collection |
|---|---|
| Fixed size | Dynamically grows |
| Can hold primitives & objects | Only holds objects (autoboxing for primitives) |
| No built-in utility methods | Rich API (sort, search, sync, etc.) |
| Faster for fixed-size numeric data | Better for dynamic, complex data structures |

### Q4. What is the difference between `Collection` and `Collections`?
- `Collection` — root interface of the framework.
- `Collections` — a utility class (`java.util.Collections`) with static helper methods (`sort()`, `synchronizedList()`, `unmodifiableList()`, `emptyList()`, etc.).

---

## 2. List Interface

### Q5. ArrayList vs LinkedList
| ArrayList | LinkedList |
|---|---|
| Backed by dynamic (resizable) array | Backed by doubly linked list |
| O(1) random access (`get(i)`) | O(n) random access |
| O(n) insertion/deletion in middle (shifting) | O(1) insertion/deletion once position found |
| Less memory overhead | More memory overhead (node + 2 pointers per element) |
| Better for read-heavy workloads | Better for frequent insert/delete at ends (also implements `Deque`) |

### Q6. How does ArrayList grow internally?
- Default initial capacity = 10 (lazily created on first `add()` since Java 8; empty array `{}` until then).
- On overflow, new capacity = `oldCapacity + (oldCapacity >> 1)` (i.e., **1.5x** growth), via `Arrays.copyOf()`.
- Growth is O(n) but amortized O(1) per `add()`.

### Q7. ArrayList vs Vector vs CopyOnWriteArrayList
| ArrayList | Vector | CopyOnWriteArrayList |
|---|---|---|
| Not synchronized | Synchronized (every method) | Thread-safe via copy-on-write |
| Fast | Slower due to sync overhead | Slow writes, fast/lock-free reads |
| Grows by 50% | Grows by 100% (doubles) | Grows by copying entire array on write |
| Fail-fast iterator | Fail-fast iterator | Fail-safe iterator (snapshot) |

### Q8. What is `Arrays.asList()` and its pitfall?
Returns a **fixed-size list backed by the array** — `add()`/`remove()` throw `UnsupportedOperationException`. Also, mutating the list mutates the backing array and vice versa. To get a mutable, independent list: `new ArrayList<>(Arrays.asList(...))`.

### Q9. How to make a List immutable?
```java
List<Integer> l1 = List.of(1, 2, 3);                 // Java 9+, truly immutable
List<Integer> l2 = Collections.unmodifiableList(list); // read-only view (backing list can still change)
```

---

## 3. Set Interface

### Q10. HashSet vs LinkedHashSet vs TreeSet
| HashSet | LinkedHashSet | TreeSet |
|---|---|---|
| Backed by `HashMap` | Backed by `LinkedHashMap` | Backed by `TreeMap` (Red-Black tree) |
| No ordering guarantee | Maintains insertion order | Sorted order (natural / Comparator) |
| O(1) add/remove/contains | O(1) add/remove/contains | O(log n) add/remove/contains |
| Allows one `null` | Allows one `null` | No `null` (NPE on compare) |

### Q11. How does HashSet ensure uniqueness?
Internally, `HashSet.add(e)` calls `map.put(e, PRESENT)` where `PRESENT` is a dummy static `Object`. Uniqueness relies on the key's `hashCode()` and `equals()` — same contract as `HashMap`.

### Q12. Contract between `equals()` and `hashCode()`
- If two objects are equal via `equals()`, they **must** have the same `hashCode()`.
- Equal hash codes do **not** imply equal objects (hash collision is legal).
- Violating this contract breaks hash-based collections (e.g., duplicates appear in a `HashSet`, or `get()` fails to find an existing key).

### Q13. Can you store a mutable object as a HashSet/HashMap key and mutate it afterward?
Technically yes, but it's a bug waiting to happen — if you mutate a field used in `hashCode()`/`equals()` after insertion, the object gets "lost" (bucket lookup uses new hash, but it's stored in old bucket). Best practice: use immutable keys.

---

## 4. Map Interface

### Q14. HashMap vs Hashtable vs LinkedHashMap vs TreeMap vs ConcurrentHashMap
| | HashMap | Hashtable | LinkedHashMap | TreeMap | ConcurrentHashMap |
|---|---|---|---|---|---|
| Null keys/values | 1 null key, multiple null values | None allowed | Same as HashMap | No null key (null values ok) | No null key/value |
| Thread-safe | No | Yes (synchronized) | No | No | Yes (segment/bucket-level locking) |
| Ordering | None | None | Insertion or access order | Sorted | None |
| Performance | Fast | Slow (legacy) | Slightly slower than HashMap | O(log n) | Fast under concurrency |
| Since | 1.2 | 1.0 (legacy) | 1.4 | 1.2 | 1.5 |

### Q15. How does HashMap work internally? *(classic deep-dive — see Section 9)*

### Q16. What happens when two keys have the same hashCode but are not equal (collision)?
They land in the same bucket. Java resolves collisions via **chaining** (linked list of `Node<K,V>`); since Java 8, if a bucket's chain length exceeds `TREEIFY_THRESHOLD` (8) and table size ≥ 64, it converts to a **Red-Black tree** for O(log n) worst-case lookup instead of O(n).

### Q17. Why is HashMap's default capacity 16 and load factor 0.75?
- Capacity must be a power of 2 for efficient bit-masking (`hash & (n-1)`) instead of modulo.
- Load factor 0.75 balances time/space cost — too low wastes memory (frequent resize), too high increases collision chains.
- Resize threshold = capacity × load factor = 12; when size exceeds this, table doubles.

### Q18. How does `HashMap.get(key)` compute the bucket index?
```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
// index = hash & (capacity - 1)
```
The `h ^ (h >>> 16)` spreads high bits into low bits ("hash spreading") to reduce collisions since bucket index only uses low bits.

### Q19. TreeMap — how does it maintain order and what's the complexity?
Backed by a **Red-Black tree** (self-balancing BST). Uses natural ordering (`Comparable`) or a supplied `Comparator`. `get/put/remove` are O(log n). Implements `NavigableMap` — supports `floorKey`, `ceilingKey`, `firstKey`, `lastKey`, `headMap`, `tailMap`, `subMap`.

### Q20. Difference between `HashMap` and `IdentityHashMap`?
`IdentityHashMap` uses reference equality (`==`) instead of `equals()`/`hashCode()` for key comparison — rarely used, mainly for topology-preserving object graphs or proxies.

### Q21. What's `WeakHashMap` used for?
Keys are held via `WeakReference`. If a key has no other strong references, it becomes eligible for GC and its entry is automatically removed — useful for caches that shouldn't prevent GC of keys.

### Q22. `Map.Entry` — what is it?
A nested interface representing a single key-value pair (`getKey()`, `getValue()`, `setValue()`). Used when iterating via `map.entrySet()` — more efficient than iterating `keySet()` and calling `get()` for each (avoids extra lookups).

### Q23. `computeIfAbsent`, `computeIfPresent`, `merge`, `getOrDefault` — usage?
```java
map.computeIfAbsent(key, k -> new ArrayList<>()).add(value); // multi-map pattern
map.merge(word, 1, Integer::sum);                             // word frequency counter
map.getOrDefault(key, 0);                                     // avoid null check
```

---

## 5. Queue & Deque

### Q24. Queue vs Deque
- `Queue` — FIFO; `offer/poll/peek`.
- `Deque` (Double-Ended Queue) — insertion/removal from both ends; can act as Queue (FIFO) or Stack (LIFO) via `push/pop`.

### Q25. Why is `ArrayDeque` preferred over `Stack`/`LinkedList` for stack operations?
`Stack` extends the legacy, synchronized `Vector` (unnecessary overhead for single-threaded use). `ArrayDeque` is faster, unsynchronized, and has no capacity restrictions — the official recommendation for stack/queue use since Java 6.

### Q26. PriorityQueue — internals?
Backed by a **binary heap** (array-based, complete binary tree). Default is a min-heap (natural ordering) or ordered per supplied `Comparator`. `offer`/`poll` are O(log n); `peek` is O(1). Not thread-safe (use `PriorityBlockingQueue` for concurrency).

### Q27. BlockingQueue implementations and when to use them?
- `ArrayBlockingQueue` — bounded, array-backed, fair-lock option — for fixed-capacity producer-consumer.
- `LinkedBlockingQueue` — optionally bounded, higher throughput (separate locks for head/tail).
- `PriorityBlockingQueue` — unbounded, priority ordering.
- `SynchronousQueue` — zero capacity; each `put` waits for a `take` (direct hand-off) — used in `Executors.newCachedThreadPool()`.
- `DelayQueue` — elements become available only after a delay expires — useful for scheduling/retries.

---

## 6. Iterators & Fail-Fast vs Fail-Safe

### Q28. Fail-Fast vs Fail-Safe iterators
| Fail-Fast | Fail-Safe |
|---|---|
| Throws `ConcurrentModificationException` if the collection is structurally modified during iteration | Iterates over a clone/snapshot; no exception |
| Examples: `ArrayList`, `HashMap`, `HashSet` iterators | Examples: `CopyOnWriteArrayList`, `ConcurrentHashMap` |
| Detected via `modCount` field | No shared `modCount` check |
| Memory efficient (no copy) | Extra memory for snapshot/copy |

### Q29. How does `ConcurrentModificationException` (CME) get detected?
Every structural modification (add/remove, not `set`) increments `modCount`. The iterator captures `expectedModCount` at creation; each `next()` call checks `modCount == expectedModCount`, throwing CME on mismatch. Note: it's a **best-effort** check, not a guarantee (fail-fast is not bulletproof under concurrent access).

### Q30. How to safely remove elements while iterating?
```java
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    if (it.next().equals("x")) it.remove(); // Iterator.remove() is safe
}
// Or Java 8+:
list.removeIf(s -> s.equals("x"));
```
Calling `list.remove()` directly inside a for-each loop throws CME.

### Q31. `Iterator` vs `ListIterator` vs `Enumeration`
- `Iterator` — forward-only, `remove()` supported.
- `ListIterator` — bidirectional (`hasPrevious/previous`), supports `add()`, `set()`, and index access — only for `List`.
- `Enumeration` — legacy (pre-Java-2, used by `Vector`/`Hashtable`), no `remove()`.

---

## 7. Comparable vs Comparator

### Q32. Comparable vs Comparator
| Comparable | Comparator |
|---|---|
| `java.lang` | `java.util` |
| `int compareTo(T o)` — defines *natural ordering*, implemented by the class itself | `int compare(T o1, T o2)` — external, defines *custom ordering* |
| One ordering per class | Multiple orderings possible |
| `Collections.sort(list)` | `Collections.sort(list, comparator)` |

### Q33. Example: sort a list of employees by salary, then by name (multi-field)
```java
list.sort(Comparator.comparing(Employee::getSalary)
                     .thenComparing(Employee::getName));

list.sort(Comparator.comparing(Employee::getSalary, Comparator.reverseOrder()));
```

### Q34. What happens if `compareTo()` is inconsistent with `equals()`?
Not required by contract, but `TreeSet`/`TreeMap` use `compareTo`/`compare` for **both ordering and equality checks** — if inconsistent, `Set` may accept "duplicate" elements that are `equals()` but `compareTo() != 0` treats them as distinct, or vice versa silently drop non-equal elements treated as duplicates.

---

## 8. Concurrent Collections

### Q35. How does ConcurrentHashMap achieve thread-safety without locking the whole map?
- **Java 7 and earlier**: Segment-based locking — map divided into segments (default 16), each with its own lock; concurrent writes allowed across different segments.
- **Java 8+**: Segments removed. Uses **CAS (Compare-And-Swap)** operations for insertion into empty buckets, and synchronizes only on the **first node of a bucket (bin)** when there's a collision — much finer granularity. Reads are mostly lock-free (`volatile` reads).

### Q36. Why is `Hashtable`/`Collections.synchronizedMap()` less efficient than `ConcurrentHashMap`?
They synchronize the **entire map** on every operation (single lock) — no concurrent reads/writes allowed. `ConcurrentHashMap` allows full concurrent reads and highly concurrent writes (bucket-level locking).

### Q37. Does ConcurrentHashMap allow null keys/values?
No — for both. Reason: in a concurrent context, `map.get(key) == null` is ambiguous (key absent vs. value is null) and can't be reliably disambiguated with `containsKey()` due to race conditions between threads.

### Q38. CopyOnWriteArrayList — how does it work, and downsides?
Every mutative operation (`add`, `set`, `remove`) creates a **new copy** of the underlying array. Iterators operate on a fixed snapshot (no CME, no visibility of concurrent changes). Great for **read-heavy, write-rare** scenarios (e.g., listener lists). Downside: O(n) memory/time per write; not suitable for write-heavy use cases.

### Q39. `Collections.synchronizedList()` vs `CopyOnWriteArrayList`?
`synchronizedList` wraps the list with a single lock per operation (still fail-fast; must manually synchronize during iteration). `CopyOnWriteArrayList` is lock-free for reads and fail-safe for iteration, but expensive writes.

---

## 9. Internal Working (Deep Dive)

### Q40. Walk through `HashMap.put(key, value)` step by step.
1. Compute `hash(key)` (spread function, see Q18).
2. If table is `null`/empty, `resize()` (lazy initialization).
3. Compute bucket index: `(n - 1) & hash`.
4. If bucket is empty → insert new `Node` directly.
5. If bucket occupied:
   - If first node's key matches (hash + equals) → will overwrite value.
   - Else if node is `TreeNode` → insert into red-black tree.
   - Else traverse linked list; if key found → overwrite; if end reached → append new node. If chain length ≥ 8 and capacity ≥ 64, treeify the bin.
6. If `size > threshold` (capacity × loadFactor) → `resize()` (double capacity).
7. Return old value (or `null`).

### Q41. What happens during HashMap `resize()`?
- New table = double the old capacity.
- Every existing node is rehashed and redistributed. Java 8 optimization: since capacity is always power of 2, each old bucket's nodes split into exactly two new buckets — "low" (same index) and "high" (index + oldCapacity) — determined by checking one extra bit (`hash & oldCapacity`), avoiding full rehash computation.

### Q42. Why is HashMap not thread-safe? What can go wrong under concurrent modification?
- Lost updates: two threads calling `put()` simultaneously can overwrite each other's entries.
- **Java 7 specific bug**: concurrent `resize()` could create a **circular linked list**, causing `get()` to infinite-loop (100% CPU) — a well-known production issue. Java 8's `resize` logic (splitting into low/high lists preserving order) avoids this specific defect, but HashMap is still fundamentally not thread-safe (data corruption / lost writes still possible).

### Q43. ArrayList internal resizing — show the growth formula and why 1.5x (not 2x)?
```java
int newCapacity = oldCapacity + (oldCapacity >> 1); // 1.5x
```
1.5x growth balances between too-frequent resizing (small factor) and wasted memory (large factor like 2x). This is a classic amortized-analysis tradeoff — both 1.5x and 2x give amortized O(1) `add()`, but 1.5x wastes less memory on average and allows reuse of previously freed memory blocks in some allocators.

### Q44. How does `LinkedHashMap` implement LRU cache behavior?
`LinkedHashMap` maintains a doubly-linked list across entries. Constructor `LinkedHashMap(capacity, loadFactor, accessOrder=true)` reorders entries by **access order** (most recently used moves to the end). Override `removeEldestEntry()` to evict the oldest entry once a size threshold is exceeded:
```java
class LRUCache<K,V> extends LinkedHashMap<K,V> {
    private final int capacity;
    LRUCache(int capacity) { super(16, 0.75f, true); this.capacity = capacity; }
    protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
        return size() > capacity;
    }
}
```

### Q45. Why must `hashCode()` be overridden when `equals()` is overridden?
Explained in Q12 — but concretely: without a matching `hashCode()`, two "equal" objects could map to different buckets, so `HashSet.contains()` or `HashMap.get()` would fail to find an entry that logically exists — silently breaking the collection's correctness.

---

## 10. Generics & Collections Utility

### Q46. What is type erasure and how does it affect Collections?
Generic type information (`List<String>`) exists only at compile time; the JVM erases it to raw `List` at runtime. Consequences:
- Cannot do `new T[]` or `instanceof List<String>`.
- Can't overload methods that differ only by generic type (`foo(List<String>)` vs `foo(List<Integer>)` — same erasure).
- Runtime type of `list.getClass()` is just `ArrayList`, no generic info.

### Q47. `? extends T` vs `? super T` (PECS principle)
**P**roducer **E**xtends, **C**onsumer **S**uper:
- `List<? extends Number>` — read-only (producer); can't add (compiler doesn't know the exact subtype).
- `List<? super Integer>` — write-safe (consumer); can add `Integer` or subtypes, but reading gives only `Object`.
```java
void copy(List<? extends T> src, List<? super T> dest) { ... } // Collections.copy signature
```

### Q48. Useful `Collections` utility methods
```java
Collections.unmodifiableList(list);
Collections.synchronizedList(list);
Collections.emptyList();
Collections.singletonList(x);
Collections.reverse(list);
Collections.shuffle(list);
Collections.max(list) / Collections.min(list);
Collections.frequency(list, element);
Collections.binarySearch(sortedList, key);
```

### Q49. `Arrays.sort()` vs `Collections.sort()` — algorithm used?
- Primitives: **Dual-Pivot Quicksort** (O(n log n) avg, in-place).
- Objects (`Arrays.sort(Object[])`, `Collections.sort()`): **TimSort** (hybrid merge sort + insertion sort) — stable sort, O(n log n) worst case, important because `Comparable`/`Comparator`-based sorts must be stable for predictable multi-key sorting.

---

## 11. Performance & Complexity

| Operation | ArrayList | LinkedList | HashSet | TreeSet | HashMap | TreeMap |
|---|---|---|---|---|---|---|
| add (end) | O(1) amortized | O(1) | O(1) avg | O(log n) | O(1) avg | O(log n) |
| add (middle/specific pos) | O(n) | O(n)* | — | — | — | — |
| get by index | O(1) | O(n) | — | — | — | — |
| get by key | — | — | O(1) avg | O(log n) | O(1) avg | O(log n) |
| contains | O(n) | O(n) | O(1) avg | O(log n) | O(1) avg | O(log n) |
| remove | O(n) | O(n)* | O(1) avg | O(log n) | O(1) avg | O(log n) |

\* O(1) once you have a reference/iterator to the node; O(n) to find it first.

### Q50. Worst-case HashMap `get()`/`put()` complexity — is it really O(1)?
Average case O(1), but **worst case is O(n)** with poor hash distribution before Java 8 (all keys collide → linked list). Since Java 8, worst case improves to **O(log n)** once a bucket treeifies (≥8 collisions, capacity ≥ 64). This treeification was actually a mitigation for hash-flooding DoS attacks (e.g., malicious `String` keys engineered to collide).

---

## 12. Scenario-Based Questions

### S1. You need to store millions of key-value pairs and frequently check "does this key exist" with best average performance. Which collection?
**`HashMap`** (or `ConcurrentHashMap` if multi-threaded). O(1) average lookup. If insertion order matters for iteration, use `LinkedHashMap`. If sorted iteration is needed, `TreeMap` (O(log n) trade-off).

### S2. Multiple threads read a configuration list frequently, but it's updated rarely (e.g., feature flags refreshed every few minutes). What collection?
**`CopyOnWriteArrayList`**. Reads are lock-free and fast; the rare writes pay the O(n) copy cost, which is acceptable given the low write frequency.

### S3. You're implementing an LRU cache for a web session store with a max size of 1000. What's your approach?
Extend `LinkedHashMap` with `accessOrder=true` and override `removeEldestEntry()` (see Q44). For thread-safety, wrap with `Collections.synchronizedMap()` or use `ConcurrentLinkedHashMap`/`Caffeine` cache library in production.

### S4. A `HashMap<String, List<String>>` needs to group values by key (multi-map). Show idiomatic code.
```java
Map<String, List<String>> grouped = new HashMap<>();
for (Entry entry : data) {
    grouped.computeIfAbsent(entry.getKey(), k -> new ArrayList<>())
           .add(entry.getValue());
}
// Or with Streams:
Map<String, List<Entry>> grouped2 = data.stream()
        .collect(Collectors.groupingBy(Entry::getKey));
```

### S5. You put a custom object `Employee` into a `HashSet` but duplicates are appearing even though you overrode `equals()`. What's wrong?
Most likely `hashCode()` was **not** overridden (or overridden inconsistently) — equal objects landing in different buckets are both accepted as "new" since HashSet never even calls `equals()` on objects in different buckets. Fix: override both, based on the same fields.

### S6. You're iterating a `List` and conditionally removing elements; you get `ConcurrentModificationException`. How do you fix it, and what's the *best* fix?
- Use `Iterator.remove()` inside the loop, or
- Use `list.removeIf(predicate)` (cleanest, Java 8+), or
- Iterate a copy: `for (String s : new ArrayList<>(list))` (less efficient, avoid for large lists).
Avoid: `list.remove()` inside a for-each — that directly causes CME.

### S7. In a high-throughput producer-consumer system, which Queue implementation would you pick and why?
`LinkedBlockingQueue` (or `ArrayBlockingQueue` for bounded backpressure) — built-in blocking `put()`/`take()`, thread-safe, avoids manual `wait/notify`. If producers must never proceed without a matching consumer (backpressure/direct hand-off), use `SynchronousQueue` (used internally by `Executors.newCachedThreadPool()`).

### S8. You need a Set that maintains elements in sorted order and want to efficiently find "the next element greater than X". Which collection and method?
`TreeSet` — use `higher(X)` (strictly greater), `ceiling(X)` (≥ X), `lower(X)` / `floor(X)` for the other direction. Backed by `NavigableSet`, O(log n).

### S9. Why did your team's HashMap-based cache cause 100% CPU usage / hang in production (Java 7 era) under concurrent load?
Classic **HashMap resize race condition** in Java 7 — concurrent `put()` calls during a `resize()` could corrupt the internal linked list into a **cycle**, causing `get()` to loop forever. Fix: use `ConcurrentHashMap` for any multi-threaded map access — never share a plain `HashMap` across threads without external synchronization.

### S10. You need to remove duplicate objects from a `List<Employee>` based on a specific field (e.g., `email`), not the whole object. How?
```java
List<Employee> unique = employees.stream()
        .collect(Collectors.collectingAndThen(
                Collectors.toCollection(() -> new TreeSet<>(Comparator.comparing(Employee::getEmail))),
                ArrayList::new));

// Or manually:
Map<String, Employee> byEmail = new LinkedHashMap<>();
employees.forEach(e -> byEmail.putIfAbsent(e.getEmail(), e));
List<Employee> unique2 = new ArrayList<>(byEmail.values());
```

### S11. Should you use `Vector`/`Stack`/`Hashtable` in new code? Why do they still exist?
No — they're **legacy** (pre-Java-2, retrofitted into the framework). They're retained purely for backward compatibility. Prefer: `ArrayList`/`ArrayDeque` + explicit synchronization or `Collections.synchronizedList()`/`ConcurrentHashMap` for thread-safety, which offer better performance and more flexibility (e.g., choice of lock granularity).

### S12. Design a thread-safe counter/frequency map (e.g., counting word occurrences) accessed by many threads concurrently.
```java
ConcurrentHashMap<String, LongAdder> counts = new ConcurrentHashMap<>();
counts.computeIfAbsent(word, w -> new LongAdder()).increment();
```
`LongAdder` outperforms `AtomicLong` under high contention (internally stripes counters across cells to reduce CAS contention, sums on read).

### S13. Given a `TreeMap`, how would you efficiently get all entries within a range, e.g., ages 18–30?
```java
SortedMap<Integer, Person> range = treeMap.subMap(18, true, 30, true); // inclusive both ends
```
O(log n) to locate boundaries, then a view (not a copy) backed by the original map.

### S14. Your application needs a Set where insertion order must be preserved for predictable UI display, but also needs fast lookups. Which collection?
`LinkedHashSet` — O(1) average lookup like `HashSet`, plus maintains insertion order via internal doubly-linked list (unlike `HashSet`, which has no ordering guarantee).

### S15. How would you make an existing non-thread-safe `HashMap` usage thread-safe with minimal code change, and what's the trade-off vs. rewriting with `ConcurrentHashMap`?
Minimal change: `Map<K,V> map = Collections.synchronizedMap(new HashMap<>());` — but you must manually synchronize on the map object during iteration (still fail-fast). Trade-off: single lock for all operations (no concurrent reads), vs. `ConcurrentHashMap`'s fine-grained locking/CAS which scales far better under contention — prefer `ConcurrentHashMap` unless working with legacy code that can't be changed.

---

## Quick Cheat Sheet

- **Need fast random access?** → `ArrayList`
- **Need frequent insert/delete at both ends?** → `ArrayDeque`
- **Need uniqueness + no order?** → `HashSet`
- **Need uniqueness + insertion order?** → `LinkedHashSet`
- **Need uniqueness + sorted order?** → `TreeSet`
- **Need key-value + fastest lookup?** → `HashMap`
- **Need key-value + insertion/access order (LRU)?** → `LinkedHashMap`
- **Need key-value + sorted keys/range queries?** → `TreeMap`
- **Thread-safe, read-heavy?** → `CopyOnWriteArrayList` / `ConcurrentHashMap`
- **Thread-safe, write-heavy / producer-consumer?** → `ConcurrentHashMap`, `BlockingQueue` implementations
- **Never use in new code:** `Vector`, `Stack`, `Hashtable` (legacy, use modern equivalents)
