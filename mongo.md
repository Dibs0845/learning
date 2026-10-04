# MongoDB Senior Developer / Architect — Interview Questions & Answers

> Depth-first prep covering data modeling, indexing, aggregation, transactions,
> replication, sharding, performance, security, and architecture/scenario design.

---

## 1. Fundamentals & Data Modeling

### Q1. When would you embed documents vs. reference them?

**Embed** when:
- Data is accessed together ("has-a" / contains relationship).
- One-to-few or one-to-many with bounded growth (e.g., a user's addresses).
- You want atomic single-document updates (a single doc write is always atomic).
- Read performance matters more than duplication cost.

**Reference** when:
- One-to-many with **unbounded** growth (e.g., a post's comments that can grow forever) — avoids the 16 MB document limit and the "massive array" anti-pattern.
- Many-to-many relationships.
- The referenced entity is large, updated independently, or shared across many parents.

**Rule of thumb:** *Data that is queried together should be stored together.* Model around your access patterns (read/write ratio, query shape), not around normalization theory.

### Q2. What is the 16 MB document size limit and how do you work around it?

Every BSON document is capped at 16 MB. Workarounds:
- **Referencing / bucketing** instead of unbounded arrays.
- **GridFS** for large binary blobs (files > 16 MB) — splits into 255 KB chunks across `fs.files` and `fs.chunks`.
- **Outlier pattern** — move rare huge cases into a separate collection.

### Q3. Explain common MongoDB schema design patterns.

- **Bucket pattern** — group time-series/IoT readings into buckets (e.g., one doc per sensor per hour holding an array of readings). Reduces doc count and index size.
- **Computed pattern** — precompute expensive aggregates (sums, averages) on write so reads are cheap.
- **Subset pattern** — store the hot subset (e.g., latest 10 reviews) embedded, the rest in another collection.
- **Extended reference** — duplicate a few frequently-read fields from the referenced doc to avoid a join (denormalization).
- **Schema versioning** — carry a `schema_version` field so you can evolve schema without a big-bang migration.
- **Outlier pattern** — handle the few documents that break your assumptions separately.
- **Polymorphic pattern** — different shapes in one collection with a discriminator field.

### Q4. Is MongoDB schemaless? What is schema validation?

It's **flexible-schema**, not schemaless — documents in a collection can differ, but you should still design intentionally. Use **JSON Schema validation** (`$jsonSchema`) via `validator` on `createCollection`/`collMod` with `validationLevel` (strict/moderate) and `validationAction` (error/warn) to enforce structure where needed.

---

## 2. Indexing

### Q5. What index types does MongoDB support?

Single-field, **compound**, multikey (on arrays), **text**, **geospatial** (`2d`, `2dsphere`), **hashed** (for hashed sharding), **wildcard** (`$**`), **TTL** (auto-expire docs), **partial** (index subset matching a filter), **sparse** (only docs where field exists), and **unique**.

### Q6. Explain the ESR rule for compound indexes.

Order compound index keys as **Equality → Sort → Range**:
1. **Equality** fields first (exact matches, e.g., `status: "active"`).
2. **Sort** fields next (so the index also satisfies the sort, avoiding an in-memory `SORT` stage).
3. **Range** fields last (`$gt`, `$lt`, `$in`).

Following ESR lets a single index serve filtering, sorting, and range scanning efficiently.

### Q7. What is a covered query?

A query where **all fields** (filter, sort, projection) are satisfied by the index, so MongoDB never touches the documents (returns data straight from the index). Requires the projection to exclude `_id` unless `_id` is in the index. Check for `totalDocsExamined: 0` in `explain()`.

### Q8. How do you diagnose a slow query?

- `db.coll.find(...).explain("executionStats")` — inspect `winningPlan`, `stage` (`COLLSCAN` = no index used, bad), `totalDocsExamined` vs `nReturned` (ratio near 1 is ideal), `executionTimeMillis`, and whether a `SORT` stage appears (in-memory sort, 32 MB limit).
- **Database Profiler** (`db.setProfilingLevel(1, { slowms: 100 })`) logs slow ops to `system.profile`.
- `mongotop` / `mongostat` for live server stats.
- Atlas Performance Advisor / query analytics in production.

### Q9. What is a multikey index and its limitations?

An index on an array field — MongoDB creates an index entry per array element. Limits: you **cannot** create a compound multikey index on **two array fields** (that would be a cartesian explosion). Multikey indexes can't be used to cover a query for the array field, and bounds behavior with `$elemMatch` differs.

### Q10. Index trade-offs — what's the cost of over-indexing?

Each index consumes RAM (working set) and disk, and **slows writes** (every insert/update/delete must maintain all relevant indexes). Aim for indexes that support your real query patterns; drop unused ones (`$indexStats` shows usage). The working set of indexes should ideally fit in RAM.

---

## 3. Aggregation Framework

### Q11. Walk through the aggregation pipeline. Name key stages.

A pipeline processes documents through ordered stages, each transforming the stream:
`$match`, `$project`, `$group`, `$sort`, `$limit`, `$skip`, `$unwind`, `$lookup` (left outer join), `$addFields`/`$set`, `$facet` (multiple sub-pipelines in one pass), `$bucket`/`$bucketAuto`, `$graphLookup` (recursive/hierarchical), `$merge`/`$out` (write results to a collection), `$replaceRoot`, `$count`, `$sample`, `$setWindowFields` (window functions).

### Q12. How do you optimize an aggregation pipeline?

- Put `$match` and `$limit` **as early as possible** so filtering happens before heavy stages (the query planner also does some reordering automatically).
- `$match` early can use an index; a `$match` after `$group`/`$project` usually cannot.
- Use `$project` early to drop unneeded fields and shrink documents in the pipeline.
- Avoid `$unwind` + `$group` when an array operator (`$reduce`, `$map`, `$filter`) works in-place.
- Be aware of the **100 MB per-stage memory limit** — enable `allowDiskUse: true` for large sorts/groups (spills to disk).
- Use `explain()` on the pipeline; index usage only helps the initial `$match`/`$sort`.

### Q13. $lookup — how does it work and what are its performance concerns?

`$lookup` performs a left outer join to another collection in the **same database**. Performance concerns: it runs per input document (can be O(n) sub-queries) — ensure the foreign field is **indexed**. For large joins prefer the correlated sub-pipeline form and filter within it. Heavy joins often signal that the schema should have embedded/denormalized the data.

### Q14. Difference between $group and $bucket? And $facet use case?

- `$group` — groups by an arbitrary key and computes accumulators (`$sum`, `$avg`, `$push`, `$addToSet`, `$max`).
- `$bucket` / `$bucketAuto` — groups numeric/date values into ranges (histograms).
- `$facet` — runs **multiple independent pipelines** over the same input in a single stage; ideal for dashboards (e.g., faceted search: counts by category + price ranges + total, all at once).

### Q15. What are accumulator vs window functions?

Accumulators (`$sum`, `$avg`, ...) collapse groups. `$setWindowFields` (3.2+ semantics, GA in 5.0) computes over a sliding **window** of documents (moving averages, running totals, rank, `$shift`, `$derivative`) **without** collapsing — like SQL window functions.

---

## 4. Transactions & Consistency

### Q16. Does MongoDB support ACID transactions?

Yes. Single-document operations have always been atomic. **Multi-document ACID transactions** are supported on replica sets (4.0+) and sharded clusters (4.2+). Use them sparingly — they carry overhead and a default 60-second time limit. Prefer good schema design (embedding) so most operations stay single-document.

### Q17. Explain read concern and write concern.

**Write concern** (`w`) — durability/acknowledgment guarantee:
- `w: 1` — acknowledged by primary.
- `w: "majority"` — acknowledged by a majority of replica-set members (survives failover; default since 5.0).
- `j: true` — wait for on-disk journal commit.
- `wtimeout` — cap the wait.

**Read concern** — consistency/isolation of reads:
- `local` — latest data on the queried node (may roll back).
- `majority` — data acknowledged by a majority (won't be rolled back).
- `linearizable` — reflects all majority-acknowledged writes before it (strongest, primary only).
- `snapshot` — consistent snapshot, used in transactions.

### Q18. What is read preference?

Controls **which replica-set members** serve reads: `primary` (default), `primaryPreferred`, `secondary`, `secondaryPreferred`, `nearest`. Reading from secondaries scales reads but risks **stale data** (replication lag) and breaks read-your-writes unless using causal consistency.

### Q19. What are causal consistency and sessions?

A **client session** with causal consistency guarantees read-your-writes, monotonic reads/writes, and writes-follow-reads across operations — even when reading from secondaries — by tracking cluster/operation time. Essential when mixing primary writes and secondary reads.

### Q20. How does MongoDB handle transaction conflicts?

WiredTiger uses **optimistic concurrency / MVCC** with snapshot isolation. On a write conflict, one transaction aborts with a `TransientTransactionError`; the driver's retry logic (or your code) should retry the whole transaction. Keep transactions short and touch few documents to minimize conflicts.

---

## 5. Replication & High Availability

### Q21. How does a replica set work?

A replica set is a group of `mongod` nodes: one **primary** (accepts writes) and one or more **secondaries** (replicate via the primary's **oplog**, a capped collection). If the primary becomes unreachable, an **election** (Raft-like protocol) promotes a secondary. Odd number of voting members recommended for clean majorities.

### Q22. What is the oplog and how does replication stay consistent?

The **oplog** (`local.oplog.rs`) is a capped collection of idempotent operations. Secondaries tail it and apply operations in order. Oplog size determines how long a secondary can be offline before needing a full resync. Idempotency means re-applying an op is safe.

### Q23. What is an arbiter and when should you avoid it?

An **arbiter** votes in elections but holds no data. Use it to reach an odd voting count cheaply. Avoid when possible: it can't serve `w: "majority"` durability well (no data copy), and with an arbiter a two-data-node set can lose majority durability if one data node fails. Prefer a real data-bearing member.

### Q24. What happens during a failover and how do you minimize disruption?

Primary steps down → secondaries hold an election (~10–12s by default) → new primary elected. Minimize disruption with **retryable writes/reads** (default in modern drivers), majority write concern, and application-side connection retry. Rollbacks can occur for writes not replicated before failover (mitigated by `w: majority`).

### Q25. Hidden, delayed, and priority-0 members?

- **Priority 0** — never becomes primary (e.g., a DR/backup node in another region).
- **Hidden** — priority 0 + invisible to client read routing (for dedicated backup/analytics).
- **Delayed** — hidden member applying oplog with a time delay (e.g., 1 hour) — a "rolling backup" that protects against fat-finger operations.

---

## 6. Sharding & Scaling

### Q26. What is sharding and what are the cluster components?

Sharding horizontally partitions data across shards for scale beyond one machine's capacity. Components:
- **Shards** — each a replica set holding a subset of data.
- **`mongos`** — query router the app connects to.
- **Config servers** — a replica set storing cluster metadata (chunk ranges, mapping).

### Q27. How do you choose a shard key?

A good shard key has:
- **High cardinality** — many distinct values.
- **Low frequency** — no single value dominates.
- **Non-monotonic** — avoid ever-increasing keys (like a raw timestamp or ObjectId) which create a "hot shard" because all new writes hit the max chunk. Use **hashed** sharding or a compound key to spread writes.
- **Alignment with query patterns** — queries should include the shard key to enable **targeted** (single-shard) queries instead of **scatter-gather** (broadcast to all shards).

Since 4.4 you can **refine** a shard key (add suffix fields); 5.0+ allows **reshard**ing.

### Q28. Ranged vs hashed sharding?

- **Ranged** — chunks by contiguous key ranges. Good for range queries; risk of hot spots with monotonic keys and uneven distribution.
- **Hashed** — MongoDB hashes the key for even write distribution. Loses efficient range queries (they become scatter-gather).

Choice depends on whether even write distribution or range-query locality matters more.

### Q29. What are chunks and the balancer?

Data is divided into **chunks** (default range spans). The **balancer** migrates chunks between shards to keep them balanced. Since 6.0, balancing is based on **data size** rather than chunk count, and **auto-merging** is default; explicit chunk splitting was largely removed. Migrations add overhead — schedule balancing windows for write-heavy clusters if needed.

### Q30. Targeted vs scatter-gather queries?

If a query includes the shard key, `mongos` routes it to the specific shard(s) holding the data (**targeted** — fast, scalable). Without the shard key, `mongos` broadcasts to **all** shards and merges results (**scatter-gather** — slow, doesn't scale). Design shard keys so hot queries are targeted.

### Q31. When should you shard — and when not?

Shard when a single replica set can no longer hold the working set in RAM, hits write throughput limits, or storage limits. **Don't shard prematurely** — it adds operational complexity (config servers, mongos, balancer, harder joins/uniqueness). First exhaust vertical scaling, indexing, and schema optimization.

---

## 7. Performance & Operations

### Q32. What is the WiredTiger storage engine and how does its cache work?

Default engine since 3.2. Features **document-level concurrency** (multiple writers on the same collection), **compression** (snappy default for data, prefix compression for indexes), and MVCC. The **WiredTiger cache** defaults to `max(50% of (RAM − 1GB), 256MB)`. Keep the working set (hot data + indexes) within cache/RAM to avoid disk reads.

### Q33. What is the working set and why does it matter?

The working set is the data + indexes actively accessed. If it exceeds RAM, MongoDB pages to disk, causing severe latency. Monitor page faults, cache eviction, and resident memory. Scale RAM or shard when the working set outgrows a node.

### Q34. Bulk operations — how do you optimize high-volume writes?

- Use **`bulkWrite`** with ordered/unordered batches (unordered continues past errors and can parallelize).
- Increase batch sizes to amortize round trips.
- Consider lower write concern for non-critical ingest, then verify.
- For huge imports, drop non-essential indexes first and rebuild after.

### Q35. Common MongoDB anti-patterns.

- Unbounded arrays (docs growing toward 16 MB).
- Massive number of collections/indexes.
- Using `$where` / JavaScript in queries (no index, slow, security risk).
- Case-insensitive queries without a proper **collation** index (leading regex `/^.../i` can't use an index efficiently).
- Bloated documents with rarely-used fields (hurts working set).
- Not using projections (fetching whole docs when you need two fields).
- Scatter-gather on sharded clusters.
- Monotonically increasing shard keys.

### Q36. How does the query planner work / what is a plan cache?

The planner generates candidate plans, runs them in a trial (**plan ranking**), picks the winner by works/docs examined, and **caches** it keyed by query shape. You can inspect (`$planCacheStats`), clear it, or pin a plan with **index hints/filters**. Cache is invalidated on index changes or after enough writes.

### Q37. TTL indexes — how do they expire data?

A single-field index with `expireAfterSeconds` on a date field. A background thread (runs ~every 60s) deletes expired docs. Not real-time — deletion lag is expected. Great for sessions, logs, caches. Can't be compound; use `expireAfterSeconds: 0` with a specific expiry date field for per-document expiry.

### Q38. Change Streams — what and when?

Change streams let apps subscribe to real-time data changes (insert/update/delete/replace) using the oplog, via `watch()`. Resumable via a **resume token**. Use for cache invalidation, event-driven pipelines, CDC, notifications, cross-service sync — replaces error-prone oplog tailing. Requires a replica set.

---

## 8. Security

### Q39. How do you secure a MongoDB deployment?

- **Authentication** — enable auth (`--auth`); mechanisms: SCRAM (default), x.509 certs, LDAP, Kerberos (enterprise).
- **Authorization** — Role-Based Access Control (RBAC): built-in roles (`read`, `readWrite`, `dbAdmin`, `clusterAdmin`) and custom roles; principle of least privilege.
- **Network** — bind to private IPs, firewall, VPC peering; never expose to `0.0.0.0` publicly.
- **Encryption** — TLS/SSL in transit; encryption at rest (WiredTiger, enterprise/Atlas); **Client-Side Field Level Encryption (CSFLE)** and **Queryable Encryption** for sensitive fields.
- **Auditing** (enterprise) and keeping software patched.

### Q40. What is Client-Side Field Level Encryption / Queryable Encryption?

**CSFLE** encrypts specific fields in the driver **before** they reach the server — the server (and DBAs) never see plaintext. **Queryable Encryption** (GA 7.0) allows equality (and range) queries over encrypted data while it stays encrypted server-side. Used for PII/PHI/compliance (HIPAA, PCI).

---

## 9. Architecture & Scenario Questions

### Q41. Design a schema for an e-commerce platform (products, orders, inventory).

- **Products** — one doc per product with embedded variants/attributes (polymorphic for different categories); extended-reference frequently-read fields (name, price) into orders.
- **Orders** — embed a **snapshot** of line items (product name, price at purchase time) so historical orders don't change when the product does; reference `userId`.
- **Inventory** — separate collection updated atomically; use a transaction or a single-document `$inc` with a guard (`{ qty: { $gte: n } }`) to prevent overselling.
- **Cart** — often ephemeral (TTL) or separate collection.
- Reviews — subset pattern (embed latest few, rest referenced).

### Q42. How would you model a social feed / activity stream?

Options with trade-offs:
- **Fan-out on write** — push each new post into followers' feed collections. Fast reads, expensive writes (celebrity problem — millions of followers).
- **Fan-out on read** — query posts of followed users at read time. Cheap writes, expensive reads.
- **Hybrid** — fan-out on write for normal users, fan-out on read for high-follower accounts. Bucket feed entries. This is the pragmatic production answer.

### Q43. Design a time-series / IoT ingestion system.

Use the native **time-series collections** (5.0+) — optimized columnar storage, automatic bucketing, and lower storage. Specify `timeField`, `metaField` (e.g., sensorId), `granularity`. Add TTL for retention. Pre-6.0, hand-roll the **bucket pattern**. Use `$setWindowFields` for moving averages.

### Q44. How do you handle a "hot document" or counter under high contention?

- **Sharded counters** — split a counter into N sub-counters (random shard on write, sum on read) to avoid contention on one document.
- Use atomic `$inc` (single-doc atomic) rather than read-modify-write.
- For rate/analytics, consider approximate counters or batching increments in the app then flushing.

### Q45. Migration strategy: evolving schema on a live system with billions of docs.

- Use the **schema versioning pattern** — write new docs in the new shape; read code handles both versions.
- **Lazy migration** — upgrade a doc when it's next written/read.
- **Background batch migration** — throttled `bulkWrite` in ranges to avoid overwhelming the cluster; monitor replication lag.
- Never a big-bang `updateMany` on billions of docs during peak; do it in chunks with checkpoints.

### Q46. How do you ensure uniqueness across a sharded collection?

A **unique index** on a sharded collection must include the **shard key as a prefix** (MongoDB can only enforce uniqueness within a shard for the shard key). For uniqueness on a non-shard-key field, options: keep it unsharded, use a separate collection as a uniqueness registry with the unique field as `_id`, or enforce in the application layer.

### Q47. Read-heavy vs write-heavy scaling strategies.

- **Read-heavy** — add secondaries + read preference (with causal consistency), caching layer (Redis), covered queries, and denormalization. Reads scale with more replicas.
- **Write-heavy** — writes only go to the primary, so replicas don't help; **shard** to distribute writes, choose a write-distributing shard key (hashed), use unordered bulk writes, and consider relaxed write concern for non-critical data.

### Q48. How do you back up and restore MongoDB?

- **`mongodump`/`mongorestore`** — logical backup (BSON); fine for small/medium, not point-in-time by itself.
- **Filesystem/volume snapshots** — consistent snapshot of the data directory (requires journaling or fsyncLock); fast for large data.
- **Continuous / PITR** — Atlas continuous backups or Ops Manager / Cloud Manager give point-in-time recovery via oplog.
- Test restores regularly; a backup you can't restore isn't a backup.

---

## 10. Rapid-Fire / Gotchas

| Question | Answer |
|---|---|
| Difference between `find()` returning cursor vs array? | `find()` returns a lazy **cursor**; iterated in batches (default 101 docs then ~16 MB batches). |
| `updateOne` vs `replaceOne`? | `updateOne` applies operators to specific fields; `replaceOne` swaps the whole doc (except `_id`). |
| `$set` vs `$setOnInsert`? | `$setOnInsert` only applies when an upsert actually inserts. |
| Does `_id` have to be an ObjectId? | No — any unique, immutable value. ObjectId is default (12 bytes: timestamp + machine/process + counter). |
| What's an ObjectId's first 4 bytes? | A **timestamp** (creation second) — so ObjectIds are roughly time-sortable. |
| `countDocuments` vs `estimatedDocumentCount`? | `countDocuments` runs an accurate query (can be slow); `estimatedDocumentCount` uses metadata (fast, approximate). |
| Can you index a field inside an array of subdocuments? | Yes — multikey index on `arr.field`; query with `$elemMatch` for correct multi-condition matching. |
| What does `$exists` cost? | Can use a sparse/partial index but often scans; model presence explicitly if hot. |
| Capped collection? | Fixed-size, insertion-ordered, auto-overwrites oldest (used by oplog). No arbitrary deletes. |
| Difference `$in` vs `$or`? | `$in` on one field is index-friendly and preferred; `$or` across fields may need multiple indexes (index intersection). |
| Retryable writes — what makes an op retryable? | Idempotent, single-doc-targeting writes; the driver retries once on a transient network/failover error using a transaction number. |

---

## 11. Behavioral / Architect-Level

- **Trade-off articulation:** Be ready to justify embedding vs referencing, sharding vs vertical scaling, and consistency vs availability (CAP — MongoDB is CP by default with majority concerns, tunable toward AP).
- **Failure design:** Talk about retryable writes, majority concerns, monitoring (Atlas, Prometheus exporter), alerting on replication lag / oplog window / cache pressure.
- **Cost/perf:** Working-set sizing, index hygiene (`$indexStats`), read/write concern tuning, connection pooling (drivers pool by default; tune `maxPoolSize`).
- **When NOT to use MongoDB:** Heavy multi-entity transactional workloads with rigid relational integrity, complex ad-hoc JOIN-heavy analytics, or when strong normalized relational modeling is the natural fit — be honest that a RDBMS may suit better.

---

### Quick prep checklist
- [ ] Can whiteboard embed-vs-reference for any given access pattern.
- [ ] Can explain ESR and read an `explain()` plan out loud.
- [ ] Can design a shard key and defend it.
- [ ] Know read/write concern + read preference combinations for a given consistency requirement.
- [ ] Can describe a failover end-to-end and how the app survives it.
- [ ] Have one real story of diagnosing and fixing a slow query / hot shard.
