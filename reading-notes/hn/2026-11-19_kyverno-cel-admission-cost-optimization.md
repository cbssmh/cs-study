1. Title
Optimizing Kubernetes admission control: Where Kyverno's CEL admission cost actually goes

2. Source
Author / Organization: Raunak Madan (Nirmata) / Cloud Native Computing Foundation (CNCF) Blog (originally posted on kyverno.io)
Link: https://www.cncf.io/blog/2026/11/18/optimizing-kubernetes-admission-control-where-kyvernos-cel-admission-cost-actually-goes/
Date: November 18, 2026

3. One-line Summary
Profiling Kyverno's CEL admission engines revealed that 86.8% of allocations went to repeatedly deserializing the admitted Kubernetes object once per matched policy rather than evaluating CEL expressions, a bottleneck eliminated in Kyverno 1.19.2.

4. Key Points
- **Unexpected profiling hotspot**: Heap profiling of Kyverno's `ValidatingPolicy` engine showed `ConvertObjectToUnstructured` accounted for 86.8% of all allocated bytes while CEL evaluation barely registered; hoisting conversion out of the loop dropped conversion share to 2.9%, making `evaluateWithData` the dominant frame.
- **`O(policies)` to `O(1)` object preparation**: Because raw JSON admission bytes were parsed into CEL-readable maps inside the matched-policy loop, a Pod matching 16 Pod Security Standards policies was parsed 16 identical times; moving conversion before the loop made preparation constant per request.
- **Preserving subtle engine contracts**: Validating/generating policies expect the raw API server object while mutating policies expect a normalized object (and both disagree on empty objects for `DELETE`); Kyverno keeps each engine's contract intact using a shared implementation verified by field-by-field equivalence tests, splicing the converted object into a blanked-out request copy.
- **Engine benchmark improvements**: On a ~50 KB Pod in Go benchmarks (excluding webhook/API server/etcd), `ValidatingPolicy` allocations at 64 policies dropped from `148,131` (`20.01 ms`) to `8,672` (`0.36 ms`), and `MutatingPolicy` dropped from `258,943` (`10.59 ms`) to `52,689` (`2.97 ms`), with the image verification engine receiving the same fix.
- **Kyverno vs. native `ValidatingAdmissionPolicy`**: On a single-host loopback cluster running 16 Pod Security Standards policies (measured on Kyverno 1.19.0 before the fix), native admission led at 1 policy (~`9.2 ms` vs. ~`12.8 ms`), but Kyverno led at 16 policies (~`20 ms` vs. ~`29 ms`) and used ~`3.2` CPU cores vs. `8.2` cores on the API server, where 67% of CPU went to policy dispatch machinery and only 5.5% to CEL bytecode execution.
- **Legacy `ClusterPolicy` regression fix**: A 12%–23% admission latency regression introduced between `1.16.4` and `1.17.0` stemmed from copying the entire evaluation context on every rule checkpoint; replacing eager copies with shallow copy-on-write snapshots restored baseline speed while preserving rule isolation.
- **Explicit remaining gaps**: Kyverno 1.19.2 still pays one JSON round-trip per request to preserve raw `request.object` semantics, mutating policies still pass full objects instead of patch diffs, and `p95` tail-latency swings under sustained single-replica load remain unaddressed.

5. Deep Dive (Structured Understanding)
**Problem**
Public Kubernetes policy-engine benchmarks showed Kyverno trailing alternatives. While maintainers initially pointed users from the legacy JMESPath `ClusterPolicy` engine to the newer CEL-based engines, they had not profiled where the CEL engines spent CPU and memory—nor diagnosed a 12%–23% latency regression in `ClusterPolicy` since `1.16.4`.

**Approach**
The author profiled `ValidatingPolicy`, `MutatingPolicy`, and image verification engines using Go heap profiles and isolated engine benchmarks (`pkg/cel/policies/vpol/engine`, `pkg/engine/context`), plus cluster-level comparisons against Kubernetes native `ValidatingAdmissionPolicy`. The fixes—shipping in Kyverno 1.19.2 (PRs `#17572`, `#17585`, `#17565`)—hoist CEL request-activation construction outside the per-policy loop, splice the heavy `object` field into a lightweight request copy, enforce behavioral parity with reference-output equivalence tests, and switch legacy context checkpoints to lazy copy-on-write snapshots.

