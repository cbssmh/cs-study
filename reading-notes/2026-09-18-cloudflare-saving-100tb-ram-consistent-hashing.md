## 1. Title

Saving Another 100TB of RAM with Math and Rust

## 2. Source

* Author / Organization: Kevin Guthrie, Mariia Iurchenko, Zaidoon Abd Al Hadi, Ivan Babrou / Cloudflare
* Link: Cloudflare Blog — Saving another 100TB of RAM with math (and Rust)
* Date: 2026-09-18

## 3. One-line Summary

* Cloudflare reclaimed roughly 100TB of RAM by mathematically reducing unnecessary consistent-hashing points by 90%, compacting their Rust representation, and safely migrating the new hash rings across its global infrastructure.

## 4. Key Points

* Cloudflare's Pingora Backend Router (PBR) used unexpectedly large amounts of memory for `pingora-ketama` consistent-hashing structures, reaching roughly 6GB in some processes.
* Consistent hashing maps requests and servers into the same hash space, limiting remapping when servers are added or removed.
* A single hash per server creates highly uneven load distribution, so implementations use many virtual points per server; NGINX and Pingora traditionally use a base of 160.
* Ketama-style weighting multiplies virtual points according to server capacity, allowing machines with more storage to receive proportionally more traffic.
* Feature and compliance constraints require separate rings for different server subsets, multiplying the number of stored hash points.
* Each original Rust `Point` stored a 32-bit hash and 32-bit server index, consuming 8 bytes.
* Reducing the server index to 16 bits did not automatically shrink the struct because Rust's alignment rules still padded it to 8 bytes.
* Encoding the hash and index in a 6-byte array eliminated the padding and reduced consistent-hashing storage by 25%.
* Mathematical analysis showed sharply diminishing returns from increasing virtual points; at very high counts, 32-bit hash collisions can actually increase distribution error.
* Cloudflare concluded that it could reduce hashes per server by 90% without appreciably degrading load distribution, then rolled out the change gradually while temporarily supporting both old and new rings.
* After the old rings were removed, measured PBR memory consumption fell by approximately 100TB across Cloudflare's fleet.

## 5. Deep Dive (Structured Understanding)

### Problem

PBR relies on consistent hashing to route cacheable requests predictably across servers. Real infrastructure complicates the basic algorithm:

1. Servers need reasonably balanced workloads.
2. Machines have different storage capacities and therefore require weighting.
3. Compliance and feature differences mean not every server can handle every request.
4. Different feature combinations require separate hash rings.

The resulting system accumulated huge numbers of virtual hash points across many rings. At Cloudflare's scale, seemingly small per-point overhead expanded into hundreds of terabytes of fleet-wide memory consumption.

### Approach

Cloudflare attacked the problem at two layers.

**Storage representation**

The original structure was conceptually:

`u32 hash + u32 server index = 8 bytes`

The server population did not require a 32-bit index, so the index could fit in 16 bits. Normal Rust struct alignment prevented the expected reduction, however.

Cloudflare therefore represented each point explicitly as six raw bytes:

`4-byte hash + 2-byte index = 6 bytes`

This reduced hash-point storage by 25%.

**Algorithmic cardinality**

The larger optimization questioned why so many virtual points existed in the first place.

More virtual points reduce variance between server workloads, but Cloudflare derived the relationship between point count and the coefficient of variation rather than relying only on conventional defaults.

The analysis exposed diminishing returns: achieving progressively smaller improvements requires roughly order-of-magnitude increases in hash count.

At sufficiently high counts, another limitation appears. The implementation uses a finite 32-bit hash space, so birthday-paradox effects make collisions increasingly probable. Collisions discard effective points and introduce additional distribution error.

The practical conclusion was counterintuitive:

**More hash points eventually consume substantially more memory while providing negligible — or potentially negative — balancing benefit.**

Cloudflare therefore reduced the generated hash count by approximately 90%.

### Key Insight

The important optimization was not replacing consistent hashing with a radically different system.

It was challenging two assumptions hidden inside an established implementation:

* Does every stored field need its current width?
* Does the conventional number of virtual hash points still provide meaningful marginal benefit at production scale?

Mathematical modeling converted the second question from intuition into an engineering constraint.

### Result / Impact

Cloudflare combined:

* 25% smaller hash-point representations
* ~90% fewer generated hash points
* improved sorting in the new implementation
* configurable hash scaling
* controlled dual-ring migration

After completing the rollout and removing the legacy rings, PBR's global memory usage fell by roughly **100TB**.

The optimized implementation is available through the `v2` functionality in `pingora-ketama`.

## 6. Why It Matters

