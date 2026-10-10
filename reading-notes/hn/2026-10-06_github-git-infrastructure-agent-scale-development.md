1. Title

Building Git Infrastructure for Agent-Scale Development

2. Source

Author / Organization: Brian Celenza / GitHub (The GitHub Blog)
Link: https://github.blog/engineering/architecture-optimization/building-git-infrastructure-for-agent-scale-development/
Date: 2026-10-06

3. One-line Summary

GitHub is executing a live, zero-downtime architectural replatforming of its core Git storage and serving tier (Spokes), decoupling compute from durable cloud object storage and minimizing consensus coordination to scale write throughput up to 35x for concurrent AI agent swarms and continuous CI pipelines.

4. Key Points

• Surge in Git Workload Volume: GitHub’s total Git activity exceeded 473.3 billion events per month in August 2026 (more than 2x YoY), monthly commits climbed past 7.38 billion (>5x YoY), pushes grew 4.9x to 3.35 billion, and GitHub Actions ran 3.26 billion times in September 2026.
• The Agentic Traffic Pattern: Autonomous coding agents execute rapid commit-checkpoint loops where single-push latency directly bounds execution speed, generating high-frequency concurrent branch pushes and severe merge contention on trunk references.
• Limitations of Legacy Spokes Architecture: Spokes couples durability and serving capacity by keeping full repository copies on the local NVMe disks of five fileservers per repo, synchronizing reference updates via a three-phase commit (3PC) quorum.
• The Replication vs. Write-Speed Bottleneck: In Spokes, every replica participates in every write; consequently, adding read replicas to absorb massive CI and agent clone fan-out directly degrades push latency and caps total write throughput.
• Disaggregating Compute from Storage: Replaces monolithic storage servers with stateless, horizontally scalable compute workers that serve Git traffic and maintain local caches, while authoritative repository data resides in Azure Blob Storage.
• Pruning Distributed Coordination: Shrinks the synchronous critical path of a push exclusively to the atomic reference update, allowing object persistence, graph connectivity checks, and secret scanning to run asynchronously and in parallel.
• Offloading Repository Maintenance: Heavy compaction, garbage collection, and reachability graph generation are extracted completely from serving nodes and delegated to background workers operating directly against durable blob storage.
• Resilient Node Failure Recovery: Losing a compute host transitions from an expensive multi-hour disk re-replication to a lightweight cache miss, enabling replacement pods to spin up immediately and warm caches on demand.
• 35x Write Throughput Improvement: Internal production benchmarks demonstrate up to a 35x gain in write throughput alongside independently scalable read capacity.

5. Deep Dive (Structured Understanding)

Problem
For over a decade, GitHub relied on Spokes to store and serve repositories. Spokes colocated storage durability and compute serving on local fileserver disks, delivering microsecond-level NVMe access for human developers. However, the rise of agentic software development broke the assumptions underlying this architecture. While human developers push intermittently, autonomous agents commit after nearly every micro-step or test execution, making round-trip push latency the primary constraint on agent speed. Furthermore, thousands of concurrent agents working across branches generate massive write streams that funnel into merge queues, which in turn fan out into thousands of CI container fetches per minute. Under Spokes, adding read replicas to handle this fan-out slowed down writes, because every replica had to participate in the synchronous 3PC quorum.

Approach
GitHub is rebuilding its core infrastructure while serving planetary-scale traffic without maintenance windows or changes to Git developer workflows. The new architecture disaggregates compute from storage while systematically reducing coordination. Authoritative repository objects are persisted directly to Azure Blob Storage, leveraging cloud-native multi-zone durability. The serving tier consists of stateless compute workers that scale elastically with incoming Git traffic. The critical path of a push is stripped down solely to the atomic compare-and-swap (CAS) reference update, while object transmission, verification, and scanning execute concurrently. Repository maintenance tasks (such as repackaging and garbage collection) are removed from serving nodes entirely and run as asynchronous background jobs against durable storage.