**Key Insight**
In admission control, policy evaluation itself is cheap compared to input preparation and dispatch overhead. Once object deserialization is amortized across policies (`O(1)` per webhook call), an external webhook engine can outperform in-process API server `ValidatingAdmissionPolicy` at realistic policy counts (e.g., 16 policies) because the API server pays per-policy dispatch overhead directly on the non-horizontally-scalable control plane.

**Result / Impact**
Allocation growth from 1 to 64 matched policies fell from ~63x to ~10x (leaving only genuine CEL evaluation scaling), engine latency at 64 policies dropped by ~98% in `ValidatingPolicy` (`20.01 ms` to `0.36 ms`) and ~72% in `MutatingPolicy` (`10.59 ms` to `2.97 ms`), and the reproducible harness was published under `docs/perf-testing/` in the Kyverno repository.

6. Why It Matters
- **Kubernetes & Platform Engineering**: Clarifies the real architectural trade-off between native `ValidatingAdmissionPolicy` and external webhook engines like Kyverno—showing that control-plane CPU saturation from policy dispatch can matter more than webhook network hops once a cluster enforces dozens of policies.
- **Backend Engineering & Infrastructure**: Demonstrates a classic systems-performance pitfall where per-item serialization inside a fan-out loop dominates runtime cost, and illustrates how to optimize shared hot paths without breaking subtle legacy contract differences across policy types.
- **Reliability / Observability**: Highlights the value of reproducible profiling harnesses and equivalence testing when fixing performance regressions in critical cluster admission paths.

7. Critical Analysis
- **Benchmark isolation vs. real-cluster conditions**: The dramatic headline speedups (`20.01 ms` to `0.36 ms`) come from single-machine Go benchmarks driving the engine loop directly, excluding webhook TLS/network serialization, API server overhead, and `etcd` writes. As the author explicitly notes, loopback cluster tests also make the webhook hop artificially free, whereas multi-node clusters pay real network latency on every Kyverno admission call.
- **Version mismatch in the native comparison**: The 16-policy comparison against native `ValidatingAdmissionPolicy` was measured on Kyverno `1.19.0` (before the `1.19.2` fix), and the author notes it has not been re-run since—leaving the exact post-fix crossover point between native and Kyverno unmeasured.
- **Open tail-latency and memory questions**: Single-replica `p95` tail-latency instability under sustained load was not re-measured or resolved, and mutating policies still thread full objects rather than patch diffs, which may keep memory pressure elevated for large manifests or heavy mutation pipelines.

8. Connections
- **Kubernetes Control-Plane Offloading**: The finding that API server `ValidatingAdmissionPolicy` spent 67% of its CPU in dispatch machinery aligns with broader Kubernetes scaling patterns (like moving heavy ingress, rate limiting, or policy evaluation to horizontally scalable worker-node deployments rather than overloading the control plane).
- **Copy-on-Write (CoW) & Structural Sharing**: Replacing full evaluation-context copies with shallow CoW snapshots in `ClusterPolicy` mirrors classic CoW optimizations in filesystems (ZFS/Btrfs), OS `fork()`, and immutable data structures to achieve isolation without upfront `O(N)` memory duplication.
- **Amortized Serialization in Middleware/Gateways**: Hoisting `ConvertObjectToUnstructured` out of the policy loop parallels how API gateways (Envoy/Kong) and Wasm plugin hosts parse request headers or JSON bodies once into a shared representation before executing multiple middleware filters.

9. Keywords
Kubernetes Admission Control, Kyverno, Common Expression Language (CEL), ValidatingAdmissionPolicy, Go Heap Profiling, Serialization Overhead, Copy-on-Write, Control Plane Scaling, Policy-as-Code, Performance Benchmarking

10. TL;DR
Profiling Kyverno's CEL admission engines showed 86.8% of allocations came from re-parsing the same admitted object once per matched policy rather than evaluating CEL expressions.
Kyverno 1.19.2 hoists object conversion to once per request (verified by field-level equivalence tests) and switches legacy `ClusterPolicy` checkpoints to copy-on-write snapshots.
Once deserialization is `O(1)` per request, Kyverno can beat native `ValidatingAdmissionPolicy` in CPU and latency at realistic policy counts (e.g., 16 policies), though multi-node network hops and `p95` tail latency under sustained load remain separate considerations.
