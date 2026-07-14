# Apache Kafka — Fundamentals Notes

> A structured study guide covering Kafka concepts, architecture, and hands-on Docker setup.

---

## 1. What is Apache Kafka?

**Apache Kafka** is an **open source, distributed, event streaming platform**.

Three keywords to remember:

| Keyword | Meaning |
|---|---|
| **Open source** | Freely available to use |
| **Distributed** | Can run across multiple machines/instances (a cluster) |
| **Event streaming** | A way of continuously producing and consuming data |

---

## 2. Why Kafka? — The Coffee Shop Analogy

**Old, unscalable model:**
A cashier takes your order (an *event*) and personally has to run and tell the chef, manager, and waiter individually. As order volume grows, this breaks down — too much manual, point-to-point communication.

**Improved model (Kafka):**
A digital display board is introduced. The cashier pushes the order to the board once. Chef, manager, and waiter each watch the board independently and react on their own schedule.

| Restaurant | Kafka |
|---|---|
| You / Cashier | **Producer** (creates & sends the event) |
| Digital display board | **Kafka** (stores the event) |
| Chef / Manager / Waiter | **Consumers** (listen and react to events) |

### Benefits this illustrates
- **Decoupling** — no direct dependency between producer and each consumer
- **Scalability** — consumers can be added/replaced without disrupting others
- **Asynchronous processing** — chef reacts immediately, manager/waiter can lag a bit
- **Replayability** — you can go back and check past events
- **Efficiency** — one connection per service to Kafka, instead of many-to-many

---

## 3. What is Event Streaming?

**Definition:** Continuously capturing data in real time, storing it in a log (like a timeline), and letting systems react immediately or later.

In tech terms:
- **Producer** creates an event (e.g., Order Service placing an order)
- **Kafka** stores it in a **topic**
- **Consumers** (Notification Service, SMS Service, Invoice Service, etc.) listen and process independently, each on its own timeline

### Ways to pass data (comparison)

| Method | Description | Example |
|---|---|---|
| Request-Response | One system calls another and waits for a reply | REST API / HTTP |
| Batch Processing | Large chunks of data processed at intervals | Nightly jobs, reports |
| Message Queues | One system sends to a queue; another pulls from it | RabbitMQ |
| Event Streaming | Constant flow of events sent/reacted to | Kafka |

---

## 4. The WhatsApp Group Analogy

| WhatsApp | Kafka |
|---|---|
| Group chat | **Topic** |
| Person sending a message | **Producer** |
| Person reading a message | **Consumer** |
| "Unread messages" marker | **Offset** |
| WhatsApp's backend server | **Broker** |

**Key insight:** Even if a group member goes offline, they don't lose messages — when they come back online, they resume from where they left off (unread messages). Kafka works the same way via **offsets**: if a consumer is offline, messages aren't lost; when it's back up, it resumes consuming from its last committed offset. Messages are also stored **in order**, and multiple consumers can read the same message independently.

---

## 5. Core Concepts

### Event
An event = **something happened + its data**. Not just a notification — the payload matters too.
- Example: `order placed` event carries order ID, payment ID, amount, etc. — whatever consumers need to act on.

### Producer
Any app/service that sends data to Kafka. There can be any number of producers (Order Service, Payment Service, etc., all producing simultaneously).

### Consumer
Any app/service that subscribes to and reads events from Kafka, then acts on the data.
- Example: Order Service produces → Notification Service, SMS Service, and Invoice Service all consume the same event and act independently.

### Topic
A named channel/category for segregating messages. Without topics, all producer messages would dump into Kafka as chaos.
- Example topics: `order-placed`, `user-signed-up`, `payment-done`
- Consumers subscribe only to the topic(s) relevant to them.

### Partition
Within a single topic, messages are further split into partitions to enable parallel processing — important when volume is huge (e.g., 1 million orders on Black Friday flooding the `order` topic).

### Consumer Group
A team of consumers sharing the workload of reading from a topic's partitions. Each consumer in the group is assigned specific partition(s), so messages are processed in parallel instead of by one overloaded consumer.

### Consumer Rebalancing
The automatic process of redistributing partitions among consumers in a group when:
- A consumer joins or leaves the group
- New partitions are added

Ensures each consumer gets a fair share of the load.

### Offset
A bookmark/number marking the last message position a consumer has read within a partition.

### Broker
A single Kafka instance/server that stores and routes messages. Multiple brokers run together as a **cluster** to handle heavy load.

### Zookeeper
A coordination service that manages the Kafka cluster — tracks which broker holds what data, handles leader election, manages metadata/configuration.

> **Note:** Newer Kafka versions are moving away from Zookeeper (KRaft mode), integrating coordination internally. Still worth knowing, since many production deployments still use Zookeeper.

