1. Title

KV Caching on Valkey: 2× Faster Responses and 25% More Requests

2. Source

Author / Organization: Omer Rubinstein, Chaitanya Nuthalapati, Jeff Duffy / Valkey Project (Linux Foundation / AWS)
Link: https://valkey.io/blog/kv-caching-on-valkey/
Date: 2026-10-20

3. One-line Summary

The Valkey engineering team demonstrates how offloading intermediate attention key-value (KV) tensors from constrained GPU and host memory to an external shared Valkey L2 cache reduces prompt recomputation from 10% to 1%, cutting time-to-first-token from 8.5 seconds to 130 milliseconds for multi-turn agentic workloads.

4. Key Points

• The Prefill Recomputation Bottleneck: In agentic multi-turn conversations, coding assistants, and RAG systems, expanding contexts (MCP schemas, system prompts, chat history) force inference engines to re-evaluate prior tokens during the compute-heavy prefill phase, leaving GPUs idle on decoding while re-reading identical context.
• Massive Memory Footprint of KV Tensors: Key-value states scale as `layers × KV heads × head_dim × bytes per element × 2`; for Llama-3.1-70B in BF16, a single token consumes ~0.33 MB of memory, meaning a 20k–30k token conversation requires 6.5 GB to 10 GB of raw tensor storage.
• Failure of In-Host Caching (L0/L1): Local GPU high-bandwidth memory (L0) and host CPU RAM (L1) are quickly exhausted (even 500 GB of host RAM holds only 50–75 long sessions), and dynamic fleet autoscaling, pod restarts, and load balancers frequently route returning users to cold nodes where local checkpoints are absent.
• Flaws of Peer-to-Peer Node Sharing: Attempting to share KV states peer-to-peer across GPU servers provides marginal capacity expansion while consuming the host CPU cycles, network interfaces, and memory buses needed for active model execution.
• Disaggregated Shared L2 Tier Architecture: Attaching an external, in-memory Valkey 9 cluster via LMCache creates an elastic storage tier accessible by any GPU node in the cluster, decoupling cache retention from GPU count and eliminating recomputation penalties during fleet rebalancing.
• Drastic Latency and Throughput Gains: In an 8 req/sec benchmark on two 8×H100 nodes running Llama-3.1-70B with 100 concurrent growing conversations (12k–28k tokens), adding a 3-shard Valkey cluster dropped prompt recompute share from ~10% to ~1%, accelerated reply streaming 2.1x (from 11.9 to 24.8 tokens/sec), and plunged time-to-first-token (TTFT) from 8.5s to 0.13s for contexts under 16k tokens.
• Chunk Size and Glibc Memory Thresholds: Sizing KV chunks to 256 tokens proved optimal; coarser 512-token chunks nearly doubled recomputation due to boundary fragmentation, while chunk sizes must stay under 33 MB to prevent exceeding glibc’s `mmap` allocation threshold, which forces page faulting and slows retrieval by 3x.
• Bandwidth Sizing over Pure Capacity: Because returning conversations fetch multi-gigabyte checkpoints in parallel, provisioning shards must prioritize aggregate network read throughput (the 3-shard test sustained 7.4 GB/s of reads, peaking at 9.4 GB/s) rather than storage volume alone.
• Favorable FinOps Ratio: Adding the three-shard memory-optimized Valkey cluster increased total fleet infrastructure cost by roughly 15%, while increasing total served request volume by 25% and output token generation by 28%.

5. Deep Dive (Structured Understanding)

Problem
Unlike traditional microservices whose resource consumption scales sub-linearly with traffic, LLM inference costs scale directly with token volume. In agentic workflows, long context windows (tool definitions, system instructions, and multi-turn chat history) cause inference engines (vLLM, TensorRT-LLM, SGLang) to spend substantial GPU cycles on the quadratic prefill phase on every user turn. While engines support in-memory KV caching in GPU HBM (L0) and host CPU memory (L1), these local tiers are severely constrained: a 30,000-token session on a 70B parameter model consumes up to 10 GB of memory. Consequently, sessions are rapidly evicted between user conversational turns. When users resume sessions or load balancers direct traffic to a different GPU node, the engine must recompute the entire prefix from scratch, causing multi-second delays before the first response token streams.

Approach
The authors decoupled tensor caching from compute nodes by implementing a distributed, external L2 tier using Valkey (the open-source, Linux Foundation-hosted in-memory database). Using LMCache integrated with vLLM, the inference engine asynchronously pushes evicted L1 KV tensors into a Valkey 9 cluster. Fronting the GPU fleet with a prefix-cache-aware router ensures requests route to nodes holding local L0/L1 states; if a node restart or load spike routes the request to a cold node, the engine pulls the multi-gigabyte KV tensor chunks in parallel from the Valkey L2 cluster over a 100 Gbps network fabric, bypassing prompt recomputation.

