# 1. Title

How We Saved 100 Terabytes of Memory by Optimizing 1.1.1.1’s DNS Cache

# 2. Source

- Author / Organization: Sebastiaan Neuteboom / Cloudflare
- Link: https://blog.cloudflare.com/dns-cache-memory-optimization-1111/
- Date: 2026-08-27

# 3. One-line Summary

Cloudflare redesigned the in-memory representation of 1.1.1.1’s 250B+ DNS cache entries, cutting per-entry memory by 56%, freeing roughly 100 TB of RAM, and simultaneously improving insert throughput and lookup latency.

# 4. Key Points

- Cloudflare’s Big Pineapple platform holds more than 250 billion DNS cache entries, making even one wasted byte per entry equivalent to over 250 GB fleet-wide.
- Five storage-layout optimizations reduced the benchmarked per-entry footprint from 953 bytes to 420 bytes, a 56% reduction.
- Replacing growable `Vec<T>` and `String` values with fixed-size `Box<[T]>` and `Box<str>` removed unused capacity metadata and over-allocation.
- Combining separate DNS record lists into one contiguous list with small offsets reduced pointers, lengths, allocations, and struct padding.
- Record owner names identical to the query name were removed and reconstructed from the cache key, eliminating widespread duplicate data.
- Rust’s enum representation caused small, common DNS records such as `A` and `AAAA` to inherit the size of rare large variants; boxing large variants reduced this waste.
- Cloudflare ultimately stored record data largely in DNS wire format inside one contiguous `Box<[u8]>`, avoiding many allocations and improving CPU cache locality.
- The wire-format representation also avoided unnecessary parse → store → serialize cycles for common record types.
- Production p99 process memory fell from 9.3 GB to 5.3 GB; fleet-wide working-set memory fell by roughly 100 TB.
- Cache insert throughput increased from 625K to 893K entries/s (+43%), while lookup latency fell from 828 ns to 670 ns (-19%).

# 5. Deep Dive (Structured Understanding)

## Problem

Big Pineapple serves Cloudflare DNS products including 1.1.1.1 and maintains hundreds of billions of cache entries.

Its original representation favored convenient, general-purpose Rust structures:

- growable `Vec<T>` collections,
- multiple independent record lists,
- duplicated owner names,
- parsed record objects,
- enums sized according to their largest variant.

Individually these overheads were small. At 250B+ entries, however, bytes of metadata, padding, spare capacity, and extra allocations multiplied into tens of terabytes.

The workload also had an important property: once a DNS response entered the cache, its record data was effectively immutable.

## Approach

Cloudflare progressively redesigned the representation around the actual workload.

**1. Remove unused mutability**

`Vec<T>` → `Box<[T]>`  
`String` → `Box<str>`

Cached data does not grow after insertion, so capacity metadata and reserved space provide no value.

**2. Consolidate collections**

Answer, authority, and additional records were placed into one collection.

Small `u16` offsets identify section boundaries instead of maintaining independent pointer/length pairs.

**3. Eliminate predictable duplication**

When a record owner equals the queried domain, the owner is omitted.

The cache key already contains that domain, so it can be restored during response construction.

**4. Reduce enum inflation**

Rust enums occupy enough space for their largest variant.

Rare large DNS record variants therefore inflated common `A` and `AAAA` records. Moving large variants behind pointers reduced the common-case footprint.

**5. Store closer to the final representation**

Instead of retaining fully parsed record objects, Cloudflare packed encoded records into a contiguous byte buffer:

`[length][record][length][record]...`

Many records can then be copied directly into outgoing DNS responses instead of being serialized field by field.

## Key Insight

The optimal representation depends on how data is actually used.

The original structures supported properties such as growth, independent ownership, random access, and rich parsed representations. The cache workload needed relatively little of that flexibility.

For a read-heavy, immutable, extremely large cache, compact contiguous data can outperform richer object representations in both memory and speed.

The key optimization was therefore not a new algorithm but changing the **physical representation of data**.

## Result / Impact

Benchmark results:

- Per-entry footprint: 953 B → 420 B (-56%)
- Per-entry allocations: 1.1 KB → 461 B (-58%)
- Insert throughput: 625K → 893K entries/s (+43%)
- Lookup latency: 828 ns → 670 ns (-19%)

Production results:

- p99 memory: 9.3 GB → 5.3 GB
- p90 memory: 6.5 GB → 3.8 GB
- Fleet-wide memory freed: ~100 TB
- Equivalent RAM: ~130 Cloudflare Gen 13 servers