---

## 6. Full Analogy Reference Table

| Term | WhatsApp Analogy | Kafka Definition |
|---|---|---|
| Event | Message sent | Single unit of data flowing through Kafka |
| Producer | Sender | App/service publishing events to a topic |
| Consumer | Reader | App/service subscribed to a topic |
| Topic | Group chat | Named category/feed for records |
| Partition | Sub-thread in a group | Split of a topic for parallel processing/scaling |
| Consumer Group | Team processing messages together | Group of consumers sharing read workload for a topic |
| Offset | Last-read bookmark | Marker tracking a consumer's read position in a partition |
| Broker | WhatsApp's server | Kafka server storing/serving messages |
| Zookeeper | Group admin | Manages metadata, leader election, config across brokers |

---

## 7. Additional Terms

| Term | Meaning |
|---|---|
| **Retention** | How long Kafka stores messages (e.g., 30/90 days) — like chat history |
| **Replication** | Kafka keeps copies of messages across brokers for safety/fault tolerance |
| **Producer Acknowledgement** | Confirmation that a sent message was successfully received by Kafka |
| **Dead Letter Queue (DLQ)** | Where failed messages are stored instead of being lost |
| **Kafka Connect** | Tool for connecting Kafka to external systems (databases, cloud, etc.) |
| **Kafka Streams** | Library for real-time data processing directly within Kafka |

---

## 8. Kafka vs. RabbitMQ

| Feature | Kafka | RabbitMQ |
|---|---|---|
| Model | Event streaming (log-based) | Message queuing |
| Message storage | Retained for a set time, replayable | Deleted after consumption |
| Consumers | Consumer groups; can also read independently outside a group | Message removed once delivered |
| Best for | High throughput, real-time analytics, data replay | Task queuing, short-lived jobs |
| Persistence | Long-term storage and replay | Short-term delivery |
| Ordering | Guaranteed **within a partition** | No strict ordering unless manually handled |
| Throughput | Very high | Moderate |

---

## 9. Hands-On: Local Kafka Setup with Docker

### 9.1 Start Zookeeper

```bash
docker run -d --name zookeeper -p 2181:2181 \
  -e ZOOKEEPER_CLIENT_PORT=2181 \
  -e ZOOKEEPER_TICK_TIME=2000 \
  confluentinc/cp-zookeeper:7.5.0
```

| Flag | Meaning |
|---|---|
| `-d` | Detached mode — runs in background |
| `--name zookeeper` | Names the container so other containers can reference it by hostname |
| `-p 2181:2181` | Maps Zookeeper's default client port to the host |
| `ZOOKEEPER_CLIENT_PORT` | Port Zookeeper listens on internally |
| `ZOOKEEPER_TICK_TIME` | Base time unit (ms) for heartbeats/session timeouts |

**Output:** Just a container ID hash (no logs shown due to `-d`):
```
a1b2c3d4e5f67890abcdef1234567890abcdef1234567890abcdef123456789
```

---

### 9.2 Start Kafka

```bash
docker run -d --name kafka -p 9092:9092 \
  -e KAFKA_BROKER_ID=1 \
  -e KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181 \
  -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://localhost:9092 \
  -e KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1 \
  --link zookeeper \
  confluentinc/cp-kafka:7.5.0
```

| Flag | Meaning |
|---|---|
| `KAFKA_BROKER_ID` | Unique ID for this broker within the cluster |
| `KAFKA_ZOOKEEPER_CONNECT` | Tells Kafka where to find Zookeeper |
| `KAFKA_ADVERTISED_LISTENERS` | Address Kafka tells clients to connect to (`PLAINTEXT` = no encryption, fine for local dev) |
| `KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR` | Replication for the internal `__consumer_offsets` topic — must be `1` with a single broker |
| `--link zookeeper` | Legacy Docker networking flag so `kafka` can resolve the `zookeeper` hostname |

**Output:** Container ID hash, e.g.:
```
f9e8d7c6b5a4938271605f4e3d2c1b0a9887766554433221100ffeeddccbb
```

> ⚠️ **Gotcha:** If Kafka starts before Zookeeper is fully ready, the Kafka container may crash-loop. Consider a short delay between the two commands, or use `docker-compose` with `depends_on` + healthchecks for reliability.

---

### 9.3 Enter the Kafka container

```bash
docker exec -it kafka bash
```

- `docker exec` — runs a command inside an **already running** container
- `-it` — interactive terminal (keeps STDIN open + allocates a pseudo-TTY)

**Output:** Prompt changes to reflect you're inside the container:
```
root@f9e8d7c6b5a4:/#
```

