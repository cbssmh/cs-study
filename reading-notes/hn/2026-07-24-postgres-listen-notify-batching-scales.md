# 1. Title



Postgres LISTEN/NOTIFY Actually Scales



---



# 2. Source



- **Author / Organization:** Peter Kraft / DBOS

- **Link:** https://dbos.dev/blog/postgres-listen-notify-actually-scales

- **Date:** 2026-07-24



---



# 3. One-line Summary



Postgres LISTEN/NOTIFY is bottlenecked by a global commit lock, but application-level notification batching can increase throughput from 2.9K to 60K writes/sec while maintaining low latency.



---



# 4. Key Points



- LISTEN/NOTIFY enables low-latency event delivery without aggressive polling.

- A naive implementation issuing `NOTIFY` on every write saturated at roughly **2.9K writes/sec**.

- The bottleneck is not CPU, memory, or storage but a **global exclusive lock** acquired during transaction commit.

- PostgreSQL serializes transactions containing `NOTIFY` to preserve notification commit order.

- Because of this serialization, transactions cannot fully benefit from **group commit**, reducing throughput.

- DBOS buffers notifications in memory and flushes them periodically as batched `NOTIFY` operations.

- Since notifications are only wake-up signals—not the source of truth—strict durability and ordering are unnecessary.

- Readers combine LISTEN/NOTIFY with low-frequency polling to recover from notification loss after crashes.

- The optimized design achieves approximately **60K writes/sec (20× improvement)** with **15–100 ms latency**.

- At peak throughput, PostgreSQL becomes CPU-bound rather than lock-bound, indicating the original contention was removed.



---



# 5. Deep Dive (Structured Understanding)



## Problem



Applications often use LISTEN/NOTIFY to avoid expensive polling when implementing streaming or pub/sub on PostgreSQL.



However, issuing a `NOTIFY` for every inserted event causes severe throughput degradation despite low CPU and I/O utilization.



The root cause is PostgreSQL's requirement to serialize commits containing `NOTIFY` so notifications preserve global transaction commit order.



---



## Approach



Instead of sending notifications inside every transaction:



- Store actual data normally in PostgreSQL.

- Buffer notification requests in application memory.

- Flush multiple notifications together in periodic batch transactions.

- Add low-frequency polling as a recovery mechanism if buffered notifications are lost after a process crash.



---



## Key Insight



The notification itself is **not** the source of truth.



The database table already contains durable state.



Therefore:



- Perfect notification durability is unnecessary.

- Global notification ordering is unnecessary.

- Notifications only need to wake readers so they can query new rows.



Recognizing this distinction removes unnecessary synchronization overhead.



---



## Result / Impact



The optimization reduced contention on PostgreSQL's global notification lock and restored normal transaction concurrency.



Results:



- Throughput increased from **2.9K → 60K writes/sec**

- Roughly **20× higher throughput**

- **15–100 ms** notification latency maintained

- PostgreSQL utilization shifted from lock contention to actual CPU saturation



---



# 6. Why It Matters



- Challenges the common belief that PostgreSQL LISTEN/NOTIFY is inherently unsuitable for production-scale streaming.

- Demonstrates that many database bottlenecks originate from usage patterns rather than database architecture itself.

- Reinforces an important systems design principle: distinguish **data durability** from **event notification** to optimize independently.

- Shows that application-layer architectural changes can outperform database-level tuning.



---



# 7. Critical Analysis



- The optimization changes application semantics rather than improving PostgreSQL itself.

- Buffered notifications intentionally sacrifice immediate durability, making fallback polling mandatory.

- Benchmarks were conducted on a very large PostgreSQL server; smaller deployments may see different limits.

- The approach is suitable only when notifications are advisory signals, not business-critical events.

- Systems requiring durable ordered messaging (financial events, distributed logs, event sourcing) still require dedicated messaging infrastructure.



---



# 8. Connections



### PostgreSQL LISTEN/NOTIFY



Uses PostgreSQL as a lightweight pub/sub mechanism while exposing commit-order synchronization costs.



### Group Commit



Illustrates how global locking prevents PostgreSQL's normal group commit optimization.



### Redis Pub/Sub & Kafka



Redis and Kafka treat messaging as the primary data source, whereas this design keeps PostgreSQL tables as the source of truth and notifications as auxiliary signals.



### CQRS / Event-Driven Systems



Separates durable state from notification mechanisms, a common architectural principle in event-driven systems.



### LLM Streaming



Applicable to AI token streaming where notification latency matters but notification persistence does not.



---



# 9. Keywords



- PostgreSQL

- LISTEN

- NOTIFY

- Group Commit

- Global Lock

- Transaction Commit

- Event Streaming

- Pub/Sub

- Batching

- Database Performance



---



# 10. TL;DR



- PostgreSQL `NOTIFY` becomes a bottleneck because commits requiring notifications are globally serialized.

- Buffering and batching notifications eliminate most lock contention while preserving low latency.

- The key architectural insight is treating notifications as **wake-up signals**, not the authoritative data source.
