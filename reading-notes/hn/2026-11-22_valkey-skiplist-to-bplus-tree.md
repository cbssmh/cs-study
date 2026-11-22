1. Title
From Skiplists to B+ Trees: Making Valkey Sorted Sets More Memory Efficient

2. Source
Author / Organization: Rain Valentine (AWS) and Dragos Andriciuc (Percona) / Valkey Blog
Link: https://valkey.io/blog/from-skiplist-to-b+tree/
Date: November 16, 2026
HN discussion: https://news.ycombinator.com/item?id=50534387

3. One-line Summary
Valkey 9.2 replaces the probabilistic skiplist backing large sorted sets with `fbtree`, a 61-way cache-aligned B+ tree variant that cuts per-member index overhead by 36%–43% while boosting `ZADD` and `ZREM` throughput by 105% and 76%.

4. Key Points
- **Why the skiplist was costly**: Large Valkey sorted sets pair a hashtable (`O(1)` member lookup) with an ordered index (`O(log n)` rank/score lookup); the legacy skiplist allocated one heap node per member carrying an 8B backward pointer, ~1.33 average levels of forward pointer + span pairs (16B/level, ~29B total), and an 8B score (~37B before the member string), while scattering dependent pointer chases across DRAM.
- **`fbtree` node sizing and packed layout**: `fbtree` (Feature B+ Tree) uses a 61-way fanout—versus the skiplist's effective branching factor of ~4—with 512-byte leaf nodes holding up to 61 packed `score + member` pointers on 64-bit builds, aligned to `jemalloc` size classes (`innerNode` rounds up to 1,280 bytes on 32-bit builds).
- **Feature-based SIMD routing**: Inner nodes store each child's anchor alongside a shared prefix and four distinguishing "feature" bytes, allowing parallel SIMD comparisons to identify or narrow the target child before dereferencing child nodes or full variable-length member strings.
- **Preserving scan and pop complexity**: Doubly linked leaf nodes allow `ZRANGE` scans to stream across 61 contiguous entries per leaf without climbing the tree, cached leftmost/rightmost leaf pointers keep `ZPOPMIN` / append operations `O(1)`, and range deletions avoid full structure rebuilds.
- **Measured memory savings**: On 5 million 20-byte members (comparing only the ordered index, with hashtable and member embedding unchanged), per-member overhead dropped from `50.3B` to `28.5B` (43%, ~109 MB saved) for sequential inserts and `32.0B` (36%, ~92 MB saved) for random inserts due to partially filled nodes.
- **Command throughput gains**: On a 64-core AWS Graviton3 (`c7g.metal`) with 9 I/O threads, pipeline depth 10, and 3M members (measured on PR `#4206` branch), `ZADD` score updates rose `+105%`, `ZREM` `+76%`, `ZCOUNT` `+27%`, `ZRANK` `+19%`, and `ZRANGE` `+6%`, while hashtable-backed `ZSCORE`/`ZRANDMEMBER` (`+2%`) and `ZPOPMIN` (`~0%`) remained near parity.
- **Minimal operational surface change**: Sorted set command semantics are unchanged; only `OBJECT ENCODING` changes from `skiplist` to `btree` for large sorted sets (small sets still use `listpack`).

5. Deep Dive (Structured Understanding)
**Problem**
In-memory data stores paying ~37B–50.3B of index metadata per sorted-set member can spend more RAM on ordering pointers than on small member payloads themselves. Moreover, scattered single-element skiplist allocations defeat CPU cache prefetching and cause dependent L3/DRAM stalls on every traversal step.

**Approach**
Valkey 9.2 replaces the skiplist in the `hashtable + ordered index` sorted-set pairing with `fbtree`, a custom B+ tree engineered around Valkey's memory allocator and command patterns:
1. Pack 61 entries per `jemalloc`-aligned node, co-locating normalized 8-byte scores with member bytes to remove an indirection hop.
2. Compress variable-length string routing in inner nodes via shared prefixes plus 4-byte SIMD-comparable feature tags.
3. Link leaves doubly for sequential scans, cache boundary leaves for `O(1)` min/max pops, and tune against full-server benchmarks until every major sorted-set command matches or beats the skiplist.

**Key Insight**
Although B+ trees originated to minimize disk I/O, the exact same high-fanout, contiguous-node principle minimizes *CPU cache-line misses* in main-memory databases—and in single-threaded or partitioned event-loop engines like Valkey, lock-free concurrency (the main traditional advantage of skiplists) is unnecessary, making B+ trees strictly superior on memory density and cache locality.