Key Insight
1. Memory Eviction Decoupling: In high-concurrency environments, eviction is driven by unique contexts over time, not just active connections. Moving KV states to an independent, horizontally scalable in-memory cluster breaks the coupling between GPU count and cache capacity, allowing memory-optimized instances (which are orders of magnitude cheaper than HBM) to absorb session pauses.
2. The Glibc `mmap` Allocation Cliff in Tensor Streaming: In high-throughput serialization pipelines, chunk sizes must be calibrated not only for token reuse boundaries but also for low-level memory allocators. Values exceeding glibc's dynamic `mmap` threshold trigger page faults and kernel allocation overhead, tripling deserialization latency. Setting chunk size to 256 tokens kept serialized chunks well below this cliff.
3. Network Bandwidth as the Primary Sizing Metric: While databases are conventionally sized for memory capacity, an L2 KV cache functions as a high-bandwidth burst pipe. A returning 20k-token session demands pulling 6.6 GB of tensor data in hundreds of milliseconds. Sizing Valkey shards to sustain 10+ GB/s aggregate network throughput is mandatory to ensure TTFT remains sub-second.

Result / Impact
Demonstrated that disaggregating KV caching via Valkey transforms open-weight LLM inference economics, enabling production fleets to serve 25% more traffic, slash TTFT from seconds to milliseconds, and double token streaming throughput without acquiring additional expensive accelerator hardware.

6. Why It Matters

• Distributed Systems & Databases: Repurposes distributed in-memory key-value databases (Valkey) from traditional web session storage into high-bandwidth distributed tensor stores for deep learning runtime engines.
• AI Engineering & Inference Infrastructure: Solves the primary latency and cost obstacle in agentic computing (prefill recomputation on expanding contexts), validating hierarchical memory tiering (L0 GPU -> L1 Host RAM -> L2 Distributed Valkey) as the future architecture for LLM serving.
• Platform Engineering & FinOps: Offers a concrete, cost-effective optimization strategy for self-hosted enterprise model serving, yielding a 25% throughput gain for a modest 15% cache infrastructure investment.
• Systems Programming & Performance Engineering: Highlights the critical importance of low-level systems tuning (allocator thresholds, network fabric placement, chunk alignment) when bridging software application runtimes with high-throughput hardware pipelines.

7. Critical Analysis

• Network Saturation Risks at Scale: Pulling multi-gigabyte tensors over standard datacenter networks places intense pressure on top-of-rack (ToR) switches and network interfaces. In massive clusters running hundreds of concurrent large-context queries, network congestion could cause fetch times to exceed local prefill computation times unless dedicated RoCE/InfiniBand fabrics are deployed.
• Tail-Latency Degradation on Long Contexts: As the authors note, for conversations exceeding 16,000 tokens under heavy load (8 req/sec), the compute required to prefill genuinely new tokens began outstripping GPU capacity, causing TTFT improvements to plateau. An L2 cache accelerates prefix recovery but cannot generate new compute capacity for novel tokens.
• Eviction Policy Complexity: Valkey's default `noeviction` rejects writes once memory fills, while LMCache historically lacked native TTL management. In high-churn production setups, missing TTL expirations or misconfigured LRU policies can lead to cache pollution, requiring diligent operational monitoring of memory fragmentation.

8. Connections

• vLLM PagedAttention & Chunked Prefill: Directly complements vLLM's memory management paradigms by extending virtual memory paging from host RAM across the network to distributed key-value storage.
• LMCache Open-Source Project: Leverages the emerging open-source caching middleware designed specifically to interface AI inference runtimes with distributed storage engines.
• Redis / Valkey Clustering & Sharding: Re-applies proven distributed database sharding and asynchronous replication protocols to manage non-traditional, multi-megabyte binary tensor payloads.

9. Keywords

• Valkey
• Key-Value (KV) Caching
• LLM Inference Optimization
• vLLM
• LMCache
• Time to First Token (TTFT)
• PagedAttention
• In-Memory Database

10. TL;DR

• The Valkey Project demonstrated that offloading LLM attention KV tensors to an external Valkey L2 cache reduces prompt recomputation from 10% to 1% across multi-turn agent workloads.
• Benchmarks on Llama-3.1-70B showed a 25% increase in served requests, a 2.1x speedup in reply streaming, and a plunge in TTFT from 8.5s to 0.13s for a 15% infrastructure cost addition.
• Emphasizes that sizing shards for aggregate network read bandwidth (7.4+ GB/s) and tuning chunk sizes to avoid glibc `mmap` thresholds are vital to keeping tensor retrieval faster than GPU prefill.
