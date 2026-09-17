## 1. Title

Training a 4B Model to Produce 81% Faster Query Plans Than Postgres

## 2. Source

- Author / Organization: Rohan Bansal
- Link: https://rohanbansal.com/qorl
- Date: 2026-09-16

## 3. One-line Summary

- A 4B Qwen-based model trained through frontier-model distillation and agentic reinforcement learning learned to generate PostgreSQL query-plan hints that achieved a 1.81× geometric-mean speedup and 44.7% lower aggregate latency on 113 join-heavy JOB queries under a best-of-15 evaluation.

## 4. Key Points

- PostgreSQL query optimization is difficult because join ordering is NP-hard and the possible combinations of join orders, join algorithms, and scan methods grow combinatorially.
- PostgreSQL therefore relies on cost-based optimization and estimated cardinalities rather than executing every possible plan; inaccurate statistics and distribution assumptions can propagate into poor plan choices.
- The experiment used `pg_hint_plan` to let an LLM influence join ordering, join algorithms, and scan strategies without changing PostgreSQL's query semantics.
- A lightweight `qo-agent` harness exposed schema inspection, statistics, plan inspection, candidate execution, and plan-selection tools to the model.
- Training used the ~13.6k-query Cardinality Estimation Benchmark (CEB), while evaluation used the 113-query Join Order Benchmark (JOB); their join-graph topologies did not overlap.
- The vanilla 4B model largely failed to operate the agent harness, but supervised fine-tuning on GPT-6 Astra trajectories taught it valid tool use and plan construction.
- SFT alone eventually reached 1.16× geometric-mean speedup on JOB, showing that teacher trajectories transferred both harness behavior and some optimization capability.
- Agentic RL used actual PostgreSQL execution latency as a verifiable reward, with a custom anchored GRPO-style advantage calculation designed to discourage invalid, duplicate, and default-equivalent plans.
- After 1,200 RL updates, a single trajectory achieved about 1.41× geometric-mean speedup; selecting the best measured candidate across three trajectories increased this to 1.81× with no reported regressions in that selection.
- The experiment cost about $1,200: roughly $800 for 95 hours on a 2× H100 node and $400 for Astra-generated training trajectories.

## 5. Deep Dive (Structured Understanding)

### Problem

PostgreSQL must choose an execution plan without exhaustively executing every candidate.

Even relatively small multi-table queries create enormous search spaces because the optimizer must consider:

- join ordering
- join orientation
- hash / merge / nested-loop joins
- sequential / index / bitmap scans
- parallelism and other execution strategies

The optimizer therefore estimates costs using table statistics and cardinality estimates. These estimates can fail when data contains correlations or distributions that violate PostgreSQL's assumptions.

The experiment targets a narrower problem than replacing PostgreSQL's optimizer: repeatedly executed analytical queries where expensive offline optimization can be amortized over many future executions.

### Approach

The system keeps PostgreSQL responsible for executing and validating SQL while allowing the model to propose optimization hints through `pg_hint_plan`.

The model operates through `qo-agent`, which lets it:

1. inspect relations and indexes
2. inspect PostgreSQL statistics
3. inspect execution plans
4. propose candidate plan actions
5. execute and measure candidates
6. retain PostgreSQL's default plan when appropriate

Training occurs in two stages.

**Stage 1 — Off-policy distillation / SFT**

GPT-6 Astra generated hundreds of agent trajectories over CEB queries.

These trajectories were rendered into Qwen's token format, loss-masked, unrolled into training examples, and used to train a small LoRA adapter.

The main purpose was behavioral bootstrapping: teaching the 4B model the agent protocol, valid structured outputs, tool sequencing, and construction of legal plan hints.

**Stage 2 — Agentic RL**

The SFT-trained model generated multiple candidate trajectories for training queries.

Candidate plans were actually executed by PostgreSQL, making query latency the reward signal.

A custom reward and anchored GRPO-style mechanism handled:

- execution-time improvements
- measurement noise
- invalid candidates
- plans equivalent to PostgreSQL's default
- pathological incentives to always select the safe default

The training architecture separated GPU-heavy inference/training on rented H100s from PostgreSQL benchmarking on a local machine.

### Key Insight

Query optimization has an unusually useful property for reinforcement learning:

**finding a good plan is difficult, but evaluating a candidate can be straightforward — execute it and measure latency.**

This creates a verifiable reward environment without requiring humans to label whether a proposed optimization is good.

The experiment also shows an important distinction between online and offline optimization.

PostgreSQL's optimizer must return a reasonable plan extremely quickly. The learned optimizer can spend far more compute exploring alternatives when the resulting plan will subsequently be reused many times.

The model therefore does not necessarily replace PostgreSQL's optimizer; it operates more like an expensive profile-guided tuning system layered above it.

### Result / Impact

The progression was substantial:

- Vanilla 4B: largely unable to operate the harness
- SFT: learned valid agent behavior and reached ~1.16× geometric-mean speedup
- RL after 600 updates: ~1.35×
- RL after 1,200 updates: ~1.41×
- Best candidate across three trajectories / up to 15 candidates: **1.81× geometric-mean speedup**
- Aggregate workload latency reduction: **44.7%**

The result demonstrates that frontier-model trajectories plus environment feedback can specialize a relatively small open-weight model for a narrow systems task.

## 6. Why It Matters

