# Event-Driven Architecture (EDA)

## 1. Core Idea
- **Request-driven**: Service A *calls* Service B and waits. A must know B, and is blocked until B replies.
- **Event-driven**: A service *announces a fact* (an event) and moves on. It doesn't know who listens. Others react if interested.

**Command vs Event**
- **Command** = "do this" — imperative, one known receiver, expects a result.
- **Event** = "this happened" — past tense, 0..N receivers, fire-and-forget.

> An event is an **immutable fact about the past**: `OrderPlaced`, `PaymentCaptured`, `UserSignedUp`. Always name events in past tense.

## 2. Three Building Blocks
| Role | Job |
|------|-----|
| **Producer / Publisher** | Emits an event when something notable happens. Doesn't know consumers. |
| **Broker / Event Bus** | Receives and delivers events. (Kafka, RabbitMQ, SNS/SQS, NATS, Pub/Sub) |
| **Consumer / Subscriber** | Subscribes to event types and reacts. Doesn't know producers. |

The broker in the middle gives **decoupling** — the most important property. Producers/consumers share only the *event schema*, never each other.

## 3. Concrete Example (checkout)
```
Order Service --emits--> "OrderPlaced" --> [ BROKER ]
                                              |
             +--------------------------------+-----------------------+
             v                                v                       v
      Payment Service               Inventory Service          Email Service
      (charge card)                 (reserve stock)            (send receipt)
```
- Order service publishes ONE event, returns to customer instantly.
- Consumers react independently & in parallel.
- Add fraud-detection later? Just subscribe it to `OrderPlaced` — change nothing in the order service.

## 4. Benefits vs Costs
**Benefits**
- Loose coupling — services evolve independently.
- Scalability — consumers scale on their own; broker buffers spikes.
- Resilience — if a consumer is down, events queue and process on recovery.
- Extensibility — new features subscribe instead of editing existing code.

**Costs**
- Eventual consistency — state is "in between" for a window; design for it.
- Harder debugging — no single stack trace; need distributed tracing + correlation IDs.
- Complex failure modes — lost / duplicate / out-of-order events.
- Operational overhead — you run and monitor a broker.

> Use EDA when many independent reactions to one thing, or to absorb load spikes. For simple linear CRUD, it's over-engineering.

## 5. Hard Problems (production reality)
- **Delivery guarantees**: most brokers are *at-least-once* → events can arrive twice → consumers must be **idempotent** (dedupe on unique event ID).
- **Ordering**: events may arrive out of order. Kafka orders only *within a partition* → partition by key (e.g. `orderId`).
- **Dual-write problem**: DB commit succeeds but publish fails = lost event. Fix with the **Outbox pattern** (write event to an `outbox` table in the SAME transaction as the data; a separate process ships it to the broker).
- **Dead-letter queue (DLQ)**: events that fail after N retries go here for inspection, instead of blocking or vanishing.
- **Schema evolution**: events are a contract. Only backward-compatible changes (add optional fields; never rename/remove). Use a schema registry.

## 6. Patterns Built on EDA
- **Event Notification** — event carries just an ID; consumers call back for details. (small events, more chatter)
- **Event-Carried State Transfer** — event carries full data; consumers never call back. (fatter events, more autonomy)
- **Event Sourcing** — store the *sequence of events* as source of truth; rebuild state by replaying. (audit log, time travel; big commitment)
- **CQRS** — separate write model from read model, synced via events. Often paired with event sourcing.
- **Saga** — manage a multi-service transaction via chained events + **compensating events** to undo steps on failure.

> Start with the first two. Event sourcing / CQRS are advanced — adopt only when you feel the pain they solve.

## 7. Learning Path
1. Nail the command vs event distinction.
2. Build a toy: 1 producer, 1 broker, 2 consumers (RabbitMQ/Kafka in Docker).
3. Break things on purpose — kill a consumer mid-flow, publish a duplicate — observe idempotency & retries.
4. Then read Martin Fowler "What do you mean by Event-Driven?" and *Enterprise Integration Patterns*.
