1. Title

AWS Contributes a Large-Transaction Binary Log Optimization to MySQL

2. Source

Author / Organization: Xueting Wu, Omar Farhat / Amazon Web Services (AWS Database Blog)
Link: https://aws.amazon.com/blogs/database/aws-contributes-a-large-transaction-binary-log-optimization-to-mysql/
Date: 2026-11-02

3. One-line Summary

AWS upstreamed Binary Log Optimization for Large Transactions (BOLT) to community MySQL, eliminating head-of-line commit stalls by promoting multi-gigabyte disk spill files directly into the active binlog sequence instead of copying them under a global lock, reducing 50 GiB commit latency from 7 minutes to 2.5 seconds.

4. Key Points

• The Binlog Global Commit Bottleneck: In MySQL, transactions buffer events in a local memory cache (`binlog_cache_size`, default 32 KiB) or disk "spill file" in parallel; however, at commit time, MySQL acquires a global lock on the active binlog file to copy events sequentially, serializing every transaction on the server.
• Head-of-Line Blocking at Scale: When committing large transactions (such as bulk data loads or batch deletes), a 50 GiB transaction copying at 120 MB/s holds the global binlog lock for roughly 7 minutes, completely freezing unrelated small transactions and doubling disk space consumption during the copy.
• Upstreaming Aurora MySQL Production Code: AWS contributed its internal optimization—Binary Log Optimization for Large Transactions (BOLT)—which has operated in production across Amazon Aurora MySQL since November 2019 (Aurora v1.22.0, 2.x, 3.x, and 8.4.x), planning for inclusion in a future MySQL innovation release.
• Zero-Copy Spill File Promotion: Rather than copying events into the main binlog file, BOLT promotes the transaction's existing temporary spill file directly into the binlog sequence, assigning it the next sequential file name and registering it in `binlog.index` by reusing the standard file rotation mechanism.
• Size-Independent Commit Latency: Because promotion replaces a linear byte copy with a constant-time file metadata rename and rotation, commit latency for large transactions becomes near-constant (~2.0 to 2.5 seconds across 1 GiB to 50 GiB), representing only the fixed time required to sync the spill file and commit to InnoDB redo logs.
• Instant Crash Recovery via Header Offset: Introduces `Large_transaction_header_event` at the start of promoted files, which records the exact byte offset of the transaction's terminating event; during server restart, recovery logic seeks directly to the end in 0.1 seconds instead of scanning the 50 GiB file body for 4 minutes.
• Backward Compatibility with CDC and Downstream Replicas: Flags `Large_transaction_header_event` as ignorable, allowing older replicas, standard `mysqlbinlog` utilities, and change data capture (CDC) pipelines (Debezium, Maxwell, AWS DMS) to transparently skip the event and read downstream events normally.
• Dramatic Write Throughput Improvements: In sysbench benchmarks on an r7i.16xlarge instance, BOLT boosted write throughput by ~3.1x on 5 GiB transaction workloads (19.3 to 60.8 MB/s) and ~2.8x on mixed workloads (19.9 to 56.1 MB/s) by removing head-of-line serialization stalls.
• Persistent Replication Order Challenges: AWS explicitly cautions that while BOLT solves primary commit latency and recovery, large transactions still degrade replica performance when `replica_preserve_commit_order` is enabled, advising developers to continue batching mutations into smaller sets.

5. Deep Dive (Structured Understanding)

Problem
Relational databases rely on write-ahead logging for ACID durability and binary logging for point-in-time recovery (PITR) and replica synchronization. In MySQL, transactions write generated events to a private `IO_CACHE` or spill file on disk in parallel without cross-transaction contention. However, to maintain a total global order across transactions, MySQL serializes the commit path through a global lock on the active binlog file. When an application commits a large transaction (e.g., ETL data migrations, batch archive purges, or bulk updates), the committing thread holds this global lock while copying every byte from its local spill file to the shared binlog file. During this transfer, no other thread—even those updating completely orthogonal tables—can commit. Furthermore, copying consumes double the disk storage, and restarting after a crash requires the server to sequentially scan the multi-gigabyte binlog file to verify transaction completion, severely impairing database availability.

Approach
AWS upstreamed BOLT, an architectural redesign of the binlog commit and recovery engine. Rather than copying bytes into the currently open binlog file, BOLT detects when a transaction's spill file exceeds a threshold (`binlog_large_transaction_optimization_threshold`, defaulting to 128 MB). At commit time, the server promotes the spill file directly into the global binlog sequence. The engine reuses MySQL’s native binlog file rotation logic: it closes the current binlog, renames the spill file to the next binlog sequence number, and appends the new filename to the binlog index file. Because the spill file is already fully persisted on disk, the operation requires no data movement. To protect crash recovery, the engine prepends a `Large_transaction_header_event` containing the byte offset of the commit's terminating event, allowing the recovery subsystem to skip the entire body.

