1. Title

Real-Time Java: Control Loops the Garbage Collector Never Stops

2. Source

Author / Organization: Shai Almog / Codename One
Link: https://www.codenameone.com/blog/real-time-java-control-loops/
Date: 2026-11-09

3. One-line Summary

Codename One introduced a firm real-time execution model in ParparVM that runs 1 kHz Java control loops on mainline PREEMPT_RT Linux by replacing JIT compilation with ahead-of-time C generation, releasing real-time threads from stop-the-world GC pauses, and failing builds on loop memory allocations.

4. Key Points

• The Essence of Real-Time Systems: Real-time execution is defined by determinism ("on time, every time") rather than raw average speed; a control loop taking 50 µs on average but occasionally spiking to 30 ms is unusable for physical actuators, whereas bounded 900 µs execution within a 1 ms period is reliable.
• Firm Real-Time on Mainline Linux: Targets firm real-time workloads (e.g., motor drivers, robotics, sensor telemetry) running on Linux 6.12+ with PREEMPT_RT, leaving hard safety-critical certified domains (IEC 61508, ISO 26262, DO-178C) to dedicated hardware interlocks or certified RTOSes.
• The Four Managed Language Hazards: Managed runtimes historically fail real-time deadlines due to: 1) garbage collector stop-the-world pauses, 2) hidden allocation stalls (autoboxing, string concatenation), 3) JIT compilation/tiering de-optimizations, and 4) hidden slow paths (dynamic class initialization, exception stack unwinding, page faults).
• JIT Elimination via Ahead-of-Time C Translation: ParparVM compiles Java ahead-of-time (AOT) to native C, guaranteeing that the machine instructions executed in the first period are byte-identical to those executed in period one million.
• Immune Real-Time Threads: While ordinary threads pause briefly during garbage collection stack scans, real-time threads are released immediately; references are tracked concurrently via write barriers, preventing the collector from ever entering stop-the-world phases while real-time tasks are active.
• Compile-Time Allocation Prohibition: The build-time translator analyzes the entire call graph reachable from `RealtimeTask.cycle()` and fails the build with exact file and line references if it encounters `new`, array instantiation, primitive boxing, or string concatenation.
• Runtime Fail-Safe for Opaque Dynamic Dispatches: For dynamic call paths that static analysis cannot resolve (e.g., polymorphic interface calls), an un-annotated allocation instantly triggers a non-ignorable `RealtimeAllocationError` that fails the thread safely rather than stalling.
• Absolute Clock Scheduling and Overrun Policy: Loops execute on an un-drifting absolute clock; when a cycle overruns its period, the runtime invokes `periodsMissed(n)` to hold current actuator state rather than firing a burst of late cycles that would cause motors to violently lurch.
• POSIX Integration & Verification: Combines Linux `mlockall()` (preventing page faults), CPU core pinning, and `PTHREAD_PRIO_INHERIT` mutexes, while asserting `CAP_SYS_NICE` at boot to prevent silent fallback to non-deterministic `SCHED_OTHER` scheduling.

5. Deep Dive (Structured Understanding)

Problem
Writing low-latency, deterministic control loops (e.g., a 1 kHz motor actuator or robotic balancing algorithm) has long been the exclusive domain of C, C++, or Rust. In physical systems, missing a deadline means hardware stutters, stale telemetry, or mechanical collapse. Conventional wisdom dictates that managed, garbage-collected languages like Java are unsuitable for real-time control. Even on Linux kernels patched with PREEMPT_RT, standard JVMs introduce fatal sources of latency jitter: garbage collectors pause all threads to scan root sets, JIT compilers trigger asynchronous optimization spikes, memory allocations force threads to wait for heap sweeps, and hidden language semantics (autoboxing an `int` to `Integer`, or throwing an exception) allocate heap objects unpredictably.

Approach
Codename One leveraged its ownership of ParparVM—an ahead-of-time Java-to-C translator—to bypass the JVM's architectural limitations. The runtime establishes a strict separation between ordinary application threads (e.g., UI, networking) and `RealtimeTask` workers. Real-time tasks execute inside dedicated OS threads configured with Linux `SCHED_FIFO` real-time scheduling, pinned to isolated CPU cores, and locked into physical RAM via `mlockall()`. To eliminate GC pauses, the runtime redesigns the garbage collector's interaction with real-time threads: stack scanning releases real-time threads instantly without blocking, write barriers track object references concurrently, and the collector is barred from entering global stop-the-world phases while any real-time loop is running. To guarantee the collector is never invoked by the real-time thread itself, static analysis verifies that `cycle()` performs zero heap allocations.

