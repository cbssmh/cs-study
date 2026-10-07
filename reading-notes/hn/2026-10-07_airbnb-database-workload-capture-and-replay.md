1. Title

Beyond Synthetic Testing: Capturing and Replaying Real Database Workloads at Airbnb

2. Source

Author / Organization: Zuofei Wang, Erluo Li / Airbnb Engineering & Data Science (The Airbnb Tech Blog)
Link: https://airbnb.tech/infrastructure/beyond-synthetic-testing-capturing-and-replaying-real-database-workloads-at-airbnb/
Date: 2026-10-07

3. One-line Summary

Airbnb engineered an end-to-end database traffic capture and offline replay platform leveraging ProxySQL, an asynchronous storage offloader, and a horizontally scalable worker fleet to validate major database engine migrations (MySQL 5.7 to 8.0) and accurately capacity-plan for seasonal peaks without the inaccuracies of synthetic benchmarking.

4. Key Points

• Limitations of Client-Side Capture: Airbnb previously attempted query capture using language-specific client logging libraries emitting to Kafka; this created high maintenance overhead across service-oriented architecture (SOA) teams and failed to capture complete transactional boundaries needed for deterministic replays.
• Transparent Wire-Protocol Capture: Shifted capture to ProxySQL, which already sits between applications and databases, enabling dynamic, cluster-scoped binary query logging at the network boundary with zero application code changes and minimal query performance impact.
• Decoupled Storage and Processing Pipeline: A Log Mover sidecar offloads raw logs to cloud object storage to prevent local disk exhaustion, while an offline batch Log Processor parses, groups, and buckets interleaved query streams into 5-minute per-cluster datasets.
• Query Rewriting for Deterministic State: The Log Processor rewrites `INSERT` statements to explicitly pin the captured `last_insert_id` (e.g., `INSERT INTO users (id, name) VALUES (1, 'bob')`), preventing downstream foreign-key lookups from failing due to diverging auto-increment behaviors across engine versions.
• Synchronized Distributed Replay: The Replay Task Scheduler stamps task batches with a shared future "expected start time," aligning thousands of distributed worker pods to replay traffic simultaneously and faithfully reproduce real-world concurrency, bursts, and temporal spacing.
• Dual Replay Operational Modes: Provides "Replay Only" for capacity planning (replaying at 1x, 2x, or 3x speed to measure resource saturation and latency shifts) and "Replay and Compare" for compatibility validation (executing identical workloads against dual target engines restored from the same snapshot to diff result sets).
• Concrete Production Regressions Caught Offline: Uncovered severe MySQL 8.0 regressions prior to deployment, including a join query whose latency degraded from 0.03s to 2.6s (reading 273 MB vs. 5 MB due to sort-row changes in MySQL 8.0.20+), non-deterministic rows returned by unindexed `ORDER BY ... LIMIT` queries, and un-cached duplicate queries previously masked by MySQL 5.7's query cache.
• Proactive Capacity Ceiling Discovery: Replaying 80% higher write traffic against a large cluster increased average commit latency by nearly 500% (from 6 ms to 34 ms), revealing exact scaling limits prior to peak travel season traffic.
• Strict Data Privacy and Governance: Logs containing production personal data are encrypted in transit and at rest, captured only during active testing windows, and replayed strictly in production-equivalent environments—never copied into local or insecure development environments.

5. Deep Dive (Structured Understanding)

Problem
Airbnb operates hundreds of MySQL-compatible database clusters handling thousands of use cases at millions of queries per second (QPS). Major infrastructure operations—such as fleet-wide upgrades from MySQL 5.7 to 8.0 or capacity sizing ahead of peak travel seasons—carry high blast-radius risks. Synthetic benchmarks like sysbench test simplified, idealized query patterns that fail to replicate real-world locking contention, subtle dialect regressions, or complex data interactions. Furthermore, Airbnb's earlier approach of capturing queries via language-specific client loggers was fragmented, unmaintainable across hundreds of microservices, and lacked the transactional context required to verify whether queries produce identical results on new database engines.

Approach
Airbnb built an infrastructure-native capture and replay platform centered on their existing ProxySQL proxy layer deployed on Kubernetes. A local Log Mover sidecar monitors ProxySQL query logs and streams them to cloud object storage. An offline Log Processor decodes the binary logs, reassembles concurrent queries into coherent per-cluster transactions, partitions them into 5-minute buckets, and rewrites `INSERT` statements with explicit primary keys. Finally, a distributed Log Replayer—consisting of a web API Server, a Replay Task Scheduler, and a fleet of worker pods—replays workloads against target databases restored from identical snapshots. Workers synchronize using shared start timestamps to faithfully replicate concurrency at configured speed multipliers, comparing result sets when validating engine upgrades.