Key Insight
1. Decoupling Durability from Serving Scale: In traditional monolithic fileservers, the number of physical disk replicas determines both data durability and read capacity. Delegating baseline durability to cloud object storage allows read capacity to scale via ephemeral caching instances without expanding the quorum size or penalizing write latency.
2. Narrowing the Coordination Boundary: Under Git semantics, the only invariant requiring strict synchronous consensus is the reference pointer (the branch head). Ingesting object packs and verifying reachability can proceed independently, dramatically shrinking the critical lock window.
3. Node Failure as a Cache Miss: In coupled architectures, a dead host requires a full repository rebuild over the network. In a disaggregated model, host failure is merely a cache miss: a replacement worker boots immediately and pulls required packfiles on demand from blob storage.

Result / Impact
Delivered up to 35x higher write throughput in internal benchmarks, eliminated the architectural conflict between read scaling and write latency, and established an elastic foundation capable of absorbing billions of agent-generated commits while maintaining branch protections and security controls.

6. Why It Matters

• Distributed Systems & Infrastructure: Provides a rare, concrete case study of decomposing a stateful, POSIX-centric monolithic storage system (Spokes/DGit) into modern disaggregated cloud-native storage while serving live planetary traffic.
• Platform Engineering & Developer Tooling: Signals a major inflection point where developer platforms must re-engineer core version-control infrastructure to accommodate non-human developers generating millions of automated branches and PRs daily.
• Backend Engineering & Reliability: Demonstrates the practical power of isolating minimal consensus state updates while offloading heavy computation and compaction to asynchronous, out-of-band workers.
• AI Engineering / Autonomous Systems: Addresses the foundational infrastructure bottleneck of AI coding agents, ensuring that repository I/O and push latency do not choke agentic development loops.

7. Critical Analysis

• Latency Floor of Object Storage: While blob storage enables elastic capacity, cloud object storage API latency (tens of milliseconds) is significantly higher than local NVMe access (microseconds). The user-perceived performance of this architecture relies heavily on high cache-hit ratios; cache misses on deep repository histories or complex `git log`/`blame` traversals risk tail-latency spikes.
• Cold-Start Storms on Massive Monorepos: For monorepos containing hundreds of gigabytes of data, provisioning fresh compute workers during traffic spikes could trigger severe network I/O storms against blob storage until local worker caches are sufficiently warmed.
• Dual-Stack Operational Risk: Rebuilding the engine while the plane is flying requires running Spokes and the new architecture concurrently, synchronizing data across heterogeneous storage models while ensuring absolute ref consistency and zero data loss.
• Hyperscaler Cloud Dependency: Transitioning from bare-metal fileservers to Azure Blob Storage couples GitHub's primary data plane directly to a single cloud provider’s regional availability zones, egress economics, and object-storage service levels.

8. Connections

• Cloud-Native Disaggregated Databases (e.g., Amazon Aurora, Google Cloud Spanner): Adopts the distributed database design pattern of separating compute query processing from shared, log-structured cloud storage.
• Git Internals & Content-Addressed Storage (CAS): Leverages Git’s native immutable object model (blobs, trees, commits, tags identified by SHA hashes) to store files cleanly as write-once, content-addressed objects in cloud storage.
• Virtualized Monorepo Systems (e.g., Microsoft Scalar, Uber GitFarm): Complements client-side virtualized file systems by addressing the corresponding server-side bottleneck: scaling server ingest and reference consensus under extreme concurrency.

9. Keywords

• Git Infrastructure
• Disaggregated Storage
• Spokes Architecture
• Agentic Software Development
• Write Throughput
• Azure Blob Storage
• Three-Phase Commit (3PC)
• Content-Addressed Storage

10. TL;DR

• GitHub is rebuilding its core Git storage engine during live production to support an exponential influx of automated AI agent commits and CI fan-out.
• The new architecture decouples stateless compute caching workers from durable Azure Blob Storage, shrinking synchronous consensus strictly to atomic reference updates.
• Internal benchmarks achieve up to 35x higher write throughput, removing the historical trade-off where scaling read replicas degraded write performance.