**Result / Impact**
Large sorted sets gain 36%–43% lower per-member index overhead and up to 2x write-update throughput (`ZADD`), while `ZPOPMIN` regressions are prevented via cached boundary leaves and downstream tooling only needs to recognize `btree` in `OBJECT ENCODING`.

6. Why It Matters
- **Databases & Infrastructure**: Demonstrates how modern memory-hierarchy realities (1 ns L1 vs. ~100 ns DRAM) push in-memory engines away from pointer-heavy structures (skiplists, linked lists) toward `jemalloc`-aligned, SIMD-assisted B+ trees—converging with disk-engine designs.
- **Backend Engineering**: Directly reduces RAM footprint and cloud instance sizing for workloads heavy on large sorted sets (leaderboards, rate limiters, time-windowed queues, secondary indexes) with zero application code changes.
- **Performance Engineering**: Shows the discipline of benchmarking full-server command mixes rather than microbenchmarking an isolated tree, ensuring tail operations (`ZPOPMIN`, `ZSCORE`) do not regress.

7. Critical Analysis
- **Benchmark currency and hardware specificity**: The authors explicitly note that command throughput benchmarks were run on PR `#4206`'s development branch (`oi/pr3-fbtree-v2`) and were *not* re-run on the final merged commit; furthermore, tests ran solely on ARM64 Graviton3 (`c7g.metal`) with pipeline depth 10, leaving x86_64 SIMD behavior and unpipelined latency unquantified in the post.
- **Workload dependence of memory savings**: Because savings are fixed per member (~18–22B) rather than proportional to payload size, workloads with large member strings will see a much smaller percentage reduction in total RAM than the 5M × 20-byte test.
- **Missing implementation details**: The post does not explain how `fbtree` maintains subtree/slot counts for `O(log n)` `ZRANK` index calculations or how leaf compaction behaves under heavy churn (though Valkey 9.2's release notes separately note sparse-leaf compaction on deletes).
- **HN community perspective**: Commenters (including Redis creator Salvatore Sanfilippo `antirez`) noted parallel B+ tree efforts—`antirez` stated Redis recently adopted his own B+ tree + specialized dictionary design storing multiple items per hashtable allocation to cut RAM even further, and DragonflyDB implemented a B+ tree sorted set in 2024. Several practitioners praised `jemalloc`-class alignment and SIMD prefix features while noting skiplists remain relevant only where lock-free concurrent writers are needed; others criticized the blog post's prose style as overly polished compared to the deeper technical design in PR `#4206`.

8. Connections
- **Redis & DragonflyDB Convergence**: Both DragonflyDB (2024) and upstream Redis (`antirez`'s recent sorted-set rewrite) independently converged on replacing skiplists with B+ trees for sorted sets, confirming an industry-wide shift from implementation simplicity to cache-line and allocator efficiency.
- **Cache-Oblivious & SIMD-Accelerated Indexes (ART, B-Trees)**: Storing 4-byte distinguishing "feature" prefixes in inner nodes for SIMD comparison parallels Adaptive Radix Trees (ART) and prefix-compressed B-trees in analytical/OLTP engines like DuckDB, PostgreSQL, and LeanStore.
- **Allocator-Aware Data Structure Design**: Sizing `fbtree` nodes explicitly to `jemalloc` size classes (512B leaves) builds directly on Valkey 9.1's `embstr` threshold and `robj->ptr` optimizations, treating the memory allocator's rounding boundaries as a first-class design constraint.

9. Keywords
Valkey 9.2, B+ Tree, Skiplist, Sorted Sets, fbtree, Cache Locality, jemalloc, SIMD, In-Memory Databases, Memory Efficiency

10. TL;DR
Valkey 9.2 replaces its probabilistic skiplist for large sorted sets with `fbtree`, a 61-way `jemalloc`-aligned B+ tree that packs scores and members together and routes lookups using SIMD-compared 4-byte prefix features.
In 5M-member tests, `fbtree` cuts per-member index overhead by 36%–43% (saving 92–109 MB) and improves `ZADD` (+105%), `ZREM` (+76%), and `ZCOUNT` (+27%) throughput while changing only `OBJECT ENCODING` output (`skiplist` to `btree`).
Because Valkey does not need lock-free concurrent writes in its sorted-set index, trading skiplists for cache-friendly B+ trees aligns with similar moves in Redis and DragonflyDB—though benchmarks were measured on an ARM64 PR branch rather than the final GA commit.