Key Insight
1. Ownership of the Virtual Machine Unlocks Custom Concurrency Semantics: Third-party libraries running on top of standard OpenJDK cannot alter the fundamental stop-the-world invariants of the HotSpot garbage collector. Owning the underlying VM translator (ParparVM) allowed the team to alter thread-scheduling and write-barrier contracts, treating real-time threads as first-class unmanaged citizens within a managed runtime.
2. Build-Time Static Enforcement of Allocation Invariants: Hoping developers remember not to allocate in a control loop is an operational anti-pattern. Enforcing zero-allocation rules at compile time by failing the build on byte-code allocations (boxing, array creation, string concatenation) eliminates entire classes of runtime latency regressions before code reaches hardware.
3. Actuator-Friendly Overrun Semantics ("Hold, Don't Lurch"): When high-frequency control loops encounter unexpected physical or algorithmic overruns, naive schedulers attempt to execute back-to-back iterations to catch up with wall-clock time. In physical robotics and motor control, burst execution sends erratic, rapid step changes to hardware; explicitly dropping missed periods and invoking `periodsMissed(count)` preserves actuator stability.

Result / Impact
Proved that high-level, ergonomic Java code can reliably execute 1 kHz firm real-time control loops on low-cost embedded Linux hardware, achieving microsecond-bounded release latency and zero GC interruptions while retaining the developer velocity of a managed language.

6. Why It Matters

• Systems Programming & Embedded Systems: Challenges the long-standing industry dogma that managed languages cannot drive real-time physical actuators, bridging high-level application development with low-level deterministic control.
• Linux Kernel & Operating System Evolution: Capitalizes on the formal merging of PREEMPT_RT into mainline Linux 6.12, showing how application runtimes can utilize kernel real-time scheduling primitives without bespoke out-of-tree patches.
• Compiler Construction & Static Analysis: Serves as an architectural template for using static call-graph analysis to enforce strict operational constraints (zero-allocation) in high-level languages.
• Platform Engineering & Hardware-in-the-Loop Testing: Demonstrates rigorous benchmarking methodologies for embedded systems, emphasizing that real-time claims must be verified via in-kernel latency histograms and runtime capability auditing (`CAP_SYS_NICE`).

7. Critical Analysis

• Firm vs. Hard Real-Time Liability Boundaries: The author explicitly emphasizes that the system targets firm real-time on PREEMPT_RT Linux, not certified hard real-time (DO-178C, ISO 26262). Deploying this architecture in life-critical systems (automotive braking, medical devices, avionics) without independent hardware watchdogs or physical interlocks would represent an irresponsible safety failure.
• Polymorphism and Reflection Blind Spots: While the static analyzer inspects direct call graphs, Java's dynamic nature (reflection, interfaces with multiple dynamic implementations) limits build-time visibility. Falling back to runtime `RealtimeAllocationError` halts the thread safely, but in an unmonitored production environment, a halted motor thread is functionally equivalent to an actuator failure.
• Process Memory Leak Blast Radius: While real-time threads do not allocate, memory leaks in adjacent UI or networking threads can exhaust physical RAM. Although `setMemoryLimit()` protects the process from system OOM panics, high memory pressure in non-real-time threads can trigger CPU memory bus contention, introducing subtle hardware-level latency jitter on shared multi-core processors.

8. Connections

• Linux Mainline PREEMPT_RT (Kernel 6.12): Directly leverages the newly mainlined real-time kernel architecture that turns spinlocks into sleepable mutexes and forces IRQs into schedulable kernel threads.
• Real-Time Specification for Java (RTSJ / JSR-1): Echoes historical real-time Java efforts (JamaicaVM, Aonix Perc) which used scoped memory areas (`NoHeapRealtimeThread`), but simplifies the developer model by enforcing zero-allocation at build time rather than introducing complex scoped-memory APIs.
• POSIX Real-Time Extensions: Extends POSIX real-time primitives—`SCHED_FIFO`, `mlockall()`, and `pthread_mutexattr_setprotocol(PTHREAD_PRIO_INHERIT)`—into Java language constructs.

9. Keywords

• Real-Time Java
• PREEMPT_RT
• ParparVM
• Control Loops
• Stop-the-World GC
• Zero Allocation
• Priority Inversion
• Linux 6.12

10. TL;DR

• Codename One introduced a firm real-time execution model in ParparVM, running 1 kHz Java control loops on mainline PREEMPT_RT Linux without GC pauses.
• AOT compilation to C eliminates JIT deoptimizations, while real-time threads are exempt from stop-the-world stack scans via write-barrier tracking.
• Enforces zero allocations at build time, incorporates priority-inheriting mutexes, and uses absolute clock scheduling with overrun callbacks to ensure deterministic motor and robotics control.