- This is an example of **verifiable-reward RL applied to systems engineering**, rather than conventional language or reasoning benchmarks.
- It illustrates a broader shift from prompting large general-purpose models toward training smaller models against company-specific environments and workloads.
- Distillation separates capability discovery from deployment: an expensive frontier model can generate demonstrations while a smaller model becomes the specialized runtime policy.
- The approach resembles profile-guided optimization: expensive experimentation happens offline, while the discovered configuration can potentially be reused cheaply.
- The experiment also demonstrates why infrastructure matters in applied RL. PostgreSQL cache behavior and measurement noise directly affected reward correctness, making benchmark engineering part of the ML problem.
- More broadly, it suggests a useful architecture for AI-assisted systems optimization: **model proposes → deterministic system validates → real environment measures → reward trains model**.

## 7. Critical Analysis

- The headline "81% faster than Postgres" needs qualification. The strongest 1.81× result comes from selecting the best candidate across three trajectories and up to 15 candidates per query, not from a single model inference.
- The benchmark database is only about 8.5 GB and measurements were deliberately warmed. This does not establish equivalent improvements on multi-terabyte databases, disk-bound workloads, distributed databases, or production OLTP systems.
- The evaluation focuses on 113 join-heavy read queries from JOB. INSERT/UPDATE workloads, concurrency, locks, changing statistics, parameter-sensitive plans, and production traffic are outside the demonstrated scope.
- Planning and training costs are largely outside the reported query-latency speedup. This is acceptable for the stated repeated-analytics use case, but makes direct comparison with PostgreSQL's real-time optimizer misleading without amortization analysis.
- Training and evaluation use the same IMDb database. Although CEB and JOB join topologies do not overlap, the model can still learn database-specific distributions, relations, and structural regularities. This is intentional for specialization but limits claims about general query optimization.
- PostgreSQL was configured with 2 GB `shared_buffers`, and queries were warmed before measurement. This substantially reduced benchmark noise but creates a controlled workload that may differ from real deployments.
- The experiment does not establish that a 4B language model is the most compute-efficient solution. Smaller specialized neural networks, learned cost models, structured search, or systems such as Bao may provide comparable improvements with less inference overhead.
- HN discussion highlights another missing baseline: improving indexes, statistics, extended statistics, cardinality estimation, or conventional optimizer configuration could remove some bad plans without introducing an LLM.
- Workload drift remains a major operational problem. A previously good hint may become poor as table sizes, correlations, indexes, predicates, and parameter distributions change.
- The strongest result therefore demonstrates **offline workload-specific plan search**, not a general replacement for PostgreSQL's native query planner.

## 8. Connections

### 1. Cost-Based Query Optimization (CBO)

PostgreSQL's planner estimates cardinalities and operator costs to search for a sufficiently good plan without executing every possibility.

The learned system attacks the same optimization space from another direction: instead of relying exclusively on an analytical cost model, it receives empirical execution feedback.

This resembles combining model-based optimization with profile-guided search.

### 2. Bao and Learned Query Optimization

Bao demonstrated a more constrained learned approach to steering existing query optimizers rather than replacing them.

That connection raises a central architectural question from this experiment: whether a general 4B language model is necessary, or whether structured candidate generation plus a much smaller learned policy would be more efficient.

### 3. Profile-Guided Optimization

The project's intended workload resembles PGO in compilers.

Instead of requiring every optimization decision to be cheap enough for runtime planning:

`workload → execute alternatives → collect measurements → optimize → reuse`

Expensive exploration becomes rational when the optimized artifact is executed repeatedly.

### 4. AlphaZero-Style Search + Learned Heuristics

Query planning and board-game search share a structural pattern:

- enormous combinatorial search space
- legal actions can be constrained
- candidate outcomes can be evaluated
- exhaustive search is impractical

This suggests architectures combining deterministic search with learned ranking/value functions rather than asking a language model to directly solve the entire optimization problem.

### 5. Distillation + Domain-Specific Small Models

The experiment follows an increasingly important deployment pattern:

`Frontier Model → Demonstrations → SFT → Environment RL → Small Specialized Model`

The frontier model provides expensive general intelligence during bootstrapping, while the smaller model becomes specialized around a narrow task and environment.

### 6. Verifiable-Reward Reinforcement Learning

Unlike subjective tasks, database optimization provides an objective environment signal:

`candidate plan → execute → measure latency → reward`

This places query optimization alongside domains such as code execution, theorem proving, games, and compiler optimization where candidate solutions can be automatically checked or scored.

### 7. Benchmark Engineering as Part of ML

The PostgreSQL cache experiments show that reward quality depends on systems measurement.

Page-cache contention and insufficient `shared_buffers` initially produced phantom speedups and slowdowns. Increasing cache residency reduced the no-op error rate dramatically.

In environment-based RL, unreliable measurement effectively becomes label noise.

## 9. Keywords

- PostgreSQL
- Query Optimization
- Query Planner
- Agentic Reinforcement Learning
- GRPO
- Model Distillation
- Supervised Fine-Tuning
- LoRA
- pg_hint_plan
- Learned Query Optimization

## 10. TL;DR

- A 4B Qwen model was bootstrapped with frontier-model trajectories, then trained with PostgreSQL execution latency as an RL reward.
- It reached a 1.81× speedup and 44.7% aggregate latency reduction on 113 join-heavy JOB queries when selecting the best candidate from up to 15 plans.
- The result supports workload-specific offline learned optimization, but does not yet demonstrate a general or production-scale replacement for PostgreSQL's real-time query planner.