* This is an example of **scale amplification**: saving only a few bytes from a frequently replicated structure can become a fleet-level infrastructure optimization.
* It demonstrates why algorithmic complexity and data representation cannot always be optimized independently; reducing object size helped, but reducing object count produced the larger structural gain.
* It challenges inherited constants such as "160 virtual nodes per server." Defaults originating in earlier systems may survive long after their original assumptions stop matching production workloads.
* The case illustrates the growing importance of **FinOps and infrastructure efficiency**: unused RAM is not merely an implementation detail when software runs across thousands of machines.
* It also shows that optimization at hyperscale increasingly involves statistics, probability, systems programming, and operational rollout design rather than isolated microbenchmarks.

## 7. Critical Analysis

* The reported 100TB reduction is fleet-wide, so the headline sounds more dramatic than the per-machine optimization. The operational scale is essential context.
* The article establishes that Cloudflare's previous point counts were excessive for its workload, but this does not imply that reducing virtual nodes by 90% is generally safe for other consistent-hashing deployments.
* Distribution quality depends on server count, weighting, hash width, topology, workload characteristics, and acceptable imbalance; Cloudflare's chosen operating point is environment-specific.
* The analysis focuses heavily on coefficient of variation. Production systems can also care about tail imbalance, hot keys, correlated workloads, heterogeneous hardware behavior, and failure scenarios that a single aggregate metric may not capture.
* Compact six-byte storage improves density but introduces less readable representation and accessor logic. The memory/performance benefit therefore trades some implementation simplicity for efficiency.
* The article identifies the explosion of separate feature-specific rings as a major source of memory usage but optimizes their representation rather than fundamentally eliminating that combinatorial architecture.
* The Hacker News discussion raises a broader engineering concern: highly specialized optimizations can increase system complexity and institutional knowledge requirements, even when their resource savings are substantial.
* Alternative approaches such as hierarchical rendezvous hashing are mentioned in the discussion but are not evaluated against Cloudflare's requirements, so the article does not establish that optimized Ketama is the theoretically or operationally best architecture.

## 8. Connections

### 1. Consistent Hashing and Virtual Nodes

Distributed systems such as caches, databases, and load balancers commonly use virtual nodes to smooth the random imbalance created by basic consistent hashing.

Cloudflare's result highlights the central trade-off:

`more virtual nodes → smoother distribution → more metadata`

The useful engineering question is therefore not "How many points maximize uniformity?" but "What is the minimum number that satisfies the required imbalance bound?"

### 2. Birthday Paradox and Finite Hash Spaces

Hash collisions are often treated as negligible, but collision probability grows approximately with the square of the number of sampled values.

A design that keeps increasing virtual points eventually collides with the limitations of a 32-bit hash space.

This connects a textbook probability concept directly to distributed-systems capacity planning.

### 3. Rust Memory Layout and Data-Oriented Design

Changing:

`u32 → u16`

does not necessarily reduce a struct's actual memory footprint because alignment and padding determine physical layout.

This connects the optimization to broader data-oriented design principles:

* field width
* alignment
* padding
* cache density
* representation overhead

At large cardinalities, physical representation can matter as much as logical type choice.

### 4. Diminishing Returns in Systems Optimization

Virtual-node counts exhibit diminishing marginal returns: progressively smaller improvements in balancing require disproportionately more state.

The same pattern appears in replication factors, cache sizes, retry counts, redundancy, indexing, and observability retention.

Optimization therefore requires identifying the point where marginal reliability or performance gains stop justifying resource costs.

### 5. Safe Distributed-System Migration

Changing a hash ring remaps cache keys. Replacing it globally would effectively cause mass cache misses and potentially overwhelm origin infrastructure.

Cloudflare therefore ran old and new rings simultaneously and controlled rollout across both traffic percentage and data-center scope.

This resembles techniques such as:

* canary deployment
* feature flags
* shadow infrastructure
* deterministic traffic splitting
* staged database migrations

The optimization was therefore as much a migration problem as an algorithm problem.

## 9. Keywords

* Consistent Hashing
* Ketama
* Virtual Nodes
* Pingora
* Rust Memory Layout
* Hash Collision
* Birthday Paradox
* Load Balancing
* Memory Optimization
* Distributed Systems

## 10. TL;DR

* Cloudflare discovered that PBR stored far more consistent-hashing state than necessary across many feature-specific Ketama rings.
* Compact Rust storage cut each point from 8 to 6 bytes, while mathematical analysis justified reducing hash counts by roughly 90%.
* A staged dual-ring migration avoided massive cache churn, and removing the legacy rings ultimately reclaimed about **100TB of RAM globally**.