Key Insight
1. Proxy Layer as the Architectural Narrow Waist: Capturing traffic at the MySQL wire-protocol proxy level completely decoupled database testing from application code, frameworks, and language bindings, capturing byte-exact statements and transactional context transparently.
2. Synchronized Start Times for Distributed Concurrency: Real-world database performance depends heavily on query concurrency and burstiness. Rather than allowing workers to replay files upon queue ingestion, coordinating workers around a synchronized future start epoch ensures accurate multi-pod traffic reproduction.
3. Deterministic State Pinning via Primary Key Rewriting: Database engines handle auto-increment generation differently across versions (e.g., changes in `innodb_autoinc_lock_mode`). Injecting explicit IDs into `INSERT` statements deliberately sacrifices testing native auto-increment locking in exchange for preventing cascading read failures on dependent queries during replay.
4. Stepped Bisection for Regression Root-Causing: When migrating across major versions with multiple compounding changes (e.g., query cache removal combined with sorting algorithm modifications), systematically bisecting workloads (5.7 with cache vs. 5.7 without cache, then 5.7 without cache vs. 8.0) is essential to isolate hidden regressions.

Result / Impact
Enabled a seamless, zero-incident fleet migration from MySQL 5.7 to 8.0 across hundreds of production clusters. Provided a self-serve load testing and capacity planning tool that pinpoints database throughput and latency ceilings prior to seasonal demand surges.

6. Why It Matters

• Databases & Reliability: Establishes a proven methodology to eliminate the uncertainty of major database version migrations, moving testing from synthetic assumptions to empirical production evidence.
• Platform Engineering & Infrastructure: Demonstrates how existing architectural components (reverse proxies like ProxySQL or Envoy) can be repurposed as foundational data-plane platforms for observability, traffic shadowing, and validation.
• Backend Engineering & Distributed Systems: Surfaces subtle cross-version application pitfalls, such as relying on non-deterministic row ordering in queries without explicit `ORDER BY` tie-breakers or unmonitored query amplification previously hidden by transparent caching.
• IT Risk / Governance: Models enterprise-grade data handling for production traffic, demonstrating that realistic replay testing can be achieved without compromising data encryption, access controls, or PII compliance.

7. Critical Analysis

• State Drift During Write Replays: Because write queries mutate the state of the target database during replay, the target environment gradually drifts from the original production state over extended replay windows. While restoring from a snapshot establishes baseline parity, long-running replays can encounter artificial primary key or unique constraint collisions.
• Bypassing Native Auto-Increment Contention: By explicitly rewriting `INSERT` statements to pin `last_insert_id`, the system deliberately avoids testing the target database engine's native auto-increment locks under high concurrency. While necessary to avoid false-positive read failures, this leaves native sequence generation unexercised until production cutover.
• Resource and Infrastructure Overhead: Operating production-equivalent test clusters, taking full database snapshots, and provisioning thousands of replay worker pods incurs substantial cloud infrastructure costs that may be prohibitive for smaller engineering organizations without sampling strategies.
• Proxy Performance and Storage Failure Modes: While ProxySQL logging is lightweight, running sidecar log movers introduces local disk I/O and network transfer overhead. If downstream object storage slows or sidecars stall, local disk saturation on ProxySQL pods could directly threaten live database traffic without robust backpressure controls.

8. Connections

• Shadow Traffic / Dark Launching: Adapts the concept of live request shadowing (e.g., Envoy request mirroring) to stateful, multi-statement database transactions with offline storage and controlled temporal replay.
• Trace-Driven Workload Simulation: Replaces synthetic load generators (sysbench, JMeter, k6) with empirical trace replays, aligning database validation with realistic long-tail production distributions.
• Smart Database Proxies: Reinforces the expanding role of database proxies (ProxySQL, Vitess, PgBouncer) as intelligent platform intermediaries capable of query rewriting, traffic routing, security filtering, and offline forensics.

9. Keywords

• Database Workload Replay
• Traffic Shadowing
• ProxySQL
• MySQL 5.7 to 8.0 Upgrade
• Capacity Planning
• Query Rewriting
• Synthetic Benchmarking
• Production Load Testing

10. TL;DR

• Airbnb developed a production database traffic capture and offline replay architecture using ProxySQL, cloud storage offloading, and a distributed worker fleet to replace inadequate synthetic benchmarks.
• The system reassembles raw binary logs, pins auto-increment IDs via query rewriting for deterministic execution, and synchronizes distributed workers to faithfully reproduce concurrent production spikes and diff engine responses.
• The platform enabled a zero-incident migration across hundreds of clusters from MySQL 5.7 to 8.0 while allowing self-service capacity planning ahead of peak travel traffic.