The Kafka CLI tools (`kafka-topics`, `kafka-console-producer`, etc.) are pre-installed here.

---

### 9.4 Topic Management

**Create a topic:**
```bash
kafka-topics --create --topic my-topic --bootstrap-server localhost:9092 \
  --partitions 3 --replication-factor 1
```
**Output:**
```
Created topic my-topic.
```

**List topics:**
```bash
kafka-topics --list --bootstrap-server localhost:9092
```
**Output:**
```
__consumer_offsets
my-topic
```
> `__consumer_offsets` is Kafka's internal system topic tracking consumer group offsets — auto-created on first offset commit.

**Describe a topic:**
```bash
kafka-topics --describe --topic my-topic --bootstrap-server localhost:9092
```
> ⚠️ Original command had a typo: `--bootstarp-server` → should be `--bootstrap-server`

**Output:**
```
Topic: my-topic  TopicId: xY9zAbC2QdCn...  PartitionCount: 3  ReplicationFactor: 1  Configs: segment.bytes=1073741824
	Topic: my-topic  Partition: 0  Leader: 1  Replicas: 1  Isr: 1
	Topic: my-topic  Partition: 1  Leader: 1  Replicas: 1  Isr: 1
	Topic: my-topic  Partition: 2  Leader: 1  Replicas: 1  Isr: 1
```
- **Leader** — broker responsible for that partition (broker `1`, since it's the only one)
- **Replicas** — brokers holding a copy of the partition
- **Isr** (In-Sync Replicas) — brokers fully caught up with the leader

---

### 9.5 Producing Messages

**Basic producer:**
```bash
kafka-console-producer --topic my-topic --bootstrap-server localhost:9092
```
**Output:** Blank interactive prompt — no confirmation per message sent:
```
>hello kafka
>this is my second message
>^C
```
Exit with `Ctrl+C`.

**Producer with keys:**
```bash
kafka-console-producer --topic my-topic --bootstrap-server localhost:9092 \
  --property "parse.key=true" --property "key.separator=:"
```
> ⚠️ Original command had a typo: `"key. Separator=:"` (stray space, wrong case) → should be `"key.separator=:"`

- `parse.key=true` — tells the producer each line has a key and value
- `key.separator=:` — splits each typed line on `:` into key/value

Example input:
```
>user123:hello world
>user456:another message
```
Kafka uses the key to decide which **partition** the message lands in (same key → same partition, preserving order for that key).

---

### 9.6 Consuming Messages

```bash
kafka-console-consumer --topic my-topic --bootstrap-server localhost:9092 \
  --group my-group --from-beginning
```

| Flag | Meaning |
|---|---|
| `--group my-group` | Joins/creates a consumer group; Kafka tracks this group's offset per partition |
| `--from-beginning` | Reads from the earliest available offset, not just new messages |

**Output:**
```
hello kafka
this is my second message
hello world
another message
```
> Note: keys won't display unless you also pass `--property print.key=true --property key.separator=:` on the consumer side.

The consumer blocks and waits for new messages until you `Ctrl+C`. Re-running with the same `--group` (without `--from-beginning`) resumes from the last committed offset — only new messages are shown.

---

### 9.7 Inspecting Consumer Group Lag

```bash
kafka-consumer-groups --bootstrap-server localhost:9092 --describe --group my-group
```

**Output:**
```
GROUP      TOPIC      PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG   CONSUMER-ID   HOST      CLIENT-ID
my-group   my-topic   0          1               1               0     -             -         -
my-group   my-topic   1          2               2               0     -             -         -
my-group   my-topic   2          1               1               0     -             -         -
```

| Column | Meaning |
|---|---|
| **CURRENT-OFFSET** | How far the group has read in that partition |
| **LOG-END-OFFSET** | Latest offset available (total messages produced) |
| **LAG** | `LOG-END-OFFSET - CURRENT-OFFSET` — growing lag = consumer falling behind |
| **CONSUMER-ID / HOST / CLIENT-ID** | Show `-` if no consumer is actively connected at the time this command runs |

> This is the exact metric used in production to monitor/alert on slow or stuck consumers.

---

## 10. Quick Fixes for Typos in Original Commands

| Incorrect | Correct |
|---|---|
| `--bootstarp-server` | `--bootstrap-server` |
| `"key. Separator=:"` | `"key.separator=:"` |

---

## 11. Next Steps

- Convert this manual Docker setup into a `docker-compose.yml` for repeatability
- Write actual Kafka producer/consumer code (Java / Spring Boot — relevant to EasyNocks)
- Explore partitioning strategy and keying decisions for production use
- Set up consumer lag monitoring/alerting
- Look into Kafka Connect and Kafka Streams for real-world data pipelines