Key Insight
1. Replacing Data Copying with Metadata Promotion: In storage and database systems, physical data copying across layers is an architectural anti-pattern. Treating an existing, fully populated temporary spill file as a first-class segment in the append-only log sequence converts an $O(N)$ linear data copy into an $O(1)$ constant-time metadata rename.
2. The Decoupling of Primary Commit from Replica Execution: BOLT decouples the primary server's write path from transaction size. However, the authors maintain an honest systems perspective: eliminating commit latency on the writer does not eliminate replication lag. Replicas enforcing strict serial commit ordering (`replica_preserve_commit_order`) still process the large transaction as a single unit, proving that low-level engine optimizations must still be complemented by application-level batching.
3. Ignorable Protocol Events for Zero-Ecosystem Disruption: Adding proprietary or optimized header formats to binary logs risks breaking the surrounding data ecosystem (CDC pipelines, ETL connectors, older read replicas). By adhering to MySQL’s "ignorable event" specification, BOLT embeds critical fast-seek offsets for modernized recovery while preserving binary compatibility across legacy consumers.

Result / Impact
Transforms large transaction commit latency from an unbounded operational risk (7 minutes at 50 GiB) into a predictable 2.5-second operation, slashes binlog crash recovery from 4 minutes to 0.1 seconds, and triples overall write throughput for mixed enterprise workloads in community MySQL.

6. Why It Matters

• Databases & Storage Engine Architecture: Solves a foundational, 20-year-old architectural bottleneck in MySQL’s replication and logging subsystem, moving one of the most celebrated proprietary features of Amazon Aurora into the open-source community.
• Platform Engineering & Reliability: Eliminates sudden, catastrophic database freezes caused by un-batched developer scripts or background batch jobs, protecting low-latency OLTP services from unexpected global lock starvation.
• Cloud Infrastructure & Cost Optimization: Removes the requirement to provision 2x disk headroom for large transaction commit buffers, reducing EBS/gp3 volume provisioning waste across cloud database fleets.
• Open-Source Stewardship & Hyperscaler Contributions: Demonstrates a healthy open-source collaboration model where cloud providers (AWS) upstream battle-tested enterprise enhancements (running in Aurora since 2019) to core community software (MySQL/Oracle, Alibaba Cloud, OurSQL Foundation).

7. Critical Analysis

• Downstream Replication Inversion: While BOLT removes primary commit serialization, it introduces potential replication asymmetry. Because a 50 GiB transaction commits in 2.5 seconds on the primary, developers may be tempted to run massive batch writes frequently. However, read replicas evaluating events sequentially cannot easily parallelize a single monolithic transaction, resulting in severe replication lag and read-replica stale reads.
• File System Fragmentation from Small "Large" Transactions: Promoting a spill file to an independent binlog file on transactions exceeding 128 MB means that workloads triggering frequent ~150 MB transactions will generate hundreds of small binlog files rather than standard 1 GB rotation files, potentially fragmenting `binlog.index` and increasing metadata management overhead on underlying filesystems.
• Storage Engine Scope Limitations: BOLT accelerates the binlog write path and binlog recovery, but does not alter InnoDB's internal commit path. As the authors note, the remaining 2 to 2.5 seconds of commit latency still belongs to flushing InnoDB undo/redo logs and fsyncing the spill file to durable disk.

8. Connections

• Log-Structured Storage & File Promotion: Parallels file-promotion techniques in LSM-tree databases (such as RocksDB/Pebble SSTable ingestion) where bulk-written external files are promoted into the active database without copying.
• Change Data Capture (CDC) Ecosystems: Integrates cleanly with modern streaming data ingestion frameworks (Debezium, Apache Kafka Connect, AWS DMS) through backward-compatible binary log event design.
• Amazon Aurora Storage Engine Innovations: Highlights the continuous backporting of distributed cloud database optimizations (originally designed for Aurora's decoupled storage fabric) into traditional monolithic relational runtimes.

9. Keywords

• MySQL Binary Log (binlog)
• BOLT Optimization
• Head-of-Line Blocking
• Transaction Commit Latency
• Amazon Aurora MySQL
• Spill File Promotion
• Crash Recovery
• Relational Database Internals

10. TL;DR

• AWS contributed BOLT to community MySQL, eliminating the global lock bottleneck that froze database commits during multi-gigabyte transactions.
• Instead of copying spill file events into the active binlog under a global lock, BOLT promotes the spill file directly into the binlog sequence, cutting 50 GiB commit latency from 7 minutes to 2.5 seconds.
• Prepends an ignorable header event to enable 0.1-second crash recovery seeks, delivering up to 3.1x higher write throughput while maintaining full compatibility with downstream CDC readers.