Cloudflare plans to use the freed memory to increase cache capacity, potentially improving hit rates and reducing upstream DNS queries.

# 6. Why It Matters

This is a concrete example of how abstraction overhead changes meaning at hyperscale.

A capacity field, pointer, padding byte, or allocation is insignificant for thousands of objects but becomes infrastructure when multiplied across hundreds of billions.

The case also demonstrates that memory optimization and performance optimization are often aligned. Fewer allocations and more contiguous data improve memory locality and reduce CPU work, so reducing RAM consumption can also reduce latency.

More broadly, it reinforces a systems-programming principle:

> Data representation can matter as much as algorithmic complexity.

The article also connects to the continuing tension between developer-friendly abstractions and workload-specific representations. High-level containers are excellent defaults, but extreme scale can justify replacing them with specialized layouts.

# 7. Critical Analysis

- The headline “saved 100 TB” is fleet-wide aggregate memory, not a single 100 TB cache. The number is technically meaningful but easy to misinterpret without deployment-scale context.
- Production process memory fell by roughly 42–43%, smaller than the 56% per-entry benchmark reduction because the cache is only part of total process memory.
- Several optimizations appear straightforward in hindsight, particularly replacing growable structures for immutable data. However, whether implementing them earlier would have justified additional development and operational complexity is unclear.
- The article does not quantify engineering cost, rollout risk, or the monetary value of the saved memory, making ROI difficult to evaluate precisely.
- Moving toward packed representations increases implementation complexity and reduces some conveniences of typed, independently owned structures.
- Sequential record access is acceptable because DNS responses contain relatively few records, but the same design would not automatically generalize to workloads requiring frequent random access.
- The Hacker News discussion highlights an unresolved engineering tradeoff: avoiding premature optimization versus choosing scale-appropriate data structures from the beginning.
- The production rollout itself matters: changing hot-path data structures safely across a global service is substantially harder than demonstrating the optimization in an isolated benchmark.

# 8. Connections

## 1. Object Representation vs. Serialization Formats

General-purpose in-memory objects optimize for programmability, mutation, and random access.

Wire and serialization formats such as TLV-style encodings instead emphasize compactness and sequential processing.

Cloudflare effectively moved its cache representation toward the latter because cached DNS records are usually read and transmitted rather than modified.

## 2. CPU Cache Locality and Data-Oriented Design

Replacing scattered heap objects with contiguous byte buffers improves spatial locality.

This resembles data-oriented design used in databases, game engines, and high-performance systems: organize memory according to access patterns rather than object-oriented conceptual boundaries.

The result can reduce both memory consumption and cache misses.

## 3. Database Row Storage

Database engines frequently pack variable-length fields into compact rows and store offsets rather than maintaining separate heap objects for every field.

Cloudflare's consolidated record buffer follows a similar principle: variable-sized data is packed together while lightweight metadata identifies boundaries.

## 4. Rust `Vec`, `Box`, and Enum Layout

The case demonstrates that Rust's safe abstractions still have concrete memory layouts.

`Vec<T>` carries pointer, length, and capacity, while `Box<[T]>` only needs information required for a fixed-size allocation.

Likewise, enum size depends on the largest variant, making boxing rare large variants a useful optimization when variant sizes are highly skewed.

## 5. Premature Optimization vs. Production Profiling

The Hacker News discussion mirrors the classic optimization debate.

One position favors simple implementations until measurements identify meaningful bottlenecks. The opposing position argues that obvious workload properties — such as billions of immutable objects — should influence data structure selection from the beginning.

The practical middle ground is to design for future evolution while optimizing aggressively only after production measurements establish the payoff.

# 9. Keywords

- DNS Cache
- Cloudflare 1.1.1.1
- Rust Memory Layout
- Data-Oriented Design
- Memory Optimization
- Cache Locality
- Boxed Slice
- Heap Allocation
- Wire Format
- Systems Programming

# 10. TL;DR

Cloudflare redesigned how 250B+ DNS cache entries are represented in memory, reducing per-entry footprint by 56% and freeing roughly 100 TB of RAM.
The biggest gains came from removing unnecessary capacity, pointers, duplication, enum padding, and heap allocations while storing records closer to DNS wire format.
The broader lesson is that at hyperscale, tiny data-structure overheads become infrastructure costs, and better memory layout can improve both space efficiency and speed.
